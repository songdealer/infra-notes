# UTM 공유 네트워크를 브리지로 변경하기

Intel MacBook의 UTM에 설치한 Ubuntu VM을 집 공유기 네트워크에 연결하는 방법입니다. Mac Studio의 Ubuntu VM과 K3s 클러스터를 구성하기 전에 진행하는 네트워크 준비 단계입니다.

작성일: 2026년 11월 1일

이 문서는 설정 방법을 정리한 안내입니다. 실제 VM의 브리지 전환과 통신 성공 여부는 아래 확인 명령으로 검증해야 합니다.

이전 단계: [linuxVM 설치방법](./README.md)

## 공유기에 연결된 것은 Mac인데, VM도 별도 기기로 연결될 수 있나요?

**브리지 모드에서는 가능합니다.** VM은 MacBook의 Wi-Fi 또는 유선 네트워크 인터페이스를 통해 같은 LAN에 참여합니다. 물리적인 랜선을 VM에 따로 연결하지 않아도, VM의 가상 네트워크 카드가 자체 MAC 주소를 갖고 공유기에 IP를 요청할 수 있습니다.

| UTM 네트워크 모드 | VM의 IP와 통신 방식 |
|---|---|
| 공유 네트워크 / Shared Network | Mac 내부의 가상 네트워크에서 IP를 받고, Mac을 통해 외부와 통신합니다. 집 공유기가 VM에 직접 IP를 배정하는 구성은 아닙니다. |
| 브리지 / Bridged | Mac과 같은 공유기 네트워크에 참여하며, 공유기의 DHCP 서버에서 VM용 IP를 받을 수 있습니다. |

다음은 브리지가 정상 작동했을 때의 **논리적인 연결 예시**입니다. 실제 IP 주소는 공유기 설정에 따라 다릅니다.

```text
공유기가 IP를 배정하는 대상
├─ MacBook          192.168.35.10
├─ MacBook의 VM     192.168.35.11
├─ Mac Studio       192.168.35.20
└─ Mac Studio의 VM  192.168.35.21
```

따라서 이번 설정 순서는 **브리지 전환 → VM의 IP와 통신 확인 → DHCP 예약 → K3s 설치**입니다. 브리지는 두 VM 사이의 통신을 구성하는 한 가지 방법이며, Wi-Fi 환경에 따라 추가 설정이 필요할 수 있습니다. [UTM 공식 네트워크 설명](https://docs.getutm.app/settings-qemu/devices/network/network/)

## 1. Ubuntu 정상 종료

**실행 위치: Ubuntu VM 터미널**

```bash
sudo poweroff
```

UTM에서 VM 상태가 **정지됨**으로 바뀔 때까지 기다립니다. 일시정지 상태에서는 설정을 변경하지 말고 완전히 종료합니다.

## 2. UTM의 VM 설정 열기

1. UTM 메인 화면에서 **Linux** 가상머신을 선택합니다.
2. 오른쪽 위의 **슬라이더 모양 설정 버튼**을 누릅니다.
3. 왼쪽 목록에서 기존 **네트워크** 장치를 선택합니다.

이 문서는 기존 설치 화면의 QEMU 기반 VM을 기준으로 합니다. UTM 버전과 표시 언어에 따라 메뉴 이름이 조금 다를 수 있습니다.

## 3. 네트워크 모드를 브리지로 변경

**네트워크 모드**를 다음과 같이 변경합니다.

```text
공유 네트워크 / Shared Network
              ↓
브리지 / Bridged
```

일부 버전에서는 **Bridged (Advanced)**처럼 표시될 수 있습니다.

## 4. 실제 연결에 맞는 브리지 인터페이스 선택

**브리지 인터페이스 / Bridged Interface**에는 MacBook이 현재 공유기와 연결된 네트워크 인터페이스를 선택합니다.

| MacBook의 연결 방식 | 선택할 인터페이스 |
|---|---|
| Wi-Fi | 해당 Wi-Fi 인터페이스 |
| 내장 또는 USB 유선 랜 | 실제 연결에 사용하는 Ethernet 인터페이스 |

### en0, en1처럼 장치 이름만 표시될 때

**실행 위치: macOS 터미널**

```bash
networksetup -listallhardwareports
```

출력 예시:

```text
Hardware Port: Wi-Fi
Device: en0
```

이 예시에서는 Wi-Fi에 해당하는 장치가 `en0`이므로, Wi-Fi로 연결 중이라면 UTM에서 `en0`를 선택합니다. **모든 Mac의 Wi-Fi가 en0인 것은 아니므로 실제 출력을 확인합니다.**

기존 VM의 MAC 주소와 네트워크 카드 종류는 그대로 유지합니다. 브리지 모드와 연결 인터페이스를 선택한 뒤 **저장**합니다.

## 5. Ubuntu 실행 후 IP 확인

UTM에서 **▶ 실행**을 누르고 Ubuntu에 로그인합니다.

**실행 위치: Ubuntu VM 터미널**

```bash
ip -4 -br addr
ip route
```

브리지가 정상 작동했을 때의 출력 예시:

```text
lo       UNKNOWN  127.0.0.1/8
enp0s1   UP       192.168.35.11/24
```

```text
default via 192.168.35.1 dev enp0s1 proto dhcp src 192.168.35.11
192.168.35.0/24 dev enp0s1 proto kernel scope link src 192.168.35.11
```

| 확인할 값 | 예시의 의미 |
|---|---|
| `192.168.35.11` | Ubuntu VM에 배정된 IPv4 주소 |
| `default via 192.168.35.1` | 기본 게이트웨이 주소. 일반적인 집 공유기 구성에서는 공유기 주소 |
| `enp0s1` | Ubuntu에서 사용하는 가상 네트워크 인터페이스 이름 |
| `127.0.0.1` | VM 자신을 가리키는 루프백 주소. 공유기에서 배정한 주소가 아님 |

위 IP와 인터페이스 이름은 모두 예시입니다. SK 공유기라도 모델과 설정에 따라 주소 대역이 다르므로 예시 주소를 그대로 입력하지 않습니다.

Ubuntu 네트워크 설정은 **DHCP 자동 할당**으로 유지합니다. 공유기 관리자 화면의 DHCP 할당 목록에서 VM의 IP와 MAC 주소가 일치하는지 함께 확인합니다. VM의 MAC 주소는 UTM 상세 정보 또는 Ubuntu의 `ip link` 명령으로 확인할 수 있습니다.

## 6. 기본 통신 확인

Ubuntu VM에서 `ip route`에 표시된 실제 게이트웨이로 ping을 보냅니다. 아래 주소를 자신의 게이트웨이로 바꿉니다.

```bash
ping -c 3 192.168.35.1
```

Mac Studio의 VM까지 준비되면 각 Ubuntu에서 **상대 VM의 IP**로도 ping을 보내 양방향 응답을 확인합니다. ping만으로 K3s의 모든 통신이 검증되지는 않으며, 클러스터 설치 시 필요한 포트와 Pod 간 통신도 별도로 확인해야 합니다.

## IP가 안 나오거나 통신이 안 될 때

| 증상 | 먼저 확인할 내용 |
|---|---|
| VM에 IPv4 주소가 없음 | UTM 브리지 인터페이스가 실제 연결된 Wi-Fi 또는 Ethernet인지, Ubuntu가 DHCP 자동 설정인지 확인 |
| 이전 가상 네트워크 주소가 계속 표시됨 | 네트워크 모드를 저장했는지, VM을 완전히 종료 후 다시 실행했는지 확인 |
| IP는 받았지만 상대 VM에 연결되지 않음 | 두 VM이 같은 LAN에 있는지, 게스트 Wi-Fi·기기 간 통신 차단·방화벽 설정이 있는지 확인 |
| Wi-Fi 브리지가 작동하지 않음 | UTM 공식 문서처럼 추가 설정이 필요한 환경일 수 있음. 가능하면 유선 Ethernet 브리지로 확인 |

ping에 응답하지 않는 것만으로 브리지 실패를 단정하지 않습니다. ICMP 차단 여부와 공유기의 DHCP 할당 목록도 함께 확인합니다.

## 다음 단계: DHCP 예약

VM이 공유기에서 정상적으로 IP를 받은 뒤, **VM의 MAC 주소와 현재 IP**를 공유기의 DHCP 예약 기능에 등록하면 IP를 유지할 수 있습니다.

MacBook과 Mac Studio의 **VM별로 각각** 등록합니다. macOS 호스트의 MAC 주소를 대신 등록하지 않습니다. SK 공유기의 정확한 메뉴와 DHCP 예약 지원 여부는 모델 확인 후 진행합니다.

이 문서의 범위는 브리지 설정과 네트워크 확인까지입니다. DHCP 예약 완료 및 K3s 설치·클러스터 연결은 아직 이 문서에서 수행 결과로 기록하지 않았습니다.

## 참고 자료

- [UTM QEMU 네트워크 설정](https://docs.getutm.app/settings-qemu/devices/network/network/)
- [K3s 네트워크 요구사항](https://docs.k3s.io/installation/requirements#networking)
- [이전 문서: linuxVM 설치방법](./README.md)
- [전체 문서 목록](../../README.md)
