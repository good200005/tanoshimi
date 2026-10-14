# Render 배포 가이드 (포트폴리오 데모용)

구성: **Render(웹 서비스, Docker)** + **TiDB Cloud** (Render 는 MySQL 을 제공하지 않음).
GitHub Pages 정적 데모(`pages/`)와는 별개로, 로그인·파티·플래너·채팅이 실제 동작하는 서비스를 띄운다.

## 1. TiDB Cloud 준비
TiDB Cloud Starter(무료) 클러스터를 만들고 Connect → General 에서 Host / Port(4000) / Username(`xxxx.root`) / Password 를 확보한다. Allowed IP 는 `0.0.0.0/0`(Render 무료 플랜은 고정 IP 없음).
TLS 가 필수이며, 앱의 기본 JDBC URL 은 `useSSL=false` 라서 아래 3번처럼 `SPRING_DATASOURCE_URL` 로 덮어써야 한다.

## 2. DB 초기화 (최초 1회, 로컬에서 실행)
```
mysql -h <HOST> -P 4000 -u <USER> --ssl-mode=REQUIRED --default-character-set=utf8mb4 -p < src/main/resources/db/schema.sql
mysql -h <HOST> -P 4000 -u <USER> --ssl-mode=REQUIRED --default-character-set=utf8mb4 -p tanoshimi < src/main/resources/db/data.sql
```
- `schema.sql` 만으로 최신 스키마가 만들어진다. **`migration_*.sql` 은 실행하지 않는다.** 특히 `migration_v20_usernames.sql` 은 저장 프로시저(`CREATE PROCEDURE`)를 쓰는데 TiDB 는 지원하지 않는다.
- 로컬 DB 의 데이터를 그대로 옮기려면(데모 화면과 동일): 로컬에서 `mysqldump --no-create-info --complete-insert --skip-triggers tanoshimi > dump.sql` 후, TiDB 의 기존 행을 비우고(`SET FOREIGN_KEY_CHECKS=0;` 아래에서 `TRUNCATE`) `dump.sql` 을 넣는다.
- 업로드 이미지 파일은 DB 에 없어서 옮겨지지 않는다.

## 3. Render 서비스 생성
1. Render → New → **Blueprint** → 이 저장소 선택(루트 `render.yaml` 사용).
2. `sync: false` 인 값 입력: `SPRING_DATASOURCE_URL`(TLS 옵션 포함, `render.yaml` 주석 참고), `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`, OAuth·SMTP 값.
3. 배포 후 `https://<서비스>.onrender.com/login` 확인.

## 환경변수 요약
| 키 | 설명 |
|---|---|
| `SPRING_DATASOURCE_URL/USERNAME/PASSWORD` | TiDB 접속 정보(`sslMode=REQUIRED` 필수) |
| `SERVER_FORWARD_HEADERS_STRATEGY=framework` | Render 프록시(HTTPS) 뒤에서 `{baseUrl}`·WebSocket 이 https 로 인식되게 함 |
| `SERVER_SERVLET_SESSION_COOKIE_SECURE=true` | HTTPS 쿠키(application.yml 주석의 "배포 시 true") |
| `REMEMBER_ME_KEY` | 자동 로그인 서명 키 — Render 가 자동 생성 |
| `UPLOAD_DIR=/tmp/uploads` | 업로드 저장 위치. **무료 플랜은 재시작 시 사라짐** |
| `APP_EMAIL_PROVIDER`, `MAIL_HOST/USERNAME/PASSWORD/FROM` | 가입 인증 메일 실발송(smtp). Gmail 은 앱 비밀번호 사용 |
| `*_CLIENT_ID/SECRET` | 소셜 로그인. 비면 기동 실패하므로 기본 `dummy`, 발급 후 교체 |

## 소셜 로그인 키 발급 후
각 콘솔의 리디렉션 URI 에 `https://<서비스>.onrender.com/login/oauth2/code/{google|naver|line}` 를 등록하고 키를 교체한다.

## 한계 (무료 플랜)
- 15분 무요청 시 슬립 → 첫 접속 30~60초 지연.
- 업로드 이미지 휘발(영구 보관은 S3 등 필요). 이메일 인증은 SMTP 환경변수를 넣어야 실제 발송된다(미설정 시 발송 실패). 데모는 시드 계정(`yuja@test.com` / `Test1234!`)으로도 로그인 가능.
- AI 챗봇·날씨·번역은 `mock` 이 기본. 외부 AI 키를 넣으면 호출 요금이 발생한다.
