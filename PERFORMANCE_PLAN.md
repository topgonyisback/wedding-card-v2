# 성능 개선 계획

마지막 정리일: 2026-06-21

## 현재 상황

Lighthouse 모바일 Performance 점수가 낮게 나오는 상태다.

최근 확인된 주요 경고:

- Use efficient cache lifetimes
- Forced reflow
- Improve image delivery
- Render-blocking requests
- Reduce unused JavaScript
- Avoid enormous network payloads
- Reduce JavaScript execution time
- Minimize main-thread work

## 이미 완료한 개선

### 1. 외부 SDK 초기 로딩 제거

초기 `<head>`에서 다음 SDK를 제거했다.

- Firebase app compat
- Firebase firestore compat
- Kakao SDK
- Kakao Map SDK

현재 Firebase는 방명록 섹션 근처에 진입하거나 메시지 작성 시점에 동적으로 로드된다.

### 2. 신랑/신부 영상 lazy load

`groom.mp4`, `bride.mp4`는 초기 로딩하지 않는다.

현재 구조:

```html
<video data-src="gallery/groom.mp4" preload="none" autoplay muted loop playsinline></video>
```

IntersectionObserver로 섹션 근처에 왔을 때만 `src`를 넣는다.

### 3. Moments 라이트박스 최적화

`함께한 순간들` 팝업에서 큰 JPG 원본 대신 최적화 WebP를 사용하도록 변경했다.

전환 로직도 다음처럼 개선했다.

- 이미지 변경 전 긴 fade-out 제거
- 다음/이전 이미지 사전 로딩 추가
- 짧은 swap 애니메이션만 유지

## 우선순위별 남은 작업

### 1순위: 이미지 전달 방식 개선

Lighthouse의 `Improve image delivery`와 `Avoid enormous network payloads`에 직접 영향을 준다.

후보:

- 초기 화면 밖 이미지에 `loading="lazy"` 추가
- 갤러리 팝업용 `data-src`를 원본 JPG/PNG에서 WebP로 더 통일
- 모바일 전용 작은 이미지 생성 검토

주의:

- 인트로 히어로는 매우 민감하다.
- 히어로 구조, GSAP 계산, DOM 구조 변경은 피한다.
- 히어로에 적용한다면 사이드 이미지에 단순 `loading="lazy"` 추가 정도만 안전하다.

예상 개선:

- 단순 lazy 추가: +1~3점
- 이미지 크기/포맷 전반 정리: +3~8점 가능

### 2순위: JS 실행 비용 줄이기

현재 `index.html` 안에 모든 JS가 들어 있다.

후보:

- 초기 화면에 필요 없는 기능을 섹션 진입 시 초기화
- 라이트박스/방명록/계좌 팝업 등은 필요한 시점까지 이벤트 초기화 지연
- GSAP 사용 구간 최소화

주의:

- 초대합니다 섹션은 GSAP ScrollTrigger를 사용 중이다.
- Our Story / Our Moments에는 GSAP pin을 다시 넣지 않는다.

### 3순위: CSS/JS 분리 및 압축

현재는 단일 HTML 파일 구조라 편하지만, 브라우저 캐시 효율과 파싱 비용 면에서는 불리하다.

후보:

- `style.css`
- `main.js`
- `vendor`는 그대로 유지
- 배포 전 minify

주의:

- 구조 변경 범위가 커진다.
- GitHub Pages에서는 정적 파일 캐시는 가능하지만 캐시 헤더를 세밀하게 제어하기 어렵다.

### 4순위: 캐시 정책

Lighthouse의 `Use efficient cache lifetimes`는 GitHub Pages 환경에서 제어가 제한적이다.

선택지:

- GitHub Pages 유지: 개선 폭 제한적
- Cloudflare Pages / Netlify / Vercel 같은 호스팅으로 이전: 캐시 헤더 제어 가능

## 위험 구간

다음 영역은 성능 개선 중에도 조심해야 한다.

- 인트로 히어로
  - 이미지 위치, 크롭, 리사이즈, GSAP 진행률 계산이 민감하다.

- 초대합니다
  - GSAP pin 동작과 문장 전환 타이밍이 민감하다.

- Our Story / Our Moments
  - 이전에 GSAP pin과 scroll hijacking으로 모바일 스크롤 문제가 있었다.
  - 현재는 일반 가로 스크롤로 안정화되어 있다.

## 다음 추천 작업

1. 하단 갤러리 `data-src` 원본 참조 정리
2. 초기 화면 밖 이미지에 `loading="lazy"` 누락분 보강
3. Lighthouse 재측정
4. 그래도 점수가 낮으면 CSS/JS 분리 검토
