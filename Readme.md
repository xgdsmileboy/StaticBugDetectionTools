#### Testing Environment

---

* OS: Mac OSX

* Java:  version "1.8.0_211"

* Maven: Apache Maven 3.5.0

  

#### Run Infer

---

* Run in command line
  1. Download or install infer and configure environment correctly.
2. In the project directory, running `infer run -- mvn compile` or `infer run -- javac Test.java`

* References
  * [https://github.com/facebook/infer](https://github.com/facebook/infer)
  * [https://fbinfer.com/docs/getting-started.html](https://fbinfer.com/docs/getting-started.html)



#### Run Error-Prone

---


* Run ine command line

  1. Download all dependencies:

     ```shell
     wget https://repo1.maven.org/maven2/com/google/errorprone/error_prone_core/2.3.4/error_prone_core-2.3.4-with-dependencies.jar
     wget https://repo1.maven.org/maven2/org/checkerframework/dataflow/2.5.7/dataflow-2.5.7.jar
     wget https://repo1.maven.org/maven2/org/checkerframework/javacutil/2.5.7/javacutil-2.5.7.jar
     wget https://repo1.maven.org/maven2/com/google/code/findbugs/jFormatString/3.0.0/jFormatString-3.0.0.jar
     wget https://repo1.maven.org/maven2/com/google/errorprone/javac/9+181-r4173-1/javac-9+181-r4173-1.jar
     ```

  2. Example code of `ShortSet`:

     ```java
     import java.util.Set;
     import java.util.HashSet;
     
     public class ShortSet {
       public static void main (String[] args) {
         Set<Short> s = new HashSet<>();
         for (short i = 0; i < 100; i++) {
           s.add(i);
           s.remove(i - 1);
         }
         System.out.println(s.size());
       }
     }
     ```

  3. Run command line as:

     ```shell
     javac \
       -J-Xbootclasspath/p:javac-9+181-r4173-1.jar \
       -XDcompilePolicy=simple \
       -processorpath error_prone_core-2.3.4-with-dependencies.jar:dataflow-2.5.7.jar:javacutil-2.5.7.jar:jFormatString-3.0.0.jar \
       '-Xplugin:ErrorProne -XepDisableAllChecks -Xep:CollectionIncompatibleType:ERROR' \
       ShortSet.java
     ```

     If the above commond line encounters the following error:

     ```shell
     java.lang.NoClassDefFoundError: com/github/benmanes/caffeine/cache/Caffeine
     ```

     download dependency `caffeine-1.0.0.jar`, and run the following command line:

     ```shell
     javac \
       -J-Xbootclasspath/p:javac-9+181-r4173-1.jar \
       -XDcompilePolicy=simple \
       -processorpath caffeine-1.0.0.jar:error_prone_core-2.3.4-with-dependencies.jar:dataflow-2.5.7.jar:javacutil-2.5.7.jar:jFormatString-3.0.0.jar \
       '-Xplugin:ErrorProne -XepDisableAllChecks -Xep:CollectionIncompatibleType:ERROR' \
       ShortSet.java
     ```

  4. The output should be like:

     ```shell
     ShortSet.java:16: error: [CollectionIncompatibleType] Argument 'i - 1' should not be passed to this method; its type int is not compatible with its collection's type argument Short
           s.remove(i - 1);
                   ^
         (see https://errorprone.info/bugpattern/CollectionIncompatibleType)
       Did you mean '@SuppressWarnings("CollectionIncompatibleType") public static void main (String[] args) {'?
     1 error
     ```

     

* Run with maven plugin

  * Add the following dependency to the `pom.xml` file.

    ```xml
      <properties>
        <javac.version>9+181-r4173-1</javac.version>
      </properties>
    
      <!-- using github.com/google/error-prone-javac is required when running on JDK 8 -->
      <profiles>
        <profile>
          <id>jdk8</id>
          <activation>
            <jdk>1.8</jdk>
          </activation>
          <build>
            <plugins>
              <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                  <fork>true</fork>
                  <compilerArgs combine.children="append">
                    <arg>-J-Xbootclasspath/p:${settings.localRepository}/com/google/errorprone/javac/${javac.version}/javac-${javac.version}.jar</arg>
                  </compilerArgs>
                </configuration>
              </plugin>
            </plugins>
          </build>
        </profile>
      </profiles>
    ```

* References


  * [https://github.com/google/error-prone](https://github.com/google/error-prone)
  * [https://github.com/google/error-prone/issues/1445](https://github.com/google/error-prone/issues/1445)
  * [https://errorprone.info/index](https://errorprone.info/index)
  * [https://errorprone.info/docs/installation](https://errorprone.info/docs/installation)
  * [https://github.com/google/error-prone/blob/master/examples/maven/pom.xml](https://github.com/google/error-prone/blob/master/examples/maven/pom.xml)
  * [https://errorprone.info/api/latest/](https://errorprone.info/api/latest/)

#### Run SpotBugs

---

* Run as maven plugin, 

  1. add the following dependency

     ```xml
     <!-- Add the following to dependencies -->
     <dependency>
       <groupId>com.github.spotbugs</groupId>
       <artifactId>spotbugs</artifactId>
       <version>4.0.0-beta5</version>
     </dependency>
     
     <!-- Add the following for bug report -->
     <reporting>
       <plugins>
         <plugin>
           <groupId>com.github.spotbugs</groupId>
           <artifactId>spotbugs-maven-plugin</artifactId>
           <version>3.1.12.2</version>
           <configuration>
             <xmlOutput>true</xmlOutput>
             <!-- Optional directory to put spotbugs xdoc xml report -->
             <xmlOutputDirectory>target/site</xmlOutputDirectory>
           </configuration>
         </plugin>
       </plugins>
     </reporting>
     ```

  2. Then, run `mvn compile site`, if encounter the following error：

     ```shell
     [ERROR] Failed to execute goal org.apache.maven.plugins:maven-site-plugin:3.3:site (default-site) on project GenPat: Execution default-site of goal org.apache.maven.plugins:maven-site-plugin:3.3:site failed: A required class was missing while executing org.apache.maven.plugins:maven-site-plugin:3.3:site: org/apache/maven/doxia/siterenderer/DocumentContent
     ```

     Solution:  Do not use the default `maven-site-plugin:3.3` plugin, upgrade it to the latest version, for example, 3.7.1

     ```xml
     <build>
       <plugins>
     
         <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-site-plugin</artifactId>
           <version>3.7.1</version>
         </plugin>
     
         <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-project-info-reports-plugin</artifactId>
           <version>3.0.0</version>
         </plugin>
     
       </plugins>
     </build>
     ```

* References
  * [https://github.com/spotbugs/spotbugs](https://github.com/spotbugs/spotbugs)
  * [https://mkyong.com/maven/maven-spotbugs-example/](https://mkyong.com/maven/maven-spotbugs-example/)
  * [https://spotbugs.readthedocs.io/en/latest/maven.html](https://spotbugs.readthedocs.io/en/latest/maven.html)
  * [https://spotbugs.github.io/spotbugs-maven-plugin/usage.html](https://spotbugs.github.io/spotbugs-maven-plugin/usage.html)





