# Changelog

## 0.1.3

- **[breaking]** 규칙파일이 `rules.json`(선언적 JSON)에서 `rules/*.rhai`([Rhai](https://rhai.rs)
  스크립트) 디렉터리로 전면 교체됨 — 기존 `rules.json` 쓰던 사용자는 `/data/rules/` 로
  마이그레이션 필요(DOCS.md §2). 각 파일이 `on_message`/`on_cloud_push`/`on_local_inject`/
  `on_session_start`/`on_tick` 중 필요한 것만 정의.
- relay: `on_tick` 타이머 트리거 추가 — 클라우드 메시지 없이도 relay 자체적으로 주기적인
  자가생성 패킷을 락에 보낼 수 있음(완전 de-cloud 확장 대비, 파일별 독립 간격).

## 0.1.2

- add-on: HA 표준 `/ssl` 공유경로 사용(Let's Encrypt add-on 연동, cert 수동배치 대체).
- provision: 로그인 경로에서도 de-cloud 브로커 도메인을 항상 물어봄(엔터=정식클라우드 유지).
- ci: provision에 macOS(amd64/arm64) 빌드 추가, windows 릴리즈를 zip으로 패키징.

## 0.1.1

- rules.json `on_local_inject`: 관찰자 로컬 언락 주입시 클라우드로 이력 미러링(공식앱 히스토리 보정).
- provision/relay 바이너리 릴리즈 워크플로 추가(별도 공개 저장소 `zigbang_relay`).

## 0.1.0

- 최초 add-on 패키징. proxy/shadow/decloud 3모드, `/data` 단일 레이아웃(systemd/Docker 배포와 공유).
