# renovate-bug

Reproduction of a renovate bug where dependencies using a property from a parent maven module is not resolved.

## Project setup

- Parent module (parent)
  - Child module (child)

The parent module define some properties (e.g: which scala version to use):

```xml
    <properties>
        <scala-parent.scala-lib.version>2.12</scala-parent.scala-lib.version>
    </properties>
```

The child module leverage the parent properties to add some dependencies (e.g: json4s):

```xml
<parent>
        <groupId>com.coveo</groupId>
        <artifactId>parent</artifactId>
        <version>1.0-SNAPSHOT</version>
</parent>

<properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

<dependencies>
    <dependency>
        <groupId>org.json4s</groupId>
        <artifactId>json4s-core_${scala-parent.scala-lib.version}</artifactId>
        <version>4.0.7</version>
    </dependency>
</dependencies>
```

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

```json5
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
                   // This warning is due to the fact that I haven't deployed my package.
                   // Internally our parent module gets resolved properly but we still have the skipReason name-placeholder bellow.
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