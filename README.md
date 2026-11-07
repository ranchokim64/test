# 배포 연습 프로젝트

GitHub Pages와 Firebase Hosting으로 정적 웹사이트를 배포하는 흐름을 연습하기 위한 간단한 프로젝트입니다.
`index.html` 하나만으로 동작하는 페이지이며, 시계와 버튼 클릭 카운터가 있어 배포 후 정상 동작을 바로 확인할 수 있습니다.

## 1. GitHub Pages로 배포하기

### 방법 A: 웹에서 파일 업로드로 배포 (가장 간단)
1. GitHub에서 새 저장소(public)를 하나 만듭니다.
2. 저장소 메인 화면에서 **Add file → Upload files**를 클릭합니다.
3. 이 폴더의 `index.html`을 드래그해서 업로드하고 커밋합니다.
4. 저장소의 **Settings → Pages**로 이동합니다.
5. **Source**를 `Deploy from a branch`로 두고, Branch는 `main` / 폴더는 `/ (root)`를 선택한 뒤 **Save**를 누릅니다.
6. 1~2분 후 같은 화면에 표시되는 URL(`https://<username>.github.io/<repo>/`)로 접속해 확인합니다.

### 방법 B: 이 저장소를 그대로 push해서 배포
이미 이 브랜치(`claude/determined-gates-un264e`)에 `index.html`이 있으니, `main` 브랜치로 머지하거나 push한 뒤 위 5~6번 과정(Settings → Pages 활성화)만 진행하면 됩니다.

## 2. Firebase Hosting으로 배포하기

Firebase 설정과 프로젝트 연결 과정을 직접 연습하는 부분입니다.

1. Firebase CLI 설치 (최초 1회):
   ```bash
   npm install -g firebase-tools
   ```
2. Firebase 로그인:
   ```bash
   firebase login
   ```
3. [Firebase 콘솔](https://console.firebase.google.com/)에서 새 프로젝트를 생성합니다.
4. 이 프로젝트 폴더에서 Hosting 초기화:
   ```bash
   firebase init hosting
   ```
   - "Use an existing project"를 선택하고 방금 만든 Firebase 프로젝트를 선택합니다.
   - public directory는 현재 폴더(`.`)로 지정합니다.
   - "Configure as a single-page app"는 `No`로 선택합니다.
   - 기존 `index.html`을 덮어쓸지 물으면 `No`를 선택해서 지금 파일을 유지합니다.
5. 배포:
   ```bash
   firebase deploy
   ```
6. 완료 후 터미널에 출력되는 Hosting URL(`https://<project-id>.web.app`)로 접속해 확인합니다.

## 3. Firestore 연동 카운터 설정하기

`index.html`의 +/- 카운터는 Firestore의 `counter/main` 문서(`value` 필드, number 타입)와 연동되어 있습니다.
새로고침해도, 다른 브라우저에서 열어도 같은 값을 보게 됩니다. 아직은 `firebaseConfig`가 비어 있어서(`YOUR_API_KEY` 등) 바로 동작하지 않습니다.

1. Firebase 콘솔 → 해당 프로젝트 → **프로젝트 설정(⚙️) → 일반** 탭으로 이동합니다.
2. 하단 "내 앱"에 웹 앱이 없다면 `</>` 아이콘을 눌러 웹 앱을 하나 등록합니다(앱 닉네임만 입력하면 됨, Hosting 설정은 건너뛰어도 됩니다).
3. 표시되는 `firebaseConfig` 객체(`apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId`) 값을 복사합니다.
4. `index.html` 안의 `const firebaseConfig = { ... }` 부분을 복사한 값으로 교체합니다.
5. Firestore에 `counter` 컬렉션이 없다면 코드가 자동으로 `counter/main` 문서를 `value: 0`으로 만듭니다. 미리 만들어둬도 됩니다.
6. Firestore 콘솔의 **규칙(Rules)** 탭에서 테스트 모드 규칙(또는 아래처럼 `counter` 컬렉션만 열어주는 규칙)이 적용되어 있는지 확인합니다:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /counter/{docId} {
         allow read, write: if true;
       }
     }
   }
   ```
   (연습용 공개 규칙입니다. 실제 서비스에서는 이렇게 쓰면 안 됩니다.)
7. `firebaseConfig`를 수정한 뒤 다시 배포(GitHub Pages는 push, Firebase Hosting은 `firebase deploy`)하고 버튼을 눌러 값이 Firestore 콘솔에서도 바뀌는지 확인합니다.

## 4. 배포 확인 체크리스트
- [ ] 페이지 제목과 "배포 테스트 v1" 배지가 보이는가
- [ ] 시계가 1초마다 갱신되는가
- [ ] 상태 문구가 "Firestore 연결됨 ✅"으로 바뀌는가
- [ ] `+` / `-` 버튼을 누르면 숫자가 바뀌고, Firestore 콘솔의 `counter/main` 문서 값도 같이 바뀌는가
- [ ] 새로고침하거나 다른 브라우저로 열어도 같은 숫자가 보이는가

GitHub Pages와 Firebase Hosting 양쪽에 모두 배포해 두고 두 URL을 비교해 보면 배포 흐름의 차이를 이해하기 좋습니다.
