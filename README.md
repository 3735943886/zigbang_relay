# zigbang_relay

직방(삼성SDS IHP) 도어락 de-cloud 도구 배포 저장소입니다.

## 바이너리

[Releases](../../releases)에서 빌드된 바이너리를 받을 수 있습니다.

- **`relay`** — 도어락↔클라우드 사이를 중계하는 상시 데몬. `linux-amd64` / `linux-arm64`.
  Docker 이미지: [3735943886/zigbang-relay](https://hub.docker.com/repository/docker/3735943886/zigbang-relay)
- **`provision`** — 도어락 최초 등록(SoftAP 커미셔닝)용 CLI. `linux-amd64` / `linux-arm64` /
  `linux-i686`(iSH 등 32비트 x86 환경용) / `windows-amd64`.

각 릴리즈 태그의 Assets에서 자신의 아키텍처에 맞는 파일을 받으면 됩니다.

## Home Assistant Add-on

`relay`를 HA add-on으로 설치하려면 HA → 설정 → 애드온 → 애드온 스토어 → 우측 상단 ⋮ → 저장소
→ 이 저장소 URL(`https://github.com/3735943886/zigbang_relay`) 추가 → "Zigbang Doorlock Relay"
설치. 설정법은 [`zigbang-relay-addon/DOCS.md`](zigbang-relay-addon/DOCS.md) 참조.
