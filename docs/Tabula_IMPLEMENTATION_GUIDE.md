# 매일경제 디지털 광고 구현 가이드 - 2025.11.25 ver 1.1
- 작성일 : 2025.11.25
- 광고코드의 생성 및 광고 메커니즘, 광고로딩만 구현, 기타 UI 및 디자인은 별도이니 신경쓰지 마세요.

## 개요
- 매일경제 60주년 홈페이지 개편에 맞춰 매일경제 초기면, 섹션, 리스트, 뷰페이지의 광고 데모 구현
- 데모를 바탕으로 개발자와 미팅을 통해 유사한 레벨의 페이지들(증권, 부동산, 스페셜섹션 등)로 개발 확장 예정
- 개발자와 논의를 통해 공통 광고 로직(.js), CSS(.css), HTML로 리팩토링 논의후 진행 
- 페이지 레벨의 키/키밸류(동적입력)에 대한 정의 및 구현 방식에 대해서는 별로로 추가 설명 예정
  (아래와 가팅 '동적입력'이라고 임시로 표기된 부분 )
  
  googletag.pubads().setTargeting('keywords', '동적입력');


### 소스코드

- https://github.com/aura153/Tabula_Rasa
- 각 소스코드 안에 개발자를 위한 성세 주석/설명 추가

### 광고구현 데모

- 뷰페이지 : https://tabula-rasa-lac.vercel.app/view.html
- 초기면 : https://tabula-rasa-lac.vercel.app/HOME.html
- 섹션 : https://tabula-rasa-lac.vercel.app/section.html
- 리스트 : https://tabula-rasa-lac.vercel.app/list.html

### 광고구현가이드
https://github.com/aura153/Tabula_Rasa/blob/main/docs/Tabula_IMPLEMENTATION_GUIDE.md



## 1. 가이드 문서 개요

### 1.1 목적

이 문서는 다음 4개 템플릿에 공통으로 적용되는 **GAM(Google Ad Manager) 구현 규칙**과  
각 템플릿(View / Home / Section / List)별 **슬롯 구조·키/값·로딩 전략 차이**를 정리한 가이드입니다.

- View GPT: 기사 본문 뷰 페이지 (in-article 동적 삽입 포함)
- HOME GPT: 홈(메인) 페이지
- Section GPT: 섹션 프론트(예: 경제 섹션 메인)
- List GPT: 섹션 리스트(예: 경제 정책 리스트)

### 1.2 구현 목표

1. **인벤토리 구조와 타게팅의 표준화**
   - AdUnitPath / page_type / section_*_nm / device_category 등 **페이지 레벨 키/값 통일**
2. **슬롯 레벨 타게팅 명확화**
   - div_id / position / ad_area_type / load_type / default_exposure 등 **슬롯별 역할 명시**
3. **반응형 레이아웃 + 사이즈매핑 통합 정책**
   - 데스크탑/태블릿/모바일 **단일 코드**로 운영
4. **로딩 전략(Eager / Lazy) 최적화**
   - ATF는 Eager, BTF는 Lazy를 기본으로 하여 **수익 + UX 균형** 확보
5. **뷰 페이지(in-article) 수익 극대화**
   - 본문 길이·뷰포트 기준으로 **동적 삽입 규칙**을 설계하여,  
     광고 밀도는 유지하되 사용자 경험 훼손 최소화

---

## 2. 광고 수익화 전략 개요

### 2.1 반응형(Responsive) 전략

- **목표**: 하나의 AdUnitPath로 **디바이스별 인벤토리 구분하지 않고** 운영
- 구현 포인트
  - `googletag.sizeMapping()`을 사용해 **뷰포트 폭 기준**으로 지원 사이즈 그룹 정의
  - 가로형용(`sizeMapping_horizontal`) / 박스형용(`sizeMapping_square`) / in-article 혼합형(`sizeMapping_horizontal_medium`) 등 역할 구분
- 수익 영향
  - 트래픽이 분산되지 않아 **경쟁 밀도(경매 경쟁)** 유지 → CPM 증대 효과
  - 다양한 사이즈를 허용하여 **다양한 유형의 광고(동영상, 리치미디오형, 기본 배너)** 수용

### 2.2 Lazy Loading 전략

- **목표**: BTF / 스크롤 하단 슬롯은 **사용자가 실제로 볼 가능성이 있을 때만** 노출 요청, 사용자 편의성 고려
- 구현 포인트
  - `googletag.pubads().disableInitialLoad()`로 **전역 초기 로드 OFF**
  - Eager 슬롯은 명시적으로 `display + refresh`
  - Lazy 슬롯은 `IntersectionObserver`로 뷰포트 근접 시 `display + refresh` 호출
- 수익 영향
  - **뷰어블 인벤토리 비율 상승 → 뷰어블 기준 과금(뷰어블 CPM, vCPM) 최적화**
  - 불필요한 BTF 노출/호출 감소 → **페이지 체감 속도 개선**, 이탈률 감소에 따른 간접 수익 효과, SEO 및 구글디스커버 최적화 가능

### 2.3 본문 In-Article 동적 삽입(View GPT)

- **목표**: 기사 본문 내 적절한 간격으로 광고를 삽입하여
  - **광고 노출 수(Ad Impressions) 최대화**
  - 동시에 **독자 경험 훼손 최소화**
- 구현 포인트 (상세 구현 전략은 View GPT의 코드 주석 참조 할 것)
  - 가시 텍스트 높이 기준 “뷰포트 단위”로 간격을 계산
  - 이미지/YouTube/데이터박스 등 **시각 블록 인접 위치는 회피**
  - 짧은 기사와 긴 기사를 **서로 다른 정책**으로 처리
- 수익 영향
  - 본문 길이와 관계없이 **안정적인 in-article 광고 노출 수 확보**
  - 시각적 피로도가 과도하게 높지 않도록 제어 → 장기적으로 페이지 체류시간·재방문율에 긍정적 영향

---

## 3. 공통 구현 가이드 (모든 GPT 템플릿 공통)

### 3.1 AdUnitPath 설계

```js
// 예시 (View GPT)
const AdUnitPath = '/7450/www.mk.co.kr/news/economy';
```

1. **고정 구조**
   - `/7450/` : 네트워크 코드 (MK 고정)
   - `/www.mk.co.kr/` : 도메인 또는 사이트 그룹
2. **세부 경로**는 페이지 타입에 따라 다름
   - View : `/news/economy`
   - Home : `/HOME`
   - Section Front : `/news/economy/section_front`
   - List : `/news/economy/section_list`
3. **CMS/템플릿에서 동적 설정 가능 포인트**
   - 섹션/채널이 바뀔 경우, 뒤쪽 세그먼트만 변경하는 것을 원칙으로 합니다.
   - 예: `/news/realestate/section_front`, `/news/stock/section_list` 등

> 🔎 _요청_: 추후 다른 섹션/채널 추가 시 **AdUnitPath + page_type + section_*_nm** 세트로 함께 설계해 주세요.

---

### 3.2 googletag 초기화 및 명령 큐

```js
window.googletag = window.googletag || { cmd: [] };

googletag.cmd.push(function () {
  // 이 안에서 pubads, defineSlot, sizeMapping 등 모든 GPT 초기화 수행
});
```

- `googletag.cmd.push()` 안에 있는 코드는 **GPT JS가 로드된 이후**에 실행됩니다.
- 모든 `defineSlot`, `sizeMapping`, `setTargeting`, `enableServices` 등은 이 콜백 내부에 위치해야 합니다.

---

### 3.3 페이지 레벨 타게팅 (공통 키/값 구조)

#### 3.3.1 공통 키

```js
googletag.pubads().setTargeting('device_category', deviceCategory);
googletag.pubads().setTargeting('page_type', 'view' | 'HOME' | 'section_front');
googletag.pubads().setTargeting('section_home_nm', 'MK' | '');
googletag.pubads().setTargeting('section_front_nm', 'economy' | '');
googletag.pubads().setTargeting('section_list_nm', 'economic-policy' | '동적입력');
googletag.pubads().setTargeting('web_type', 'responsive');
googletag.pubads().setTargeting('is_dark_mode', isDarkMode);
googletag.pubads().setTargeting('gpt_version', '2025');
googletag.pubads().setPublisherProvidedId('PPID_DYNAMIC_HASHED_ID (SERVER_HASHED_ID)');
```

- **page_type**
  - View: `'view'`
  - Home: `'HOME'`
  - Section / List: `'section_front'` (향후 구분 필요 시 `'section_list'`로 분리 고려)
- **section_home_nm / section_front_nm / section_list_nm**
  - 섹션/리스트 구분용 세그먼트 (리포트·타게팅에서 사용)

#### 3.3.2 기사/컨텐츠 메타 정보 (동적 입력 대상)

View / Home / Section / List 모두 아래 키 구조는 동일하며, **실제 값은 CMS에서 주입**하는 것을 전제로 합니다. 추가 설명 예정

```js
googletag.pubads().setTargeting('SOURCE_ID', '동적입력');
googletag.pubads().setTargeting('source_type', '동적입력');
googletag.pubads().setTargeting('MIDDLE_CODE_ENG_NM', '동적입력');
googletag.pubads().setTargeting('CODE_ID', '동적입력');
googletag.pubads().setTargeting('article_num', '동적입력');
googletag.pubads().setTargeting('title', '동적입력');
googletag.pubads().setTargeting('title_slug', '동적입력');
googletag.pubads().setTargeting('url_slug', '동적입력');
googletag.pubads().setTargeting('publish_date', '동적입력');
googletag.pubads().setTargeting('journalist', '동적입력');
googletag.pubads().setTargeting('keywords', '동적입력');
googletag.pubads().setTargeting('ticker_name', '동적입력');
googletag.pubads().setTargeting('has_video', '동적입력');
googletag.pubads().setTargeting('is_app_viewer', '동적입력');
googletag.pubads().setTargeting('login_type', '동적입력');
googletag.pubads().setTargeting('login_status', '동적입력');
googletag.pubads().setTargeting('membership_type', '동적입력');
googletag.pubads().setTargeting('is_paid_content', '동적입력');
googletag.pubads().setTargeting('has_adult_content', '동적입력');
googletag.pubads().setTargeting('brandsensitive', '동적입력');
```

- 공통 규칙
  - **값 주입 책임**: 템플릿 렌더링 시점의 서버 또는 프론트 CMS 스크립트
  - 비어 있더라도 key는 유지 → **리포팅 스키마 일관성 유지**

---

### 3.4 디바이스 카테고리 결정 로직

```js
const userAgent = navigator.userAgent.toLowerCase();
let deviceCategory = 'desktop';

if (/mobile|android|iphone|ipad|ipod/.test(userAgent) || window.innerWidth <= 768) {
  deviceCategory = 'mobile';
} else if (window.innerWidth <= 1024) {
  deviceCategory = 'tablet';
}

googletag.pubads().setTargeting('device_category', deviceCategory);
```

- 기본 정책
  - 모바일: **UA에 모바일 기기 문자열**이 있거나, `window.innerWidth <= 768`
  - 태블릿: UA는 모바일이 아니지만 `innerWidth <= 1024`
  - 나머지: 데스크탑
- 주의사항
  - 페이지 로드 이후 **창 크기만 변하는 경우**를 실시간으로 추적하지는 않음
    - 원칙: **최초 로드 시점**의 디바이스 카테고리를 기준으로 광고 요청
    - 필요 시, SPA 환경에서는 리사이즈 기준 재평가 로직을 별도로 논의

---

### 3.5 사이즈 매핑 (반응형 구현 핵심)

#### 3.5.1 가로형 배너용: `sizeMapping_horizontal`

```js
const sizeMapping_horizontal = googletag.sizeMapping()
  .addSize([1024, 0], [[1200, 300], [1200, 100], [970, 250], [980, 120], [970, 90], [930, 180], [728, 90], [2, 1], [1, 1], 'fluid'])
  .addSize([768, 0],  [[750, 300], [750, 200], [728, 90], [480, 320], [336, 280], [300, 250], [2, 1], [1, 1], 'fluid'])
  .addSize([0, 0],    [[336, 280], [300, 250], [320, 480], [320, 100], [2, 1], [1, 1], 'fluid'])
  .build();
```
- **1024 이상 (데스크탑)**: 우측 날개 영역의 사이즈 매핑은 추후 수정 예정
- **1024 이상 (데스크탑)**: 와이드 리더보드 / 빅사이즈 지원
- **768~1023 (태블릿)**: 750폭 위주 + 728x90 등
- **0~767 (모바일)**: 320x100 / 320x480 / 300x250 / 336x280 등

#### 3.5.2 박스형 배너용: `sizeMapping_square`

```js
const sizeMapping_square = googletag.sizeMapping()
  .addSize([1024, 0], [[300, 600], [336, 280], [2, 1], [1, 1], 'fluid'])
  .addSize([768, 0],  [[336, 280], [300, 250], [2, 1], [1, 1], 'fluid'])
  .addSize([0, 0],    [[336, 280], [300, 250], [320, 480], [320, 100], [2, 1], [1, 1], 'fluid'])
  .build();
```

#### 3.5.3 In-Article 혼합형 (View GPT 전용): `sizeMapping_horizontal_medium`

- MC_article_rectangle_* 슬롯에 사용
- 데스크탑·태블릿에서 **중간 사이즈(480x320, 336x280, 다양한 300x25x) 수용**

> ⚠️ 사이즈 추가/삭제 시 주의:
> - 실제 GAM 인벤토리에 **허용된 사이즈**와 반드시 맞춰야 합니다.
> - 매체·광고주 요구에 따라 `fluid` 제거/추가 시 인벤토리 정의도 함께 수정 필요.

---

### 3.6 슬롯 정의 패턴 및 슬롯 레벨 타게팅

#### 3.6.1 패턴

```js
const slot = googletag.defineSlot(AdUnitPath, SIZE_ARRAY, 'DIV_ID')
  .defineSizeMapping(sizeMapping_xxx)
  .setTargeting('div_id', 'DIV_ID')
  .setTargeting('position', 'TC' | 'MC' | 'MR' | 'BC')
  .setTargeting('ad_area_type', 'ATF' | 'in_article' | 'below_byline' | 'below_comments' | 'below_popular_news')
  .setTargeting('default_exposure', '' | 'hidden')
  .setTargeting('load_type', 'eager' | 'lazy')
  .addService(googletag.pubads());
```

- **div_id**
  - DOM 상의 ID와 동일한 문자열 (`#TC_billboard_X` ↔ div_id = 'TC_billboard_X')
  - 리포팅/디버깅용
- **position**
  - 페이지 내 위치 계층: TC(Top Center), MC(Main Content), MR(Right Rail), BC(Below Content/Comment)
- **ad_area_type**
  - 같은 position 내에서도 역할을 구분 (예: `'in_article'`, `'ATF'`, `'below_byline'` 등)
- **default_exposure**
  - `'hidden'` 등 기본 노출 정책이 필요한 경우 사용 (필요 시 확장 가능)
- **load_type**
  - `'eager'`: 페이지 로드시 바로 요청
  - `'lazy'`: IntersectionObserver를 통해 스크롤 진입 시점에 요청

---

### 3.7 GAM 핵심 기능 사용 전략

```js
googletag.pubads().enableSingleRequest();
googletag.pubads().collapseEmptyDivs(true);
googletag.pubads().disableInitialLoad();  // View/HOME/Section/List 공통 적용
googletag.enableServices();
```

- `enableSingleRequest()`
  - 가능한 한 **모든 슬롯을 하나의 HTTP 요청**으로 묶어 호출
  - 네트워크 레이턴시 감소, 광고 응답 동기화에 유리
- `collapseEmptyDivs(true)`
  - 광고가 매칭되지 않은 슬랏은 **DOM 공간을 자동으로 접어줌**
  - 빈 슬롯으로 인한 레이아웃 공백을 최소화
- `disableInitialLoad()`
  - GPT의 **자동 로드 기능을 끄고**, 우리가 명시적으로 `display + refresh` 호출
  - Eager / Lazy 로딩 전략을 전적으로 **클라이언트 로직에서 관리**

> ❗ 이 설정 덕분에, Lazy 슬롯은 **반드시** IntersectionObserver 콜백에서 `display + refresh`를 호출해야 합니다. 빠뜨리면 영원히 노출되지 않습니다.

---

### 3.8 광고 로딩 방식: Eager vs Lazy

#### 3.8.1 Eager Load

- 사용 위치
  - View: `TC_billboard_X` (데스크탑 전용 상단 ATF)
  - Home: `TC_billboard_leaderboard`
  - Section/List: `TC_billboard_leaderboard`
- 호출 패턴

```js
// View 예시
googletag.cmd.push(function () {
  if (TC_billboard_X_slot) {
    googletag.pubads().refresh([TC_billboard_X_slot]);
  }
});
```

혹은 Section/List처럼 `googletag.display('TC_...'); googletag.pubads().refresh([...])`를 연달아 호출.

#### 3.8.2 Lazy Load

- 사용 위치
  - View: MR_side_rail_X, MC_article_rectangle_*, BC_* 등
  - Home: MC_billboard_rectangle, MR_halfpage_rectangle
  - Section/List: MR_halfpage_rectangle

- 공통 변수 구조

```js
const LAZY_AD_SLOT_IDS = ['MR_halfpage_rectangle', ...];

let lazyAdSlotsHTML = []; // DOM 요소 리스트
let lazyGamSlots = [];    // GAM Slot 객체 리스트
let lazyObserver = null;  // IntersectionObserver
```

- Lazy 흐름 요약
  1. DOM에서 `.ad-slot` 중 `LAZY_AD_SLOT_IDS`에 해당하는 요소를 수집
  2. `googletag.pubads().getSlots()`에서 Slot 객체를 가져와 같은 기준으로 필터
  3. IntersectionObserver를 생성하여 **뷰포트 근접 시점**에 콜백 실행
  4. 콜백 내부에서 해당 슬롯에 대해 `googletag.display(id) + googletag.pubads().refresh([slot])`
  5. 한 번 로드된 슬롯은 `observerInstance.unobserve(entry.target)`로 등록 해제

> 💡 **수익/UX 관점**
> - Lazy 임계값(`rootMargin`)은 수익/UX 균형에 따라 조정 가능 (`'0px 0px 100px 0px'` 등)
> - 너무 늦게 로딩하면 뷰어블 손실, 너무 빨리 로딩하면 Lazy의 의미가 약해짐

---

### 3.9 Advertisement 라벨 구현 방식

#### 3.9.1 GAM 이벤트 리스너

```js
function addAdLabel(event) {
  const slotId = event.slot.getSlotElementId();
  const slotElement = document.getElementById(slotId); 

  // 필요시 특정 슬롯 제외 (예: 네이티브 목록)
  // if (slotId === 'BC_native_list_1' || slotId === 'BC_native_list_2') return;

  if (event.isEmpty === false && slotElement) {
    const outerSlot = slotElement.closest('.ad-slot'); 
    if (!outerSlot) return;

    if (outerSlot.querySelector('.mk-ad-label')) return; 

    const labelElement = document.createElement('div');
    labelElement.className = 'mk-ad-label';
    labelElement.textContent = 'Advertisement';

    outerSlot.insertBefore(labelElement, outerSlot.firstChild);

    outerSlot.style.border = 'none'; 
    outerSlot.style.backgroundColor = 'transparent';

    console.log(`✅ [Ad Label] Label added for ${slotId}`);
  }
}

googletag.pubads().addEventListener('slotRenderEnded', addAdLabel);
```

- 동작 조건
  - `event.isEmpty === false`일 때만 라벨 생성 → 실 광고가 렌더링된 슬롯에만 표시
  - `.ad-slot` 컨테이너 기준으로 라벨 삽입

#### 3.9.2 공통 CSS

```css
.mk-ad-label {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
  padding: 0;
  background-color: transparent;
  border: none;
  color: #5A5A5A;
  font-size: 13px;
  font-weight: normal;
  text-transform: uppercase;
  line-height: 1;
  white-space: nowrap;
}

@media (prefers-color-scheme: dark) {
  .mk-ad-label {
    background-color: transparent;
    color: #A0A0A0;
    border: none;
    font-weight: normal;
  }
}
```

- 라이트/다크 모드 모두에서 **최소한의 시인성 + 레이아웃 영향 최소화**를 목표로 합니다.

---

### 3.10 .ad-slot 공통 CSS 정책

```css
.ad-slot {
  position: relative;
  text-align: center;
  display: block;
  margin: 0 auto;
  font-size: 14px;
  font-weight: bold;
  box-sizing: border-box;
  padding-top: 10px;   /* 라벨 공간 확보 */
  min-height: 1px;
  background-color: transparent;
  width: 100%;
  max-width: 1210px;
  overflow: visible;
}
```

- **min-height: 1px**
  - collapseEmptyDivs로 접힐 때도 레이아웃 충돌을 최소화하기 위한 기본값
- **background/border는 기본 투명**
  - 광고 렌더링 전후 스타일 변경은 JS 라벨 로직에서 처리

> ✏️ 페이지별로 TC/MR 등 일부 슬롯은 border를 제거하기 위해 ID 기반 CSS override를 추가하고 있습니다.

---

## 4. View GPT 구현 가이드 (기사 View 페이지)

### 4.1 역할 개요

- **기사 본문** 기반 페이지
- In-article 본문 중간 광고, 사이드 MR, Byline/댓글 하단, 인기뉴스 네이티브 등  
  가장 복잡한 구조를 가진 템플릿입니다.

### 4.2 주요 AdUnitPath 및 페이지 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/news/economy';

googletag.pubads().setTargeting('page_type', 'view');
googletag.pubads().setTargeting('section_front_nm', 'economy');
googletag.pubads().setTargeting('section_home_nm', '');
```

- View 페이지에서는 **기사 관련 메타 키(article_num, title, publish_date 등)**을 적극적으로 활용하게 됩니다.

### 4.3 슬롯 구성 요약

- Eager
  - `TC_billboard_X` (데스크탑 전용 상단 ATF)
- Lazy
  - `MR_side_rail_X` (데스크탑 Right Rail)
  - `MC_article_rectangle_1` ~ `MC_article_rectangle_8` (in-article)
  - `BC_byline_rectangle` (기자명 하단)
  - `BC_billboard_rectangle` (댓글 아래)
  - `BC_native_list_1`, `BC_native_list_2` (인기뉴스 내 네이티브)

### 4.4 In-Article 동적 삽입 로직

#### 4.4.1 실행 시점

```js
document.addEventListener('DOMContentLoaded', function () {
  setupInArticleAds();    // 본문 in-article 광고 동적 주입
  setupLazyLoading();     // Lazy 인프라
  shufflePopularNewsNativeAds(); // 인기 뉴스 내 네이티브 랜덤 배치
  startObservingLazySlots();
});
```

- DOMContentLoaded 시점에 **이미 본문 DOM 구조가 완성**되어 있다는 전제입니다.
- SPA나 지연 로딩되는 텍스트가 있을 경우, 해당 구조가 깨질 수 있으므로 주의 필요

#### 4.4.2 본문 블록 탐색 및 앵커 생성

```js
const articleBody = document.querySelector('.article-body');
const viewportHeight = window.innerHeight || 800;

const VISUAL_BLOCK_CLASSES = [
  'article-image',
  'article-embed',
  'dummy-data-box',
  'visual-block',
  'data-table',
  'chart-box'
];

const contentBlocks = Array.from(
  articleBody.querySelectorAll(
    'p, .article-image, .article-embed, .dummy-data-box, ' +
    'figure, table, iframe, video, .data-table, .chart-box'
  )
);
```

- 텍스트(p) + 시각 블록(이미지, 임베드, 차트 등)을 **같은 리스트로 관리**
- 각 블록에 대해
  - `isVisual` (시각 블록 여부)
  - `textHeight` (텍스트인 경우에만 높이 누적)
  - `cumulativeTextHeight` (지금까지 텍스트 누적 높이)
  - 를 계산하여 **뷰포트 기준 간격 계산에 활용**

#### 4.4.3 짧은 기사 예외 처리

```js
const totalTextHeight = anchors.length > 0
  ? anchors[anchors.length - 1].cumulativeTextHeight
  : 0;

const isShortArticle = totalTextHeight < viewportHeight;

if (isShortArticle && lastContentBlock) {
  // 1) 마지막 블록 뒤에 첫 번째 in-article 광고만 삽입
  // 2) 나머지 in-article 슬롯은 DOM + GAM에서 제거(destroySlots)
}
```

- 텍스트 누적 높이가 1 뷰포트보다 작으면
  - 본문 중간에 여러 개를 넣는 대신, **마지막에 1개만 삽입**
  - 나머지 MC 슬롯은 DOM 제거 + `googletag.destroySlots()`로 리소스 정리

#### 4.4.4 일반 기사 삽입 규칙

- 기준 파라미터
  - `VIEWPORT_GAP_BETWEEN_ADS = 1.0`
  - `VIEWPORT_AFTER_VISUAL_MIN = 0.5`

- 조건 요약
  1. **광고 간 최소 거리**:  
     - 텍스트 기준 누적 높이 상, 마지막 광고 이후 최소 `1.0 * viewportHeight` 이상
  2. **직전 시각 블록 이후 최소 거리**:  
     - 마지막 시각 블록 이후 최소 `0.5 * viewportHeight` 이상
  3. **인접 시각 블록 회피**:
     - 현재 후보 블록의 **위 또는 아래에 시각 블록이 있으면** 그 위치는 스킵

- 실제 삽입

```js
const slotId = MC_IN_ARTICLE_SLOT_IDS[slotIndex];
const slotEl = document.getElementById(slotId);

articleBody.insertBefore(slotEl, anchor.element);
placedSlotIds.add(slotId);
lastAdTextHeight = anchor.cumulativeTextHeight;
slotIndex++;
```

- 사용되지 않은 MC 슬롯은
  - DOM에서 제거
  - `googletag.pubads().getSlots()`에서 찾아 `googletag.destroySlots(unusedSlots)` 호출

> ✅ 이 구조 덕분에 기사 길이/구조가 달라져도 **in-article 광고 수·위치는 자동 조정**됩니다.

### 4.5 인기 뉴스 내 네이티브 랜덤 배치

```js
function shufflePopularNewsNativeAds() {
  const popularList = document.querySelector('.popular-news-list');
  const allItems = Array.from(popularList.children);
  const isAd = (item) => item.id && item.id.startsWith('BC_native');

  const articleItems = allItems.filter(item => !isAd(item));
  const adItems = allItems.filter(item => isAd(item));

  const firstArticle = articleItems.shift(); // 1위 뉴스는 고정
  const remainingItems = [...articleItems, ...adItems];

  // Fisher–Yates shuffle
  ...
}
```

- 정책
  - 인기 뉴스 1위는 **항상 기사**로 고정
  - 나머지 기사/네이티브는 함께 섞어서 **슬롯 위치 다양성** 확보

### 4.6 Lazy Load 구성 (View GPT)

- `LAZY_AD_SLOT_IDS`에 View 페이지의 Lazy 대상 슬롯들을 모두 정의
- `setupLazyLoading()` + `startObservingLazySlots()` 구조는 공통

> 슬롯 ID만 추가하면 곧바로 Lazy 대상에 포함되도록 설계되어 있습니다.

---

## 5. HOME GPT 구현 가이드 (홈 페이지)

### 5.1 역할 개요

- 메인 홈(포털 역할) 페이지
- 헤드라인/섹션 요약 + 일부 광고 슬롯으로 구성된 상대적으로 단순한 레이아웃

### 5.2 AdUnitPath 및 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/HOME';

googletag.pubads().setTargeting('page_type', 'HOME');
googletag.pubads().setTargeting('section_home_nm', 'MK');
googletag.pubads().setTargeting('section_front_nm', '동적입력');
```

### 5.3 슬롯 구성

- Eager
  - `TC_billboard_leaderboard`
- Lazy
  - `MC_billboard_rectangle`
  - `MR_halfpage_rectangle`

### 5.4 특징

- 본문 in-article 로직 없음
- Lazy 스크롤 트리거 지점이 비교적 상단에 위치해 **ATF/1스크롤 이내**에서 주요 인벤토리 로딩 완료
- View GPT와 동일한 Lazy 인프라 구조를 사용하되, **대상 ID만 다름**

---

## 6. Section GPT 구현 가이드 (섹션 프론트)

### 6.1 AdUnitPath 및 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/news/economy/section_front';

googletag.pubads().setTargeting('page_type', 'section_front');
googletag.pubads().setTargeting('section_front_nm', 'economy');
googletag.pubads().setTargeting('section_list_nm', '동적입력');
```

### 6.2 슬롯 구성

- Eager
  - `TC_billboard_leaderboard`
- Lazy
  - `MR_halfpage_rectangle`

### 6.3 특징

- HOME GPT 레이아웃을 기반으로 섹션 콘텐츠만 교체한 구조
- Lazy 인프라는 동일 (`LAZY_AD_SLOT_IDS = ['MR_halfpage_rectangle']`)
- 추후 섹션 하단에 MC/BC 광고 슬롯을 추가할 경우
  - View/HOME와 마찬가지로 **ID만 LAZY_AD_SLOT_IDS에 추가하면 재사용 가능**

---

## 7. List GPT 구현 가이드 (섹션 리스트)

### 7.1 AdUnitPath 및 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/news/economy/section_list';

googletag.pubads().setTargeting('page_type', 'section_front');   // (향후 section_list로 분리 고려)
googletag.pubads().setTargeting('section_front_nm', '');
googletag.pubads().setTargeting('section_list_nm', 'economic-policy');
```

- Section GPT와 동일한 구조이되,
  - `section_front_nm` 대신 `section_list_nm` 중심으로 타게팅하는 것이 차이점입니다.

### 7.2 슬롯 구성 및 Lazy 구조

- Section GPT와 동일
  - Eager: `TC_billboard_leaderboard`
  - Lazy: `MR_halfpage_rectangle`
- Lazy 인프라 완전히 동일

> 사실상 “Section GPT + 타게팅 값만 리스트 용으로 바꾼 버전”이라고 이해하시면 됩니다.

---

## 8. 확장 및 커스터마이징 가이드

### 8.1 새 페이지 타입 추가 체크리스트

1. AdUnitPath 정의
   - `/7450/www.mk.co.kr/...` 뒤 경로 설계
2. page_type 및 section_*_nm 설계
   - 리포트·타게팅에서 어떤 축으로 보고 싶은지 먼저 정의
3. 공통 페이지 타게팅 주입
   - device_category, web_type, gpt_version 등 공통 키
4. 슬롯 구조 설계
   - TC / MC / MR / BC 구분
   - Eager vs Lazy 분리
5. 사이즈 매핑 재사용 여부 확인
   - 기존 `sizeMapping_horizontal` / `sizeMapping_square` / `sizeMapping_horizontal_medium`을 재사용
   - 필요한 경우 별도 매핑 추가
6. Lazy 인프라 적용
   - `LAZY_AD_SLOT_IDS`에 Lazy 대상 ID 추가
   - 별도 로직 필요 없으면 `setupLazyLoading()` / `startObservingLazySlots()` 재사용

### 8.2 신규 슬롯 추가시 주의사항

- DOM
  - `<div id="NEW_SLOT_ID" class="ad-slot ..."></div>` 추가
- GPT 정의
  - `googletag.defineSlot(AdUnitPath, [...sizes...], 'NEW_SLOT_ID') ...`
- 타게팅
  - `div_id`, `position`, `ad_area_type`, `load_type` 세팅
- Lazy 여부
  - Lazy일 경우 `LAZY_AD_SLOT_IDS`에 `'NEW_SLOT_ID'` 추가 필수

---

## 9. QA 및 디버깅 팁

1. **GAM Publisher Console 사용**
   - `?google_console=1` 파라미터로 광고 디버깅 콘솔 활성화
   - 각 슬롯의 타게팅 값, 사이즈, 라인아이템 매칭 상태 확인
2. **Network 패널 확인**
   - `securepubads.g.doubleclick.net` 요청이 언제 나가는지,
   - Eager / Lazy 슬롯이 계획대로 트리거되는지 체크
3. **브레이크포인트 테스트**
   - 320 / 768 / 1024 px 기준으로 DevTools 디바이스 모드를 이용하여
   - `sizeMapping` 동작 및 노출 사이즈 확인
4. **레이아웃 검증**
   - `collapseEmptyDivs(true)`로 인해 접히는 영역이 레이아웃을 깨지 않는지
   - `.ad-slot`의 `min-height` 및 주변 마진/패딩 조합 확인
5. **로그 활용**
   - 콘솔 로그
     - `[GPT INIT]`, `[Lazy Load SUCCESS]`, `[Ad Label]`, `[In-Article] ...` 등
   - 문제 발생 시 이 로그들을 기준으로 어느 단계에서 멈췄는지 추적

---

## 10. 마무리

추후 다음과 같은 방향으로 계속 업데이트 예정

- 1st-party 데이터 강화 및 타겟팅 발굴
- PPID 적용
- Prebid/오픈빗 통합, **헤더비딩 연동 규칙** 추가, 수익화 극대화 추진

실제 구현 중 막히는 부분, 혹은 기획 단계에서 추가로 정의가 필요한 부분이 있으시면  
담당자(매일경제 광고마케팅국 디지털마케팅부 황재연) 편하게 말씀해 주세요.  

