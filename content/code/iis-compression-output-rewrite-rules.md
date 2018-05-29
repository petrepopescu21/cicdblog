---
title: "Fixing 500.52 for an Azure App Service with Outbound Rewrite Rules and Compression"
date: 2018-05-29T13:31:27+03:00

categories: ['Code', 'Tutorials']
tags: ['IIS', 'Azure', 'App Services', 'Compression', 'Rewrite', 'Outbound']
author: "Petre"
---

In an App Service Plan, there is one IIS server running per instance and each Web App will run in its own App Domain. Although compression is enabled on the entire server, the web.config file applies to the app itself. If you enable compression here, it will run in the same pipeline as the Rewrite Module so it will clash with your Outbound rules.
There is a way to work around the issue, by removing the Accept-Encoding header, performing the outbound rewrite actions and then adding the header back before passing the request through the Compression module.

##### 1. Add `HTTP_ACCEPT_ENCODING` and `HTTP_X_ORIGINAL_ACCEPT_ENCODING` as allowed server variables

Create an applicationHost.xdt file in `D:\home\site` and paste the following content

```xml
<?xml version="1.0"?>
 <configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
   <system.webServer>
     <rewrite>
       <allowedServerVariables>
         <add name="HTTP_ACCEPT_ENCODING" xdt:Transform="InsertIfMissing" xdt:Locator="Match(name)" />
         <add name="HTTP_X_ORIGINAL_ACCEPT_ENCODING" xdt:Transform="InsertIfMissing" xdt:Locator="Match(name)" />
        </allowedServerVariables>
      </rewrite>
  </system.webServer>
</configuration>
```

This will allow us to modify the two headers in various steps within the pipeline. We'll use the `HTTP_X_ORIGINAL_ACCEPT_ENCODING` as a backup for `HTTP_ACCEPT_ENCODING`
When adding multiple values to the `allowedServerVariables` definition, make sure that you add the Locator property.

##### 2. Create a catch all inbound rewrite rule in your web.config to remove the accept-encoding header

```xml
<rule name="Catch all" patternSyntax="Wildcard">
  <match url="*" ignoreCase="false"/>
  <conditions logicalGrouping="MatchAll">
    <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true"/>
    <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true"/>
  </conditions>
  <serverVariables>
     <set name="HTTP_X_ORIGINAL_ACCEPT_ENCODING" value="{HTTP_ACCEPT_ENCODING}"/>
     <set name="HTTP_ACCEPT_ENCODING" value=""/>
  </serverVariables>
  <action type="None" />
</rule>
```

##### 3. Create a new outbound rule/precondition pair which will add the header back – make sure that this is the last rule in the outboundRules definition

```xml
<rule name="Restore Headers" preCondition="NeedsRestoringAcceptEncoding">
  <match serverVariable="HTTP_ACCEPT_ENCODING" pattern="^(.*)"/>
  <action type="Rewrite" value="{HTTP_X_ORIGINAL_ACCEPT_ENCODING}" replace="true"/>
</rule>
```

```xml
<preCondition name="NeedsRestoringAcceptEncoding" patternSyntax="ECMAScript" logicalGrouping="MatchAll">
  <add input="{HTTP_X_ORIGINAL_ACCEPT_ENCODING}" matchType="Pattern" ignoreCase="true" pattern=".+"/>
</preCondition>
```

##### 4. Add your compression definition __after__ the rewrite module definition

```xml
<urlCompression doStaticCompression="true" doDynamicCompression="true" dynamicCompressionBeforeCache="false" />
```

##### 5. Restart the site (step required for the first step to take effect)
