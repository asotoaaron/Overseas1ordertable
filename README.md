# ☕ 팀 커피 주문 웹앱

Coway 팀원들을 위한 구로 G타워 인근 카페 커피 주문 도구입니다. 정적 HTML 한 파일로 동작하며 GitHub Pages로 바로 배포 가능합니다.

## 기능

### 일반 사용자 (누구나)
- **0단계 날짜 선택** → 주문자 → 카페 → 메뉴 → HOT/ICE → 추가
- 주문 텍스트 **클립보드 복사** · `.txt` **파일 저장**
- **📅 달력 / 이력 탭**: 월별 캘린더에서 날짜별 주문 건수 · 금액 한눈에 보기
- 카페별 자동 집계 + 전체 합계

### 🔒 관리자 모드 (비밀번호 `0000`)
우측 상단 **[🔒 관리자 모드]** 박스 클릭 → 비밀번호 입력하면 활성화. 노란색 배너 + 로그아웃 버튼 표시됨. localStorage 저장돼 다음 방문 시에도 유지.

- **팀원 관리**: 추가 / 삭제(✕) / 이름 변경(칩 더블클릭)
- **메뉴 관리**: 카페별 메뉴 추가(＋), 편집(✏️ — 이름·가격 수정), 삭제(✕). 기본 메뉴도 편집·삭제 가능
- **주문 기록 관리**: 개별 주문 ✕, 날짜 일괄 삭제, 전체 초기화
- **전체 기간 CSV 내보내기**

> ⚠️ 정적 HTML이라 진짜 인증이 아니며 브라우저 콘솔로 우회 가능합니다. 사내 신뢰 환경의 **실수 방지** 용도로만 사용하세요.

### 기타
- 브라우저 **localStorage** 자동 저장 (새로고침·재방문해도 유지)
- **기타** 카페는 카페명·메뉴·가격 수기 입력
- 모바일 반응형

### 관리자 비밀번호 변경
`index.html`에서 다음 줄을 찾아 원하는 값으로 수정:
```javascript
const ADMIN_PASSWORD = '0000';
```

## 등록된 카페

| 카페 | 위치 |
|---|---|
| 폴 바셋 구로 G타워점 | G타워 |
| 책다방 with 카페꼼마 | 넷마블 사옥 B1 |
| 크크다방 | 넷마블 사옥 3F |
| 우드스톤 구로 | 디지털로26길 |
| 디저트39 지타워몰점 | 지타워몰 |
| 카페아이엠티 지타워몰점 | 지타워몰 |
| 기타 (직접 입력) | - |

> 가격은 인터넷에서 수집한 참고 가격입니다. 시즌·매장에 따라 변동될 수 있으니 결제 시 매장 가격을 확인하세요.

## 등록된 팀원

최윤석, 이존수, 이승헌, 이수호, 김선태, 김동진, 엄지수, 바마유나

---

## ⚡ Firebase 셋업 (팀 공유의 핵심)

이 앱은 **Firebase Firestore**를 통해 팀원 간 주문/메뉴/팀원 데이터를 실시간 동기화합니다. 설정 안 하면 본인 브라우저에만 저장됩니다 (상단 노란 배너로 표시).

### 1단계 — Firebase 프로젝트 생성

1. 구글 계정으로 https://console.firebase.google.com 접속
2. **프로젝트 추가** → 이름(예: `coffee-order`) 입력 → Google Analytics는 **사용 안 함** 선택 → 만들기
3. 프로젝트 생성 완료 후 좌측 메뉴에서 **Firestore Database** 클릭
4. **데이터베이스 만들기** → 위치는 `asia-northeast3 (서울)` → **테스트 모드로 시작** → 사용 설정

### 2단계 — 웹 앱 등록

1. 프로젝트 홈에서 **</> (웹) 아이콘** 클릭
2. 앱 닉네임 입력 (예: `coffee-web`) → **앱 등록** (호스팅 체크는 해제)
3. **firebaseConfig** 객체가 표시됨 — 다음과 같은 형태:
   ```javascript
   const firebaseConfig = {
     apiKey: "AIzaSy....",
     authDomain: "coffee-order-xxxx.firebaseapp.com",
     projectId: "coffee-order-xxxx",
     storageBucket: "coffee-order-xxxx.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abc123..."
   };
   ```
4. 위 값들을 `firebase-config.js` 파일의 `window.firebaseConfig` 객체에 그대로 복사

### 3단계 — Firestore 보안 규칙 (사내용 간이 설정)

Firestore Database → **규칙** 탭 → 아래 내용으로 교체 → **게시**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      // 사내용 임시 설정: 누구나 읽기/쓰기 (URL을 모르는 외부인은 접근 불가)
      // 운영 시작 후 30일 지나면 기본적으로 차단되므로 아래 한 줄로 영구 허용
      allow read, write: if true;
    }
  }
}
```

> ⚠️ **보안 주의**: URL이 공개되면 누구나 읽고 쓸 수 있습니다. 사내 공유 URL이라면 큰 문제 없지만, 외부 노출이 우려되면 Firebase Auth 추가를 고려하세요.

### 4단계 — 배포 / 동작 확인

- `firebase-config.js` 값을 채워넣고 `index.html`을 열면 상단에 **✅ 팀 공유 모드** 초록 배너가 표시됩니다
- 빨간 배너가 뜨면 콘솔(F12)에서 오류 확인 — 보통 보안 규칙 미설정이거나 config 오타입니다
- 다른 PC/브라우저에서 같은 URL 접속 시 같은 데이터를 보게 됩니다 (실시간 동기화)

### 무료 한도

Firebase 무료 플랜(Spark):
- Firestore 일일 50,000 read / 20,000 write
- 8명 팀 + 일 수십 건 주문이면 한도의 1% 미만으로 사용

---

## GitHub Pages 배포 방법

### 방법 A — 명령줄 (Git 사용)

```powershell
cd C:\Users\20018202\Downloads\coffee-order-app
# (먼저 firebase-config.js에 Firebase 설정값 채워넣기)
git init
git add .
git commit -m "Initial commit: 팀 커피 주문 웹앱"
git branch -M main

# GitHub에서 미리 새 빈 레포지토리 만들고 (예: coffee-order-app)
git remote add origin https://github.com/<본인계정>/coffee-order-app.git
git push -u origin main
```

> 📌 `firebase-config.js`는 .gitignore에 넣지 마세요 — GitHub Pages에서도 이 파일을 로드해야 동작합니다. API 키가 공개되지만, Firestore 보안 규칙으로 통제됩니다.

푸시 후 GitHub 웹에서:
1. 레포 → **Settings** → **Pages**
2. **Source**: `Deploy from a branch`
3. **Branch**: `main` / `/ (root)` 선택 → **Save**
4. 1–2분 후 `https://<본인계정>.github.io/coffee-order-app/` 접속

### 방법 B — 웹에서 업로드 (Git 없이)

1. GitHub에서 새 레포 생성 (Public, `coffee-order-app`)
2. **Add file → Upload files** 로 `index.html`, `README.md` 업로드 후 Commit
3. **Settings → Pages** 에서 위와 동일하게 `main` 브랜치 활성화

### 사내망에서 외부 GitHub Pages가 차단되는 경우

- **사내 GitHub Enterprise** 가 있다면 동일한 절차로 Enterprise Pages에 배포
- 또는 `index.html` 파일 자체를 사내 메신저로 공유 → 각자 더블클릭으로 실행 (오프라인 동작 가능)

## 메뉴 수정하기

`index.html` 상단의 `CAFES` 객체에서 메뉴·가격·HOT/ICE 옵션을 자유롭게 수정하세요.

```javascript
paulbassett: {
  name: '폴 바셋 구로 G타워점',
  menus: [
    { name: '아메리카노', price: 4700, temp: true },   // HOT/ICE 모두 가능
    { name: '콜드브루', price: 4900, iceOnly: true },  // ICE 전용
    { name: '룽고', price: 4900, temp: false },        // 온도 옵션 없음
    ...
  ],
}
```

팀원 추가/변경은 `MEMBERS` 배열을 수정하세요.
