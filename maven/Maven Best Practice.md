# Maven Best Practice
source: http://www.kyleblaney.com/maven-best-practices/

## 1. Gradually Learn Maven
Maven is a great tool but it has a steep learning curve. Therefore, have Maven the complete reference by your sides:
https://books.sonatype.com/mvnref-book/pdf/mvnref-pdf.pdf

## 2. Know Maven's Command-Line Options
To learn about Maven's command-line option, run **mvn -?**. Useful options include:
- -B - runs Maven in batch mode, which is particularly useful when invoking Maven from a continuous server such as Bamboo because it avoids Maven's reporting of downloading progress.
- -e - configures Maven to report detailed information about errors, which is particularly useful to debug problems that occur when Maven is invoked from a continuous server such as Atlassian Bamboo.

## 3. Know the Standard Maven Plugins
To learn about a specific Maven plugin, browse the plugin's Maven site and read the details on each of the plugin's goals.

## 4. Prefer a one-to-one relationship between artifacts and Maven Projects.
For example, if your code base creates three JAR files, have three Maven projects. If all three JAR files share common code, use a fourth Maven project to store the common code. Have the original three projects define a dependency on the common code.

## 5. Name Your Project
Even though the <name data-preserve-html-node="true" data-preserve-html-node="true"> element is optional in a pom.xml, use it! Forge services (such as Sonar) use a project's name. In IDEs it is hard to distingusih between two unnamed projects.

## 6. Put all generated artifacts in the **target** folder
The target folder is automatically deleted when you run **mvn clean**. Therefore, it's best place to put all build artifacts, such as Java .class files, JAR files, and temporary files created during the build process. Maven plugins typically place the temporary artifacts they create somewhere within the target folder.

## 7. Use a **dependencyManagement** section in a parent **pom.xml** to avoid duplicating dependency information in child projects

Maven's dependencyManagement section allows a parent pom.xml to define dependencies that are potentially reused in child projects. This avoids duplication; without the dependencyManagement section, each child project has to define its own dependency and duplicate the version, scope, and type of the dependency. For example, suppose a multi-module Java project makes extensive use of JUnit for unit testing. To specify the project dependency on JUnit, the project's parent pom.xml can use a dependencyManagement section as follows:

### DEPENDENCY MANAGEMENT IN PARENT POM.XML
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
### DEPENDENCY IN CHILD POM.XML (VERSION AND SCOPE INHERITED FROM PARENT)
```xml
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
  </dependency>
</dependencies>
```

## 8. Use SNAPSHOT versions during development
SNAPSHOT versions take some getting used to, but they provide significant benefits:
- They reduce the usage of Forge resources. For example, they reduce the amount of disk space used by Nexus.
- They reduce the frequency of changes that dependent projects make. For example, when project A depends on project B and both projects are under development, project A can declare a dependency on the SNAPSHOT version of project B. That way, project A doesn't need to change its dependency every time there is a new build of project B.

## 9. Use the Maven Dependency Plugin's analyze goals to help idenfity issues in your project's dependency management


