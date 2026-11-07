# Intel Mac에서 Homebrew로 Java 17 설치하고 사용할 버전 선택하기

작성일: 2026-11-07

Spring Boot로 채팅 백엔드를 직접 만들기 전에 Java 개발 환경을 준비한 기록이다. Java 12가 선택된 상태에서 시작해 JDK 17을 설치하고, 현재 터미널에서 Java 17을 사용하도록 설정했다.

이 문서는 터미널의 전체 히스토리가 아닌 대화에서 진행한 흐름을 기준으로 정리했다. 사용자가 알려준 결과와 확인 방법을 함께 기록했으며, 전달받지 않은 세부 패치 버전이나 설치 로그는 만들지 않았다.

## 개발 환경과 이번 작업의 범위

| 항목 | 내용 |
|---|---|
| 컴퓨터 | MacBook Pro 13형, 2018, Thunderbolt 3 포트 4개 |
| CPU 계열 | Intel, x64 |
| macOS | Sequoia 15.7.7 |
| 기존 Java | 설치 목록에 11과 12가 있었고, `java -version`은 12를 표시 |
| 이번에 선택한 JDK | Eclipse Temurin 17 |
| 설치 방법 | Homebrew Cask |
| 최종 확인 | 환경 변수 설정 후 Java 17이 표시된다고 사용자 확인 |

이번 작업은 Java 설치와 현재 터미널의 버전 선택까지다. 셸 시작 파일에 설정을 저장하거나 Spring Boot 프로젝트를 생성하는 작업은 아직 진행하지 않았다.

## 1. 현재 실행되는 Java 버전 확인

```bash
java -version
javac -version
```

**왜 실행했나?**

Spring Boot 프로젝트를 만들기 전에 Java 실행 도구와 컴파일러가 준비되어 있는지 확인하기 위해 실행했다.

- `java`: 컴파일된 Java 프로그램을 실행하는 도구다.
- `javac`: Java 소스 코드인 `.java` 파일을 바이트코드인 `.class` 파일로 컴파일하는 도구다.
- `-version`: 해당 도구의 버전을 출력한다.

**확인한 내용**

사용자가 처음 확인한 Java 버전은 12였다. 이 결과는 현재 터미널에서 선택된 버전이 12라는 뜻이며, 다른 JDK가 함께 설치되어 있는지까지 알려주지는 않는다. `javac`의 초기 세부 출력은 대화에 별도로 남아 있지 않다.

## 2. macOS가 인식하는 JDK 설치 목록 확인

```bash
/usr/libexec/java_home -V
```

**왜 실행했나?**

전에 다운로드한 Java 17이 이미 설치되어 있는지 확인하기 위해 실행했다. 다른 JDK가 설치되어 있다면 다시 설치하는 대신 사용할 버전만 선택할 수 있기 때문이다.

**명령어의 의미**

- `/usr/libexec/java_home`: macOS에서 설치된 JDK의 위치를 찾는 도구다.
- 대문자 `-V`: macOS가 인식하는 JDK 목록과 경로를 출력한다.

**확인한 내용**

사용자가 확인한 목록에는 Java 11과 12만 보였다. 따라서 Java 17 설치를 진행하기로 했다. 목록에 보이지 않는다고 다운로드한 설치 파일이나 다른 위치에 풀어 둔 JDK까지 없다는 뜻은 아니다.

## 3. macOS 버전 확인

```bash
sw_vers -productVersion
```

**왜 실행했나?**

Intel Mac의 macOS 및 Xcode 업데이트 제한이 Java 설치에도 영향을 주는지 확인하기 위해 운영체제 버전을 확인했다.

**명령어의 의미**

- `sw_vers`: macOS의 제품 이름, 버전, 빌드 정보 등을 확인하는 도구다.
- `-productVersion`: 그중 운영체제 버전만 출력한다.

**실제 전달받은 결과**

```text
15.7.7
```

macOS Sequoia 15.7.7을 사용하고 있었다. 이 Mac 모델은 공식적으로 Sequoia까지 지원하며, Intel Mac의 Sequoia 환경에서 사용할 수 있는 Temurin 17 패키지가 제공된다. 최신 Xcode의 설치 가능 여부와 Java 17의 설치 가능 여부는 구분해서 확인해야 한다.

참고: [Apple의 MacBook Pro 모델별 지원 정보](https://support.apple.com/en-ie/108052), [Homebrew Temurin 17 패키지](https://formulae.brew.sh/cask/temurin@17).

## 4. Homebrew로 JDK 17 설치

```bash
brew install --cask temurin@17
```

**왜 실행했나?**

Java 프로그램 실행 도구와 컴파일러가 포함된 JDK 17을 설치하기 위해 실행했다.

**명령어의 의미**

- `brew`: Homebrew 명령이다.
- `install`: 지정한 패키지를 다운로드하고 설치한다.
- `--cask`: macOS용 설치 패키지 등을 관리하는 Cask 방식으로 설치한다.
- `temurin@17`: Eclipse Adoptium에서 제공하는 Temurin JDK 17 패키지다.

Intel Mac에서는 해당 아키텍처에 맞는 x64 패키지를 사용한다. 설치는 기존 Java 버전을 모두 삭제하거나, 현재 터미널의 Java 선택 설정을 반드시 변경하는 작업은 아니다.

공식 명령: [Homebrew Temurin 17](https://formulae.brew.sh/cask/temurin@17).

## 5. 설치 후에도 Java 12가 나오는 현상 확인

```bash
java -version
```

**왜 다시 실행했나?**

설치한 Java 17이 현재 터미널에서도 사용되는지 확인하기 위해 실행했다.

**확인한 내용**

설치 후에도 `java -version`은 12를 표시했다. 여기서는 다음 두 가지를 구분해야 했다.

- **JDK가 설치되어 있는 상태**: 컴퓨터에 해당 버전의 JDK 파일이 존재한다.
- **그 JDK가 선택된 상태**: 터미널이나 개발 도구가 그 JDK를 실제로 사용한다.

설치된 버전과 현재 선택된 버전이 다를 수 있으므로, 설정을 바꾸기 전에 다음 명령으로 설치 목록을 다시 확인하도록 했다.

```bash
/usr/libexec/java_home -V
```

목록에 `17.0...`이 있는지 확인하는 단계다. 목록에 17이 없다면 버전 선택보다 설치 상태를 먼저 확인해야 한다.

## 6. 사용할 JDK의 위치를 JAVA_HOME에 지정

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

**왜 실행했나?**

현재 터미널에서 실행하는 Java 관련 도구가 JDK 17의 설치 위치를 참조하도록 설정하기 위해 실행했다.

**명령어의 의미**

- `/usr/libexec/java_home -v 17`: 설치된 JDK 중 버전 17에 해당하는 JDK의 홈 경로를 찾는다.
- 소문자 `-v 17`: 찾을 Java 버전을 지정한다. 목록을 출력하는 대문자 `-V`와 용도가 다르다.
- `$(...)`: 괄호 안의 명령을 실행하고, 그 출력값을 바깥 명령에 넣는 문법이다.
- `JAVA_HOME=...`: 찾은 JDK 경로를 `JAVA_HOME` 변수에 저장한다.
- `export`: 이 터미널에서 시작하는 자식 프로세스에서도 해당 환경 변수를 사용할 수 있도록 한다.

`JAVA_HOME`은 JDK의 홈 디렉터리를 가리킨다. `bin` 디렉터리 자체를 지정하는 변수가 아니다. 이 변수만 지정해도 모든 환경에서 `java` 명령의 실행 경로가 바뀌는 것은 아니므로 다음 단계에서 `PATH`도 설정했다.

## 7. Java 17의 실행 파일을 먼저 찾도록 PATH 설정

```bash
export PATH="$JAVA_HOME/bin:$PATH"
```

**왜 실행했나?**

터미널에서 `java` 또는 `javac`를 입력했을 때 JDK 17의 실행 파일을 먼저 찾도록 하기 위해 실행했다.

**명령어의 의미**

- `PATH`: 실행 파일을 찾을 디렉터리들을 순서대로 담고 있는 환경 변수다.
- `$JAVA_HOME/bin`: 선택한 JDK의 `java`, `javac` 같은 실행 파일이 들어 있는 디렉터리다.
- `:`: PATH에서 디렉터리를 구분하는 문자다.
- 맨 뒤의 `$PATH`: 기존 경로 목록을 유지한다.

새 경로를 앞에 추가했으므로 일반적인 명령 검색에서 Java 17의 실행 파일이 기존 Java 경로보다 우선된다. `JAVA_HOME`과 `PATH`가 같은 JDK를 가리키도록 맞추는 것이 목적이다.

## 8. Java 실행 도구와 컴파일러 버전 재확인

```bash
java -version
javac -version
```

**왜 다시 실행했나?**

설정이 실제 명령 실행에 반영되었는지, Java 실행 도구와 컴파일러가 모두 의도한 버전을 사용하는지 확인하기 위해 실행했다.

**확인한 내용**

사용자가 설정 후 17이 표시된다고 확인했다. 실제 세부 패치 버전 문자열은 공유되지 않았으므로 이 기록에는 특정 `17.0.x` 버전을 단정해 적지 않는다.

## 이번에 배운 점

1. `java -version`은 현재 실행되는 Java 버전을 확인한다.
2. `/usr/libexec/java_home -V`는 macOS가 인식하는 설치된 JDK들을 확인한다.
3. JDK를 설치하는 것과 사용할 JDK를 선택하는 것은 별개의 작업이다.
4. `JAVA_HOME`은 JDK의 위치를 지정하고, `PATH`는 명령 실행 파일의 검색 순서를 정한다.
5. `export`로 지정한 설정은 현재 터미널 세션과 그 환경을 상속한 자식 프로세스에 적용된다. 새 터미널에서도 유지하려면 별도 설정이 필요하다.

## 다음에 진행할 일

- 사용하는 셸의 시작 파일에 Java 17 설정을 저장해 새 터미널에서도 유지되도록 한다.
- IDE의 프로젝트 JDK와 빌드 도구의 JDK를 확인한다. 터미널의 설정과 IDE 설정은 서로 다를 수 있다.
- Spring Initializr로 Spring Boot 프로젝트를 생성한다.

위 항목들은 아직 진행하지 않은 다음 단계다. 이 문서는 현재 터미널에서 Java 17을 사용하도록 설정하고 확인한 시점까지의 기록이다.

## 참고 문서

- [macOS에서 JDK 17 설치 및 버전 선택 — Oracle](https://docs.oracle.com/en/java/javase/17/install/installation-jdk-macos.html)
- [Temurin 17 설치 명령과 지원 플랫폼 — Homebrew](https://formulae.brew.sh/cask/temurin@17)
- [MacBook Pro 모델별 지원 운영체제 — Apple](https://support.apple.com/en-ie/108052)
- [Xcode의 macOS 요구 사항 — Apple](https://developer.apple.com/xcode/system-requirements)
