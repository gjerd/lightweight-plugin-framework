# Lightweight Plugin Framework (Fork)
![Release](https://badgen.net/badge/Release/0.0.8-SNAPSHOT/blue)
[![License](https://badgen.net/badge/License/Apache&nbsp;2.0/blue)](https://opensource.org/licenses/Apache-2.0)

## 0. Companion project for gradle
    https://github.com/gjerd/lightweight-plugin-framework-gradle


## 1. Features
- Simple api
- Sandboxing with custom java classloader
- Build plugins with maven



## 2. Usage

#### 2.1 Create plugin

Create Plugin class that implements ljpf.Plugin interface.

```java
public class CustomPlugin implements Plugin {

    @Override
    public void load() {
    }

    @Override
    public void unload() {
    }
}
```

Create descriptor file for corresponding plugin and place it in project resources.
Descriptor file must have .plugin extension.

src/main/resources/custom.plugin

```properties
id=CustomPluginId
version=0.0.1
pluginClass=ljpf.examples.plugin.CustomPlugin
description=My Custom plugin
```

#### 2.2 Create app

Load plugins in your main application using PluginManager interface. Use plugin id from descriptor file to load extensions.
Plugin repository determines way of loading plugins. Base case is to load jars from classpath plugins/ directory.


```java
public class App {

    public static void main(String[] args) {

        PluginRepository pluginRepository = new DirPluginRepository("plugins");
        PluginManager pluginManager = new DefaultPluginManager(pluginRepository);

        pluginManager.load("CustomPluginId");
    }

}
```

#### 2.3 Build with Maven

#### Build plugin with Maven

Use lightweight-plugin-framework-maven-plugin to create maven artifact (*-plugin.jar).
Maven will create fat jar with plugin code and all its dependencies. 

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.javacentric.gjerd</groupId>
                <artifactId>lightweight-plugin-framework-maven-plugin</artifactId>
                <version>0.0.8-SNAPSHOT</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>make-plugin</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
        ....    
    
    </build>
```

#### Add plugins to app

lightweight-plugin-framework-maven-plugin can also prepare plugins/ directory in your application.

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.javacentric.gjerd</groupId>
                <artifactId>lightweight-plugin-framework-maven-plugin</artifactId>
                <version>0.0.8-SNAPSHOT</version>
                <executions>
                    <execution>
                        <phase>process-sources</phase>
                        <goals>
                            <goal>make-plugin-repository</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/plugins</outputDirectory>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>ljpf.examples.plugin</groupId>
                                    <artifactId>custom-plugin</artifactId>
                                    <version>0.0.1</version>
                                </artifactItem>
                            </artifactItems>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            ... 
               
        </plugins>
        ....    
    
    </build>
```



## 3. Plugin Repositories

#### DirPluginRepository
TODO Loads plugins from given directory

#### ClasspathPluginRepository
TODO Loads plugins from java classpath

#### MultiPluginRepository
TODO Enables mixing multiple plugin reposiotories

## 4. Roadmap

- [x] Updated all maven dependencies, not maven-assembly-plugin & maven-dependency-plugin
- [x] Updated code to work with maven-assembly-plugin & maven-dependency-plugin version 3.6.0
- [ ] Make a fully working Gradle Example utilizing this framework
- [ ] Make a Gradle plugin for distributing plugins



## 5. Debugging
In app create plugins dir and run or place plugins as dependencies

```xml
    <dependencies>
        <dependency>
            <groupId>ljpf.examples.plugin</groupId>
            <artifactId>custom-plugin</artifactId>
            <version>0.0.8-SNAPSHOT</version>
        </dependency>
        ...
        
    </dependencies>
```


## 6. Examples
See example project [here](https://github.com/souzen/lightweight-plugin-framework/tree/master/examples)

Build and run example

```
mvn clean package
./run tree

Should result in the following output:

app-0.0.8-SNAPSHOT
├── config
│   └── logback.xml
├── lib
│   ├── app-0.0.8-SNAPSHOT.jar
│   ├── common-api-0.0.8-SNAPSHOT.jar
│   ├── commons-lang3-3.5.jar
│   ├── lightweight-plugin-framework-0.0.8-SNAPSHOT.jar
│   ├── logback-classic-1.4.11.jar
│   ├── logback-core-1.4.11.jar
│   ├── lombok-1.18.30.jar
│   └── slf4j-api-2.0.9.jar
├── plugins
│   ├── first-plugin
│   │   ├── first.plugin
│   │   └── lib
│   │       ├── first-plugin-0.0.8-SNAPSHOT.jar
│   │       ├── spring-core-5.2.19.RELEASE.jar
│   │       └── spring-jcl-5.2.19.RELEASE.jar
│   ├── META-INF
│   │   └── MANIFEST.MF
│   ├── second-plugin
│   │   ├── lib
│   │   │   ├── second-plugin-0.0.8-SNAPSHOT.jar
│   │   │   ├── spring-core-5.2.19.RELEASE.jar
│   │   │   └── spring-jcl-5.2.19.RELEASE.jar
│   │   └── second.plugin
│   └── third-plugin
│       ├── additional.properties
│       ├── lib
│       │   └── third-plugin-0.0.8-SNAPSHOT.jar
│       └── third.plugin
├── start
└── start.bat

11 directories, 23 files

10:22:40.648 [main] INFO  ljpf.examples.app.App - Initializing, using Lombok...
10:22:40.652 [main] WARN  DirPluginRepository - Dir Plugin Repository not found /home/user/Sources/lightweight-plugin-framework/plugins
10:22:40.652 [main] DEBUG DirPluginRepository - Loading plugins from directory: /home/user/Sources/lightweight-plugin-framework/examples/app/target/plugins
10:22:40.656 [main] DEBUG BasePluginRepository - Plugin added: SecondPlugin /home/user/Sources/lightweight-plugin-framework/examples/app/target/plugins/second-plugin
10:22:40.656 [main] DEBUG BasePluginRepository - Plugin added: FirstPlugin /home/user/Sources/lightweight-plugin-framework/examples/app/target/plugins/first-plugin
10:22:40.656 [main] DEBUG BasePluginRepository - Plugin added: ThirdPlugin /home/user/Sources/lightweight-plugin-framework/examples/app/target/plugins/third-plugin
10:22:40.659 [main] DEBUG FirstPlugin - Load [classloader ljpf.loader.ParentLastClassLoaderFactory$ParentLastClassLoader@45afc369]
10:22:40.660 [main] INFO  FirstPlugin - Spring version 5.2.19.RELEASE
10:22:40.660 [main] DEBUG DefaultPluginManager - Plugin Loaded: Plugin{id='FirstPlugin', version=0.0.8-SNAPSHOT}
10:22:40.661 [main] DEBUG SecondPlugin - Load [classloader ljpf.loader.ParentLastClassLoaderFactory$ParentLastClassLoader@1176dcec]
10:22:40.661 [main] INFO  SecondPlugin - Spring version 5.2.19.RELEASE
10:22:40.661 [main] DEBUG DefaultPluginManager - Plugin Loaded: Plugin{id='SecondPlugin', version=0.0.8-SNAPSHOT}
10:22:40.662 [main] DEBUG ThirdPlugin - Load [classloader ljpf.loader.ParentLastClassLoaderFactory$ParentLastClassLoader@6b67034]
10:22:40.662 [main] INFO  ThirdPlugin - Hello World!
10:22:40.662 [main] INFO  ThirdPlugin - Hello From Plugin Resource!
10:22:40.662 [main] DEBUG DefaultPluginManager - Plugin Loaded: Plugin{id='ThirdPlugin', version=0.0.8-SNAPSHOT}
10:22:40.662 [main] INFO  ljpf.examples.app.App - Shutting down...
10:22:40.662 [main] INFO  FirstPlugin - Unload
10:22:40.662 [main] DEBUG DefaultPluginManager - Plugin Unloaded: Plugin{id='FirstPlugin', version=0.0.8-SNAPSHOT}
10:22:40.662 [main] INFO  SecondPlugin - Unload
10:22:40.662 [main] DEBUG DefaultPluginManager - Plugin Unloaded: Plugin{id='SecondPlugin', version=0.0.8-SNAPSHOT}
10:22:40.662 [main] INFO  ThirdPlugin - Unload
10:22:40.662 [main] DEBUG DefaultPluginManager - Plugin Unloaded: Plugin{id='ThirdPlugin', version=0.0.8-SNAPSHOT}

-------------------------------

The java command to run the app, do not include the plugins, as you can see they are
dynamically loaded, so the program does it's job and works as expected.
- The individual plugins can also use different libs/dependencies.

java -classpath "$(dirname $0)/lib/*" ljpf.examples.app.App

eg. it loads and resolves all jars inside the lib directory;
(yes, the app's jar itself is there too, when it's done the apps class containing the main
method is executed) - The main program loads the plugins dynamically, executing them
the way the main program want - yay!
 
```



## Licence
Copyright 2017 Luke Sosnicki, 2023- Arild G. Gjerd

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
