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

* So then : 
  * In one shell session, run `docker-compose up -d && docker-compose logs -f`, and look at the logs.
  * Open anoher paralell shell session, goto the docker-compose home folder for step 6, and run : 
  
```bash
docker-compose exec kafka-1 bash -c "kafka-topics --zookeeper zookeeper:2181 --list"
```  
  * example output : 
  
```bash
jbl@poste-devops-typique:~/a-kafka-history/step6$ docker-compose exec kafka-1 bash -c "kafka-topics --zookeeper zookeeper:2181 --list" 
__confluent.support.metrics
__consumer_offsets
simple-stream-KSTREAM-AGGREGATE-STATE-STORE-0000000004-changelog
simple-stream-KSTREAM-AGGREGATE-STATE-STORE-0000000004-repartition
simple-stream-KSTREAM-AGGREGATE-STATE-STORE-0000000012-changelog
simple-stream-KSTREAM-AGGREGATE-STATE-STORE-0000000012-repartition
telegraf
telegraf-10s-window-count
telegraf-global-count
telegraf-input-by-thread
telegraf-length-divisible-by-3
telegraf-length-divisible-by-5
telegraf-length-divisible-by-neither-3-nor-5

```
  
  * Then in a third shell session, go and run : 
```bash
docker-compose exec kafka-1 bash -c "kafka-console-consumer --bootstrap-server localhost:9092 --topic telegraf-input-by-thread --from-beginning"
```
  * Example output : 
```bash
jbl@poste-devops-typique:~/a-kafka-history/step6$ docker-compose exec kafka-1 bash -c "kafka-console-consumer --bootstrap-server localhost:9092 --topic telegraf-input-by-thread --from-beginning" 
[...]
simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,io.confluent.docker.build.number=1,host=c81929cb333f,container_name=step6_stream_1,com.docker.compose.oneoff=False,io.confluent.docker=true,engine_host=poste-devops-typique,com.docker.compose.version=1.25.2,container_version=unknown,com.docker.compose.project=step6,com.docker.compose.container-number=1,io.confluent.docker.git.id=a8965a7,com.docker.compose.service=stream,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.project.config_files=docker-compose.yml,container_image=step6_stream,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,cpu=cpu-total usage_in_usermode=21880000000i,usage_in_kernelmode=6120000000i,usage_system=17065910000000i,throttling_throttled_periods=0i,usage_total=33021209565i,throttling_periods=0i,throttling_throttled_time=0i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d",usage_percent=4.2744764677804294 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,com.docker.compose.service=stream,engine_host=poste-devops-typique,com.docker.compose.oneoff=False,host=c81929cb333f,com.docker.compose.project=step6,com.docker.compose.container-number=1,io.confluent.docker=true,com.docker.compose.project.config_files=docker-compose.yml,container_name=step6_stream_1,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,io.confluent.docker.build.number=1,com.docker.compose.version=1.25.2,container_version=unknown,cpu=cpu0,io.confluent.docker.git.id=a8965a7,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,container_image=step6_stream usage_total=7577225470i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d" 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,io.confluent.docker.git.id=a8965a7,io.confluent.docker=true,com.docker.compose.version=1.25.2,container_version=unknown,com.docker.compose.project=step6,engine_host=poste-devops-typique,cpu=cpu1,io.confluent.docker.build.number=1,com.docker.compose.service=stream,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,com.docker.compose.oneoff=False,container_name=step6_stream_1,container_image=step6_stream,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.project.config_files=docker-compose.yml,host=c81929cb333f,com.docker.compose.container-number=1 usage_total=8057882154i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d" 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,engine_host=poste-devops-typique,com.docker.compose.project=step6,com.docker.compose.container-number=1,cpu=cpu3,io.confluent.docker=true,io.confluent.docker.build.number=1,com.docker.compose.project.config_files=docker-compose.yml,com.docker.compose.version=1.25.2,container_name=step6_stream_1,com.docker.compose.service=stream,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,host=c81929cb333f,container_version=unknown,io.confluent.docker.git.id=a8965a7,com.docker.compose.oneoff=False,container_image=step6_stream usage_total=9589098322i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d" 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_net,com.docker.compose.service=stream,container_image=step6_stream,com.docker.compose.oneoff=False,network=eth0,com.docker.compose.project.config_files=docker-compose.yml,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,com.docker.compose.project=step6,com.docker.compose.container-number=1,io.confluent.docker.build.number=1,io.confluent.docker.git.id=a8965a7,io.confluent.docker=true,engine_host=poste-devops-typique,host=c81929cb333f,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.version=1.25.2,container_name=step6_stream_1,container_version=unknown tx_bytes=16260307i,rx_bytes=14836892i,tx_packets=38061i,tx_dropped=0i,rx_packets=102659i,tx_errors=0i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d",rx_dropped=0i,rx_errors=0i 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_blkio,io.confluent.docker.git.id=a8965a7,com.docker.compose.oneoff=False,com.docker.compose.container-number=1,device=8:0,io.confluent.docker=true,com.docker.compose.version=1.25.2,engine_host=poste-devops-typique,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,container_name=step6_stream_1,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.service=stream,io.confluent.docker.build.number=1,com.docker.compose.project.config_files=docker-compose.yml,container_version=unknown,host=c81929cb333f,container_image=step6_stream,com.docker.compose.project=step6 io_queue_recursive_async=0i,io_serviced_recursive_read=0i,io_queue_recursive_write=0i,io_serviced_recursive_write=3902i,io_serviced_recursive_async=1i,io_service_time_recursive_read=0i,io_service_time_recursive_write=853397912i,io_service_time_recursive_async=149020i,io_wait_time_total=13171793167i,io_service_bytes_recursive_sync=15986688i,io_service_bytes_recursive_total=16019456i,io_merged_recursive_async=0i,sectors_recursive=31288i,io_wait_time_write=13171793167i,io_wait_time_sync=13171739466i,io_serviced_recursive_total=3902i,io_service_time_recursive_sync=853248892i,io_wait_time_read=0i,io_merged_recursive_read=0i,io_time_recursive=2978011628i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d",io_service_bytes_recursive_write=16019456i,io_service_time_recursive_total=853397912i,io_wait_time_async=53701i,io_serviced_recursive_sync=3901i,io_queue_recursive_sync=0i,io_merged_recursive_total=5i,io_service_bytes_recursive_async=32768i,io_queue_recursive_total=0i,io_merged_recursive_write=5i,io_merged_recursive_sync=5i,io_service_bytes_recursive_read=0i,io_queue_recursive_read=0i 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_mem,container_version=1.5,com.docker.compose.oneoff=False,com.docker.compose.project.config_files=docker-compose.yml,com.docker.compose.version=1.25.2,host=c81929cb333f,engine_host=poste-devops-typique,container_name=step6_telegraf_1,container_image=telegraf,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.service=telegraf,com.docker.compose.config-hash=c24dc95371fdf5872bbed43e14ca063870f00c5494c67cd821071c88cac4ed8d,com.docker.compose.container-number=1,com.docker.compose.project=step6 active_anon=11161600i,pgmajfault=0i,unevictable=0i,usage=11862016i,usage_percent=0.14157711526969094,active_file=4096i,pgfault=5740i,pgpgout=1447i,total_pgpgin=4173i,total_rss=11161600i,total_pgpgout=1447i,inactive_file=0i,total_cache=4096i,total_inactive_anon=0i,total_inactive_file=0i,total_pgfault=5740i,container_id="c81929cb333fc92d49920d5847fe148e3266ea78e4529a5633f2e652ebf705f1",cache=4096i,total_active_anon=11161600i,total_mapped_file=0i,pgpgin=4173i,mapped_file=0i,total_writeback=0i,hierarchical_memory_limit=9223372036854771712i,total_active_file=4096i,total_pgmajfault=0i,writeback=0i,limit=8375590912i,max_usage=12152832i,inactive_anon=0i,rss=11161600i,rss_huge=0i,total_rss_huge=0i,total_unevictable=0i 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-2 docker_container_cpu,com.docker.compose.config-hash=6cc49378a4197d13e6ccef769bd2fc627d344899de1305dd4816cbcd8525af48,com.docker.compose.project=step6,com.docker.compose.oneoff=False,host=c81929cb333f,container_name=step6_stream_1,com.docker.compose.service=stream,com.docker.compose.project.config_files=docker-compose.yml,cpu=cpu2,container_image=step6_stream,container_version=unknown,io.confluent.docker.git.id=a8965a7,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,io.confluent.docker.build.number=1,com.docker.compose.version=1.25.2,engine_host=poste-devops-typique,com.docker.compose.container-number=1,io.confluent.docker=true usage_total=7797003619i,container_id="7340361dd8195644fcdb9d79f186d84cd4099a761a39a1918c391fc0d191045d" 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,container_name=step6_telegraf_1,com.docker.compose.project=step6,engine_host=poste-devops-typique,com.docker.compose.container-number=1,cpu=cpu-total,com.docker.compose.service=telegraf,host=c81929cb333f,container_version=1.5,com.docker.compose.config-hash=c24dc95371fdf5872bbed43e14ca063870f00c5494c67cd821071c88cac4ed8d,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.project.config_files=docker-compose.yml,com.docker.compose.version=1.25.2,container_image=telegraf,com.docker.compose.oneoff=False usage_system=17065970000000i,throttling_throttled_periods=0i,throttling_throttled_time=0i,container_id="c81929cb333fc92d49920d5847fe148e3266ea78e4529a5633f2e652ebf705f1",usage_in_kernelmode=840000000i,usage_in_usermode=1280000000i,throttling_periods=0i,usage_percent=0.47528939597315434,usage_total=2610636597i 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,host=c81929cb333f,engine_host=poste-devops-typique,container_name=step6_telegraf_1,com.docker.compose.config-hash=c24dc95371fdf5872bbed43e14ca063870f00c5494c67cd821071c88cac4ed8d,com.docker.compose.oneoff=False,container_version=1.5,com.docker.compose.project.config_files=docker-compose.yml,com.docker.compose.version=1.25.2,container_image=telegraf,com.docker.compose.service=telegraf,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.container-number=1,com.docker.compose.project=step6,cpu=cpu0 usage_total=626772269i,container_id="c81929cb333fc92d49920d5847fe148e3266ea78e4529a5633f2e652ebf705f1" 1583632062000000000

simple-stream-72f14f2f-878c-4e6f-a9e0-cb99756f5d05-StreamThread-1 docker_container_cpu,com.docker.compose.oneoff=False,com.docker.compose.config-hash=c24dc95371fdf5872bbed43e14ca063870f00c5494c67cd821071c88cac4ed8d,com.docker.compose.container-number=1,container_version=1.5,com.docker.compose.project=step6,engine_host=poste-devops-typique,container_image=telegraf,cpu=cpu1,com.docker.compose.version=1.25.2,host=c81929cb333f,container_name=step6_telegraf_1,com.docker.compose.project.config_files=docker-compose.yml,com.docker.compose.project.working_dir=/home/jibl/a-kafka-history/step6,com.docker.compose.service=telegraf container_id="c81929cb333fc92d49920d5847fe148e3266ea78e4529a5633f2e652ebf705f1",usage_total=579930093i 1583632062000000000
[...]
```
  * Finally in a fourth shell session, go and run : 
```bash
docker-compose exec kafka-1 bash -c "kafka-console-consumer --bootstrap-server localhost:9092 --topic telegraf-10s-window-count --property print.key=true --value-deserializer org.apache.kafka.common.serialization.LongDeserializer --from-beginning"
```
  * example output (those are the metrics restreamed by producer from the `telegraf` topic to the `windows ten seconds whatever` topic) : 

```bash
jbl@poste-devops-typique:~/a-kafka-history/step6$ docker-compose exec kafka-1 kafka-console-consumer \
>     --bootstrap-server localhost:9092 \
>     --topic telegraf-10s-window-count \
>     --property print.key=true \
>     --value-deserializer org.apache.kafka.common.serialization.LongDeserializer \
>     --from-beginning
[docker_container_cpu@1583631560000/1583631570000]	1
[docker_container_cpu@1583631560000/1583631570000]	2
[docker_container_cpu@1583631560000/1583631570000]	3
[docker_container_cpu@1583631560000/1583631570000]	4
[docker_container_cpu@1583631560000/1583631570000]	5
[docker_container_cpu@1583631560000/1583631570000]	6
[docker_container_cpu@1583631560000/1583631570000]	7
[docker_container_cpu@1583631560000/1583631570000]	8
[docker_container_cpu@1583631560000/1583631570000]	9
[docker_container_cpu@1583631560000/1583631570000]	10
[docker_container_cpu@1583631560000/1583631570000]	11
[docker_container_cpu@1583631560000/1583631570000]	12
[docker_container_cpu@1583631560000/1583631570000]	13
[docker_container_cpu@1583631560000/1583631570000]	14
[docker_container_cpu@1583631560000/1583631570000]	15
[docker_container_cpu@1583631560000/1583631570000]	16
[docker_container_cpu@1583631560000/1583631570000]	17
[docker_container_cpu@1583631560000/1583631570000]	18
[docker_container_cpu@1583631560000/1583631570000]	19
[...A lot more lines...]
```
  

# What Client to use instead of the `Java` here provided example

* Exemple de Kafka Consumer / Kafka Producer en Type Script : https://github.com/CoNarrative/kafka-typescript whi I privately backed up at https://gitlab.com/second-bureau/pegasus/pokus/exterieur/kafka/kafka-typescript.git 

