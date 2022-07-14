# Maven CI/CD Example - Dependency
An example of how to do dependency versioning using Maven and a CI/CD pipeline.

This project uses the approach described in [Axel Fontaine's excellent blog post](ihttps://axelfontaine.com/blog/dead-burried.html).

## Usage

### Tagging (manual example)

To create and push a git tag with the name and version of the dependency:

`mvn scm:tag -Drevision=1.1.1`

This is roughly equivalent to:

```shell
git tag -F /tmp/maven-scm-1325820348.commit 1.1.1
git push git@github.com:dwinchell/maven-cicd-dependency.git refs/tags/1.1.1
```

The SCM plugin gets the git repo URL from pom.xml.

### Deployment (manual example)

```shell
mvn deploy -Drevision=1.1.1
```

This will deploy the artifact (i.e. jar file) to your repository (i.e. Artifactory/Nexus/similar).

### In a CI/CD Pipeline

Your pipeline should use a command like below to deploy the artifact and tag the source in the same Maven invocation. `$VERSION` should be replaced with whatever mechanism your CI tool uses to inject the version number that the jar should be published with.

```shell
mvn deploy scm:tag -Drevision=$VERSION
```

## Recommended Versioning Scheme
Caveat: It is possible for reasonable people to disagree about the best way to do this. There are probably a few good ways, but there are many bad ways.

For the purposes of these recommendations, a "release" refers to a version of the dependency for which the following are true:
1. You want to make formally available to a broad audience (like users of an API on another team), OR
2. You want to continue referring to for a long time. Short lived CI builds might be cleaned up automatically, but we want some artifacts like major releases to stick around for a while.

Recommendations:
* Use [Semantic Versioning](https://semver.org/) (semver).
* For "release" versions (as defined above):
    * Use semver, as always. If needed, use pre-release metadata for things like "beta" and "rc". I.e. "1.1.1-rc1".
* For "non-release" version (as defined above). This includes cases like "I'm build my feature branch for the 100th time today"
    * Use the semver convention for pre-release and build metadata
    * Set the pre-release metadata to the name of your branch, replacing any slashes with '-'
    * Set the build metadata to the first 7 characters of the git hash.
    * Example: "1.1.1-main-e5ea2f".

### When to Version:

* Every CI build gets a "non-release" version.
* A human should decide the major, minor, and micro versions of the non-release version, which should correspond to the next planned release. I.e. if you just released 1.0.0 and aren't planning a minor version release, non-release versions should be 1.0.1-main-A234FD1
* Record human decisions about the next release version in the pom of the thing being released. TODO: update example pom file to do this.
* When a "release build" is successful, it should be assigned a "release" version. "1.0.1-main-A234FD1" might become "1.0.1" or "1.0.1-rc1".
* "Release build" is one that:
    * Is based on the correct branch. A good way to do this is to only build releases from "main".

## How it Works

After the deployment, the following things will be true:
* Your git repository will have a new tag, with the name of the artifact and the value passed to the -Drevision argument, something like "maven-cicd-dependency-1.1.1". The scm:tag command creates and pushes this tag to the remote repository.
* Your repository will contain a new jar with a name and path like `default-libs-release-local/io/ploigos/maven-cicd-dependency/1.1.1/maven-cicd-dependency-1.1.1.jar`. Notice the version number.
* The jar will be associated with a pom file with a name and path like `default-libs-release-local/io/ploigos/maven-cicd-dependency/1.1.1/maven-cicd-dependency-1.1.1.pom`. Notice the version number.
* The published pom file will have contents like below. This is okay because Maven and the repository manager are able to determine the version using the names and paths of the jar/pom.
```xml
...
  <groupId>ploigos.io</groupId>
  <artifactId>maven-cicd-dependency</artifactId>
  <version>${revision}</version>
...
```
* The repository will also have an updated maven-metadata.xml at a path like `default-libs-release-local/io/ploigos/maven-cicd-dependency/maven-metadata.xml` with contents like:
```xml
<metadata>
  <groupId>io.ploigos</groupId>
  <artifactId>maven-cicd-dependency</artifactId>
  <versioning>
    <latest>1.1.1</latest>
    <release>1.1.1</release>
    <versions>
      <version>1.1.0</version>
      <version>1.1.1</version>
    </versions>
...
```
* Dependent projects can now refer to the jar with a dependency entry and the version number passed with the -Drevision argument:
```xml
<dependency>
  <groupId>io.ploigos</groupId>
  <artifactId>maven-cicd-dependency</artifactId>
  <version>1.1.1</version>
</dependency>
```

