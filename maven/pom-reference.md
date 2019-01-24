# POM Reference
1. Introduction
    1. What is the POM?
    2. Quick Overview
2. The Basics
    1. Maven Coordinates
    2. POM Relationships
        1. Dependencies
            1. Dependency Version Requirement Specification
            2. Version Order Specification
            3. Version Order Testing
            4. Exclusions
        2. Inheritance
            1. The Super POM
            2. Dependency Management
        3. Aggregation (or Multi-Module)
            1. Inheritance v. Aggregation
    3. Properties
3. Build Settings
    1. Build
        1. The BaseBuild Element Set
            1. Resources
            2. Plugins
            3. Plugin Management
        2. The Build Element Set
            1. Directories
            2. Extensions
    2. Reporting
        1. Report Sets
4. More Project Information
    1. Licenses
    2. Organization
    3. Developers
    4. Contributors
5. Environment Settings
    1. Issue Management
    2. Continuous Integration Management
    3. Mailing Lists
    4. SCM
    5. Prerequisites
    6. Repositories
    7. Plugin Repositories
    8. Distribution Management
        1. Repository
        2. Site Distribution
        3. Relocation
    9. Profiles
        1. Activation
        2. The BaseBuild Element Set (revisited)
6. Final

## 1. Introduction
### 1.1. What is the POM?
POM stands for "Project Object Model". It is an XML representation of a Maven project held in a file named **pom.xml**. When in the presence of Maven folds, speaking a project is speaking in the philosphical sense, beyond a mere collection of files containing code. A project contains configuration files, as well as the developers involved and the roles they play, the defect tracking system, the organization and licenses, the URL of where the project lives, the project's dependencies, and all of the other little pieces that come into play to give code life. It is a one-stop-shop for all things concerning the project. In fact, in the Maven world, a project need not contain any code at all, merely a pom.xml.

### 1.2. Quick Overview
Notices that **modelVersion** contains 4.0.0. That is currently the only supported POM version for both Maven 2 & 3, and is always required.
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <!-- The Basics -->
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <dependencies>...</dependencies>
  <parent>...</parent>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>
 
  <!-- Build Settings -->
  <build>...</build>
  <reporting>...</reporting>
 
  <!-- More Project Information -->
  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>
 
  <!-- Environment Settings -->
  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingLists>...</mailingLists>
  <scm>...</scm>
  <prerequisites>...</prerequisites>
  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>
  <distributionManagement>...</distributionManagement>
  <profiles>...</profiles>
</project>
```

## 2. The Basics
The POM contains all necessary information about a project, as well as configurations of plugins to be used during the build process. It is, effectively, the declarative manifestation of the "who", "what", and "where", while the build lifecycle is the "when" and "how". That is not to say that the POM cannot affect the flow of the lifecycle - it can. For example, by configuring the maven-antrun-plugin, one can effectively embed Apache Ant tasks inside of the POM. It is ultimately a declaration, however. Where as a **build.xml** tells Ant precisely what to do when it is run (procedural), a POM states its configuration (declarative). If some external force causes the lifecycle to skip the Ant plugin execution, it will not stop the plugins that are executed from doing their magic. This is unlike a **build.xml** file, where tasks are almost always dependant on the lines executed before it.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-project</artifactId>
  <version>1.0</version>
</project>
```
### 2.1. Maven Coordinates
The POM definition above is the minimum that both Maven will allow. **groupId:artifactId:version** are all required fields (although, groupId and version need not be explicitly definied if they are inherited from a parent - more on inheritance later). The three fields act much like an address and timestamp in one. This marks a specific place in a repository, acting like a coordinate system for Maven projects:
- **groupId**: This is generally unique amongst an organization or a projec
    - For example, all core Maven artifacts do (well, should) live under the groupId org.apache.maven. Group ID's do not necessarily use the dot notation, for example, the junit project. Note that the dot-notated groupId does not have to correspond to the package structure that the project contains. It is, however, a good practice to follow. When stored within a repository, the group acts much like the Java packaging structure does in an operating system. The dots are replaced by OS specific directory separators (such as '/' in Unix) which becomes a relative directory structure from the base repository. In the example given, the org.codehaus.mojo group lives within the directory $M2_REPO/org/codehaus/mojo.
- **artifactId**: The artifactId is generally the name that the project is known by. 
    - Although the groupId is important, people within the group will rarely mention the groupId in discussion (they are often all be the same ID, such as the MojoHaus project groupId: org.codehaus.mojo). It, along with the groupId, creates a key that separates this project from every other project in the world (at least, it should :)).
    - Along with the groupId, the artifactId fully defines the artifact's living quarters within the repository. In the case of the above project, my-project lives in $M2_REPO/org/codehaus/mojo/my-project.
- **version**: This is the last piece of the naming puzzle.
    - groupId:artifactId denotes a single project but they cannot delineate which incarnation of that project we are talking about. Do we want the junit:junit of 2018 (version 4.12), or of 2007 (version 3.8.2)? In short: code changes, those changes should be versioned, and this element keeps those versions in line. It is also used within an artifact's repository to separate versions from each other. my-project version 1.0 files live in the directory structure $M2_REPO/org/codehaus/mojo/my-project/1.0.
    
**Packaging**
Now that we have our address structure of groupId:artifactId:version, there is one more standard label to give us a really complete what: that is the project's packaging. In our case, the example POM for org.codehaus.mojo:my-project:1.0 defined above will be packaged as a jar. We could make it into a war by declaring a different packaging:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <packaging>war</packaging>
  ...
</project>
```
When no packaging is declared, Maven assumes the packaging is the default: jar. The valid types are Plexus role-hints (read more on Plexus for a explanation of roles and role-hints) of the component role org.apache.maven.lifecycle.mapping.LifecycleMapping. The current core packaging values are: pom, jar, maven-plugin, ejb, war, ear, rar. These define the default list of goals which execute to each corresponding build lifecycle stage for a particular package structure: see Plugin Bindings for default Lifecycle Reference for details.

### 2.2. POM Relationships


