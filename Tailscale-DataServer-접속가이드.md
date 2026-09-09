# 데이터 서버 접속 가이드 (Tailscale + Windows 공유 폴더)

이 문서는 Tailscale 네트워크로 연결된 Windows 데이터 서버(`DataServer` 공유 폴더)에 각 운영체제에서 접속하는 방법을 정리한 것입니다.

## 서버 정보

| 항목 | 값 |
|---|---|
| 서버 호스트명 | `desktop-nie0l72` |
| Tailscale IP | `100.111.244.100` |
| 공유 폴더 이름 | `DataServer` |
| 서버 측 실제 경로 | `C:\DataServer` |
| 접속 계정 | `정우성` (Windows 로컬 관리자 계정) |

> 서버에 접속하려는 모든 기기는 **동일한 Tailscale 계정(tailnet)에 로그인**되어 있어야 합니다. Tailscale이 없으면 아무리 같은 Wi-Fi/네트워크에 있어도 접속할 수 없습니다.

---

## 사전 준비 (모든 OS 공통)

1. 해당 기기에 Tailscale 설치: https://tailscale.com/download
2. 서버와 **같은 Tailscale 계정**으로 로그인
3. 로그인 후 아래 명령/화면으로 서버가 목록에 보이는지 확인 (선택)
   ```
   tailscale status
   ```
   `desktop-nie0l72` 항목이 보이면 정상 연결된 상태입니다.

---

## Windows에서 접속

### 방법 A — 탐색기 주소창
1. 파일 탐색기 실행 → 주소창에 입력:
   ```
   \\100.111.244.100\DataServer
   ```
2. Enter → 로그인 창이 뜨면 `정우성` / 해당 계정 암호 입력
3. "자격 증명 저장"을 체크하면 다음부터 자동 로그인

### 방법 B — 네트워크 드라이브로 연결 (지속적으로 사용할 경우 추천)
1. 탐색기 → **내 PC** 우클릭 → **네트워크 드라이브 연결**
2. 드라이브 문자 선택, 폴더 경로에 `\\100.111.244.100\DataServer` 입력
3. "다른 자격 증명으로 연결" 체크 → 로그인 정보 입력
4. "로그온할 때 다시 연결" 체크 → 재부팅 후에도 자동 연결

---

## macOS에서 접속

1. Finder 실행
2. 메뉴 바에서 **이동(Go) → 서버에 연결...** (`Cmd + K`)
3. 서버 주소에 입력:
   ```
   smb://100.111.244.100/DataServer
   ```
4. **연결** 클릭 → 사용자 이름 `정우성`, 암호 입력
5. 연결되면 Finder 사이드바 "위치"에 표시되고, 이후 즐겨찾기로 등록 가능 (연결된 상태에서 `Cmd + D`로 Finder에 드래그)

---

## Linux에서 접속

### GUI (GNOME Files / Nautilus 등)
1. 파일 관리자 실행 → 주소창에 `Ctrl + L`
2. 입력:
   ```
   smb://100.111.244.100/DataServer
   ```
3. 사용자 이름/암호 입력 후 연결

### CLI (마운트 방식)
```bash
sudo apt install cifs-utils   # Debian/Ubuntu 계열, 최초 1회
sudo mkdir -p /mnt/dataserver
sudo mount -t cifs //100.111.244.100/DataServer /mnt/dataserver \
  -o username=정우성,password=<암호>,uid=$(id -u),gid=$(id -g)
```
매번 암호를 입력하지 않으려면 자격 증명 파일을 만들어 `-o credentials=/경로/파일`로 대체할 수 있습니다.

---

## Android에서 접속

1. Play 스토어에서 SMB 지원 파일 관리자 설치 (예: **Solid Explorer**, **CX 파일 탐색기**, 또는 기기 기본 "파일" 앱의 "네트워크 저장소 추가" 기능)
2. Tailscale 앱 설치 및 로그인 (동일 tailnet)
3. 파일 관리자에서 **네트워크 저장소 추가 → SMB/CIFS** 선택
4. 서버 주소: `100.111.244.100`, 공유 이름: `DataServer`
5. 사용자 이름 `정우성`, 암호 입력 후 저장

---

## iOS / iPadOS에서 접속

1. App Store에서 Tailscale 앱 설치 및 로그인 (동일 tailnet)
2. 기본 **파일(Files)** 앱 실행
3. 오른쪽 위 **⋯(더보기)** → **서버 연결**
4. 서버 주소:
   ```
   smb://100.111.244.100/DataServer
   ```
5. 사용자 이름 `정우성`, 암호 입력 후 연결
6. 연결되면 "위치" 목록에 표시되며, 이후 파일 앱에서 바로 탐색 가능

---

## 문제 해결

| 증상 | 확인할 것 |
|---|---|
| 서버 주소가 안 열림 | 두 기기 모두 Tailscale이 **로그인** 상태인지 확인 (`tailscale status`) |
| 자격 증명 오류 | 계정명(`정우성`)과 암호가 정확한지, 공유 권한(`Get-SmbShareAccess`)에 해당 계정이 있는지 확인 |
| 목록에는 뜨는데 폴더가 안 열림 | 서버에서 Windows 방화벽의 "파일 및 프린터 공유" 규칙이 활성화되어 있는지 확인 |
| macOS/Linux에서 접속이 유독 느림 | SMB1 폴백 문제일 수 있음 — 서버 SMB 버전 확인 (`Get-SmbServerConfiguration`) |

---

*이 문서는 로컬 데이터 서버 구축 과정에서 작성되었습니다.*
