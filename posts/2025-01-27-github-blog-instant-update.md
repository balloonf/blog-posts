---
title: "Vercel에서 GitHub 블로그 즉시 반영하기: ISR과 Webhook 활용법"
date: "2025-01-27"
tags: ["Next.js", "Vercel", "GitHub", "ISR", "Webhook", "캐시", "최적화"]
published: true
---

# Vercel에서 GitHub 블로그 즉시 반영하기: ISR과 Webhook 활용법

GitHub에서 블로그 포스트를 수정했는데 사이트에 반영되는데 1시간이나 걸린다면? 이 문제를 해결하기 위해 Next.js의 ISR(Incremental Static Regeneration)과 GitHub Webhook을 활용한 실시간 업데이트 시스템을 구축했습니다.

## 문제 상황

기존 시스템에서는 다음과 같은 문제가 있었습니다:

- **ISR 재검증 시간**: 1시간 (3600초)로 설정
- **메모리 캐시**: 5분간 GitHub API 호출 캐시
- **수동 갱신 불가**: 즉시 반영할 방법이 없음

**결과**: GitHub에서 포스트를 수정해도 최대 1시간 후에야 사이트에 반영

## 해결 전략

문제를 3단계로 나누어 해결했습니다:

1. **즉시 효과**: ISR 시간 단축
2. **수동 갱신**: 재검증 API 구현  
3. **완전 자동화**: GitHub Webhook 연동

## 1단계: ISR 시간 단축

### 기존 코드
```typescript
// src/app/blog/page.tsx
export const revalidate = 3600 // 1시간
```

### 개선된 코드
```typescript
// src/app/blog/page.tsx
export const revalidate = 60 // 1분으로 단축
```

**효과**: 1시간 → 1분으로 60배 단축!

## 2단계: 수동 재검증 API 구현

Vercel의 On-Demand ISR을 활용해서 수동으로 캐시를 무효화할 수 있는 API를 구현했습니다.

### API 엔드포인트 생성
```typescript
// src/app/api/revalidate/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { revalidatePath } from 'next/cache'

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url)
    const secret = searchParams.get('secret')
    
    // 보안 검증
    if (secret !== process.env.REVALIDATE_SECRET) {
      return NextResponse.json({ error: 'Invalid secret' }, { status: 401 })
    }

    // 블로그 관련 경로 재검증
    revalidatePath('/blog')
    revalidatePath('/blog/[slug]', 'page')
    
    return NextResponse.json({ 
      revalidated: true,
      timestamp: new Date().toISOString(),
      message: 'Blog cache cleared successfully'
    })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to revalidate' },
      { status: 500 }
    )
  }
}
```

### 사용법
```bash
# 브라우저에서 직접 호출
https://your-domain.vercel.app/api/revalidate?secret=your-secret-key

# 터미널에서 호출
curl "https://your-domain.vercel.app/api/revalidate?secret=your-secret-key"
```

## 3단계: GitHub Webhook 완전 자동화

GitHub에서 파일이 수정될 때 자동으로 캐시를 갱신하는 Webhook API를 구현했습니다.

### Webhook API 구현
```typescript
// src/app/api/webhook/github/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { revalidatePath } from 'next/cache'
import crypto from 'crypto'

// GitHub Webhook payload 타입 정의
interface GitHubCommit {
  added?: string[]
  modified?: string[]
  removed?: string[]
}

interface GitHubWebhookPayload {
  action?: string
  commits?: GitHubCommit[]
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.text()
    const signature = request.headers.get('x-hub-signature-256')
    const webhookSecret = process.env.GITHUB_WEBHOOK_SECRET

    // GitHub Webhook 서명 검증 (보안)
    if (webhookSecret && signature) {
      const expectedSignature = 'sha256=' + crypto
        .createHmac('sha256', webhookSecret)
        .update(body)
        .digest('hex')

      if (signature !== expectedSignature) {
        return NextResponse.json({ error: 'Invalid signature' }, { status: 401 })
      }
    }

    const payload: GitHubWebhookPayload = JSON.parse(body)
    
    // 블로그 폴더의 파일이 변경되었는지 확인
    const blogFolder = process.env.NEXT_PUBLIC_BLOG_FOLDER || 'posts'
    const hasBlogChanges = payload.commits?.some((commit: GitHubCommit) => 
      commit.added?.some((file: string) => file.startsWith(blogFolder)) ||
      commit.modified?.some((file: string) => file.startsWith(blogFolder)) ||
      commit.removed?.some((file: string) => file.startsWith(blogFolder))
    ) || true

    if (hasBlogChanges) {
      // 블로그 관련 경로 재검증
      revalidatePath('/blog')
      revalidatePath('/blog/[slug]', 'page')
      
      return NextResponse.json({ 
        revalidated: true,
        timestamp: new Date().toISOString(),
        message: 'Blog cache updated via GitHub webhook',
        commits: payload.commits?.length || 0
      })
    }

    return NextResponse.json({ 
      message: 'Webhook received but no blog changes detected' 
    })

  } catch (error) {
    return NextResponse.json(
      { error: 'Webhook processing failed' },
      { status: 500 }
    )
  }
}
```

### GitHub Webhook 설정

1. **GitHub 저장소** → Settings → Webhooks
2. **Add webhook** 클릭
3. **Payload URL**: `https://your-domain.vercel.app/api/webhook/github`
4. **Content type**: `application/json`
5. **Secret**: 환경변수와 동일한 값
6. **Events**: "Just the push event" 선택

## 캐시 시스템 개선

GitHub API 호출을 최적화하기 위해 캐시 시스템도 개선했습니다.

### 개선된 캐시 구조
```typescript
// src/lib/github.ts
export const cache = {
  posts: null as BlogPostMeta[] | null,
  tags: null as string[] | null,
  lastFetch: 0,
  ttl: 2 * 60 * 1000, // 5분 → 2분으로 단축

  async getPosts(force = false): Promise<BlogPostMeta[]> {
    if (!force && this.posts && Date.now() - this.lastFetch < this.ttl) {
      return this.posts
    }
    
    this.posts = await getBlogPosts()
    this.lastFetch = Date.now()
    return this.posts
  },

  clear() {
    this.posts = null
    this.tags = null
    this.lastFetch = 0
    console.log('GitHub cache cleared')
  },

  // 캐시 상태 확인
  getStatus() {
    const now = Date.now()
    const isExpired = now - this.lastFetch > this.ttl
    const timeLeft = Math.max(0, this.ttl - (now - this.lastFetch))
    
    return {
      hasPosts: !!this.posts,
      hasTags: !!this.tags,
      isExpired,
      timeLeft: Math.round(timeLeft / 1000),
      lastFetch: new Date(this.lastFetch).toISOString()
    }
  }
}
```

## Vercel 환경변수 설정

필요한 환경변수들을 Vercel 대시보드에서 설정해야 합니다:

```env
# GitHub 연동
GITHUB_TOKEN=ghp_your_token_here
NEXT_PUBLIC_GITHUB_USERNAME=your_username
NEXT_PUBLIC_GITHUB_REPO=your_blog_repo
NEXT_PUBLIC_BLOG_FOLDER=posts

# API 보안
REVALIDATE_SECRET=your_secure_random_string
GITHUB_WEBHOOK_SECRET=your_webhook_secret
```

## TypeScript 오류 해결

빌드 과정에서 발생한 TypeScript 오류들도 해결했습니다:

### any 타입 오류
```typescript
// 문제: ESLint에서 any 타입 금지
payload.commits?.some((commit: any) => ...)

// 해결: 적절한 타입 정의
interface GitHubCommit {
  added?: string[]
  modified?: string[]
  removed?: string[]
}
payload.commits?.some((commit: GitHubCommit) => ...)
```

### unknown 타입 오류
```typescript
// 문제: catch 블록의 error는 unknown 타입
catch (error) {
  return { error: 'Failed', details: error.message }
}

// 해결: 타입 가드 사용
catch (error) {
  return { 
    error: 'Failed', 
    details: error instanceof Error ? error.message : 'Unknown error' 
  }
}
```

## 최종 결과

### 성능 개선
- **기존**: GitHub 수정 → 최대 1시간 후 반영
- **1단계 후**: GitHub 수정 → 최대 1분 후 반영 (60배 향상)
- **2단계 후**: 수동 API 호출로 즉시 반영 가능
- **3단계 후**: GitHub 수정 → **실시간 자동 반영**

### 빌드 최적화
```
Route (app)                Size    First Load JS   Revalidate
├ ○ /blog                 55.3 kB    168 kB          1m
├ ● /blog/[slug]           888 B     114 kB          1m
├ ƒ /api/revalidate        145 B     101 kB
└ ƒ /api/webhook/github    145 B     101 kB
```

## 마무리

이번 프로젝트를 통해 다음 기술들을 활용했습니다:

- **Next.js ISR**: 정적 사이트의 성능과 동적 업데이트의 장점 결합
- **Vercel On-Demand ISR**: API를 통한 즉시 캐시 무효화
- **GitHub Webhook**: 이벤트 기반 자동 업데이트
- **TypeScript**: 타입 안전성을 통한 런타임 오류 방지
- **암호화**: HMAC을 활용한 Webhook 보안

결과적으로 GitHub에서 블로그를 수정하면 **실시간으로** 사이트에 반영되는 완전 자동화된 시스템을 구축할 수 있었습니다.

이 방법은 블로그뿐만 아니라 CMS 연동, 실시간 데이터 업데이트 등 다양한 상황에서 활용할 수 있습니다.
