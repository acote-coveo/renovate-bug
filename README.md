# renovate-bug

Reproduction of a renovate bug where dependencies using a property from a parent maven module is not resolved.

## Project setup

- Parent module (parent)
  - Child module (child)

The parent module define some properties (e.g: which scala version to use).
The child module leverage the parent properties to add some dependencies (e.g: json4s)

Upon running renovate, dependencies using parent properties are getting skip because renovate is not able to resolve the dependencies.

## How to reproduce:

```shell
pushd parent
mvn install
popd

pushd child
mvn install

# Run renovate in debug mode
LOG_LEVEL=DEBUG npx renovate --platform=local


popd
```

Upon running the `npx renovate` command, you should notice the following error:

```json
{
         "maven": [
           {
             "datasource": "maven",
             "packageFile": "pom.xml",
             "deps": [
               {
                 "datasource": "maven",
                 "depName": "com.coveo:parent",
                 "currentValue": "1.0-SNAPSHOT",
                 "fileReplacePosition": 514,
                 "registryUrls": ["https://repo.maven.apache.org/maven2"],
                 "depType": "parent",
                 "updates": [],
                 "packageName": "com.coveo:parent",
                 "versioning": "maven",
                 "warnings": [
                   {
                     "topic": "com.coveo:parent",
                     "message": "Failed to look up maven package com.coveo:parent"
                   }
                 ]
               },
               {
                 "datasource": "maven",
                 "depName": "org.json4s:json4s-core_${scala-parent.scala-lib.version}",
                 "currentValue": "4.0.7",
                 "fileReplacePosition": 965,
                 "registryUrls": ["https://repo.maven.apache.org/maven2"],
                 "depType": "compile",
                 "skipReason": "name-placeholder",
                 "updates": [],
                 "packageName": "org.json4s:json4s-core_${scala-parent.scala-lib.version}"
               }
             ],
             "packageFileVersion": "1.0-SNAPSHOT"
           }
         ]
       }
```