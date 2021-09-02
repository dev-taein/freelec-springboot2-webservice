# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스   
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


> index.mustache
```
{{>layout/header}}

<h1>스프링부트로 시작하는 웹 서비스 Ver.2.5</h1>
<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
            {{#GN-user}}
                Logged in as: <span id="user">{{GN-user}}</span>
                <a href="/logout" class="btn btn-info active" role="button">Logout</a>
            {{/GN-user}}
            {{^GN-user}}
                <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a>
            {{/GN-user}}
        </div>
    </div>
    <br>
    <!-- 목록 출력 영역 -->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-strong">
        <tr>
            <th>게시글번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>최종수정일</th>
        </tr>
        </thead>
        <tbody id="tbody">
        {{#posts}}
            <tr>
                <td>{{id}}</td>
                <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                <td>{{author}}</td>
                <td>{{modifiedDate}}</td>
            </tr>
        {{/posts}}
        </tbody>
    </table>
</div>
{{>layout/footer}}
```

> index.js
```
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () {
            _this.update();
        });

        $('#btn-delete').on('click', function () {
            _this.delete();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    update : function () {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8'
        }).done(function() {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }

};

main.init();
```


> indexController
```
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

        if(user != null) {
            model.addAttribute("GN-user", user.getName());
        }
        return "index";
    }

    @GetMapping("posts/save")
    public String postsSave() {
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }
}
```

> 글쓰기 mustache
```
{{>layout/header}}

<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author"> 작성자 </label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content"> 내용 </label>
                <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>

{{>layout/footer}}
```


> 글 수정 mustache
```
{{>layout/header}}

<h1>게시글 수정</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">글 번호</label>
                <input type="text" class="form-control" id="id" value="{{post.id}}" readonly>
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" value="{{post.title}}">
            </div>
            <div class="form-group">
                <label for="author"> 작성자 </label>
                <input type="text" class="form-control" id="author" value="{{post.author}}" readonly>
            </div>
            <div class="form-group">
                <label for="content"> 내용 </label>
                <textarea class="form-control" id="content">{{post.content}}</textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
        <button type="button" class="btn btn-danger" id="btn-delete">삭제</button>
    </div>
</div>

{{>layout/footer}}
```


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


> putty 배포 스크립트 작성   
![putty배포스크립트작성](https://user-images.githubusercontent.com/77142806/130398855-96821bdb-2caa-4f88-be1a-5edd3fc4b7cc.PNG)


> 시큐리티, DB 스크립트   
![putty배포스크립트작성2](https://user-images.githubusercontent.com/77142806/130398856-807afae6-2976-41e2-8d46-1851e7473014.PNG)


------------


> deploy.sh
```
#! /bin/bash


REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=freelec-springboot2-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f freelec-springboot2-webservice*.jar)

echo "> 현재 구동중인 애플리케이션 pid : $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
        echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."

else
        echo "> kill -15 $CURRENT_PID"
        kill -15 $CURRENT_PID
        sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

echo "> JAR Name : $JAR_NAME"

nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \
        -Dspring.profiles.active=real \
        $REPOSITORY/$JAR_NAME 2>&1 &
~

```
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



> AWS CodeDeploy
![codeploy](https://user-images.githubusercontent.com/77142806/131336596-653df17a-93c4-44c2-9f26-68d442319491.PNG)


------------
------------
# 사용한 Spring 기능들
> 추가 기능(plugins)


+ ignore : Intelij에서 바로 Git Commit 할 수 있는 기능
+ mustache : 수 많은 언어를 제공하는 가장 심플한 엔진이다. 문법이 다른 템플릿 엔진보다 심플하고 로직 코드를 사용할 수 없어 view의 역할과 서버의 역할이 명확하게 분리된다. Mustache.js, Mustache.java 2가지가 다 있어, 하나의 문법으로 클라이언트/서버 템플릿을 모두 사용 가능합니다.
+ database Navigator
+ Lombok : Getter, Setter, 기본 생성자, toString 등을 어노테이션으로 자동 생성해 준다.
+ JPA : 수십 수백개의 테이블의 SQL를 만들고 관리하기엔 단순 반복 작업을 수백 번 해야 한다. 또한 상속, 1:N 등 다양한 객체 모델링을 데이터베이스로 구현할 수 없다. JPA는 이러한 문제를 해결 하기 위해 등장하였고 개발자는 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행한다. 개발자는 항상 객체지향적으로 코드를 표현할 수 있으니 더는 SQL에 종속적인 개발을 하지 않아도 된다. 객체 중심으로 개발을 하게 되니 생산성 향상은 물론 유지 보수하기가 정말 편해진다.
> 스프링 어노테이션


+ @SpringBootApplcation : 스프링부트의 자동 설정, 스프링 Bean 읽기와 생성 모두 자동으로 설정한다. Application이 있는 위치부터 설정을 읽어 가기 때문에 프로젝트의 최상단에 위치해야한다.
+ @RestController : 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
+ @GetMapping : HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어 준다.
+ @RunWith(SpringRunner.class) : 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.
+ @WebMvcTest : 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있다.
+ @Autowired : 스프링이 관리하는 빈을 주입 받는다.
+ private MockMvc mvc : 웹 API를 테스트할 때 사용합니다. 스프링 MVC 테스트의 시작점입니다. 이 클래스를 통해 HTTP GET, POST등에 대한 API테스트를 할 수 있다.
+ @Getter : 선언된 모든 필드의 get 메소드를 생성한다.
+ @RequiredArgsConstructor : 선언된 모든 final 필드가 포함된 생성자를 생성해 준다. final이 없는 필ㄹ드는 생성자에 포함되지 않는다.
+ @RequestParam : 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션이다. @RequestParam("name")을 name(String name)에 저장된다.
+ @Entity : 테이블과 링크될 클래스임을 나타낸다. ex) SalesManager.java -> sales_manager table
+ @Id : 해당 테이블의 PK 필드
+ @GeneratedValue : PK의 생성 규칙을 나타낸다. GeneratedType.IDENTITY 옵션을 추가하면 auto_increment가 된다.
+ @Column : 테이블의 칼럼을 나타내면 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다. 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용한다.
+ @NoArgsConstructor : 기본 생성자 자동 추가
+ @Builder : 해당 클래스의 빌더 패턴 클래스를 생성, 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함한다.
+ @After : Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정한다.
+ @MappendSuperclass : JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드를(createdDate,modifiedDate)도 칼럼으로 인식하도록 한다.
+ @EntityListerners(AuditingEntityListener.class) : BaseTimeEntity클래스에 Auditing 기능을 포함시킨다.
+ @CreateDate : Entity가 생성되어 저장될 때 시간이 자동 저장된다.
+ @LastModifiedDate : 조회한 Entity의 값을 변경할 때 시간이 자동 저장된다.
+ @WithMockUser(roles="USER") : 인증된 모의 사용자를 만들어서 사용한다.
+ @Before : 매번 테스트가 시작되기 전에 MockMvc 인스턴스를 생성한다.


> Spring Security
+ @EnableWebSecurity : Spring Security 설정들을 활성화시켜 줍니다.
+ csrf().disable().headers().frameOptions().disable() : h2-console 화면을 사용하기 위해 해당 옵션들을 disable 한다.
+ authorizeRequests : URL별 권한 관리를 설정하는 옵션의 시작점입니다.
+ antMatchers : 권한 관리 대상을 지정하는 옵션, URL, HTTP 메소드별로 관리가 가능합니다. "/" 지정된 URL들을 permitAll()옵션을 통해 전체 열람 권한을 주었습니다. "/api/v1/**"주소를 가진 API는 GUEST 권한을 가진 사람만 가능하도록 했습니다.
+ anyRequest : 설정된 값들 이외 나머지 URL들을 나타냅니다. authenticated()을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용합니다. 인증된 사용자 즉, 로그인한 사용자
+ logout().logoutSuccessUrl("/") : 로그아웃 기능에 대한 여러 설정의 진입점입니다. 로그아웃 성공 시 /주소로 이동
+ oauth2Login : OAuth 2 로그인 기능에 대한 여러 설정의 진입점
+ userInfoEndpoint : OAuth 2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당합니다.
+ userService : 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록한다. 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있다.
+ registrationId : 현재 로그인 진행 중인 서비스를 구분하는 코드
+ userNameAttributeName : Oauth2 로그인 진행 시 키가 되는 필드값을 이야기합니다. Primary Key와 같은 의미이다. 구글은 코드를 지원하지만 네이버, 카카오는 지원하지 않는다. 이후 네이버, 구글 로그인을 동시 지원할 때 사용된다.
+ OAuthAttributes : OAuth2UserService를 통해 가져온 OAuth2User의 attributte를 담을 클래스입니다. 이후 네이버 등 다른 소셜 로그인도 이 클래스를 사용한다.
+ SessionUser : 세션에 사용자 정보를 저장하기 위한 Dto 클래스이다.
+ of() : OAuth2User에서 ㅂㄴ환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야만 한다.
+ toEntity() : User엔티티를 생성한다. OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때입니다. 가입할 때의 기본 권한을 Role.GUEST로 설정합니다.
+ @Target(ElementType.PARAMETER) : 이 어노테이션이 생성될 수 있는 위치를 지정한다. PARAMETER로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용할 수 있다.
+ @interface : 이 파일을 어노테이션 클래스로 지정한다. LoginUser라는 이름을 가진 어노테이션이 생성되었다고 보면 된다..
------------


# 주요 이슈
* 로그인 후 userName이 바뀌지 않은 문제
* 프로젝트 Build Test 문제
* AWS 도메인 주소로 접속 시 페이지 문제 Whitelabel Error Page This application has no explicit mapping for /error, so you are seeing this as a fallback.
* Travis CI 배포가 안되는 문제
* putty hostName 변경 문제
* putty DB Connection 문제
* putty EC2 서버 접속 실패 문제
------------


# 느낀점


##### AWS를 이용하여 스크립트를 배포하고 Travis CI를 이용하여 Git Commit할 때마다 자동으로 배포되도록 만들었습니다.자동화 배포까지 다 마무리하고 나니 정말 편하다고 느껴졌습니다.<br><br>라인/쿠팡/우아한형제들/카카오 등 모두 자바와 스프링 프레임워크로 웹 개발하는데 저는 처음에 간단하고 쉬운 Ruby와 Rails, Python과 Django 프레임워크가 더 낫지 않겠냐고 생각했습니다.<br><br>하지만 스프링 프레임워크를 이용해서 웹 사이트를 만들어 보고 이번에는 AWS를 이용한 무중단 배포까지 하면서 생각이 바뀌었습니다. 스프링 부트가 나오면서 더는 스프링으로 만드는 웹 개발도 어렵지 않다고 느껴졌습니다. 따로 서버를 설치할 필요가 없고 Jar만 있으면 서비스를 운영할 수 있습니다. 그리고 객체 지향적이여서 재미도 있습니다.<br><br>주로 개발 툴은 Eclips로만 개발하였었지만 Intelij 개발 툴을 사용해보니 굉장히 간편하고 좋았습니다.
------------
