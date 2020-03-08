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

* https://maven.apache.org/surefire/surefire-booter/dependency-info.html 
# What Client to use instead of the `Java` here provided example

* Exemple de Kafka Consumer / Kafka Producer en Type Script : https://github.com/CoNarrative/kafka-typescript whi I privately backed up at https://gitlab.com/second-bureau/pegasus/pokus/exterieur/kafka/kafka-typescript.git 

