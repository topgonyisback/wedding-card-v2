# 배포 가이드

마지막 정리일: 2026-06-21

## 배포 정보

- 배포 방식: GitHub Pages
- 저장소: `https://github.com/topgonyisback/wedding-card-v2`
- 배포 브랜치: `main`
- 배포 URL: `https://topgonyisback.github.io/wedding-card-v2/`
- 로컬 프로젝트: `/Users/im_1531/Desktop/Claude_Wedding`

## 기본 배포 흐름

수정이 끝나면 다음 순서로 배포한다.

```bash
git status --short
git add index.html
git commit -m "변경 내용 요약"
git push origin main
```

이미지나 새 파일을 추가했다면 해당 파일도 같이 add한다.

예:

```bash
git add index.html gallery/og-thumbnail.webp
git commit -m "Add social sharing metadata"
git push origin main
```

## 배포 확인

푸시 후 아래 주소에서 확인한다.

```text
https://topgonyisback.github.io/wedding-card-v2/
```

GitHub Pages는 반영까지 약간 시간이 걸릴 수 있다.

보통:

- 빠르면 30초 이내
- 느리면 1~3분 정도

## 캐시 확인

변경이 바로 안 보이면 다음을 시도한다.

- 브라우저 새로고침
- 모바일 브라우저에서 탭 닫고 다시 열기
- URL 뒤에 임시 쿼리 추가

예:

```text
https://topgonyisback.github.io/wedding-card-v2/?v=check1
```

## 공유 썸네일 캐시

카카오톡, 문자, SNS 공유 미리보기는 페이지 캐시와 별개로 OG 캐시가 남을 수 있다.

현재 OG 이미지:

```text
https://topgonyisback.github.io/wedding-card-v2/gallery/og-thumbnail.webp
```

카카오톡 미리보기가 안 바뀌면 카카오 공유 디버거에서 URL 캐시 초기화가 필요하다.

## 작업 전 확인

작업 전에는 현재 상태를 먼저 확인한다.

```bash
git status --short
```

주의:

- 사용자가 만든 미추적 파일이 있을 수 있다.
- 현재 예시로 `gallery/Wedding_Snapshots5.webp`가 미추적 상태로 남아 있을 수 있다.
- 작업과 무관한 파일은 임의로 add하지 않는다.

## 자주 쓰는 명령

최근 커밋 확인:

```bash
git log --oneline -10
```

원격 저장소 확인:

```bash
git remote -v
```

변경 내용 확인:

```bash
git diff
```

스테이징된 변경 확인:

```bash
git diff --cached
```

## 로컬 서버

정적 확인용으로 로컬 서버를 띄울 수 있다.

```bash
python3 -m http.server 4186
```

확인 URL:

```text
http://localhost:4186/
```

다른 포트를 사용해도 된다.

## 배포 시 주의사항

- 히어로 영역은 안정화된 상태라 성능 개선 중에도 구조를 함부로 바꾸지 않는다.
- Firebase 방명록은 Firestore 규칙과 프론트 payload 구조가 맞아야 한다.
- 이미지 추가 시 가능하면 `gallery/optimized/`의 WebP를 사용한다.
- 원본 JPG/PNG는 용량이 큰 경우가 많아 초기 화면에 직접 사용하지 않는다.
