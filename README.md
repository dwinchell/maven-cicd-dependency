# Maven CI/CD Example - Dependency
An example of how to do dependency versioning using Maven and a CI/CD pipeline.

This project uses the approach described in [Axel Fontaine's excellent blog post](ihttps://axelfontaine.com/blog/dead-burried.html).

## Usage

### Tagging (manual example)

To create and push a git tag with the name and version of the dependency:

`mvn scm:tag -Drevision=1.1.1`

This is roughly equivalent to:

```shell
git tag maven-cicd-dependency-1.1.1
git push git@github.com:dwinchell/maven-cicd-dependency.git refs/tags/maven-cicd-dependency-1.1.1
```

The SCM plugin gets the git repo URL from pom.xml.

### Deployment (manual example)

```shell
mvn deploy -Drevision=1.1.1
```

This will deploy the artifact (i.e. jar file) to your repository (i.e. Artifactory/Nexus/similar).

### In a CI/CD Pipeline

Your pipeline should use a command like below to deploy the artifact and tag the source in the same Maven invocation. `$REVISION` should be replaced with whatever mechanism your CI tool uses to inject the version number that the jar should be published with.

```shell
mvn deploy scm:tag -Drevision=$REVISION
```


## How it Works

After the deployment, the following things will be true:
* Your git repository will have a new tag, with the name of the artifact and the value passed to the -Drevision argument, something like "maven-cicd-dependency-1.1.1". The scm:tag command creates and pushes this tag to the remote repository.
* Your repository will contain a new jar with a URL similar to https://ploigos2.jfrog.io/artifactory/default-libs-release/ploigos/io/maven-cicd-dependency/1.1.2/maven-cicd-dependency-1.1.2.jar.
* The jar will be associated with a pom file with a name and path like `default-libs-release-local/ploigos/io/maven-cicd-dependency/1.1.1/maven-cicd-dependency-1.1.1.pom`. Notice the version in the name.
* The published pom file will have contents like below. This is okay because Maven and the repository manager are able to determine the version using the path.
```xml
...
  <groupId>ploigos.io</groupId>
  <artifactId>maven-cicd-dependency</artifactId>
  <version>${revision}</version>
...
```
* The repository will also have an updated maven-metadata.xml at a path like `default-libs-release-local/ploigos/io/maven-cicd-dependency/maven-metadata.xml` with contents like:
```xml
<metadata>
  <groupId>ploigos.io</groupId>
  <artifactId>maven-cicd-dependency</artifactId>
  <versioning>
    <release>1.1.1</release>
    <versions>
      <version>1.1.1</version>
    </versions>
```
* Dependent projects can now refer to the jar with a dependency entry and the version number passed with the -Drevision argument:
```xml
<dependency>
  <groupId>ploigos.io</groupId>
  <artifactId>maven-cicd-dependency</artifactId>
  <version>${revision}</version>
</dependency>
```

