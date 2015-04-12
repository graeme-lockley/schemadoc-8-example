# schemadoc-8-example

A really simple example of generating a schema image off of a database all through a single maven pom file.  What makes this example cool is that:

- It uses an [H2](http://www.h2database.com) database that is packaged from dependencies,
- The database is setup using [liquibase](http://www.liquibase.org) from scratch, and
- Assuming that you have [Graphviz](http://www.graphviz.org) installed in your path with /bin/sh the following graphic image will be generated.
 
![Output.png](src/main/resources/output.png)

There are two files in this repository that need to be expanded upon:

- pom.xml, and
- jsqldsl.xml


# pom.xml

The extract from the pom file that is pertinant is

```xml
 <project ...>
     <build>
         <plugins>
             ...
             <plugin>
                 <groupId>za.co.no9</groupId>
                 <artifactId>jdbc-8-maven-plugin</artifactId>
                 <version>8.0</version>
                 <dependencies>
                     <dependency>
                         <groupId>com.h2database</groupId>
                         <artifactId>h2</artifactId>
                         <version>1.3.175</version>
                     </dependency>
                     <dependency>
                         <groupId>za.co.no9</groupId>
                         <artifactId>schemadoc-handler</artifactId>
                         <version>8.0</version>
                     </dependency>
                 </dependencies>
                 <executions>
                     <execution>
                         <id>jsqldsl-sources</id>
                         <phase>generate-sources</phase>
                         <goals>
                             <goal>jsqldsl</goal>
                         </goals>
                     </execution>
                 </executions>
             </plugin>
         </plugins>
     </build>
 </project>
```

Apart from including the above there is nothing else to configure.  The configuration of the database connection and what tables need to be included in the diagram are all contained within [jsqldsl.xml](jsqldsl.xml) in the root of the project.

# jsqldsl.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration xmlns="http://www.no9.co.za/xsd/jdbc-8-maven-plugin-configuration.xsd"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://www.no9.co.za/xsd/jdbc-8-maven-plugin-configuration.xsd">
    <source>
        <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:target/movies</url>
            <username>sa</username>
        </jdbc>

        <tables>
            <include>
                <schema>PUBLIC</schema>
            </include>
            <exclude>
                <schema>PUBLIC</schema>
                <table>DATABASECHANGE.*</table>
            </exclude>
        </tables>
    </source>

    <targets>
        <target>
            <handler>za.co.no9.jdbcdry.tools.schemadoc.Handler</handler>
            <destination>target/schemadoc</destination>
            <properties>
                <property>
                    <name>driver</name>
                    <value>za.co.no9.jdbcdry.drivers.H2</value>
                </property>
                <property>
                    <name>template-output</name>
                    <value>target/schemadoc/output.dot</value>
                </property>
                <property>
                    <name>post-command</name>
                    <value>dot -T png -o target/schemadoc/output.png target/schemadoc/output.dot</value>
                </property>
            </properties>
        </target>
    </targets>
</configuration>
```

The following describes all of the important bits of this file:

- The source/jdbc elements describe the connection details to the database.
- The source/tables refers to which tables are to be included in the diagram.  Given the combination of H2 and Liquibase all tables within the PUBLIC schema are included and then the Liquibase control tables are excluded by applying the DATABASECHANGELOG.* regular expression in combination with the exclusion tag.
- The targets/target element configures the schemadoc handler and associated properties.

