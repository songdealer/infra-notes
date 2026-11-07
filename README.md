# infra-notes

Linux, Docker, Kubernetes 등 인프라 환경의 설치와 운영 과정을 정리하는 문서 저장소입니다. 실제 설정 순서, 확인한 화면, 문제 해결 방법을 함께 기록합니다.

## 문서 목록

| 분야 | 문서 | 내용 |
|---|---|---|
| Linux | [linuxVM 설치방법](./linux/utm-ubuntu/README.md) | Intel MacBook에서 UTM으로 Ubuntu Server 설치, 스크린샷 5장, ISO 연결 해제와 재부팅 문제 해결 |
| 네트워크 | [UTM 공유 네트워크를 브리지로 변경하기](./linux/utm-ubuntu/bridged-network.md) | 공유·브리지 모드의 차이, 인터페이스 선택, VM IP와 통신 확인, DHCP 예약 전 준비 |
| Java | [Intel Mac에 Java 17 설치하고 버전 선택하기](./java/macos-java17-setup.md) | Homebrew 설치, Java 12에서 17로 전환, JAVA_HOME·PATH 설정과 명령별 설명 |
| Spring Boot | [Gradle 의존성 정리](./spring-boot/gradle-dependencies.md) | 기본·웹·테스트 스타터, Initializr 모듈, BOM과 Gradle 선언 방식, 현재 적용 상태 |
| 데이터베이스 | [MySQL과 PostgreSQL 비교](./database/mysql-vs-postgresql.md) | 채팅 프로젝트 관점의 공통점, ID 생성·중복 처리·문자열 비교·JSON 차이와 선택 기준 |

## 문서와 프로젝트 코드 분리

- **공통 설치·운영 문서:** 이 `infra-notes` 저장소에서 기술별로 관리합니다.
- **프로젝트 코드:** Forus 같은 프로젝트의 별도 저장소에서 관리합니다.
- **프로젝트 전용 실행·배포 설정:** 해당 프로젝트 저장소의 `README.md` 또는 `docs/`에 두고, 공통 설치 절차는 이 저장소의 문서를 링크합니다.
- **스크린샷:** 각 문서 옆의 `images/` 폴더에 저장하고 상대 경로로 연결합니다.

## 저장소 구조

```text
infra-notes/
├── README.md
├── database/
│   └── mysql-vs-postgresql.md
├── spring-boot/
│   └── gradle-dependencies.md
├── java/
│   └── macos-java17-setup.md
└── linux/
    └── utm-ubuntu/
        ├── README.md
        ├── bridged-network.md
        └── images/
            ├── 01-boot-menu.png
            ├── 02-install-confirmation.png
            ├── 03-reboot-cdrom.png
            ├── 04-utm-cdrom-location.png
            └── 05-eject-iso-menu.png
```

새 문서는 관련 기술 폴더 아래에 추가하고, 위의 문서 목록에 링크를 등록합니다.

