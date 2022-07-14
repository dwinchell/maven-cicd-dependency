# Maven CI/CD Example - Dependency
An example of how to do dependency versioning using Maven and a CI/CD pipeline.

This project uses the approach described in [Axel Fontaine's excellent blog post](ihttps://axelfontaine.com/blog/dead-burried.html).

## Usage

### To create and push a git tag with the name and version of the dependency

`mvn scm:tag -Drevision=1.1.1`

This is roughly equivalent to:

```shell
git tag maven-cicd-dependency-1.1.1
git push git@github.com:dwinchell/maven-cicd-dependency.git refs/tags/maven-cicd-dependency-1.1.1
```

The SCM plugin gets the git repo URL from pom.xml.

