# Firebase 방명록 설정

마지막 정리일: 2026-06-21

## 사용 목적

모바일 청첩장의 축하 메시지 방명록을 저장하고 불러오기 위해 Firebase Cloud Firestore를 사용한다.

## Firebase 프로젝트

- Project ID: `topgony-wedding`
- App ID: `1:715353561275:web:8126b1bf73f2d480a2f19e`
- Firestore 컬렉션: `guestbook`

## 프론트 설정 위치

Firebase 설정은 `index.html` 안의 `CONFIG.firebase`에 있다.

```js
const CONFIG = {
  firebase: {
    apiKey: "...",
    authDomain: "topgony-wedding.firebaseapp.com",
    projectId: "topgony-wedding",
    storageBucket: "topgony-wedding.firebasestorage.app",
    messagingSenderId: "715353561275",
    appId: "1:715353561275:web:8126b1bf73f2d480a2f19e",
    measurementId: "G-6NFQYLT4T0"
  }
};
```

## SDK 로딩 방식

성능 개선을 위해 Firebase SDK는 초기 로딩하지 않는다.

현재 동작:

- 페이지 첫 로딩 시 Firebase SDK를 받지 않는다.
- 방명록 섹션 근처에 진입하면 SDK를 동적으로 로드한다.
- 사용자가 축하 메시지를 작성하면 SDK를 동적으로 로드하고 Firestore에 저장한다.

동적 로드 대상:

```text
https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js
https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js
```

## 저장 데이터 구조

컬렉션: `guestbook`

문서 필드:

```js
{
  name: string,
  message: string,
  createdAt: string
}
```

`createdAt`은 ISO string으로 저장한다.

예:

```js
createdAt: new Date().toISOString()
```

주의:

- 예전에 `firebase.firestore.FieldValue.serverTimestamp()`를 사용했을 때 Firestore 보안 규칙과 맞지 않아 `Missing or insufficient permissions` 오류가 발생했다.
- 현재는 보안 규칙과 맞추기 위해 `createdAt`을 string으로 저장한다.

## 권장 Firestore Rules

현재 프론트 구조에 맞는 규칙은 `firestore.rules`에도 같은 내용으로 저장해두었다.
Firebase Console의 Firestore `규칙` 탭에 붙여 넣고 `게시`하면 실제 서버 제한이 적용된다.
Firebase CLI를 사용할 경우 `firebase deploy --only firestore:rules --project topgony-wedding`로 배포할 수 있다.

```js
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /guestbook/{docId} {
      allow read: if true;

      allow create: if
        request.resource.data.keys().hasOnly(['name', 'message', 'createdAt']) &&
        request.resource.data.name is string &&
        request.resource.data.message is string &&
        request.resource.data.createdAt is string &&
        request.resource.data.name.size() >= 1 &&
        request.resource.data.name.size() <= 20 &&
        request.resource.data.message.size() >= 1 &&
        request.resource.data.message.size() <= 300;

      allow update, delete: if false;
    }
  }
}
```

## 데이터 읽기

프론트에서는 최신 메시지 순으로 가져온다.

```js
db.collection('guestbook')
  .orderBy('createdAt', 'desc')
  .limit(200)
```

`createdAt`이 ISO string이면 문자열 정렬로도 날짜 순서가 유지된다.

## 오류 대응

### Missing or insufficient permissions

가능한 원인:

- Firestore Rules가 프론트 payload와 맞지 않음
- `createdAt` 타입이 규칙과 다름
- `name` 또는 `message` 길이가 규칙 범위를 벗어남
- 컬렉션 이름이 다름

현재 프론트는 다음 조건을 만족해야 한다.

- `name`: 1~30자
- `message`: 1~500자
- `createdAt`: string
- 추가 필드 없음

### 방명록이 안 보이는 경우

확인할 것:

- Firebase 프로젝트 ID가 맞는지
- Firestore 컬렉션 이름이 `guestbook`인지
- Rules에서 `allow read: if true;`인지
- 배포된 페이지가 최신인지

## 운영 주의사항

- 클라이언트에서 누구나 create 가능하므로 스팸 가능성이 있다.
- 필요하면 나중에 다음 기능을 추가한다.
  - 관리자 삭제 기능
  - 금칙어 필터
  - 간단한 rate limit
  - reCAPTCHA 또는 App Check

현재 결혼식 청첩장 용도에서는 위 규칙 정도면 우선 운영 가능하다.
