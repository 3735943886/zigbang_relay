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

## 2. rules.json / cert 배치

`/data`(add-on 데이터 폴더, Samba 공유나 File editor add-on으로 접근 가능)에 아래를 채운다 —
systemd/Docker 배포와 완전히 같은 레이아웃(`deploy/README.md` 참조):

```
/data/rules.json          # 레포 루트 rules.json 복사(핫리로드, 이후 add-on 재시작 불필요)
/data/certs/fullchain.pem
/data/certs/privkey.pem
```

cert는 certbot deploy-hook으로 자동 갱신하는 걸 권장(`deploy/README.md` §2, 경로만
`/data/certs/`로 바꿔서). mtime 핫리로드라 add-on 재시작 불필요.

## 3. 옵션 설정

애드온 "구성" 탭에서 `cert_name`(실제 주입 도메인)과 `route[].upstream`(실클라우드 호스트)을
편집. 나머지는 기본값으로 충분. 저장 후 애드온 재시작.

## 4. 확인

"로그" 탭에서 `relay 시작 — routes=2 ...` 확인. 평문 tap(관찰자 9883)은 host로 안 열려있음 —
HA 내부 도커망에서만 컨테이너명으로 접근 가능(의도적, `config.yaml` 주석 참조). 필요하면
`config.yaml`의 `ports`에 `9883/tcp` 추가해서 직접 열 수 있음(로컬신뢰망 밖으로는 절대 노출 금지).
