# Release Policy

## Versioning

We use semantic versioning (cf. https://semver.org/ ) 

Versions have the following format: MAJOR.MINOR.PATCH 

MAJOR incremented on incompatible API changes 

MINOR incremented when a backwards compatible feature is added 

PATCH incremented when a backwards compatible bug fix is done 

## Release Cycles

### Retention

Only in the 2 last MAJOR versions are maintained (including the on going one).

New features are only added in the last MAJOR version.

Only patches are added to the previous MAJOR version.

- Current major version: new features and bug fixes
- Previous major version: bug fixes
- Older major versions: unsupported

### Cycle

#### MINOR

Released every new feature is deployed.

Released every new Angular version is deployed.

No breaking changes.

A MINOR version of Checkout SDK will be supported for 12 month maximum.

#### MAJOR

Released every new breaking change is requested.

Include breaking changes compared to the previous MAJOR.

A MAJOR version is supported until 2 new MAJOR are released.

## Deprecation Notices
New delivered version will have it’s release note updated in the Userguide.

Every new version will come with an “end of support” date, published in the Userguide.

Each deprecated version is marked as such in the Userguide.

 

## Angular support

Due to PCI DSS constraints, we are requested to deprecate all the SDK version using Angular version where security support expired.

### Cycle

Angular release a new version every 6 months.

Checkout will update to new Angular versions once these are released so that we support the most recent version of Angular.

Angular migration will generate:

- a new MINOR version update if the changes are backward compatibles
- a new MAJOR version update if the changes are NOT backward compatibles (typically: Visual Changes, Material version upgrade, Breaking libraries, etc ….)