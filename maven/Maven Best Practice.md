# Maven Best Practice
source: http://www.kyleblaney.com/maven-best-practices/

### 1. Gradually Learn Maven
Maven is a great tool but it has a steep learning curve. Therefore, have Maven the complete reference by your sides:
https://books.sonatype.com/mvnref-book/pdf/mvnref-pdf.pdf

### 2. Know Maven's Command-Line Options
To learn about Maven's command-line option, run **mvn -?**. Useful options include:
- -B - runs Maven in batch mode, which is particularly useful when invoking Maven from a continuous server such as Bamboo because it avoids Maven's reporting of downloading progress.
- -e - configures Maven to report detailed information about errors, which is particularly useful to debug problems that occur when Maven is invoked from a continuous server such as Atlassian Bamboo.

### 3. Know the Standard Maven Plugins
To learn about a specific Maven plugin, browse the plugin's Maven site and read the details on each of the plugin's goals.

### 4. Prefer a one-to-one relationship between artifacts and Maven Projects.
For example, if your code base creates three JAR files, have three Maven projects. If all three JAR files share common code, use a fourth Maven project to store the common code. Have the original three projects define a dependency on the common code.

### 5. Name Your Project
Even though the <name data-preserve-html-node="true" data-preserve-html-node="true"> element is optional in a pom.xml, use it! Forge services (such as Sonar) use a project's name. In IDEs it is hard to distingusih between two unnamed projects.

### 6. Put all generated artifacts in the **target** folder
The target folder is automatically deleted when you run **mvn clean**. Therefore, it's best place to put all build artifacts, such as Java .class files, JAR files, and temporary files created during the build process. Maven plugins typically place the temporary artifacts they create somewhere within the target folder.

### 7. Use a **dependencyManagement** section in a parent **pom.xml** to avoid duplicating dependency information in child projects

Maven's dependencyManagement section allows a parent pom.xml to define dependencies that are potentially reused in child projects. This avoids duplication; without the dependencyManagement section, each child project has to define its own dependency and duplicate the version, scope, and type of the dependency. For example, suppose a multi-module Java project makes extensive use of JUnit for unit testing. To specify the project dependency on JUnit, the project's parent pom.xml can use a dependencyManagement section as follows:

#### DEPENDENCY MANAGEMENT IN PARENT POM.XML
```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```
Note that each child project that uses JUnit must still specify its dependency on JUnit. However, the version and scope of the dependency should be omitted as this avoids duplication and ensures that the same dependency is used throughout the project.
#### DEPENDENCY IN CHILD POM.XML (VERSION AND SCOPE INHERITED FROM PARENT)
```xml
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
  </dependency>
</dependencies>
```

### 8. Use SNAPSHOT versions during development
SNAPSHOT versions take some getting used to, but they provide significant benefits:
- They reduce the usage of Forge resources. For example, they reduce the amount of disk space used by Nexus.
- They reduce the frequency of changes that dependent projects make. For example, when project A depends on project B and both projects are under development, project A can declare a dependency on the SNAPSHOT version of project B. That way, project A doesn't need to change its dependency every time there is a new build of project B.

### 9. Use the Maven Dependency Plugin's analyze goals to help idenfity issues in your project's dependency management

Maven Dependency Plugin - http://maven.apache.org/plugins/maven-dependency-plugin/

The Maven Dependency Plugin has an analyze goal that identifies two types of dependency issues in a project:
- Dependencies that are directly used but are not declared. (The project still compiles because it gets the dependencies transitively.)
- Dependencies that are declared but are unused.
You can configure a project's build to fail if it has any dependency warnings - http://maven.apache.org/plugins/maven-dependency-plugin/examples/failing-the-build-on-dependency-analysis-warnings.html

#### 9.1. USED UNDECLARED DEPENDENCIES
Suppose the following:
- Project A uses classes from projects B and C.
- Project A only declares a Maven Dependency on project B
- Project B uses classes from project C
- Project B declares a Maven dependency on project C.

The situation is a lurking problem because project A is relying on project B for its dependency on project C. Project A will compile until project B removes its definition of its dependency on project C. This problem is made even worse when the dependency is on a SNAPSHOT version.

When project A is analyzed suing **mvn dependency:analyze**, Maven produces the following output:
```bash
[WARNING] Used undeclared dependencies found:
[WARNING]    com.avaya.ace.aaft:foundation_services_client_api:jar:6.2.0-SNAPSHOT:compile
```
To fix the problem, project A should define a direct Maven dependency on project C.

#### 9.2. UNUSED DECLARED DEPENDENCIES
If a project declares a dependency and then does not use that dependency, mvn dependency:analyze produces the following output:
```bash
[WARNING] Unused declared dependencies found:
[WARNING]    com.gigaspaces:gs-openspaces:jar:7.1.1-b4534:compile
```
To fix the problem, the project should remove the dependency from its POM.
Unused dependencies sometimes occur ebcause the developers are unaware of Maven's dependencyManagement mechanism.

### 10. Use Maven for Dependency Management rather than writing custom code for it
For example, if one Maven project depends on a zip file created by another Maven project, have the second project create a Maven artifact and use a Maven dependency rather than using a relative path.

### 11. Use Maven for dependency management rather than using Subverson externals
For example, if one Maven project depends on resources stored in another project's repository, have the second project create a Maven artifact. Use a Maven dependency to allow the first project to access the artifact.

### 12. Do not write custom Maven code to copy or unzip files.
Use maven's standard plugins instead.
