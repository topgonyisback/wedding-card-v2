# 모바일 성능 개선 계획

이 문서는 모바일 청첩장 성능 개선을 위한 외부 공유용 템플릿이다.
실제 파일명과 저장소명은 예시값으로 치환했다.

## 목표

Lighthouse 모바일 성능 점수를 개선하되, 히어로 인터랙션과 주요 디자인은 깨지지 않도록 한다.

우선순위:

1. 초기 로딩 리소스 줄이기
2. 모바일 이미지 전달 최적화
3. JavaScript 실행 비용 줄이기
4. 캐시 정책과 배포 구조 정리

## 현재 발견된 주요 이슈

Lighthouse에서 자주 보이는 항목:

- Use efficient cache lifetimes
- Improve image delivery
- Render-blocking requests
- Reduce unused JavaScript
- Minimize main-thread work
- Avoid enormous network payloads
- Reduce JavaScript execution time
- Avoid long main-thread tasks

## 완료 또는 권장된 개선

### 1. Firebase SDK 지연 로딩

Firebase SDK는 페이지 첫 로딩 시 바로 받지 않고, 방명록 섹션 근처에 진입하거나 메시지 작성 시점에 로드한다.

기대 효과:

- 초기 JS 다운로드 감소
- 초기 main-thread 작업 감소
- 사용자가 방명록까지 가지 않으면 Firebase SDK를 받지 않음

### 2. 지도/공유 SDK 지연 로딩

지도, 공유 SDK는 해당 기능 버튼을 누를 때만 로드한다.

예시:

- 지도 팝업을 열 때 지도 SDK 로드
- 공유 버튼을 누를 때 공유 SDK 로드

### 3. 영상 lazy loading

신랑/신부 소개 영상은 처음부터 `src`를 넣지 않고 `data-src`로 둔 뒤, 해당 섹션 근처에서 실제 로드한다.

```html
<video data-src="assets/groom.mp4" preload="none" autoplay muted loop playsinline></video>
```

### 4. 갤러리 팝업 최적화

이미지 팝업에서 좌우 이동 시 두 번 로딩되는 느낌이 생길 수 있다.

개선 방향:

- 같은 이미지에 중복으로 `src`를 다시 넣지 않는다.
- 다음/이전 이미지만 미리 preload한다.
- 팝업 전환은 opacity/transform 위주로 처리한다.

## 다음 개선 우선순위

### Priority 1. 이미지 전달 최적화

가장 효과가 큰 영역이다.

할 일:

- 원본 JPEG/PNG를 WebP로 변환
- 실제 화면 크기보다 과도하게 큰 이미지는 별도 축소본 생성
- 모바일에서는 섹션 아래쪽 이미지를 lazy loading
- 히어로 이미지는 eager 유지

예시 구조:

```text
assets/
assets/optimized/
assets/optimized/hero-main.webp
assets/optimized/gallery-01.webp
assets/optimized/gallery-01-mobile.webp
```

주의:

- Intro Hero는 모바일/PC 레이아웃이 민감하므로 이미지 교체 후 반드시 확인한다.
- PC에서는 필요 이상으로 lazy 처리하지 않아도 된다.

### Priority 2. JavaScript 실행 비용 줄이기

현재 모든 기능이 한 파일에 몰려 있으면 초기 파싱/실행 비용이 커질 수 있다.

가능한 방향:

- 방명록, 지도, 갤러리 팝업, 계좌 팝업 로직을 기능별로 분리
- 초기 실행 함수 최소화
- 섹션 진입 시 필요한 기능만 초기화

주의:

- 히어로 스크롤 로직은 안정성이 중요하므로 마지막에 건드린다.

### Priority 3. CSS/JS minify

GitHub Pages에 올릴 최종 파일은 minify하면 전송량을 줄일 수 있다.

가능한 방향:

- 빌드 도구 도입
- 또는 단순 minify 스크립트 사용

주의:

- 디버깅이 어려워질 수 있으므로 원본 파일과 배포 파일을 나누는 방식을 권장한다.

### Priority 4. 캐시 정책

GitHub Pages는 세밀한 Cache-Control 설정이 어렵다.

대안:

- 파일명에 버전/hash 포함
- 이미지 교체 시 파일명 변경
- OG 이미지는 캐시가 오래 남을 수 있으므로 새 파일명 사용

## 안정성 체크리스트

수정 후 반드시 확인할 영역:

- Intro Hero: PC 넓은 화면, 중간 화면, 모바일
- Greeting: 3장 문장 전환 타이밍
- Our Story: 모바일/PC 가로 스크롤
- Our Moments: 모바일/PC 가로 스크롤 및 팝업
- Guestbook: Firestore 저장/조회
- Account: 계좌 복사 토스트
- Outro: 텍스트 위치와 OG 이미지

## 건드릴 때 조심해야 하는 영역

- Intro Hero
- Greeting GSAP ScrollTrigger
- 모바일 가로 스크롤 섹션
- 팝업 공통 닫기 버튼
- Firebase guestbook payload

## 성능 개선 예상

보통 가장 큰 개선은 이미지 최적화와 SDK 지연 로딩에서 나온다.

예상:

- 이미지 전달 최적화: 가장 큰 폭의 개선 가능
- SDK 지연 로딩: 초기 JS 비용 감소
- JS 분리/minify: 중간 정도 개선
- 캐시 정책: 반복 방문 성능 개선

실제 점수는 네트워크 상태, 기기 성능, Lighthouse 실행 환경에 따라 달라진다.
