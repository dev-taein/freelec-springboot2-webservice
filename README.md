# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스   
http://ec2-15-165-198-250.ap-northeast-2.compute.amazonaws.com/   
#### AWS의 무료 이용 기간 1년이므로 링크의 주소가 만료될 수 있습니다.<br>(소셜 로그인 후 글 등록은 사용 할 수없습니다.제가 DB에서 직접 권한을 바꾸어야 등록할 수 있습니다.
------------
## 목적
+ 스프링부트와 AWS 무중단 배포 경험 쌓기
------------
# Build.gradle 설정
+ Tools : Intellij IDEA, Gradle 4.8, Maria DB
> build.gradle 소스 코드
```
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}


apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'


group 'com.sproject.book'
version '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
    compile('org.springframework.boot:spring-boot-starter-mustache')
    compile('org.springframework.boot:spring-boot-starter-oauth2-client')
    compile('org.springframework.session:spring-session-jdbc')
    compile('org.mariadb.jdbc:mariadb-java-client')
    testCompile("org.springframework.security:spring-security-test")
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```


------------
# 동작 (로그인, 글 등록, 수정, 삭제)
![글등록](https://user-images.githubusercontent.com/77142806/130397525-f0ec15f3-9ee3-419c-a1a7-c9a98ccd7464.gif)

------------
> 로그인 시 MariaDB user 테이블에 사용자가 기록되며 ROLE를 GUEST에서 USER로 변경하면 글 등록이 가능합니다. 
![mariadb](https://user-images.githubusercontent.com/77142806/130398845-2facea14-3e71-4d9d-93f1-57a942da2cf1.PNG)


------------
> 소셜로그인을 위해 네이버와 구글의 로그인 API를 등록하였습니다.   
![naverAPI](https://user-images.githubusercontent.com/77142806/130398852-468db268-5a2b-4127-a943-7d8a0b8d6483.PNG)
![googleAPI](https://user-images.githubusercontent.com/77142806/130398864-f246ea2d-1f7b-4e91-881c-8b0877542672.PNG)


------------
> 아마존 AWS 인스턴스 생성
![EC2인스턴스](https://user-images.githubusercontent.com/77142806/130398862-e0214538-ac52-407c-b78c-e16e7ad07619.PNG)


------------
> putty를 이용하여 아마존 EC2 서버 접속   
![puttyEC2서버접속](https://user-images.githubusercontent.com/77142806/130398853-57711309-ceeb-498a-8819-a8788e9e30b2.PNG)
------------


> putty 배포 스크립트 작성 deploy.sh 등   
![putty배포스크립트작성](https://user-images.githubusercontent.com/77142806/130398855-96821bdb-2caa-4f88-be1a-5edd3fc4b7cc.PNG)
![putty배포스크립트작성2](https://user-images.githubusercontent.com/77142806/130398856-807afae6-2976-41e2-8d46-1851e7473014.PNG)
------------


> 아마존 RDS 설정
![RDS](https://user-images.githubusercontent.com/77142806/130398857-ef5b9c23-83c0-4bb9-9787-bbdc07799bfa.PNG)
------------


> CI 배포 자동화
![Travis CI](https://user-images.githubusercontent.com/77142806/130398859-6c41a9f3-19a4-45a1-bf77-94a213bbc865.PNG)
> 실제 배포 성공 시 로그
```
Build system information
0.21s0.05s0.00s0.04s0.00s0.01s0.01s0.01s0.01s0.01s0.00s0.00s0.02s0.00s0.01s0.33s0.00s0.00s0.00s0.01s0.00s0.10s0.00s0.76s0.00s0.11s6.03s0.00s2.87s0.00s2.58s
docker_mtu_and_registry_mirrors
resolvconf
install_jdk
Installing openjdk8
0.00s
git.checkout
0.47s$ git clone --depth=50 --branch=main https://github.com/dev-taein/freelec-springboot2-webservice.git dev-taein/freelec-springboot2-webservice
0.01s
Setting environment variables from repository settings
$ export AWS_ACCESS_KEY=[secure]
$ export AWS_SECRET_KEY=[secure]
$ export TERM=dumb
cache.1
Setting up build cache
$ java -Xmx32m -version
openjdk version "1.8.0_252"
OpenJDK Runtime Environment (build 1.8.0_252-8u252-b09-1~16.04-b09)
OpenJDK 64-Bit Server VM (build 25.252-b09, mixed mode)
$ javac -J-Xmx32m -version
javac 1.8.0_252
before_install
0.00s$ chmod +x gradlew
install
10.30s$ ./gradlew assemble
17.93s$ ./gradlew clean build
> Task :clean
> Task :compileJava
Note: /home/travis/build/dev-taein/freelec-springboot2-webservice/src/main/java/com/sproject/boot/springboot/config/auth/dto/OAuthAttributes.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
> Task :processResources
> Task :classes
> Task :bootJar
> Task :jar SKIPPED
> Task :assemble
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
2021-08-23 06:02:18.848  INFO 4636 --- [       Thread-6] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
2021-08-23 06:02:18.850  INFO 4636 --- [       Thread-6] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-08-23 06:02:18.851  INFO 4636 --- [       Thread-6] .SchemaDropperImpl$DelayedDropActionImpl : HHH000477: Starting delayed evictData of schema as part of SessionFactory shut-down'
Hibernate: drop table if exists posts
Hibernate: drop table if exists user
2021-08-23 06:02:18.863  INFO 4636 --- [       Thread-8] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
2021-08-23 06:02:18.875  INFO 4636 --- [       Thread-6] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-08-23 06:02:18.877  INFO 4636 --- [      Thread-10] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
2021-08-23 06:02:18.879  INFO 4636 --- [      Thread-10] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-08-23 06:02:18.882  INFO 4636 --- [      Thread-10] .SchemaDropperImpl$DelayedDropActionImpl : HHH000477: Starting delayed evictData of schema as part of SessionFactory shut-down'
Hibernate: drop table if exists posts
Hibernate: drop table if exists user
2021-08-23 06:02:18.888  INFO 4636 --- [      Thread-10] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Shutdown initiated...
2021-08-23 06:02:18.899  INFO 4636 --- [      Thread-10] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Shutdown completed.
2021-08-23 06:02:18.901  INFO 4636 --- [       Thread-6] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
> Task :check
> Task :build
Deprecated Gradle features were used in this build, making it incompatible with Gradle 5.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/4.10.2/userguide/command_line_interface.html#sec:command_line_warnings
BUILD SUCCESSFUL in 17s
6 actionable tasks: 6 executed
The command "./gradlew clean build" exited with 0.
cache.2
store build cache
before_deploy.1
0.01s$ mkdir -p before-deploy
before_deploy.2
0.01s$ cp scripts/*.sh before-deploy/
before_deploy.3
0.00s$ cp appspec.yml before-deploy/
before_deploy.4
0.03s$ cp build/libs/*.jar before-deploy/
before_deploy.5
1.35s$ cd before-deploy && zip -r before-deploy *
before_deploy.6
0.00s$ cd ../ && mkdir -p deploy
before_deploy.7
0.00s$ mv before-deploy/before-deploy.zip deploy/freelec-springboot2-webservice.zip
dpl_0
1.26s$ rvm $(travis_internal_ruby) --fuzzy do ruby -S gem install dpl
11.28s
dpl.1
Installing deploy dependencies
Logging in with Access Key: ****************VGHX
Beginning upload of 1 files with 5 threads.
dpl.2
Preparing deploy
dpl.3
Deploying application
before_deploy.1
0.00s$ mkdir -p before-deploy
before_deploy.2
0.00s$ cp scripts/*.sh before-deploy/
before_deploy.3
0.00s$ cp appspec.yml before-deploy/
before_deploy.4
0.18s$ cp build/libs/*.jar before-deploy/
before_deploy.5
1.34s$ cd before-deploy && zip -r before-deploy *
before_deploy.6
0.00s$ cd ../ && mkdir -p deploy
before_deploy.7
0.15s$ mv before-deploy/before-deploy.zip deploy/freelec-springboot2-webservice.zip
dpl_1
1.18s$ rvm $(travis_internal_ruby) --fuzzy do ruby -S gem install dpl
46.70s
dpl.1
Installing deploy dependencies
Logging in with Access Key: ****************VGHX
Version 2 of the Ruby SDK will enter maintenance mode as of November 20, 2020. To continue receiving service updates and new features, please upgrade to Version 3. More information can be found here: https://aws.amazon.com/blogs/developer/deprecation-schedule-for-aws-sdk-for-ruby-v2/
Registering app revision with version=wxePuZ.JrOV3XG4vsZ1FEMNB9H4W0ZZ4, etag="db99f7ec89508cfd117c79d8ede27142-8"
Triggered deployment "d-NYXRRJSUB".
Deployment successful.
dpl.2
Preparing deploy
dpl.3
Deploying application
Done. Your build exited with 0.
```


------------
------------
# 사용한 기능
> 추가 기능(plugins)


+ignore +mustache + database Navigator
> 스프링 기능
------------
# 주요 이슈
* 
------------
# 느낀점

------------

 
