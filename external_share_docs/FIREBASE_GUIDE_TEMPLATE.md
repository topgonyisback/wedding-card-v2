# Firebase 방명록 설정 가이드

이 문서는 모바일 청첩장의 축하 메시지 방명록을 Firebase Cloud Firestore로 연결하기 위한 외부 공유용 템플릿이다.
실제 Firebase 프로젝트 ID, App ID, API Key는 예시값으로 치환했다.

## 목적

방문자가 남긴 축하 메시지를 Firestore에 저장하고, 페이지에서 최신 메시지를 불러와 보여준다.

## Firebase 프로젝트 예시

- Project ID: `<FIREBASE_PROJECT_ID>`
- App ID: `<FIREBASE_APP_ID>`
- Collection: `guestbook`

## 프론트 설정 예시

Firebase 설정은 보통 메인 JS 또는 `CONFIG.firebase`에 둔다.

```js
const firebaseConfig = {
  apiKey: "<FIREBASE_API_KEY>",
  authDomain: "<FIREBASE_PROJECT_ID>.firebaseapp.com",
  projectId: "<FIREBASE_PROJECT_ID>",
  storageBucket: "<FIREBASE_PROJECT_ID>.firebasestorage.app",
  messagingSenderId: "<FIREBASE_MESSAGING_SENDER_ID>",
  appId: "<FIREBASE_APP_ID>",
  measurementId: "<FIREBASE_MEASUREMENT_ID>"
};
```

## SDK 로딩 방식

성능 개선을 위해 Firebase SDK는 초기 로딩하지 않고 필요할 때 동적으로 로드한다.

예시:

- 방명록 섹션 근처에 진입할 때
- 메시지 작성 버튼을 눌렀을 때

장점:

- 첫 화면 로딩이 가벼워진다.
- 방명록을 사용하지 않는 방문자는 Firebase SDK를 받지 않을 수 있다.

## 데이터 구조

Firestore 컬렉션:

```text
guestbook
```

문서 구조:

```json
{
  "name": "홍길동",
  "message": "두 분의 결혼을 축하합니다.",
  "createdAt": "2026-06-08T00:04:59.200Z"
}
```

필드:

- `name`: 작성자 이름
- `message`: 축하 메시지
- `createdAt`: ISO 문자열 날짜

## Firestore 보안 규칙

기본 권장 규칙:

```txt
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
        request.resource.data.name.size() <= 30 &&
        request.resource.data.message.size() >= 1 &&
        request.resource.data.message.size() <= 500;

      allow update, delete: if false;
    }
  }
}
```

의미:

- 누구나 읽을 수 있다.
- 누구나 새 메시지는 작성할 수 있다.
- 허용된 필드만 저장할 수 있다.
- 이름과 메시지 길이를 제한한다.
- 수정과 삭제는 막는다.

## serverTimestamp 대신 ISO 문자열을 쓰는 이유

비로그인 사용자가 메시지를 작성하는 구조에서는 Firestore 규칙에서 `serverTimestamp()` 검증이 번거로울 수 있다.

간단한 정적 사이트라면 프론트에서 ISO 문자열을 만들어 저장하는 방식이 관리하기 쉽다.

```js
createdAt: new Date().toISOString()
```

## 화면 표시 규칙

추천 방식:

- 최신순으로 정렬
- 기본 화면에는 최대 9개 표시
- 더보기 버튼으로 전체 화면 팝업 표시
- 작성 완료 후 토스트 표시
- 입력 필드 초기화
- 새 메시지는 부드럽게 나타나는 transition 적용

## 흔한 오류

### Missing or insufficient permissions

확인할 것:

- Firestore 규칙이 실제로 게시되었는지
- 컬렉션명이 `guestbook`인지
- 저장 payload가 규칙의 필드명과 일치하는지
- `createdAt`이 string인지
- `name`, `message` 길이가 규칙 범위 안인지

### 저장은 되는데 화면에 안 보임

확인할 것:

- Firestore read가 허용되어 있는지
- 정렬 기준 필드가 존재하는지
- 프론트에서 빈 배열 처리 로직이 맞는지
- 캐시된 이전 JS를 보고 있지 않은지

### 스팸 방지

공개 청첩장은 누구나 쓸 수 있으므로 스팸 가능성이 있다.

가벼운 대응:

- 이름/메시지 길이 제한
- 동일 브라우저에서 짧은 시간 내 반복 작성 제한
- 욕설 필터

강한 대응:

- Firebase App Check
- Cloud Functions 검증
- 관리자 삭제 기능

단순 청첩장에서는 처음에는 가벼운 대응부터 시작해도 충분하다.

## 운영 체크리스트

- Firebase 프로젝트 생성
- Web App 추가
- Firebase config 확인
- Cloud Firestore 생성
- 보안 규칙 게시
- 프론트 config 적용
- 로컬에서 작성 테스트
- GitHub Pages 배포 후 실제 URL에서 작성 테스트
- 모바일 브라우저에서 작성 테스트
