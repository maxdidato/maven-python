# MAVEN PYTHON INTEGRATION

This simple project shows how to package a python project in a zip file, push it to the local maven repository (it can be deployed on nexus for example setting the proper deployment on maven), and use it via another project having the python artifact as dependency.

# Execution

Simply execute 
```
mvn clean install
```
in the root folder. 
This will generate the zip version of the simple hello world python script and utilise it in a different maven module

### Packaging a python project with Maven

The module "python" included in this codebase contains a very simple python script which prints on the stdout the string "Hello, this is my packaged python app". This is coded in the file __helloworld.py__.
To define a default entrypoint a ____main__.py__ module is provided which is simply including helloworld module. 
This is required by python interpreter when launching a zip package, so to provide a default entrypoint.

This project also include a __pom.xml__ file which, using the maven assembly plugin, will create a package using the assembly definition described in the file __dep.xml__

The assembly descriptor simply package all the python scripts in a zip file, not including the root folder, so that the files are directly included in the zip file without any further folder. 
__It assigns bin as classifier and it will add the bin suffix to the artifact__

### Using a packaged python program

The second module included in this project is __pythondependent__
This is a maven project which will include the zip file created previously and installed in the local maven repo and execute the python script using the __exec_maven_plugin_ 

The first thing to do is to include the zip dependency

```xml
 <dependency>
        <groupId>test</groupId>
        <artifactId>pythonscript</artifactId>
        <classifier>bin</classifier>//same classifier as in the assembly 
        <version>1.0</version>
        <type>zip</type>//this specify that the dependency is a zip
</dependency>
```

The second thing is to include all the dependencies in the target folder. In this case all the dependencies boil down to only the zipped python app, if a project with more dependencies doesn't need all of them to be packaged, the filter plugin can be used.

```xml
<executions>
        <execution>
            <phase>compile</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}</outputDirectory>
            </configuration>
        </execution>
    </executions>
```
Here we are saying to copy all the dependencies and place them in the target directory before starting compiling (I might need the dependency to generate more code)

Eventually we can execute the python script using maven exec plugin. 

```xml
<plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.5.0</version>
        <executions>
            <execution>
                <id>python-build</id>
                <phase>compile</phase>
                <goals>
                    <goal>exec</goal>
                </goals>
                <configuration>
                    <executable>${env.PYTHONPATH}</executable>
                    <arguments>
                        <argument>${project.build.directory}/pythonscript-1.0-bin.zip</argument>
                    </arguments>
                </configuration>
            </execution>
        </executions>
</plugin>
```
During the compilation goal I want to execute my python script (of course the goal can be everything needed). 
The zip file in the __python__ project is generated using the standard <artifactId>-<version>-<classifier>.zip but can be modified using maven plugins. 

If we run 
```
mvn clean install 
```
from the root project, everything will compile nicely and we should see 
```
[INFO] --- exec-maven-plugin:1.5.0:exec (python-build) @ python-dependent ---
Hello, this is my packaged python app