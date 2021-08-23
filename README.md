# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스   
http://ec2-15-165-198-250.ap-northeast-2.compute.amazonaws.com/   
*AWS의 무료 이용 기간 1년이므로 링크의 주소가 만료될 수 있습니다.<br>(소셜 로그인 후 글 등록은 사용 할 수없습니다.제가 DB에서 직접 권한을 바꾸어야 등록할 수 있습니다.*
------------
## 목적
+ 스프링부트와 AWS 무중단 배포 경험 쌓기
------------
# Build.gradle 설정
+ Tools : Intellij IDEA, Gradle 4.8
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
#

------------

------------
------------
# 사용한 기능
------------
# 주요 이슈
* 
------------
# 느낀점

------------

 
