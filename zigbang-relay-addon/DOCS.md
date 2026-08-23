# 설정

## 0. 사전빌드 이미지 — CI가 자동으로 올림, 수동 빌드 불필요

이 add-on은 HA가 직접 빌드하지 않는다(루트 `Dockerfile`이 `FROM scratch`라 HA 표준 빌드
파이프라인 전제인 s6-overlay 베이스가 아님) — 대신 `config.yaml`의 `image:`(`3735943886/zigbang-relay`,
Docker Hub)가 가리키는 사전빌드 이미지를 pull한다. `.github/workflows/docker-publish.yml` 이
`v*` 태그 푸시마다 멀티아치(amd64+arm64)로 자동 빌드+푸시함 — 관리자가 수동으로 할 일 없음.

버전을 올릴 땐 `config.yaml`의 `version`을 올리고 **같은 값으로 git tag(`vX.Y.Z`)를 찍기만**
하면 CI가 알아서 그 버전 태그로 이미지를 올림(태그=버전이 곧 pull될 이미지, `config.yaml` 주석 참조).

## 1. 저장소 추가 + 설치

HA → 설정 → 애드온 → 애드온 스토어 → 우측 상단 ⋮ → 저장소 → 이 GitHub repo URL 추가 →
"Zigbang Doorlock Relay" 설치.

## 2. rules/ 배치

`/data`(add-on 데이터 폴더, Samba 공유나 File editor add-on으로 접근 가능)에 레포 루트의
`rules/` 디렉터리를 통째로 복사한다:

```
/data/rules/99-default.rhai
```

이 디렉터리 안의 `*.rhai`(파일명순 우선순위)가 락↔클라우드 메시지에 어떻게 응답/중계할지
정하는 [Rhai](https://rhai.rs) 스크립트다 — **mtime 핫리로드**라 파일만 갈아끼우면 add-on
재시작 없이 바로 반영된다. 각 파일이 정의할 수 있는 함수는 레포 루트 README §3 참고.

## 3. cert — Let's Encrypt add-on 연동 (권장)

도어락은 공인 CA 인증서만 수락한다(자체서명 무조건 거부, 호스트명도 검사함). HAOS엔 보통
certbot을 직접 돌릴 셸이 없으니, **커뮤니티 "Let's Encrypt" add-on**을 같이 설치해서 쓰는 걸
권장한다 — 그 add-on이 발급한 인증서는 HA 표준 공유경로 `/ssl/fullchain.pem`+`/ssl/privkey.pem`
에 떨어지고, 이 add-on은 `map: ["ssl:ro"]`로 그 경로를 읽기전용으로 이미 가져다 쓰게 기본
설정돼있다(`config.yaml`의 `cert_file`/`cert_key` 기본값 참조). 갱신도 mtime 핫리로드라 재시작
불필요 — Let's Encrypt add-on이 갱신하면 자동으로 반영됨.

**HAOS가 아니거나(예: Docker Compose로 직접 관리) `/ssl`을 안 쓰고 싶으면**: add-on "구성" 탭에서
`cert_file`/`cert_key`를 `/data/certs/fullchain.pem`/`/data/certs/privkey.pem` 등 원하는 경로로
바꾸고, 그 경로에 직접 인증서를 넣어주면 된다(certbot deploy-hook 등으로 자동화 가능, 마찬가지로
mtime 핫리로드).

## 4. 옵션 설정

애드온 "구성" 탭에서 `cert_name`(실제 주입 도메인)과 `route[].upstream`(실클라우드 호스트)을
편집. 나머지는 기본값으로 충분. 저장 후 애드온 재시작.

## 5. 확인

"로그" 탭에서 `relay 시작 — routes=2 ...` 확인. 평문 tap(관찰자 9883)은 host로 안 열려있음 —
HA 내부 도커망에서만 컨테이너명으로 접근 가능(의도적, `config.yaml` 주석 참조). 필요하면
`config.yaml`의 `ports`에 `9883/tcp` 추가해서 직접 열 수 있음(로컬신뢰망 밖으로는 절대 노출 금지).
