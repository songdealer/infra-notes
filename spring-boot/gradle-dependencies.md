# Spring Boot·Gradle 의존성 정리

작성일: 2026-11-07  
확인 기준: `chatDemo` 프로젝트의 저장된 `build.gradle`, `application.yaml`과 학습 중 확인한 공식 문서.

Spring Boot 프로젝트를 만들고 Spring Initializr 생성 서비스를 구성하면서 사용한 의존성의 역할을 정리한다. 직접 선언한 항목과 웹 서버 실행을 위해 변경을 안내한 항목을 함께 기록한다.

## 1. 현재 확인한 구성

- Java toolchain: 17
- Spring Boot 플러그인: 4.1.2
- Spring Dependency Management 플러그인: 1.1.7
- Spring Initializr BOM: 0.24.0
- 빌드 스크립트: Gradle Groovy DSL

문서 작성 시 저장된 `build.gradle`에는 아래 6개 항목이 선언되어 있었다. BOM도 포함한 선언 수이며, 실제 내려받는 라이브러리 수는 전이 의존성 때문에 더 많다.

| 항목 | 선언 방식 | 역할 | 확인 상태 |
|---|---|---|---|
| `spring-boot-starter` | `implementation` | Spring Boot 기본 실행 기능 | 현재 선언됨 |
| `spring-boot-starter-test` | `testImplementation` | 테스트 작성·실행에 필요한 기본 도구 | 현재 선언됨 |
| `junit-platform-launcher` | `testRuntimeOnly` | JUnit 테스트 탐색·실행을 연결하는 런처 | 현재 선언됨 |
| `initializr-bom:0.24.0` | `implementation platform(...)` | Initializr 모듈 버전 관리 | 현재 선언됨 |
| `initializr-web` | `implementation` | 생성 서비스의 HTTP API | 현재 선언됨 |
| `initializr-generator-spring` | `implementation` | Spring Boot 프로젝트 파일 생성 규칙 | 현재 선언됨 |
| `spring-boot-starter-web` | `implementation` | Spring MVC 기반 웹 기능 | 앞서 안내함. YAML의 생성 대상 의존성 목록에 있음 |
| `spring-boot-starter-webmvc` | `implementation` | Boot 4에서 권장하는 Spring MVC 웹 스타터 | 변경을 안내했으나 저장된 Gradle 파일에는 아직 없음 |

의존성을 추가한 뒤의 서버 실행 성공이나 테스트 통과를 이 문서에서 확인한 것으로 간주하지 않는다.

## 2. dependency와 starter란?

**Dependency(의존성)**는 내 프로그램이 사용하는 다른 라이브러리다. Gradle에 필요한 항목을 선언하면 Gradle이 저장소에서 찾아 빌드에 연결한다.

**Starter(스타터)**는 특정 기능에 필요한 의존성을 모아 둔 구성이다. 예를 들어 웹 스타터를 사용하면 웹 개발에 필요한 라이브러리를 하나씩 고를 필요가 줄어든다. 스타터가 함께 가져오는 라이브러리를 **전이 의존성**이라고 한다. [Spring Boot 공식 설명](https://docs.spring.io/spring-boot/reference/using/build-systems.html#using.build-systems.starters)

의존성의 기본 표기 형태는 다음과 같다.

```text
group:artifact:version
```

예를 들어 `io.spring.initializr:initializr-bom:0.24.0`에서 `io.spring.initializr`는 그룹, `initializr-bom`은 모듈 이름, `0.24.0`은 해당 모듈 버전이다. 프로젝트 자체의 `version = '0.0.1-SNAPSHOT'`과는 별개다.

## 3. 각각 어떤 목적으로 사용하는가?

### spring-boot-starter

Spring Boot의 기본 스타터다. 자동 설정 지원, 로그 출력, YAML 설정 처리 등에 필요한 기본 구성을 제공한다.

자체적으로 내장 HTTP 서버를 포함하지 않는다. 웹 요청을 받을 필요가 없는 프로그램의 기본 구성으로 사용할 수 있다. `server.port`를 적는 것만으로 웹 서버가 추가되지는 않는다.

### spring-boot-starter-web / spring-boot-starter-webmvc

웹사이트와 REST API처럼 HTTP 요청을 받아 응답하는 애플리케이션에 사용한다.

- **Spring MVC:** 요청 주소와 처리 코드를 연결한다.
- **Tomcat:** 기본 내장 서버로 HTTP 요청을 받는다.
- **JSON 처리:** 웹 구성에 포함되는 변환 기능으로 요청·응답 데이터를 자바 객체와 연결한다.

웹 스타터가 기본 스타터도 함께 가져오므로, 기본 스타터를 별도로 선언할 필요는 없다.

Boot **4.1.2**의 공식 목록에서 `spring-boot-starter-web`은 deprecated이며 `spring-boot-starter-webmvc`로 대체하도록 안내한다. 이전 학습에서는 `starter-web`을 안내했지만, 현재 버전에 맞춘 권장 선언은 다음과 같다. [스타터 목록](https://docs.spring.io/spring-boot/reference/using/build-systems.html#using.build-systems.starters), [Spring MVC와 메시지 변환](https://docs.spring.io/spring-boot/reference/web/servlet.html)

```groovy
implementation 'org.springframework.boot:spring-boot-starter-webmvc'
```

Deprecated는 즉시 사용할 수 없다는 의미가 아니라, 새 코드에서는 대체 항목 사용을 권장한다는 뜻이다.

### spring-boot-starter-test

테스트 코드에서 사용하는 기본 도구 모음이다. Spring Boot 테스트 지원과 JUnit Jupiter, AssertJ, Mockito 등의 도구를 이용할 수 있게 한다.

예를 들어 JUnit은 테스트를 정의하고 실행하는 데, AssertJ는 결과를 검증하는 데, Mockito는 실제 객체를 대신할 테스트용 객체를 만드는 데 사용한다. `testImplementation`으로 선언하므로 일반 애플리케이션의 실행 의존성에는 포함되지 않는다.

```groovy
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

의존성을 추가해도 업무 기능의 테스트가 자동으로 작성되는 것은 아니다. [Spring Boot 스타터 목록](https://docs.spring.io/spring-boot/reference/using/build-systems.html#using.build-systems.starters)

### junit-platform-launcher

JUnit Platform에서 테스트를 찾아 실행하고 결과를 전달하는 데 사용하는 런처 API다. 테스트를 작성하는 JUnit Jupiter와 역할이 다르다. Gradle 같은 실행 도구와 테스트 엔진을 연결하는 데 쓰인다.

```groovy
testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

tasks.named('test') {
    useJUnitPlatform()
}
```

`testRuntimeOnly`는 테스트 실행 시 사용할 의존성이라는 뜻이다. `useJUnitPlatform()`은 Gradle의 test 작업에서 JUnit Platform을 사용하도록 지정한다. [JUnit Launcher API](https://docs.junit.org/6.1.3/advanced-topics/launcher-api.html), [JUnit의 Gradle 설정](https://docs.junit.org/6.1.3/running-tests/build-support.html)

### initializr-bom

Initializr 관련 모듈의 버전을 묶어서 관리하는 **BOM(Bill of Materials)**이다. 프로젝트 생성 기능을 실행하는 라이브러리 자체가 아니라, 어떤 버전들을 사용할지 정하는 정보다.

```groovy
implementation platform('io.spring.initializr:initializr-bom:0.24.0')
```

Gradle의 `platform(...)`으로 BOM을 가져오면 그 BOM이 관리하는 라이브러리들의 버전을 각 줄에 반복해서 적지 않아도 된다. BOM만 추가한다고 Initializr 모듈 전체가 설치되는 것은 아니다. 필요한 모듈은 별도로 선언한다. [Gradle BOM 설명](https://docs.gradle.org/current/userguide/platforms.html#sub:bom_import)

Initializr **0.24.0**은 Spring Boot **4.0.x**를 기준으로 출시되었다. 현재 프로젝트의 Boot 4.1.2와의 실제 실행·생성 결과는 별도로 확인해야 한다. YAML의 `bootVersions`는 생성할 프로젝트의 목록이며, 생성 서비스 자체의 라이브러리 호환성을 바꾸지 않는다. [Initializr 0.24.0 릴리스](https://github.com/spring-io/initializr/releases/tag/v0.24.0)

### initializr-web

STS 같은 클라이언트가 사용할 Initializr HTTP API를 제공한다. 선택 가능한 Java 버전·언어·의존성 등의 메타데이터를 제공하고 프로젝트 생성 요청을 받는 역할이다.

```groovy
implementation 'io.spring.initializr:initializr-web'
```

이름에 `web`이 있다고 해서 내장 서버 구성이 모두 준비되는 것은 아니다. 생성 서비스를 웹 애플리케이션으로 실행하기 위한 웹 스타터도 필요하다.

### initializr-generator-spring

Spring Boot 프로젝트에 맞는 파일 생성 규칙을 제공한다. 선택한 빌드 도구와 언어 등에 따라 빌드 파일, 애플리케이션 소스 등 초기 프로젝트 구성 요소를 만드는 데 사용한다.

```groovy
implementation 'io.spring.initializr:initializr-generator-spring'
```

두 Initializr 모듈은 이 연습의 **프로젝트 생성 서비스**에 필요하다. 일반적인 채팅 API 서버를 구현할 때 필수로 넣는 라이브러리는 아니다. [Initializr 모듈과 자체 서비스 구성](https://docs.spring.io/initializr/docs/current/reference/html/index.html)

## 4. implementation과 테스트 의존성의 차이

| 선언 방식 | 사용 범위 |
|---|---|
| `implementation` | 메인 코드의 컴파일·실행에 사용 |
| `testImplementation` | 테스트 코드의 컴파일·실행에 사용 |
| `testRuntimeOnly` | 테스트 실행에 사용하며 테스트 컴파일에는 직접 추가하지 않음 |
| `platform(...)` | 위 선언에서 BOM을 버전 관리 정보로 가져오는 방식 |

테스트 의존성은 일반적인 `bootJar`의 애플리케이션 실행 라이브러리에 포함되지 않는다. [Gradle Java 플러그인의 의존성 구성](https://docs.gradle.org/current/userguide/java_plugin.html)

## 5. 현재 파일과 변경을 안내한 부분

작성 시점에 저장된 `build.gradle`의 의존성 블록이다. 들여쓰기만 정리했다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

    implementation platform('io.spring.initializr:initializr-bom:0.24.0')
    implementation 'io.spring.initializr:initializr-web'
    implementation 'io.spring.initializr:initializr-generator-spring'
}
```

웹 서버 구성을 위해 첫 번째 `implementation` 줄을 다음으로 변경하도록 안내했다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-webmvc'
```

당시 화면에는 `Started ChatDemoApplication`이 있었지만 Tomcat 시작 로그가 없었다. 파일 확인에서도 웹 스타터가 빠져 있었고, 확인 시점에 8081 포트의 수신 서버도 없었다. 애플리케이션 초기화 성공과 HTTP 서버의 시작은 따로 확인해야 한다.

## 6. Gradle 설정과 YAML 설정의 차이

| 위치 | 의미 |
|---|---|
| 현재 프로젝트의 `build.gradle → dependencies` | 지금 실행할 생성 서비스가 사용하는 라이브러리 |
| `application.yaml → initializr → dependencies` | 생성 화면에 제공할 의존성 선택 목록 |
| `application.yaml → server.port` | 생성 서비스 자체가 사용할 포트 |
| `application.yaml → initializr → bootVersions` | 앞으로 생성할 프로젝트에서 선택 가능한 Boot 버전 |

현재 YAML에는 생성 대상 의존성으로 `spring-boot-starter-web`이 적혀 있다. 생성 서비스의 Gradle 파일을 `starter-webmvc`로 바꾸더라도 YAML의 목록은 자동으로 바뀌지 않는다. Boot 4용 생성 프로젝트도 같은 권장 스타터를 사용하려면 해당 YAML 항목의 `artifactId`를 `spring-boot-starter-webmvc`로 별도로 수정한다.

## 7. plugins와 repositories도 함께 이해하기

현재 `plugins` 블록은 다음과 같다.

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '4.1.2'
    id 'io.spring.dependency-management' version '1.1.7'
}
```

| 플러그인 | 역할 |
|---|---|
| `java` | 자바 컴파일, 테스트, JAR 생성 같은 Gradle 작업 제공 |
| `org.springframework.boot` | `bootRun`, `bootJar` 등 Boot 실행·패키징 작업 제공 |
| `io.spring.dependency-management` | BOM 등에 따른 의존성 버전 관리 제공 |

플러그인은 빌드 도구인 Gradle의 기능을 확장한다. 애플리케이션에서 사용하는 라이브러리는 `dependencies`에 선언한다. 현재 Boot 플러그인과 dependency-management 플러그인의 조합에서는 Boot BOM을 통해 Spring Boot 관련 라이브러리의 버전을 관리한다. Initializr 모듈의 버전은 별도로 가져온 Initializr BOM이 관리한다. [Spring Boot Gradle 플러그인](https://docs.spring.io/spring-boot/gradle-plugin/index.html), [Dependency Management 플러그인](https://docs.spring.io/dependency-management-plugin/docs/current/reference/html/)

```groovy
repositories {
    mavenCentral()
}
```

`mavenCentral()`은 애플리케이션 의존성을 찾을 저장소를 지정한다. Maven Central이라는 이름의 저장소는 Gradle에서도 사용할 수 있다.

## 8. 이후 확인에 사용할 명령

아래는 학습용 확인 명령이며, 이번 문서 작성 과정에서 서버 실행이나 테스트를 수행한 기록은 아니다. 프로젝트 루트에서 실행한다.

```bash
# 애플리케이션 실행 의존성과 전이 의존성 보기
./gradlew dependencies --configuration runtimeClasspath

# 테스트 실행에 사용하는 의존성 보기
./gradlew dependencies --configuration testRuntimeClasspath

# Initializr 모듈의 실제 선택 버전과 선택 이유 보기
./gradlew dependencyInsight --dependency initializr-web --configuration runtimeClasspath

# 테스트 실행
./gradlew test
```

의존성 보고서에 `FAILED`가 있는지도 확인한다. 버전을 생략한 항목의 실제 선택 버전은 보고서에서 확인할 수 있다. STS에서는 Gradle 설정을 바꾼 뒤 **프로젝트 우클릭 → Gradle → Refresh Gradle Project**로 동기화하고, 애플리케이션을 다시 실행한다.

[이전 기록: Intel Mac에 Java 17 설치하기](../java/macos-java17-setup.md)
