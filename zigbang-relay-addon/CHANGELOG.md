# Changelog

## 0.1.7

- fix(add-on): 구성 탭 설정(`/data/options.json`)을 안 읽고 `relay.toml`을 찾다가 기동 실패하던
  문제 — 이미지 기본 CMD를 없애고, 인자 없이 뜨면 `/data/options.json`이 있을 때 자동으로 읽음
  (없으면 기존처럼 `relay.toml`). (#1)
  **주의**: 우회로 `/data/relay.toml`을 직접 만들어 쓰던 경우, 업데이트 후엔 구성 탭 값이 우선
  적용되므로 그 내용을 구성 탭으로 옮긴 뒤 업데이트할 것.
- fix(add-on): 이미지가 UID 10001로 돌아서 `/data` 아래 `logs/`·`state/` 생성 실패 + Let's Encrypt
  add-on의 `/ssl/privkey.pem`(root 600)을 못 읽던 문제 — 이미지 기본 유저를 root로 변경(HA add-on
  관례). 일반 Docker는 `--user $(id -u):$(id -g)`로 비root 실행 권장(기존 안내 그대로). (#1)
- docs: DOCS §2 rules 경로를 실제 위치(`docker/rules/`)로 수정. (#1)

## 0.1.6

- fix: `cert_file`/`cert_key`가 없을 때 자체서명 인증서를 만들어 그 경로에 써버리던 동작 제거 —
  하나만 있을 때(예: 공유경로 `/ssl`에 개인키만 있음) 남은 파일까지 덮어쓰는 문제가 있었음. 이제
  인증서가 없으면 에러로 기동 실패. 랩/테스트용 자체서명이 필요하면 `cert_self_signed = true`
  (그때도 둘 다 없을 때만 생성, 하나라도 있으면 절대 덮어쓰지 않음).
- fix: 18883/5683/9883 포트 bind 실패 등으로 listener가 죽어도 프로세스는 FAIL 로그만 남기고
  반쪽 상태로 계속 떠있던 문제 — 이제 에러로 종료해서 Docker `--restart`/add-on/HA 내장 relay가
  재기동함.
- 배포: relay 바이너리를 PyPI 패키지 `zigbang-relay`(Linux x86_64/aarch64 wheel)로도 배포 —
  HA 커스텀 컴포넌트 zigbang_doorlock의 "내장 relay" 모드용.

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
