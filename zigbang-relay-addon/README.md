# Zigbang Doorlock Relay

직방(삼성SDS IHP) 도어락을 클라우드 경유 없이(de-cloud) 로컬에서 투명 릴레이 — 락은 그대로
동작하면서 전체 MQTTS/UDP 트래픽을 관찰하고, `shadow` 모드에서는 서버 장애시에도 락이 계속
동작하도록 로컬에서 대신 응답한다.

- **proxy**: 순수 통과 + 관찰(tap). 첫 설치 안전값.
- **shadow**: proxy + ack 선응답/dedup + 업스트림 장애시 로컬 fallback.
- **decloud**: 업스트림 없이 완전 로컬 브로커.

설치/설정은 `DOCS.md` 참조 — **사전빌드 이미지를 직접 올려야 동작함**(이 add-on 저장소는
Dockerfile을 HA가 빌드하지 않고 `image:`로 지정된 사전빌드 이미지를 pull함).
