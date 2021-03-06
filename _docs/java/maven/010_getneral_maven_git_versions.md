---
title: General - Maven, Versions and Git
tags: [maven,git]
category: maven
summary: "Maven artifact versions, git branches and builds."
---

# Overview

## The Need for Different Maven SNAPSHOT Versions

Git can be used with many different workflows, see for instance <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>. Usually you work with a master branch and different feature branches.
If we work with different git branches, it can be useful to build identifiable artifacts from different branches. For instance 
the different branches can be related to individual (test) environments.

# Assumptions

* Only CI Server deploys to artifact repository
* Artifacts from different branches should be distinguishable by their version
* Should work in dev environment like eclipse

# Implementation

{: .success title="No custom maven plugins"} 
In order to reduce external dependencies, we try not to use or create custom maven plugins. The main drawback of self written 
maven plugins is, that they have to be maintained (like production code), which is rarely the case.

* For all `pom.xml` the version is `<version>local-SNAPSHOT</version>`
* The CI server can set individual versions

{: .success title="No pom.xml merges"} 
There are no changes of the `pom.xml` files due to different version configurations 
in different branches. Therefore merges do not produce conflicts or unwanted overrides with respect to the version configuration.

{: .success title="No numbers for SNAPSHOT"}
When we have a dependency to a SNAPSHOT version does usually mean that the dependency should be to the newest SNAPSHOT version, 
not to an older one. When we use the traditional maven version scheme, we have SNAPSHOT versions like 1.2.3-SNAPSHOT. 
After a release build, the number is changed (to e.g. 1.2.4-SNAPSHOT). Most of the time, the dependencies to SNAPSHOTs have 
to be updated (e.g from 1.2.3-SNAPSHOT to 1.2.4-SNAPSHT). This is error-prone work.


# Versioning with CI Friendly Versions

See <https://maven.apache.org/maven-ci-friendly.html>

{: .code title="pom.xml"}
~~~xml
<groupId>...</groupId>
<artifactId>...</artifactId>
<version>${revision}</version>
  ...
<properties>
  <revision>local-SNAPSHOT</revision>
</properties>
~~~

In a multi module setup, the child module's `pom.xml` references the parent:

{: .code title="child pom.xml"}
~~~xml
...
<parent>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>${revision}</version>
</parent>
...
~~~
Note that the version of the parent is given by `${revision}`.

{: .success title="Single Place"}
With the CI friendly versions there is only one place in a maven multi module setup, where 
a concrete version is defined. The parent's version is identified by the `${revision}` property. 

{: .success title="Flatten Maven Plugin "}
From <https://maven.apache.org/maven-ci-friendly.html> section "Install / Deploy":  
> If you like to install or deploy artifacts by using the above setup you have to use the flatten-maven-plugin 
> otherwise you will install/deploy artifacts in your repository which will not be consumable by Maven anymore. 


### Eclipse
The `${revision}` is resolved to the configured value, in our case to "local-SNAPSHOT".

# CI SNAPSHOT Builds

In order to identify SNAPSHOT artifacts of the different branches, the maven version can be the <name of the branch> + "-SNAPSHOT". 
Examples:

* master: master-SNAPSHOT
* feature-123: feature-123-SNAPSHOT

The version can be set as maven command line argument `-Drevision=my-version`. 

A short script can look like:
~~~
branch=$(git rev-parse --abbrev-ref HEAD)
mvn -Drevision=$branch-SNAPSHOT ...
~~~
(Jenkins has a variable with the branch name)

# Dependency Management and Development

## Multi Module Setup

### Development
In a multi module setup all modules have the same version. The version of another module from the setup is specified by `${project.version}` 
(see <https://maven.apache.org/maven-ci-friendly.html>).

### CI SNAPHSOT Feature Build
All modules of the multi module setup have the same version which references to the revision value. 

### Merge
There are no version changes in the different `pom.xml` files.

## Multiple Independent Modules

### Development

* Modules:
* m_1 and m_2, can be in different git repositories
* m_2 depends an m_1
* Checked out with the same feature branch (same name)
* In the `pom.xml` of m_2 the version of the m_1 dependency is changed to `${project.version}`

### CI SNAPHSOT Feature Build
CI must ensure, that the builds are called with the same revision value.

### Merge
In this case, each independent module is merged individually. After each merge the dependency versions are adjusted

# CI Release Builds

Release builds are builds which have a numbered version instead of "-SNAPSHOT" suffix. In order to be reproducible, a tag 
in git is advisable.

A nice article about how to build a release can be found at <https://axelfontaine.com/blog/dead-burried.html>.

## Steps
* Checkout from git repository
* Tag
* Build (same like SNAPSHOT)
* Push to git repository

{: .info title="No source changes"}  
Because the release version is set by a command line parameter and not inside the `pom.xml` files, there are no source code 
changes. On the opposite the maven release plugin (http://maven.apache.org/maven-release/maven-release-plugin/index.html) changes 
the `pom.xml` two times. 

## Additional Optional Steps
* Calculate the revision depending on branch, available tags
* Include git information in the artifact with <https://github.com/git-commit-id/maven-git-commit-id-plugin>

