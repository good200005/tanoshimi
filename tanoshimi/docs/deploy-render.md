# Render 배포 가이드 (포트폴리오 데모용)

구성: **Render(웹 서비스, Docker)** + **외부 MySQL** (Render 는 MySQL 을 제공하지 않음).
GitHub Pages 정적 데모(`pages/`)와는 별개로, 로그인·파티·플래너·채팅이 실제 동작하는 서비스를 띄운다.

## 1. MySQL 준비
Aiven / TiDB Cloud 등 무료 MySQL 8 하나를 만들고 호스트·포트·계정·비밀번호를 확보한다(`utf8mb4`).

## 2. DB 초기화 (최초 1회, 로컬에서 실행)
앱을 먼저 켜지 말고 아래 순서로 넣는다(`src/main/resources/db/README.md` 의 필수 순서와 동일).

```
cd tanoshimi
for f in schema data migration_v16_mypage_titles migration_v16_planner_manner_ai \
         migration_v17_titles_catalog migration_v18_email_verification \
         migration_v18_titles_distance_tiers migration_v19_my_trips \
         migration_v19_temporary_password migration_v20_usernames; do
  mysql -h <HOST> -P <PORT> -u <USER> -p --default-character-set=utf8mb4 < src/main/resources/db/$f.sql
done
```
- 매니지드 DB 가 데이터베이스 생성을 막으면 `schema.sql` 상단의 `CREATE DATABASE`/`USE` 를 지우고 DB 이름을 맞춘다.
- 데모 데이터가 필요하면 `demo_mypage_seed.sql` 도 실행(DB README 참고).

## 3. Render 서비스 생성
1. Render → New → **Blueprint** → 이 저장소 선택(루트 `render.yaml` 사용).
2. `sync: false` 인 값 입력: `DB_HOST`, `DB_USERNAME`, `DB_PASSWORD` (`DB_PORT`/`DB_NAME` 이 다르면 수정).
3. 배포 후 `https://<서비스>.onrender.com/login` 확인.

## 환경변수 요약
| 키 | 설명 |
|---|---|
| `DB_HOST/PORT/NAME/USERNAME/PASSWORD` | 외부 MySQL 접속 정보 |
| `SERVER_FORWARD_HEADERS_STRATEGY=framework` | Render 프록시(HTTPS) 뒤에서 `{baseUrl}`·WebSocket 이 https 로 인식되게 함 |
| `SERVER_SERVLET_SESSION_COOKIE_SECURE=true` | HTTPS 쿠키(application.yml 주석의 "배포 시 true") |
| `REMEMBER_ME_KEY` | 자동 로그인 서명 키 — Render 가 자동 생성 |
| `UPLOAD_DIR=/tmp/uploads` | 업로드 저장 위치. **무료 플랜은 재시작 시 사라짐** |
| `*_CLIENT_ID/SECRET` | 소셜 로그인. 비면 기동 실패하므로 기본 `dummy`, 발급 후 교체 |

## 소셜 로그인 키 발급 후
각 콘솔의 리디렉션 URI 에 `https://<서비스>.onrender.com/login/oauth2/code/{google|naver|line}` 를 등록하고 키를 교체한다.

## 한계 (무료 플랜)
- 15분 무요청 시 슬립 → 첫 접속 30~60초 지연.
- 업로드 이미지 휘발(영구 보관은 S3 등 필요). 이메일 인증은 `log` 모드라 인증번호가 Render 로그에만 찍힌다. 데모는 시드 계정(`yuja@test.com` / `Test1234!`)으로 로그인.
- AI 챗봇·날씨·번역은 `mock` 이 기본. 외부 AI 키를 넣으면 호출 요금이 발생한다.
