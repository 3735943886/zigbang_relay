# Changelog

## 0.1.10

- **보안 fix(shadow/decloud)**: 기본 규칙 `on_cloud_push`가 msgCode `MSGBA0300001`이면 전부 replay
  저장했는데, 이 코드로는 크리덴셜 등록(func 408)뿐 아니라 **원격 열림(407)·삭제(420)·402·508**도 옴.
  pinName이 없는 이것들이 `keysync//` 한 슬롯을 덮어써서, 마지막이 407이면 장애 후 락 재접속 부트스트랩
  때 relay가 **열림 명령을 재생**할 수 있었음. 이제 이름 있는 func 408만 저장. proxy 모드(기본값)는
  정책 규칙이 꺼져 있어 영향 없음. 기존 shadow/decloud 사용자는 `replay_store` 파일에서 `keysync//`
  로 끝나는 키를 지울 것.
- relay core 범용화: 기기 식별(clientId 해석)·dedup 키·토큰 기억·관찰자 retain(dataBus)·주입 대상을
  Rust 하드코딩에서 rules 훅(`identify`/`correlation_id`/`learn`/`retain_key`/`retain_merge`/
  `inject_target`/`decode`)으로 이전 — 직방 동작은 `99-default.rhai`가 그대로 재현. 다른 도어락은
  core 수정 없이 규칙 파일 추가로 지원(비-JSON payload, `payload` 원문 응답 포함). README §3.
- proxy 모드도 같은 `rules/`를 로드하되 정책 훅은 끔(프로토콜 훅만) — 커스텀 규칙의 기기 식별이
  proxy에서도 적용됨. 타이머(`on_tick`)는 이제 정책 모드 세션에만 감(이전엔 proxy 세션에도 감).
- 호환: `on_local_inject(msg, topic, ctx)` 3-인자 추가(2-인자도 호출됨). 단, 예전엔 relay가 주입 메시지의
  `msg.authToken`을 클라우드 토큰으로 몰래 덮어써줬는데 이제 안 함 — 커스텀 2-인자 규칙이 이 값을 썼다면
  `ctx.cloudAuthToken`(기본 규칙의 `learn`이 채움)으로 바꿀 것.
- **breaking(관찰자 API)**: 세션 이벤트(`relay/session/<id>`) payload의 기기 id 필드가 `tpId` → `device`
  (`{"device","online","peer","at"}`). zigbang_doorlock 통합은 `device`→`tpId`→토픽 순으로 읽는 버전부터
  호환 — **"외부" 연결(add-on/Docker/systemd) 사용자는 통합을 먼저 업데이트**할 것(구버전 통합은 "relay
  연결" 센서가 갱신 안 됨). 통합 내장 relay는 버전 고정이라 무관. 직접 구독하는 스크립트도 `device`로.

## 0.1.9

- add-on: 지원 중단 철회. HAOS에서는 HA core와 별개로 실행되는 이 add-on에 zigbang_doorlock 통합을
  "외부" 방식으로 연결하는 것을 권장(DOCS §6). 통합의 내장 relay는 HA와 함께 시작·종료됨.
- fix: `upstream_insecure = false`가 신뢰 루트 없이(빈 RootCertStore) 접속해서 어떤 업스트림에도 붙지
  못하던 문제 — 이제 공인CA(webpki-roots)로 정상 검증. 기본값(true, 검증 안 함)은 그대로. 참고로 직방
  클라우드(`*.homeiot.zigbang-iot.com`) 인증서가 2026-07-27 만료된 채 서빙 중이라 false면 현재 접속
  실패(업스트림 장애로 보고 Local로 동작).

## 0.1.8

- relay: 락 세션 접속/종료를 관찰자(9883)에 이벤트로 알림 — `relay/session/<tpId>`
  `{"tpId","online","peer","at"}`. 관찰자가 구독하는 순간엔 접속중 목록 스냅샷 `relay/sessions`
  `{"online":[...]}`(비어있어도 보냄)과 락별 상세를 재생. 같은 락이 재접속한 뒤 옛 세션이 늦게 끝나도
  offline은 안 보냄. zigbang_doorlock 통합의 "relay 연결" 센서가 이걸 씀.

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
