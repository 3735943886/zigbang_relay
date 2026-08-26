# Changelog

## 0.1.5

- add-on: armv7·armhf(armv6) 아키텍처 지원 추가(라즈베리파이2/3/4 32비트, 라즈베리파이1/Zero 등
  구형 보드 포함) — 이제 amd64/aarch64/armv7/armhf 4종.
- fix: 9883 관찰자의 retain 캐시(신규 구독시 "마지막 상태" 즉시 재생)가 이전엔 최근 payload를
  통째로 덮어써서, 도어락의 25분 주기 상태 풀덤프 뒤에 짧은 델타 하나만 와도 신규 구독자가
  firmInfo/wifiStrength 같은 필드를 못 보는 문제가 있었음 — 이제 필드단위로 병합해서 유지됨.
- add-on: 9883 관찰자에 선택적 id/pass 인증 추가(`observer_user`/`observer_pass`, 둘 다
  설정해야 활성화) — 9883을 host로 노출하는 경우를 위한 것으로, 기본은 예전처럼 무인증.

## 0.1.4

- relay: 도어락 여러 대 동시접속 지원 — 세션 상태(원격주입 대상, on_tick 타이머, replay 저장소)가
  전부 tpId(락 고유id)로 스코핑됨. 락 1대 배포엔 동작 변화 없음.
- fix: 락이 여러 대일 때 서로의 remote-unlock 명령주입/재시작 타이밍이 뒤섞이던 레이스, 그리고
  한 락의 keysync/credential replay가 다른 락에 잘못 재생되던 문제를 해결.

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
