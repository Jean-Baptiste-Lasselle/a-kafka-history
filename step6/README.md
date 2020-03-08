# Objective

1. Streams

# Docker

```
$ docker-compose build
$ docker-compose up -d
$ docker-compose ps
$ docker-compose exec kafka-1 kafka-topics \
    --zookeeper zookeeper:2181 \ 
    --list
$ docker-compose exec kafka-1 kafka-console-consumer \
    --bootstrap-server localhost:9092 \
    --topic telegraf-input-by-thread \
    --from-beginning
$ docker-compose exec kafka-1 kafka-console-consumer \
    --bootstrap-server localhost:9092 \
    --topic telegraf-10s-window-count \
    --property print.key=true \
    --value-deserializer org.apache.kafka.common.serialization.LongDeserializer \
    --from-beginning
```

# The full action ?

[![screencast](https://asciinema.org/a/jfXXzDMSrGNplXj3MLdPOGEAt.png)](https://asciinema.org/a/jfXXzDMSrGNplXj3MLdPOGEAt?autoplay=1)

# Last Error to repair the initial `docker-compose up -d`

```bash
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-providers/2.12.4/surefire-providers-2.12.4.pom (2.3 kB at 235 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit4/2.12.4/surefire-junit4-2.12.4.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit4/2.12.4/surefire-junit4-2.12.4.jar (37 kB at 1.7 MB/s)

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Error: Could not find or load main class org.apache.maven.surefire.booter.ForkedBooter

Results :

Tests run: 0, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 15.241 s
[INFO] Finished at: 2020-03-08T00:41:30Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test (default-test) on project simple-streams: Execution default-test of goal org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test failed: The forked VM terminated without saying properly goodbye. VM crash or System.exit called ? -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginExecutionException
ERROR: Service 'stream' failed to build: The command 'mvn install' returned a non-zero code: 1
jbl@poste-devops-typique:~/a-kafka-history/step6$ sudo df -Th /var
[sudo] Mot de passe de jbl : 
Sys. de fichiers Type Taille Utilisé Dispo Uti% Monté sur
/dev/sda5        ext4   9,2G    2,1G  6,6G  24% /var
jbl@poste-devops-typique:~/a-kafka-history/step6$ 
```
* potential fix 1 naive : add `surefire-booter` to `<devDependencies>` in `pom.xml` : 

```xml
    <dependency>
      <groupId>org.apache.maven.surefire</groupId>
      <artifactId>surefire-booter</artifactId>
      <version>3.0.0-M4</version>
    </dependency>
```
* potential fix 2 , less naive : the `surefire-booter` is probably tobe installed as dependency of the `maven-surfire-plugin`. So the fix would be to upgrade version of the `maven-surefire-plugin`  : 
  * first, in error logs we have he running version of surefire : 

```bash
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-providers/2.12.4/surefire-providers-2.12.4.pom
```
  * second thing, surfire plugin version is not set in pom.xml, so i defaults to a default value, induced by the maven version itelf, which is maven `3.5` (pretty up to date, latest is `3.6` when I write this in `march 2020`) which seems to be `2.12.4`.
  * In `march 2020`, maven surefire plugin latest release is version `3.0.0-M4`, and to install this version we should add the following to [the `pom.xml` for the `SimpleStream` java executable](https://github.com/Jean-Baptiste-Lasselle/a-kafka-history/blob/master/step6/streams/pom.xml) : 
  
```xml
    <project>
      [...]
      <build>
        <pluginManagement>
          <plugins>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-surefire-plugin</artifactId>
              <version>3.0.0-M4</version>
            </plugin>
          </plugins>
        </pluginManagement>
      </build>
      [...]
    </project>
```

* the second non-naive fix worked gracefully, :), and : 
  * here is the `pom.xml` before my fix : 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.github.framiere</groupId>
    <artifactId>simple-streams</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!-- test -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.github.charithe</groupId>
            <artifactId>kafka-junit</artifactId>
            <version>4.1.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.8.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>install</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <appendAssemblyId>false</appendAssemblyId>
                            <archive>
                                <manifest>
                                    <mainClass>com.github.framiere.SimpleStream</mainClass>
                                </manifest>
                            </archive>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```
  * here is the `pom.xml` after I added the forced explicit surefire plugin version, in my fix : 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.github.framiere</groupId>
    <artifactId>simple-streams</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!-- test -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.github.charithe</groupId>
            <artifactId>kafka-junit</artifactId>
            <version>4.1.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.8.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>install</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <appendAssemblyId>false</appendAssemblyId>
                            <archive>
                                <manifest>
                                    <mainClass>com.github.framiere.SimpleStream</mainClass>
                                </manifest>
                            </archive>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-surefire-plugin</artifactId>
              <version>3.0.0-M4</version>
            </plugin>

        </plugins>
    </build>
</project>

```



# What Client to use instead of the `Java` here provided example

* Exemple de Kafka Consumer / Kafka Producer en Type Script : https://github.com/CoNarrative/kafka-typescript whi I privately backed up at https://gitlab.com/second-bureau/pegasus/pokus/exterieur/kafka/kafka-typescript.git 

