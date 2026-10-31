# zigbang_relay

직방(삼성SDS) 도어락 de-cloud 도구 배포 저장소입니다.

## 바이너리

[Releases](../../releases)에서 빌드된 바이너리를 받을 수 있습니다.

- **`provision`** — 도어락 최초 등록(SoftAP 커미셔닝)용 CLI. `linux-amd64` / `linux-arm64` /
  `linux-i686`(iSH 등 32비트 x86 환경용) / `windows-amd64` / `macos-amd64`(Intel) /
  `macos-arm64`(Apple Silicon).
- **`relay`** — 도어락↔클라우드 사이를 중계하는 상시 데몬. `linux-amd64` / `linux-arm64`.
  Docker container 또는 직접 실행. Home Assistant에선
  [zigbang_doorlock](https://github.com/3735943886/zigbang_doorlock) 통합의 내장 relay
  (PyPI [`zigbang-relay`](https://pypi.org/project/zigbang-relay/))를 쓰세요.

각 릴리즈 태그의 Assets에서 자신의 아키텍처에 맞는 파일을 받으면 됩니다.

## Docker Container

Docker 이미지: [3735943886/zigbang-relay](https://hub.docker.com/repository/docker/3735943886/zigbang-relay)

## Home Assistant App(Add-on) — 지원 중단

HA App은 **지원 중단(deprecated)**됐습니다. 새로 설치하지 말고
[zigbang_doorlock](https://github.com/3735943886/zigbang_doorlock) 통합의 **내장 relay** 모드를
쓰세요(HA가 relay를 직접 실행 — App·포트 매핑·권한 설정 불필요). 기존 App 사용자는
[`zigbang-relay-addon/DOCS.md`](zigbang-relay-addon/DOCS.md) 맨 위의 이전 방법을 참조하세요
(도어락 재프로비저닝 불필요). App은 기존 설치를 위해 당분간 남겨두지만 새 기능은 추가하지 않습니다.

