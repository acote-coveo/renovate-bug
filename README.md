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