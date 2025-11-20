
# 커밋마켓(온라인 쇼핑몰) 플랫폼

## ✏️ 과제의 개요

| 카테고리 | 난이도 | 제한시간 |
|----------|--------|----------|
| frontend | normal | 7d |

### 💻 과제에서 요구하는 개발언어

- react
- nextjs
- javascript
- typescript


## 📜 과제의 내용

> 과제 설명과 요구사항을 참고하여, 구현해 주세요.

### 👀 과제의 설명

# 과제 가이드

## 개요

커밋마켓(온라인 쇼핑몰)의 핵심 사용자 여정을 **상품 탐색 → 상세 → 장바구니 → 체크아웃(모의 결제) → 주문 완료**까지 구현합니다.

라이브러리나 상태관리 도구는 **자유 선택**이며, 요구사항은 “무엇이 구현되었는지”만 체크할 수 있도록 설계했습니다.

### 사용자 시나리오

* 사용자는 `/products`에서 검색어·카테고리·정렬을 조합해 상품을 탐색한다.
* 상품 카드를 클릭해 `/products/:id` 상세로 이동, 이미지/스펙/재고/태그를 확인한다.
* 상세에서 수량을 선택해 장바구니에 담는다. 재고 부족·품절이면 적절한 안내가 뜬다.
* `/cart`에서 수량 변경·삭제·쿠폰 적용, 배송 방법 선택으로 **총액**이 즉시 갱신된다.
* `/checkout`에서 배송지 폼을 입력·검증하고 **모의 주문 제출**을 완료한다.
* `/order/success?orderId=...`에서 주문 요약(주문번호/금액/배송방법/ETA)을 확인한다.

### 필요한 화면

* **상품 리스트** `/products` 검색 인풋, 카테고리/브랜드/태그 필터, 정렬(신상/평점/가격↑/↓), 반응형 그리드, 로딩·빈·에러 상태.&#x20;
* **상품 상세** `/products/:id` 이미지 갤러리, 가격·재고·평점, 태그, 수량 선택, “장바구니 담기”, 관련 상품(동일 카테고리 일부).&#x20;
* **장바구니** `/cart` 라인아이템(썸네일/단가/수량/소계), 삭제, 쿠폰 적용, 배송 방법 선택, 총액 영역(상품 소계/할인/배송/총합).&#x20;
* **체크아웃** `/checkout` 배송지 폼(이름/전화/우편번호/주소), 결제(모의) 제출, 폼 검증·에러 메시지.&#x20;
* **주문 완료** `/order/success` 주문번호/내역/총액/배송방법/예상 도착일(ETA) 표시, 홈/주문내역 이동 버튼.&#x20;
* **기타**: 404, 공통 토스트/알럿.&#x20;

### 데이터 인터페이스 및 예시

```typescript
// types.ts
export interface Product {
  id: string;
  title: string;
  priceKRW: number;
  rating: number;   // 0.0 ~ 5.0
  stock: number;    // 0 이하면 품절
  brand: string;
  category: string; // Category.id
  thumbnail: string;
  tags: string[];
}

export interface Category {
  id: string;   // 예: "keyboard"
  name: string; // 예: "Keyboards"
}

export interface ShippingMethod {
  id: "standard" | "express" | "free_over_50000";
  name: string;
  feeKRW: number;
  etaDays: [number, number]; // [최소, 최대] 일
  condition?: { minSubtotalKRW?: number };
}

export interface Coupon {
  code: string; // 예: "HELLO10"
  type: "percent" | "fixed" | "free_shipping";
  value: number; // percent면 %, fixed면 KRW
  minSubtotalKRW: number;
  expiresAt: string; // YYYY-MM-DD
}

// 클라이언트 상태 예시(참고)
export interface CartItem {
  productId: string;
  qty: number;
}

export interface OrderDraft {
  items: CartItem[];
  shippingMethodId: ShippingMethod["id"];
  couponCode?: string;
  shippingAddress: {
    name: string;
    phone: string;
    postcode: string;
    address1: string;
    address2?: string;
  };
}

```

```json
// products.json (최소 예시)
[
  {
    "id": "p_101",
    "title": "65% Mechanical Keyboard - Brown Switch",
    "priceKRW": 129000,
    "rating": 4.6,
    "stock": 42,
    "brand": "ZenKeys",
    "category": "keyboard",
    "thumbnail": "/images/products/keyboard_101.png",
    "tags": ["mechanical","brown","wireless","usb-c","compact"]
  },
  {
    "id": "p_102",
    "title": "Wireless Mouse",
    "priceKRW": 59000,
    "rating": 4.4,
    "stock": 63,
    "brand": "AeroLink",
    "category": "mouse",
    "thumbnail": "/images/products/mouse_102.png",
    "tags": ["wireless","low-latency","usb-c","office"]
  },
  {
    "id": "p_103",
    "title": "27\\" 4K Monitor",
    "priceKRW": 489000,
    "rating": 4.6,
    "stock": 11,
    "brand": "PixelWave",
    "category": "monitor",
    "thumbnail": "/images/products/monitor_103.png",
    "tags": ["4k","usb-c","office","gaming"]
  }
]

```

```json
// categories.json (최소 예시)
[
  { "id": "keyboard", "name": "Keyboards" },
  { "id": "mouse", "name": "Mice" },
  { "id": "monitor", "name": "Monitors" }
]

```

```json
// shipping.json (최소 예시)
[
  { "id": "standard", "name": "Standard", "feeKRW": 3000, "etaDays": [2, 4] },
  { "id": "express", "name": "Express", "feeKRW": 6000, "etaDays": [1, 2] },
  { "id": "free_over_50000", "name": "Free over 50,000", "feeKRW": 0, "etaDays": [2, 4], "condition": { "minSubtotalKRW": 50000 } }
]

```

```json
// coupons.json (최소 예시)
[
  { "code": "HELLO10", "type": "percent", "value": 10, "minSubtotalKRW": 0, "expiresAt": "2026-12-31" }
]

```

```json
// order.request.example.json (참고용)
{
  "items": [{ "productId": "p_101", "qty": 2 }],
  "shippingMethodId": "standard",
  "couponCode": "HELLO10",
  "shippingAddress": {
    "name": "홍길동",
    "phone": "010-1234-5678",
    "postcode": "06236",
    "address1": "서울시 강남구 테헤란로 123",
    "address2": "5층"
  }
}

```

```json
// order.response.example.json (참고용 - 모의 응답)
{
  "orderId": "O2025-000123",
  "subtotalKRW": 258000,
  "discountKRW": 25800,
  "shippingKRW": 3000,
  "totalKRW": 235200,
  "shippingMethod": "standard",
  "etaDays": [2, 4]
}

```

​


### 🎯 과제의 요구사항

- 상품 리스트: 검색/카테고리/정렬이 동작하고, 로딩·빈·에러 상태가 구분된다.
- URL 동기화: 리스트 상태(검색/필터/정렬)가 URL 쿼리에 반영되어 새로고침·공유 시 복원된다.
- 상품 상세: 가격/재고/평점/태그/이미지를 표시하고, 수량 선택 및 장바구니 담기가 가능하다.
- 장바구니: 추가/삭제/수량 변경, 재고 초과 방지, 소계·할인·배송비·총합 계산이 즉시 반영된다.
- 체크아웃: 배송지 폼 검증(필수값/형식), 배송 방법 선택, 모의 주문 제출 성공/실패 피드백.
- 주문 완료: 주문번호·요약·ETA 표시.
- 접근성: 폼 레이블·포커스·키보드 탐색 등 기본 기준 충족.
