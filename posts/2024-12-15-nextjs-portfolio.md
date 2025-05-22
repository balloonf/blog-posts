---
title: "Next.js와 TypeScript로 포트폴리오 만들기"
date: "2024-12-15"
tags: ["Next.js", "TypeScript", "포트폴리오", "웹개발"]
published: true
---

# Next.js와 TypeScript로 포트폴리오 만들기

이번 포스트에서는 Next.js 15와 TypeScript를 사용해서 개인 포트폴리오 사이트를 만드는 과정을 소개합니다.

## 주요 기술 스택

- **Next.js 15**: 최신 React 프레임워크
- **TypeScript**: 타입 안전성을 위한 필수 도구
- **Tailwind CSS**: 빠른 스타일링을 위한 유틸리티 CSS
- **shadcn/ui**: 재사용 가능한 컴포넌트 라이브러리

## 프로젝트 구조

```
portfolio/
├── src/
│   ├── app/
│   ├── components/
│   ├── lib/
│   └── types/
└── public/
```

## 핵심 기능들

### 1. 반응형 디자인
모든 기기에서 완벽하게 동작하는 반응형 레이아웃을 구현했습니다.

### 2. 다크모드 지원
next-themes를 사용해서 다크/라이트 모드를 지원합니다.

### 3. SEO 최적화
- 메타데이터 최적화
- 구조화된 데이터 (JSON-LD)
- 사이트맵 자동 생성

### 4. GitHub API 연동
블로그 포스트를 GitHub 레포지토리에서 자동으로 가져와 표시합니다.

## 배포

Vercel을 사용해서 간단하게 배포할 수 있습니다:

```bash
git push origin main
```

main 브랜치에 푸시하면 자동으로 배포됩니다.

## 마무리

Next.js 15의 새로운 기능들과 TypeScript의 타입 안전성 덕분에 안정적이고 확장 가능한 포트폴리오를 만들 수 있었습니다.

다음 포스트에서는 GitHub API 연동 방법에 대해 더 자세히 다뤄보겠습니다.
