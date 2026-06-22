# 모바일 청첩장 작업 노트

마지막 정리일: 2026-06-21

## 프로젝트 개요

- 프로젝트 위치: `/Users/im_1531/Desktop/Claude_Wedding`
- 배포 방식: GitHub Pages
- 배포 URL: `https://topgonyisback.github.io/wedding-card-v2/`
- GitHub 저장소: `https://github.com/topgonyisback/wedding-card-v2`
- 메인 파일: `index.html`
- 주요 이미지 폴더: `gallery/`, `gallery/optimized/`

## 현재 운영 구조

- 정적 HTML 기반 모바일 청첩장이다.
- GitHub Pages에 `main` 브랜치를 푸시하면 배포된다.
- 방명록은 Firebase Cloud Firestore의 `guestbook` 컬렉션을 사용한다.
- 대부분의 시각 요소와 인터랙션은 `index.html` 안의 CSS/JS에 포함되어 있다.

## 주요 섹션

- 인트로 히어로
  - Cordially 데모를 참고한 이미지 모임 애니메이션 구조.
  - 현재 상태가 안정적이므로 불필요한 구조 변경 금지.
  - 모바일/리사이즈 대응이 민감한 영역이다.

- 초대합니다
  - GSAP ScrollTrigger로 3개의 문장이 순차 노출된다.
  - 강조 단어:
    - 사랑의 의미
    - 선명하게
    - 불 / 물
    - 증인
    - 체크인

- 두 사람을 소개합니다
  - 신랑/신부 카드에 영상 사용.
  - `groom.mp4`, `bride.mp4`는 초기 로딩을 막기 위해 lazy load 처리되어 있다.

- Our Story
  - 일반 가로 스크롤 캐러셀.
  - GSAP pin/scroll hijacking은 오류가 많아 제거한 상태.

- Our Moments
  - 일반 가로 스크롤 캐러셀.
  - 팝업 라이트박스 지원.
  - 팝업 전환 지연을 줄이기 위해 최적화 WebP를 사용하도록 변경했다.

- Guestbook
  - Firebase Firestore 사용.
  - Firebase SDK는 초기 로딩하지 않고, 방명록 근처 진입 또는 메시지 작성 시점에 동적 로드한다.

- Outro
  - `gallery/optimized/outro-hero.webp` 사용.
  - 문구: `더할 나위 없이 멋진 제2의 시작을 축하해주세요`

## Firebase 설정

현재 Firebase 프로젝트:

- Project ID: `topgony-wedding`
- Collection: `guestbook`
- 사용 필드:
  - `name`: string
  - `message`: string
  - `createdAt`: ISO string

프론트에서는 다음 형태로 저장한다.

```js
{
  name,
  message,
  createdAt: new Date().toISOString()
}
```

주의:

- Firestore 보안 규칙에서 `createdAt`을 string으로 허용해야 한다.
- 예전에 `serverTimestamp()`를 쓰다가 `Missing or insufficient permissions` 오류가 발생했기 때문에 현재는 ISO string으로 맞춰두었다.

## 공유 썸네일 / OG 태그

링크 공유 미리보기는 Open Graph 메타태그로 관리한다.

현재 사용 이미지:

- `gallery/og-thumbnail.webp`
- 배포 URL 기준: `https://topgonyisback.github.io/wedding-card-v2/gallery/og-thumbnail.webp`

주의:

- 카카오톡은 OG 캐시가 남을 수 있다.
- 썸네일이 바로 바뀌지 않으면 카카오 공유 디버거에서 URL 캐시 초기화가 필요하다.
- WebP가 일부 공유 앱에서 불안정하면 JPG 버전을 추가하는 것이 좋다.

## 성능 최적화 진행 상황

완료한 작업:

- Firebase SDK 초기 로딩 제거.
- Firebase SDK를 방명록 섹션 진입 또는 작성 시점에만 동적 로드.
- 사용하지 않는 카카오맵 SDK 초기 로딩 제거.
- 사용하지 않는 카카오 공유 SDK 초기 로딩 제거.
- 신랑/신부 영상 lazy load 적용.
- Moments 라이트박스 이미지 전환 지연 개선.

남은 후보:

- 히어로 사이드 이미지 `loading="lazy"` 검토.
  - 다만 히어로는 매우 민감하므로 구조 변경 금지.
  - 하려면 사이드 이미지에 단순 lazy만 추가하는 정도가 안전하다.
- 하단 갤러리 이미지의 원본 참조를 WebP로 더 정리.
- CSS/JS 파일 분리 및 minify.
- 이미지 캐시 정책 개선.
  - GitHub Pages에서는 서버 캐시 헤더 제어가 제한적이다.
  - 커스텀 호스팅으로 옮기면 개선 가능.

## 최근 주요 커밋

- `cf8daf6` Lazy load couple videos
- `764032d` Defer guestbook SDK loading
- `c174cdf` Adjust intro side photo crop
- `dc21e42` Fix guestbook timestamp format
- `afbfdb4` Speed up moments lightbox
- `36255d0` Adjust greeting and outro text
- `aebf8d5` Add social sharing metadata
- `e3402e2` Tune outro and moments scrolling

## 작업 시 주의사항

- 인트로 히어로는 안정화된 상태이므로 구조, 위치 계산, GSAP 진행률 계산을 함부로 바꾸지 않는다.
- Our Story / Our Moments에는 GSAP pin을 다시 넣지 않는다.
- 모바일에서 세로 스크롤을 가로 스크롤로 강제 변환하는 wheel/touch hijacking은 피한다.
- 이미지 원본 파일은 큰 것이 많으므로 실제 화면에는 `gallery/optimized/`의 WebP를 우선 사용한다.
- `gallery/Wedding_Snapshots5.webp`는 현재 미추적 파일로 남아 있을 수 있다. 필요한 파일인지 확인 후 처리한다.

## 배포 방법

수정 후 배포는 보통 다음 순서로 진행한다.

```bash
git add index.html
git commit -m "변경 내용 요약"
git push origin main
```

배포 확인:

- `https://topgonyisback.github.io/wedding-card-v2/`
- GitHub Pages 반영에는 약간의 지연이 있을 수 있다.
