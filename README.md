## 스프링 부트와 웹 서버 - 프로젝트 생성

스프링 부트는 지금까지 고민한 문제를 깔끔하게 해결해준다.

내장 톰캣을 사용해서 빌드와 배포를 편리하게 한다.

빌드시 하나의 Jar를 사용하면서, 동시에 Fat Jar 문제도 해결한다.

지금까지 진행한 내장 톰캣 서버를 실행하기 위한 복잡한 과정을 모두 자동으로 처리한다.

스프링 부트로 프로젝트를 만들어보고 스프링 부트가 어떤 식으로 편리한 기능을 제공하는지 하나씩 알아보자.

### 프로젝트 생성

**프로젝트 설정 순서**

1. `boot-start` 의 폴더 이름을 `boot` 로 변경하자.

2. **프로젝트 임포트**

File Open 해당 프로젝트의 `build.gradle` 을 선택하자. 그 다음에 선택창이 뜨는데, Open as Project를 선 택하자.

**또는 스프링 부트 스타터 사이트로 이동해서 스프링 프로젝트를 생성해도 된다.**


### **내장 톰캣 의존관계 확인**

`spring-boot-starter-web` 를 사용하면 내부에서 내장 톰캣을 사용한다.

### **라이브러리 버전**

```groovy
 dependencies {
     implementation 'org.springframework.boot:spring-boot-starter-web'
     testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
스프링 부트를 사용하면 라이브러리 뒤에 버전 정보가 없는 것을 확인할 수 있다.

스프링 부트는 현재 부트 버전에 가장 적절한 외부 라이브러리 버전을 자동으로 선택해준다. (이 부분에 대한 자세한 내용은 뒤에서 다룬다.)

## 스프링 부트와 웹 서버 - 실행 과정

### 스프링 부트의 실행 과정

```java
@SpringBootApplication
public class BootApplication {
   public static void main(String[] args) {
      SpringApplication.run(BootApplication.class, args);
    }
}
```
스프링부트를 실행할 때는 자바 `main()` 메서드에서 `SpringApplication.run()` 을 호출해주면 된다.

여기에 메인 설정 정보를 넘겨주는데, 보통 `@SpringBootApplication` 애노테이션이 있는 현재 클래스를 지정해주면 된다.

참고로 현재 클래스에는 `@SpringBootApplication` 애노테이션이 있는데, 이 애노테이션 안에는 컴포넌트 스캔을 포함한 여러 기능이 설정되어 있다. 

기본 설정은 현재 패키지와 그 하위 패키지 모두를 컴포넌트 스캔한다.

이 단순해 보이는 코드 한줄 안에서는 수 많은 일들이 발생하지만 핵심은 2가지다.

1. 스프링 컨테이너를 생성한다.
2. WAS(내장 톰캣)를 생성한다.

**스프링 부트 내부에서 스프링 컨테이너를 생성하는 코드** 

`org.springframework.boot.web.servlet.context.ServletWebServerApplicationContextFactory` 

```java
class ServletWebServerApplicationContextFactory implements
ApplicationContextFactory {
    //...
    private ConfigurableApplicationContext createContext() {
        if(!AotDetector.useGeneratedArtifacts()) {
            return new AnnotationConfigServletWebServerApplicationContext();
        }

        return new ServletWebServerApplicationContext();
    }
}
```
`new AnnotationConfigServletWebServerApplicationContext()` 이 부분이 바로 스프링 부트가 생성하는 스프링 컨테이너이다.

이름 그대로 애노테이션 기반 설정이 가능하고, 서블릿 웹 서버를 지원하는 스프링 컨테이너이다.

스프링 부트 내부에서 내장 톰캣을 생성하는 코드** 

`org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory`

```java
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
   //...
     Tomcat tomcat = new Tomcat();
   //...
     Connector connector = new Connector(this.protocol);
   //...
    return getTomcatWebServer(tomcat);
}
```
`Tomcat tomcat = new Tomcat()` 으로 내장 톰캣을 생성한다.

그리고 어디선가 내장 톰캣에 디스패처 서블릿을 등록하고, 스프링 컨테이너와 연결해서 동작할 수 있게 한다.

어디서 많이 본 것 같지 않은가?

스프링 부트도 우리가 앞서 내장 톰캣에서 진행했던 것과 동일한 방식으로 스프링 컨테이너를 만들고, 내장 톰캣을 생성하고 그 둘을 연결하는 과정을 진행한다.

**참고**
스프링 부트는 너무 큰 라이브러리이기 때문에 스프링 부트를 이해하기 위해 모든 코드를 하나하나 파보는 것은 추천하지 않는다.

스프링 부트가 어떤 식으로 동작하는지 개념을 이해하고, 꼭 필요한 부분의 코드를 확인하자.

지금까지 스프링 부트가 어떻게 톰캣 서버를 내장해서 실행하는지 스프링 부트의 비밀 하나를 풀어보았다. 

다음에는 스프링 부트의 빌드와 배포 그리고 스프링 부트가 제공하는 `jar` 의 비밀을 알아보자.

## 스프링 부트와 웹 서버 - 빌드와 배포

내장 톰캣이 포함된 스프링 부트를 직접 빌드해보자.

**jar 빌드**

```shell
./gradlew clean build
```

다음위치에 `jar` 파일이만들어진다. 

`build/libs/boot-0.0.1-SNAPSHOT.jar`

**jar 파일 실행**

`jar` 파일이 있는 폴더로 이동한 후에 다음 명령어로 `jar` 파일을 실행해보자.

```shell
java -jar boot-0.0.1-SNAPSHOT.jar
```
**실행 결과**

```shell
... % java -jar boot-0.0.1-SNAPSHOT.jar
o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
hello.boot.BootApplication               : Started BootApplication in 0.961 seconds
```

스프링 부트 애플리케이션이 실행되고, 내장 톰캣이 8080 포트로 실행된 것을 확인할 수 있다.

컨트롤러가 잘 호출되는지 확인해보자.

**실행**

http://localhost:8080/hello-spring

### 스프링 부트 jar 분석

```shell
ls -arlth
```

`boot-0.0.1-SNAPSHOT.jar` 파일 크기를 보면 대략 `18M` 정도 된다. 참고로 버전에 따라서 용량은 변할 수 있다.

아마도 앞서 배운 Fat Jar와 비슷한 방식으로 만들어져 있지 않을까? 생각될 것이다. 비밀을 확인하기 위해 `jar` 파일의 압축을 풀어보자.

**jar 압축 풀기**

`build/libs` 폴더로 이동하자.

다음 명령어를 사용해서 압축을 풀자

```shell
jar -xvf boot-0.0.1-SNAPSHOT.jar
```

**JAR를 푼 결과** 

`boot-0.0.1-SNAPSHOT.jar`
* `META-INF`
  * `MANIFEST.MF`
* `org/springframework/boot/loader` 
  * `JarLauncher.class` : 스프링 부트 `main()` 실행 클래스 
* `BOOT-INF`
  * `classes` : 우리가 개발한 class 파일과 리소스 파일 
  * `hello/boot/BootApplication.class`
  * `hello/boot/controller/HelloController.class` ...
* `lib` : 외부 라이브러리
  * `spring-webmvc-6.0.4.jar` 
  * `tomcat-embed-core-10.1.5.jar`
  * ...
* `classpath.idx` : 외부 라이브러리 경로 
* `layers.idx` : 스프링 부트 구조 경로

JAR를 푼 결과를 보면 Fat Jar가 아니라 처음보는 새로운 구조로 만들어져 있다. 

심지어 jar 내부에 jar를 담아서 인식 하는 것이 불가능한데, jar가 포함되어 있고, 인식까지 되었다. 지금부터 스프링 부트가 제공하는 jar에 대해서 알아보\자.

**참고**

빌드 결과를 보면 `boot-0.0.1-SNAPSHOT-plain.jar` 파일도 보이는데, 이것은 우리가 개발한 코드만 순수한 jar로 빌드한 것이다. 무시하면 된다.

## 스프링 부트 실행 가능 Jar

Fat Jar는 하나의 Jar 파일에 라이브러리의 클래스와 리소스를 모두 포함했다. 

그래서 실행에 필요한 모든 내용을 하나의 JAR로 만들어서 배포하는 것이 가능했다. 하지만 Fat Jar는 다음과 같은 문제를 가지고 있다.

### **Fat Jar의 단점**

어떤 라이브러리가 포함되어 있는지 확인하기 어렵다.

모두 `class` 로 풀려있으니 어떤 라이브러리가 사용되고 있는지 추적하기 어렵다. 파일명 중복을 해결할 수 없다.

클래스나 리소스 명이 같은 경우 하나를 포기해야 한다. 이것은 심각한 문제를 발생한다. 

예를 들어서 서블릿 컨테이너 초기화에서 학습한 부분을 떠올려 보자.

`META-INF/services/jakarta.servlet.ServletContainerInitializer` 이 파일이 여러 라이브러리( `jar` )에 있을 수 있다.

`A` 라이브러리와 `B` 라이브러리 둘다 해당 파일을 사용해서 서블릿 컨테이너 초기화를 시도한다. 

둘다 해당 파일을 `jar` 안에 포함한다.

`Fat Jar` 를 만들면 파일명이 같으므로 `A` , `B` 둘중 하나의 파일만 선택된다. 결과적으로 나머지는 정상 동작하지 않는다.

**실행 가능 Jar**

스프링 부트는 이런 문제를 해결하기 위해 jar 내부에 jar를 포함할 수 있는 특별한 구조의 jar를 만들고 동시에 만든 jar 를 내부 jar를 포함해서 실행할 수 있게 했다. 

이것을 실행 가능 Jar(Executable Jar)라 한다. 이 실행 가능 Jar를 사용 하면 다음 문제들을 깔끔하게 해결할 수 있다.

* 문제: 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다.
  * 해결: jar 내부에 jar를 포함하기 때문에 어떤 라이브러리가 포함되어 있는지 쉽게 확인할 수 있다.
* 문제: 파일명 중복을 해결할 수 없다.
  * 해결: jar 내부에 jar를 포함하기 때문에 `a.jar` , `b.jar` 내부에 같은 경로의 파일이 있어도 둘다 인식할 수 있다.

  * 참고로 실행 가능 Jar는 자바 표준은 아니고, 스프링 부트에서 새롭게 정의한 것이다. 지금부터 실행 가능 Jar를 자세히 알아보자.

### 실행 가능 Jar 내부 구조


**Jar 실행 정보**

`java -jar xxx.jar` 를 실행하게 되면 우선 `META-INF/MANIFEST.MF` 파일을 찾는다. 

그리고 여기에 있는 `Main-Class` 를 읽어서 `main()` 메서드를 실행하게 된다. 

스프링 부트가 만든 `MANIFEST.MF` 를 확인해보자.

`META-INF/MANIFEST.MF` 

```
Manifest-Version: 1.0
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: hello.boot.BootApplication
Spring-Boot-Version: 3.0.2
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Build-Jdk-Spec: 17
```
* `Main-Class`
  * 우리가 기대한 `main()` 이 있는 `hello.boot.BootApplication` 이 아니라 `JarLauncher` 라 는 전혀 다른 클래스를 실행하고 있다.
  * `JarLauncher` 는 스프링 부트가 빌드시에 넣어준다. `org/springframework/boot/loader/ JarLauncher` 에 실제로 포함되어 있다.
  * 스프링 부트는 jar 내부에 jar를 읽어들이는 기능이 필요하다. 또 특별한 구조에 맞게 클래스 정보도 읽어들여야 한다. 
  * 바로 `JarLauncher` 가 이런 일을 처리해준다. 
  * 이런 작업을 먼저 처리한 다음 `Start- Class:` 에 지정된 `main()` 을 호출한다.
* `Start-Class` : 우리가 기대한 `main()` 이 있는 `hello.boot.BootApplication` 가 적혀있다. 
* 기타: 스프링 부트가 내부에서 사용하는 정보들이다.
  * `Spring-Boot-Version` : 스프링 부트 버전
  * `Spring-Boot-Classes` : 개발한 클래스 경로 
  * `Spring-Boot-Lib` : 라이브러리 경로 
  * `Spring-Boot-Classpath-Index` : 외부 라이브러리 모음 
  * `Spring-Boot-Layers-Index` : 스프링 부트 구조 정보
* 참고: `Main-Class` 를 제외한 나머지는 자바 표준이 아니다. 스프링 부트가 임의로 사용하는 정보이다.

**스프링 부트 로더**

`org/springframework/boot/loader` 하위에 있는 클래스들이다.

`JarLauncher` 를 포함한 스프링 부트가 제공하는 실행 가능 Jar를 실제로 구동시키는 클래스들이 포함되어 있다.

스프링 부트는 빌드시에 이 클래스들을 포함해서 만들어준다.

**BOOT-INF**

`classes` : 우리가 개발한 class 파일과 리소스 파일 

`lib` : 외부 라이브러리

`classpath.idx` : 외부 라이브러리 모음 

`layers.idx` : 스프링 부트 구조 정보

WAR구조는 `WEB-INF` 라는 내부 폴더에 사용자 클래스와 라이브러리를 포함하고 있는데, 실행 가능 Jar도 그 구조를 본따서 만들었다. 이름도 유사하게 `BOOT-INF` 이다.

`JarLauncher` 를 통해서 여기에 있는 `classes` 와 `lib` 에 있는 jar 파일들을 읽어들인다.

**실행 과정 정리**

1. `java -jar xxx.jar`
2. `MANIFEST.MF` 인식
3. `JarLauncher.main()` 실행
   * `BOOT-INF/classes/` 인식
   * `BOOT-INF/lib/` 인식
4. `BootApplication.main()` 실행

**참고**

실행 가능 Jar가 아니라, IDE에서 직접 실행할 때는 `BootApplication.main()` 을 바로 실행한다. 

IDE가 필요한 라이브러리를 모두 인식할 수 있게 도와주기 때문에 `JarLauncher` 가 필요하지 않다.

