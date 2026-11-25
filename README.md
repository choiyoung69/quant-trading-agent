# agent

프론트엔드와 백엔드를 한 프로젝트 폴더에서 관리합니다.

```text
agent/
├── frontend/          # React + TypeScript + Vite (npm workspace)
├── backend/           # Spring Boot + Java 21 (Gradle 모듈)
│   ├── src/
│   └── build.gradle
├── package.json       # 프론트·백 통합 실행 명령
├── package-lock.json
├── settings.gradle    # backend 모듈 등록
├── build.gradle
└── gradlew
```

## 준비

Java 21과 Node.js 22.12 이상이 필요합니다. 최초 실행 시 루트에서 설치합니다.

```sh
npm ci
```

현재 백엔드에는 JPA/JDBC 의존성이 있으므로 실행 전 DB 연결이 필요합니다.
`backend/src/main/resources/application.properties`에 실제 DB 설정을 추가하고,
`backend/build.gradle`에 사용하는 DB의 JDBC 드라이버를 추가하세요.
설정이 없으면 백엔드 실행과 기존 `contextLoads` 테스트가 실패합니다.

## 실행

루트에서 프론트와 백을 함께 실행합니다. 종료는 Ctrl+C입니다.
한 서버가 종료되면 다른 서버도 함께 종료합니다.

```sh
npm run dev
```

- 프론트: http://localhost:5173
- 백엔드: http://localhost:8080

따로 실행할 수도 있습니다.

```sh
npm run dev:frontend
npm run dev:backend
```

프론트에서 `/api/...`로 요청하면 Vite 개발 서버가 백엔드 8080 포트로 전달합니다.
백엔드에는 아직 API가 구현되어 있지 않습니다. 이 프록시는 개발 환경 전용이며,
배포할 때는 별도로 API 주소 또는 리버스 프록시를 설정해야 합니다.

## 빌드와 검사

```sh
npm run build           # 프론트 빌드 후 백엔드 빌드·테스트
npm run build:frontend  # frontend/dist
npm run build:backend   # backend/build/libs (DB 설정 필요)
npm run lint            # 프론트 정적 검사
npm run test:backend    # 백엔드 테스트 (DB 설정 필요)
```

Gradle 명령도 루트에서 실행합니다.

```sh
./gradlew :backend:bootRun
./gradlew :backend:build
```

프론트 빌드 결과와 백엔드 JAR는 별도로 생성됩니다.
프론트의 기본 화면은 Vite 시작 템플릿입니다.

## 토스페이먼츠 환경변수

현재는 키를 읽는 환경 설정만 준비되어 있습니다. 결제 SDK와 결제 승인 API는 아직 구현하지 않았습니다.

1. [토스페이먼츠 개발자센터](https://developers.tosspayments.com/)에서 테스트 키를 확인합니다.
   결제창/API 개별 연동인지 결제위젯 연동인지에 맞춰 같은 연동용 클라이언트 키와 시크릿 키를 사용하세요.
2. `backend/.env`에 서버에서 사용할 시크릿 키를 입력합니다.

   ```dotenv
   TOSS_SECRET_KEY=발급받은_테스트_시크릿_키
   ```

3. `frontend/.env`에 공개 가능한 클라이언트 키를 입력합니다.

   ```dotenv
   VITE_TOSS_CLIENT_KEY=발급받은_테스트_클라이언트_키
   ```

4. 변경 후 개발 서버를 재시작합니다. 기존 DB 설정은 별도로 필요합니다.

키를 입력할 때 따옴표나 `export`를 붙이지 마세요.
백엔드는 `application.properties`의 `spring.config.import`로 `.env`를 properties 형식으로 읽습니다.
Gradle 실행은 backend 폴더 기준으로, 루트에서 JAR를 실행하면 backend/.env 경로로 읽습니다.
다른 폴더에서 JAR를 실행할 때는 환경변수 `TOSS_SECRET_KEY`를 직접 주입하세요.
운영 환경변수는 로컬 파일보다 우선합니다. 백엔드는 `.env.local` 등은 자동으로 읽지 않습니다.

백엔드 서비스나 컴포넌트에서는 다음처럼 주입합니다.

```java
@Value("${toss.payments.secret-key}")
private String tossSecretKey;
```

프론트에서는 Vite가 `.env`를 자동으로 읽습니다.

```ts
const clientKey = import.meta.env.VITE_TOSS_CLIENT_KEY
```

`VITE_`로 시작하는 변수는 브라우저에 공개됩니다. 시크릿 키는 반드시 `backend/.env`에만 보관하세요.
`.gitignore`는 모든 패키지의 `.env`, `.env.*`를 제외하고 빈 `.env.example`만 공유하도록 설정되어 있습니다.
새로 내려받은 환경에서는 각 패키지의 `.env.example`을 `.env`로 복사해서 값을 채우세요.

공식 문서: [키 관리](https://docs.tosspayments.com/guides/v2/get-started/llms-quick-reference),
[Spring 외부 설정](https://docs.spring.io/spring-boot/reference/features/external-config.html),
[Vite 환경변수](https://vite.dev/guide/env-and-mode).
