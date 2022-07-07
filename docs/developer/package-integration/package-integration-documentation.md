# Big Bang Package: Documentation

Big Bang requires some additional documentation for supported packages to help user's understand how it interacts with other components.  The following are documents that should be created or updated for integration into Big Bang:


- Reference Package Architecture: See Big Bang's [Reference Package Architecture here](/docs/understanding_bigbang/package_architecture/ref-package/).  
- Overviews of other package architectures are located in the [package achitecture folder](/docs/understanding_bigbang/package_architecture/)
- [Default Credentials](/docs/guides/using_bigbang/default_credentials.md)
- [Licensing](/docs/understanding_bigbang/licensing_model.md))
- [Minimum Hardware Requirements](/docs/prerequisites/minimum_hardware_requirements.md)

## Documentation for how to upgrade the Package helm charts

The Package chart should include maintenance documentation that describes
1. The process for upgrading
2. How to test the upgrade
3. Documentation for changes made to the upstream chart.

## Configuration

For every package, we need to inform users how to configure the application.
This includes:

* Pre-installation configuration parameters
* Post-installation package configuration

## User Experience

* Documentation should be easily accessible to an end user when BigBang is installed
* Needs to be air-gapped friendly for those in disconnected environments
* Links should be clickable within the documentation. i.e. sonarqube.bigbang.dev might be sonarqube.customer.mil