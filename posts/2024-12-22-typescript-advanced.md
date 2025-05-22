---
title: "TypeScript 고급 타입 시스템 마스터하기"
date: "2024-12-22"
tags: ["TypeScript", "고급", "타입", "제네릭", "유틸리티타입"]
published: true
---

# TypeScript 고급 타입 시스템 마스터하기

TypeScript의 고급 타입 시스템을 활용하면 더 안전하고 표현력 있는 코드를 작성할 수 있습니다.

## 제네릭 (Generics)

### 기본 제네릭
```typescript
function identity<T>(arg: T): T {
  return arg;
}

const result = identity<string>("hello");
```

### 제네릭 제약 조건
```typescript
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```

## 유틸리티 타입들

### Partial<T>
모든 속성을 선택적으로 만듭니다.

```typescript
interface User {
  name: string;
  email: string;
  age: number;
}

type PartialUser = Partial<User>; // 모든 속성이 선택적
```

### Pick<T, K>
특정 속성만 선택합니다.

```typescript
type UserNameAndEmail = Pick<User, 'name' | 'email'>;
```

### Omit<T, K>
특정 속성을 제외합니다.

```typescript
type UserWithoutAge = Omit<User, 'age'>;
```

## 조건부 타입

```typescript
type ApiResponse<T> = T extends string
  ? { message: T }
  : { data: T };

type StringResponse = ApiResponse<string>; // { message: string }
type DataResponse = ApiResponse<number>; // { data: number }
```

## 매핑된 타입

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};
```

## 템플릿 리터럴 타입

```typescript
type EventName<T extends string> = `on${Capitalize<T>}`;

type ClickEvent = EventName<"click">; // "onClick"
type HoverEvent = EventName<"hover">; // "onHover"
```

## 실제 사용 예시

### API 응답 타입 정의
```typescript
interface ApiResponse<T = any> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
  email: string;
}

type UserResponse = ApiResponse<User>;
type UsersResponse = ApiResponse<User[]>;
```

### 폼 상태 관리
```typescript
type FormState<T> = {
  [K in keyof T]: {
    value: T[K];
    error?: string;
    touched: boolean;
  };
};

interface LoginForm {
  email: string;
  password: string;
}

type LoginFormState = FormState<LoginForm>;
```

## 고급 패턴

### 함수 오버로딩
```typescript
function createElement(tag: "div"): HTMLDivElement;
function createElement(tag: "span"): HTMLSpanElement;
function createElement(tag: string): HTMLElement;
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}
```

### 브랜드 타입
```typescript
type UserId = number & { readonly brand: unique symbol };
type ProductId = number & { readonly brand: unique symbol };

function getUserById(id: UserId): User { ... }
function getProductById(id: ProductId): Product { ... }
```

## 마무리

TypeScript의 고급 타입 시스템을 마스터하면 런타임 에러를 컴파일 타임에 잡아낼 수 있고, 더 표현력 있는 API를 설계할 수 있습니다.

타입 시스템을 활용해서 더 안전한 코드를 작성해보세요!
