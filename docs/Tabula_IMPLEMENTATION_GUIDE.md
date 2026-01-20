# 매일경제 디지털 광고 구현 가이드 - ver 2.1
- 작성일 : 2025.11.25  
- 2차 업데이트 : 2025.12.02 18:01 PM
- 3차 업데이트 : 2025.12.09 15:00 PM
- 4차 업데이트 : 2025.12.16
- 광고코드의 생성 및 광고 메커니즘, 광고로딩만 구현  
- UI 및 디자인에 대한 고려는 제외, 광고 사이즈, 형태, 위치 등은 추후 최적화 진행  

# ----------------# 3차 업데이트 주요 내용--------------------------------
### 1. 모든 페이지 전체 넓이 1246px로 조정 (View / HOME / Section / List 공통)
.container { max-width: 1246px; margin: 0 auto; padding: 0 10px; }

### 2. HOME/Section/List의 .sidebar 폭 300 → 336px (우측 MR 영역)으로 조정, 
@media (min-width: 1025px) {
    .sidebar {
        flex: 0 0 336px;
        min-width: 336px;
        /* max-width: 336px;  ← 꼭 필요하진 않지만 추가해도 무방 */
    }
}

### 3. 사이즈 매핑 업데이트, 통일 (HOME/Section/List/View 전역)
  #### 3.1 용도별 사이즈 매핑 업데이트, 네이밍 통일 (상세 내용은 본 가이드 374줄 또는 각 데모 소스 확인)
  - 가로형 : `sizeMapping_horizontal` : 'TC_billboard_leaderboard', 'TC_billboard_X', 'BC_byline_rectangle' 등 수평 가로형 광고 슬롯 정의
  - 세로 박스형 / 사이드 레일 :  `sizeMapping_siderail_square` : 'MR_halfpage_rectangle' 우측 사이드레일 세로 박스형/세로형 슬롯 정의
  - 본문 박스형 : `sizeMapping_article_horizontal` : 'MC_article_rectangle_1~8' 본문 박스형 슬롯
  #### 3.2 사이즈 매핑으로 인한 각 슬롯별 광고 사이즈 모두 업데이트
  #### 3.2. sizeMapping_right_square → sizeMapping_siderail_square 로 교체, 그리고 더 이상 쓰이지 않는 sizeMapping_right_square 선언은 삭제.

### 4 HOME/Section/List/View 페이지에서 상단 영역에서  "Advertisement" 라벨 제외
- HOME/Section/List 상단 'TC_billboard_leaderboard' 슬롯에 "Advertisement" 라벨 제외
-  View  TC_billboard_X  슬롯에서  "Advertisement" 라벨 제외

### 5. HOME만 해당(나머지는 해당사항 없음)  '체크박스 + 오늘하루 | 닫기 X 버튼' 은 데스크탑 전용으로만 변경

  #### 5.1 '오늘 하루 닫기' / 디바이스 정책 관련 전역 헬퍼들
        // ---------------------------------------------------
        // - HOME GPT 상단 `TC_billboard_leaderboard` 전용 로직
        // - 정책:
        //   · 초기면(HOME)의 상단 TC에만 `오늘 하루 닫기` 기능을 제공
        //   · 현재는 데스크탑 전용으로 운용
        //     (모바일/태블릿에서는 레이아웃 안정성 및 수익성 관점에서 닫기 기능 비활성화)
        //   · 상위 플래그(window.__MK_TC_CLOSE_ENABLED__ !== false)를 통해
        //    **** 로드블로킹 캠페인 등에서 "닫기 기능 전체 OFF"가 가능하도록 설계
  #### 5.2 상단 닫기 버튼 UI (.mk-ad-close-*) CSS 최적화
# 끝--------------------------------------------------------------------------

# 4차 업데이트 ; article_horizontal에서 사이즈 일부 삭제 ---

 ## sizeMapping_article_horizontal 에서 1024 폭에서 [750, 300] , 768 폭에서 '[750, 300], [750, 200], [728, 90], ' 사이즈 삭제 ##
 MC_article_rectangle_1~2 슬롯 사이즈에 삭제된 사이즈가 존재하는지 확인하고, 삭제 요망
    .addSize([1024, 0], [
        [480, 320], [336, 280],[320, 480], 

const sizeMapping_article_horizontal = googletag.sizeMapping()
    .addSize([768, 0], [
        [480, 320], [336, 280], [320, 480], 
(이하생략)

 ## sizeMapping_article_horizontal 이 적용되는 광고슬롯(MC_article_rectangle_1~8)에서 [750, 300] , 768 폭에서 '[750, 300], [750, 200], [728, 90], ' 사이즈 삭제 ##

# 5차 업데이트(2026.01.05)
            // --- [rootMargin 최적화: 디바이스별 rootMargin 동적 할당] ---
            /*
             * [업데이트 된 로직 설명]
             * - 기존에는 100px 고정이었으나, 한국의 네트워크 환경과 스크롤 패턴을 반영하여
             * 디바이스별 최적의 마진 값을 적용합니다.
             * - Smartphone: 600px (빠른 플릭 대응)
             * - Tablet: 500px
             * - Desktop: 400px

# 6차 업데이트
## 디바이스 카테고리 결정 로직 - 2026.1.6 업데이트
- UA(User Agent)와 viewport 폭을 결합하여 디바이스를 3단계로 구분합니다.
  1. smartphone : 스마트폰용 UA 문자열이 포함되어 있거나, 화면 폭이 768px 이하인 경우
  2. tablet     : 스마트폰은 아니지만 태블릿용 UA 문자열이 포함되어 있거나, 화면 폭이 1024px 이하인 경우
  3. desktop    : 위 조건에 해당하지 않는 모든 경우 (기본값)

  - 이 값은 슬롯 정의 여부(if 조건)와 GAM 타게팅 값(device_category)으로 활용됩니다.
```js
const userAgent = navigator.userAgent.toLowerCase();
const width = window.innerWidth;
let deviceCategory = 'desktop';

// 1. Smartphone 판별: 모바일 UA(iPad 제외)이거나 폭이 768px 이하인 경우
if ((/mobile|android|iphone|ipod/.test(userAgent) && !/ipad/.test(userAgent)) || width <= 768) {
    deviceCategory = 'smartphone';
} 
// 2. Tablet 판별: iPad/Tablet UA이거나 폭이 1024px 이하인 경우
else if (/ipad|tablet/.test(userAgent) || width <= 1024) {
    deviceCategory = 'tablet';
})

## 0. 개요

- 매일경제 60주년 홈페이지 개편에 맞춰 **매일경제 초기면(HOME), 섹션(Section), 리스트(List), 뷰(View)** 페이지의 광고 데모 구현
- 이 데모를 바탕으로 개발자와 미팅을 진행하고,  
  동일/유사 레벨의 페이지들(증권, 부동산, 스페셜섹션 등)로 확장 개발 예정
- 개발자와 협의하여 **공통 광고 로직(.js), 공통 CSS(.css), 공통 HTML 구조**로 리팩토링 방향을 설계한 뒤,
  실제 서비스 반영 전 단계에서 재협의
- **페이지 레벨 키/키밸류(동적 입력)**에 대한 구체 정의 및 구현 방식은 별도 문서로 추가 예정  
  (현재는 `'동적입력'` placeholder 사용)

```js
googletag.pubads().setTargeting('keywords', '동적입력');
```

### 0.1 소스코드

- GitHub 저장소: https://github.com/aura153/Tabula_Rasa  
- 각 템플릿(View / HOME / Section / List) 소스코드 내부에 **개발자를 위한 상세 주석**을 충분히 추가

### 0.2 광고 구현 데모 URL

- View 페이지 : https://aura153.github.io/Tabula_Rasa/view.html  
- HOME(초기면) : https://aura153.github.io/Tabula_Rasa/HOME.html  
- Section(섹션 프론트) : https://aura153.github.io/Tabula_Rasa/section.html  
- List(섹션 리스트) : https://aura153.github.io/Tabula_Rasa/list.html  

### 0.3 추가 문서

- 광고 구현 가이드(원본):  
  https://github.com/aura153/Tabula_Rasa/blob/main/docs/Tabula_IMPLEMENTATION_GUIDE.md  

---

## 1. 가이드 문서 개요

### 1.1 목적

이 문서는 다음 4개 템플릿에 공통으로 적용되는 **GAM(Google Ad Manager) 구현 규칙**과  
각 템플릿(View / HOME / Section / List)별 **슬롯 구조·키/값·로딩 전략 차이**를 정리한 종합 가이드입니다.

- **View GPT**: 기사 본문 뷰 페이지 (본문 in-article 동적 삽입 포함)
- **HOME GPT**: 홈(메인) 페이지
- **Section GPT**: 섹션 프론트(예: 경제 섹션 메인)
- **List GPT**: 섹션 리스트(예: 경제 정책 리스트 등)

### 1.2 구현 목표

1. **인벤토리 구조와 타게팅의 표준화**
   - `AdUnitPath / page_type / section_*_nm / device_category` 등 **페이지 레벨 키/값 통일**
2. **슬롯 레벨 타게팅 명확화**
   - `div_id / position / ad_area_type / load_type / default_exposure` 등 **슬롯별 역할 명시**
3. **반응형 레이아웃 + 사이즈 매핑 통합 정책**
   - 데스크탑/태블릿/모바일을 **하나의 통합 코드**로 운영
4. **로딩 전략(Eager / Lazy) 최적화**
   - ATF는 Eager, BTF는 Lazy를 기본으로 하여 **수익 + UX 균형** 확보
5. **뷰 페이지(in-article) 수익 극대화**
   - 본문 길이·뷰포트 기준으로 **동적 in-article 삽입 규칙**을 설계하여,  
     광고 밀도는 유지하되 사용자 경험 훼손 최소화

---

## 2. 프로젝트 개요 및 전략 (Overview & Strategy)

### 2.1 작업 목적
이번 템플릿(View / HOME / Section / List)의 공통 목표는 다음과 같습니다.

- **GAM 360 인벤토리 구조 정리**
- View / HOME / Section / List 4종 페이지를 하나의 **공통 패턴**으로 운용
- **수익화 · 성능(Core Web Vitals/SEO) · UX**를 동시에 고려한 **표준 구현 레퍼런스** 구축

### 2.2 코드 상 공통 특징

#### 2.2.1 공통 AdUnitPath 패턴
- HOME: `/7450/www.mk.co.kr/HOME`
- View: `/7450/www.mk.co.kr/news/economy`
- Section: `/7450/www.mk.co.kr/news/economy/section_front`
- List: `/7450/www.mk.co.kr/news/economy/section_list` (메모리 기준)

#### 2.2.2 공통 타게팅 키 구조

- `device_category, page_type, section_home_nm, section_front_nm, section_list_nm, SOURCE_ID` 등

#### 2.2.3 공통 GAM 설정 패턴
    ```javascript
    googletag.setConfig({
        singleRequest: true,        // SRA(Single Request Architecture) 활성화
        disableInitialLoad: true,   // 초기 로드 방지 (Lazy Load 필수 전제) ; 모든 슬롯은 **display() + refresh()**가 호출될 때만 요청
        centering: true,            // 크리에이티브 수평 중앙 정렬
        collapseDiv: 'BEFORE_FETCH' // 빈 슬롯 영역 선제적 붕괴 (CLS 방지); 광고가 없거나 요청 전인 슬롯은 **0px로 접힘**
    });

googletag.pubads().addEventListener('slotRenderEnded', addAdLabel); //`slotRenderEnded + addAdLabel()`; 렌더 완료 시 자동으로 “Advertisement” 라벨 삽입
googletag.enableServices();
```

### 2.3 수익화 관점 (Monetization)

#### 2.3.1 Viewability 기반 수익 최적화

- `disableInitialLoad() + IntersectionObserver` 기반 **Lazy Load**
  - 실제로 뷰포트 근처까지 스크롤된 슬롯만 `display + refresh`
- Lazy 대상 예
  - View GPT : in-article, MR 레일, BC 하단, Native 리스트
  - HOME / Section / List GPT : MR/MC 슬롯

#### 2.3.2 위치·역할별 세부 타게팅

- 각 슬롯별로 다음과 같은 타게팅 사용
  - `position`: `TC / MC / MR / BC`
  - `ad_area_type`: `ATF, in_article, below_byline, below_comments, below_popular_news` 등
- 장점
  - 리포트/입찰에서 **위치별 성과 분석 및 최적화** 용이

### 2.4 SEO & Performance (Core Web Vitals 관점)

1. **초기 로딩 부담 최소화**
   - 상단 핵심 슬롯(예: HOME `TC_billboard_leaderboard`, View `TC_billboard_X`) 등 **Eager**
   - 그 외 중단/하단/사이드/인아티클 광고는 모두 Lazy
   - => HTML / 주요 기사 콘텐츠가 먼저 렌더링 → **LCP, FCP 개선**
2. **화이트 스페이스 및 CLS 최소화**
   - `collapseDiv: 'BEFORE_FETCH'`로 광고 응답 전까지 슬롯 높이를 0으로 유지
   - 빈 슬롯에 의한 공백 없음, **레이아웃 이동(CLS) 최소화**

### 2.5 사용자 경험 (UI/UX)

1. **시각적 안정성**
   - `.ad-slot`에 height/min-height를 강제하지 않고,  
     `collapseDiv`에 의해 0px → 광고 로드 시 자연스럽게 펼침
2. **자연스러운 본문 흐름 (View GPT)**
   - `setupInArticleAds()`에서 문단/시각 블록 구조를 분석
   - 텍스트 기준 누적 높이로 광고 삽입 위치 계산
   - 짧은 기사에는 1개만 사용 후 나머지 슬롯은 `destroy`
3. **리소스 효율**
   - **IntersectionObserver** 기반 Lazy Load로  
     보지 않는 영역 광고에 대한 네트워크 요청 자체를 지연/생략
   - 모바일에서 자원(통신, 트래픽/배터리) 소비 절감

---

## 3. 광고 수익화 전략 개요

### 3.1 반응형(Responsive) 전략

- **목표**: 하나의 AdUnitPath로 **디바이스별 인벤토리를 분리하지 않고** 운영

#### 3.1.1 구현 포인트
- `googletag.sizeMapping()`을 사용해 **뷰포트 폭 기준**으로 지원 사이즈 그룹 정의
- 용도별 사이즈 매핑
  - 가로형 : `sizeMapping_horizontal` : 'TC_billboard_leaderboard', 'TC_billboard_X', 'BC_byline_rectangle' 등 수평 가로형 광고 슬롯 정의
  - 세로 박스형 / 사이드 레일 :  `sizeMapping_siderail_square` : 'MR_halfpage_rectangle' 우측 사이드레일 세로 박스형/세로형 슬롯 정의
  - 본문 박스형 : `sizeMapping_article_horizontal` : 'MC_article_rectangle_1~8' 본문 박스형 슬롯

#### 3.1.2 수익 영향
- 트래픽이 분산되지 않아 **경쟁 밀도(Ad Auction 경쟁)** 유지 → CPM 상승 효과
- 다양한 사이즈를 허용하여 **동영상, 리치미디어, 기본 배너** 등 다양한 캠페인 수용

### 3.2 Lazy Loading 전략

- **목표**: BTF / 스크롤 하단 슬롯은 **사용자가 실제로 볼 가능성이 있을 때, 유효 노출 시에만** 노출 요청

#### 3.2.1 구현 포인트
- `googletag.pubads().disableInitialLoad()`로 **전역 자동 로드 OFF**
- Eager 슬롯
  - 명시적으로 `googletag.display(id); googletag.pubads().refresh([slot]);` : disableInitialLoad() 을 무력화 하고 바로 광고 요청
- Lazy 슬롯
  - `IntersectionObserver`로 **뷰포트 근접 시점**에 `display + refresh` 호출

#### 3.2.2 수익 및 UX 영향
- 뷰어블 인벤토리 비율 상승 → **vCPM/뷰어블 과금 기준 최적화**
- 불필요한 BTF 호출 감소 → SEO 측면에서 **페이지 체감 속도 개선** 및 이탈률 감소  
- 결과적으로 **SEO 및 구글 디스커버 노출 최적화**에 긍정적

### 3.3 본문 In-Article 동적 삽입 (View GPT)

- 목표
  - 기사 본문 내 **적절한 간격**으로 광고를 삽입하고,
  - **광고 노출 수 극대화**와 **독자 경험**을 동시에 달성

#### 3.3.1 구현 포인트
- “가시 텍스트 높이” 기준으로 **뷰포트 단위 간격** 계산
- 이미지/YouTube/데이터박스 등 **시각 블록 인접 위치 회피**
- 짧은 기사와 긴 기사에 **각기 다른 정책** 적용

#### 3.3.2 수익 영향
- 기사 길이와 관계없이 **안정적인 in-article 광고 노출 수** 확보
- 광고 간 적절한 간격을 유지하여 사용자 피로도 감소 → **체류시간/재방문율 개선**

---

## 4. 공통 구현 가이드 (모든 GPT 템플릿 공통)

### 4.1 AdUnitPath 설계
```js
// 예시 (View GPT)
const AdUnitPath = '/7450/www.mk.co.kr/news/economy';
```

1. **AdUnitPath 구조**
   - `/7450/` : 네트워크 코드 (MK 고정)
   - `/www.mk.co.kr/` : 도메인 또는 사이트 그룹
   - **세부 경로**
2. **세부 경로**는 페이지 타입별로 구분
   - View : `/news/economy`
   - HOME : `/HOME`
   - Section Front : `/news/economy/section_front`
   - List : `/news/economy/section_list`
3. **CMS/템플릿에서 동적 설정 예시**
   - 예시
   - 초기면 : `/7450/www.mk.co.kr/HOME`
   - 뉴스 섹션 홈 : `/7450/www.mk.co.kr/news/realestate/section_front` 'realesate'와 같이 **서비스 메뉴명을 동적 경로**로 사용 하고 section_front라는 'page_type' 표기
   - 뉴스 섹션 리스트 `/7450/www.mk.co.kr/news/realestate/section_list` 'realesate'와 같이 **서비스 메뉴명을 동적 경로**로 사 용하고 section_list라는 'page_type' 표기
   - **중요** 뉴스 기사뷰 `/7450/www.mk.co.kr/news/realestate/` CMS, WMS의 **MIDDLE_CODE_ENG_NM 기사 카테고리 영문명을 동적 경로**로 사용
   
--

### 4.2 googletag 초기화 및 명령 큐

```js
window.googletag = window.googletag || { cmd: [] };

googletag.cmd.push(function () {
  // 이 안에서 pubads, defineSlot, sizeMapping, setTargeting, enableServices 등
  // 모든 GPT 초기화 수행
});
```

- `googletag.cmd.push()` 내부 코드는 **GPT JS 로드 이후** 실행
- `defineSlot`, `sizeMapping`, `setTargeting`, `enableServices`는 반드시 이 콜백 내부에 위치

---

### 4.3 페이지 레벨 타게팅 (공통 키/값 구조)

#### 4.3.1 공통 키 예시

```js
googletag.pubads().setTargeting('device_category', deviceCategory);
googletag.pubads().setTargeting('page_type', 'view' | 'HOME' | 'section_front' | 'section_list');
googletag.pubads().setTargeting('section_home_nm', 'MK' | '');
googletag.pubads().setTargeting('section_front_nm', 'economy' | '');
googletag.pubads().setTargeting('section_list_nm', 'economic-policy' | '동적입력');
googletag.pubads().setTargeting('web_type', 'responsive');
googletag.pubads().setTargeting('is_dark_mode', isDarkMode);
googletag.pubads().setTargeting('gpt_version', '2025');

// 공통 도메인/URL 정보
googletag.pubads().setTargeting('domain', window.location.hostname);
googletag.pubads().setTargeting('url', window.location.href);
googletag.pubads().setTargeting('referer', document.referrer || '');

// PPID (서버에서 해시/익명화된 값 주입)
googletag.pubads().setPublisherProvidedId('PPID_DYNAMIC_HASHED_ID (SERVER_HASHED_ID)');
```

- **page_type**
  - View: `'view'`
  - HOME: `'HOME'`
  - Section Front: `'section_front'`
  - List: `'section_list'` (최신 코드 기준으로 명확히 분리 권장)
- **section_*_nm**
  - 섹션/리스트 구분용 세그먼트 (리포트·타게팅 축으로 활용)

#### 4.3.2 기사/컨텐츠 메타 정보 (동적 입력 대상)

모든 템플릿에서 공통으로 사용하는 키 구조이며, 실제 값은 CMS/백엔드에서 주입: 

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

- **값 주입 책임**: 템플릿 렌더링 시점의 서버 또는 CMS 스크립트
- 비어 있어도 key는 유지 → **리포팅 스키마 일관성 유지**
**공통모듈 구성시 편의상 HOME, section,  list, 기타 페이지에 동일하게 설정해도 무방**
---

### 4.4 디바이스 카테고리 결정 로직 - 2026.1.6 업데이트
- UA(User Agent)와 viewport 폭을 결합하여 디바이스를 3단계로 구분합니다.
  1. smartphone : 스마트폰용 UA 문자열이 포함되어 있거나, 화면 폭이 768px 이하인 경우
  2. tablet     : 스마트폰은 아니지만 태블릿용 UA 문자열이 포함되어 있거나, 화면 폭이 1024px 이하인 경우
  3. desktop    : 위 조건에 해당하지 않는 모든 경우 (기본값)

  - 이 값은 슬롯 정의 여부(if 조건)와 GAM 타게팅 값(device_category)으로 활용됩니다.
```js
const userAgent = navigator.userAgent.toLowerCase();
const width = window.innerWidth;
let deviceCategory = 'desktop';

// 1. Smartphone 판별: 모바일 UA(iPad 제외)이거나 폭이 768px 이하인 경우
if ((/mobile|android|iphone|ipod/.test(userAgent) && !/ipad/.test(userAgent)) || width <= 768) {
    deviceCategory = 'smartphone';
} 
// 2. Tablet 판별: iPad/Tablet UA이거나 폭이 1024px 이하인 경우
else if (/ipad|tablet/.test(userAgent) || width <= 1024) {
    deviceCategory = 'tablet';
}

googletag.pubads().setTargeting('device_category', deviceCategory);
```
- **smartphone**: UA 상 스마트폰 문자열 포함 OR `innerWidth <= 768`
- **tablet**: 스마트폰은 아니지만 `innerWidth <= 1024`
- **desktop**: 그 외
- SPA/리사이즈 환경에서는 필요 시 별도 재평가 로직 협의

---

### 4.5 사이즈 매핑 (반응형 구현 핵심)

View / HOME / Section / List GPT 코드를 기준으로 정리한 최신 가이드입니다.  
(코드는 **공통 가이드용 기준값**이고, 각 템플릿별로 일부 사이즈 추가·삭제가 있을 수 있습니다.)
---

#### 4.5.1 가로형 배너용 – HOME / Section / List / View 공통 개념(sizeMapping_horizontal)

```js
const sizeMapping_horizontal = googletag.sizeMapping()
    .addSize([1024, 0], [
        [1246, 100], [1200, 300], [1200, 100], 
        [980, 120], [970, 250], [930, 180], [970, 90], 
        [728, 90], 
        [2, 1], [1, 1], 
        'fluid'
    ])
    .addSize([768, 0], [
        [750, 300], [750, 200], [728, 90], 
        [480, 320], 
        [336, 280], [320, 480], [300, 250], 
        [2, 1], [1, 1], 
        'fluid'
    ])
    .addSize([0, 0], [
        [336, 280], [320, 480],
        [320, 100], [320, 50], 
        [300, 250], [250, 250], [200, 200], 
        [2, 1], [1, 1], 
        'fluid'
    ])
    .build();
```

- **용도**
  - HOME GPT:  
    - `TC_billboard_leaderboard`, `MC_billboard_rectangle`에 적용
  - Section GPT / List GPT:  
    - 상단 `TC_billboard_leaderboard`  가로형 배너에 적용
  - View GPT:  
    - 상단 `TC_billboard_X`, 하단 `BC_billboard_rectangle`, `BC_byline_rectangle` 등 가로 영역에 적용
- **브레이크포인트 개념**
  - `1024px 이상 (데스크탑)`  
    - 1200×300 / 1200×100 / 970×250 / 980×120 / 970×90 / 930×180 / 728×90 등  
    - 빌보드·리더보드·와이드형 고단가 인벤토리 위주
  - `768~1023px (태블릿)`  
    - 750×300 / 750×200 / 728×90 + 480×320·336×280·300×250  
    - 750폭·728폭 중심 + 중형 직사각형 혼합
  - `0~767px (모바일)`  
    - 320×100 / 320×50 / 320×480 / 300×250 / 336×280 등  
    - 세로 스크롤 환경에 맞춘 모바일 전용 배너·박스형 중심
- **템플릿별 세부 차이 (참고)**
  - View GPT의 `sizeMapping_horizontal`에는 데스크탑 구간에  
    750×300, 750×200 등이 추가되어 있어, 하단 빌보드를 좀 더 유연하게 수용합니다.
  - HOME / Section / List GPT는 위 코드와 거의 동일한 구조를 사용하되,  
    일부 사이즈(예: 320×50, 250×250, 200×200)는 템플릿별로 포함/제외될 수 있습니다.

---

#### 4.5.2 박스형 / 사이드 레일 – MR 우측 레일 공통 적용

```js
const sizeMapping_siderail_square = googletag.sizeMapping()
    .addSize([1024, 0], [
        [336, 280], [320, 480], [300, 600], [300, 250], 
        [250, 250], [200, 200],
        [2, 1], [1, 1], 
        'fluid'
    ])
    .addSize([768, 0], [
        [750, 300], [750, 200], [728, 90], 
        [480, 320], 
        [336, 280],[320, 480],
        [320, 100], [320, 50],
        [300, 250], [250, 250], [200, 200],
        [2, 1], [1, 1], 
        'fluid'
    ])
    .addSize([0, 0], [
        [336, 280], [320, 480],
        [320, 100], [320, 50],
        [300, 250], [250, 250], [200, 200],
        [2, 1], [1, 1], 
        'fluid'
    ])
    .build();
```

- **용도**
  - HOME GPT  
    - `MR_halfpage_rectangle`에 사용 
  - Section GPT / List GPT  
    - 우측 `MR_halfpage_rectangle`(사이드 레일 광고)에 동일 정책 적용
  - View GPT  
    - 우측 레일 `MR_side_rail_X`에 동일 적용

- **디자인/레이아웃 전제**
  - HOME / Section / List / View 공통 CSS에서  
    사이드바/우측 레일 폭은 **336px**로 고정 또는 최소 보장:
    - HOME: `.sidebar { flex: 0 0 336px; min-width: 3336px; }`
    - View: `.related-area { flex: 0 0 336px; min-width: 336px; }`
  - 이 폭 정책을 전제로 300×600, 300×250, 336×280, 320x480 등이 안정적으로 노출되도록 설계

---

#### 4.5.3 In-Article 혼합형 – View GPT in-article 전용 (sizeMapping_article_horizontal)

```js
const sizeMapping_article_horizontal = googletag.sizeMapping()
    .addSize([1024, 0], [
        [480, 320], [336, 280],[320,480], 
        [320, 100], [320, 50],
        [300, 250],
        // 300x25x 시리즈: 라인아이템/캠페인 분리를 위한 미세 구분용 사이즈
        [300, 251], [300, 252], [300, 253], [300, 254], [300, 255], [300, 256], [300, 257], [300, 258],
        [250, 250],[200, 200],
        [2, 1], [1, 1], 'fluid'
    ])
    .addSize([768, 0], [
        [480, 320], [336, 280], [320,480], 
        [320, 100], [320, 50],
        [300, 250],
        [300, 251], [300, 252], [300, 253], [300, 254], [300, 255], [300, 256], [300, 257], [300, 258],
        [250, 250], [200, 200],
        [2, 1], [1, 1], 'fluid'
    ])
    .addSize([0, 0], [
        [336, 280], [320, 480],
        [320, 100], [320, 50],
        [300, 250],
        [300, 251], [300, 252], [300, 253], [300, 254], [300, 255], [300, 256], [300, 257], [300, 258],
        [250, 250], [200, 200],
        [2, 1], [1, 1], 'fluid'
    ])
    .build();
```

- **용도 (View GPT 전용)**
  - `MC_article_rectangle_1` ~ `MC_article_rectangle_8`  
    - 모두 `sizeMapping_article_horizontal`을 사용
    - 각 슬롯은 300×251~258 등 **서로 다른 300×25x 사이즈**를 하나씩 포함해  
      **라인아이템/캠페인 단위 구분에 활용 가능**
- **특징**
  - **In-article 전용 혼합형 매핑**
    - 300×250 / 336×280 / 480×320 / 320x480 같은 전형적인 중형 배너 +  
      300×25x(251~258) 변형 사이즈를 함께 지원
    - 텍스트 환경/뷰포트에 따라 다양한 비율의 중형 바디 배너 집행 가능
  - **3단계 브레이크포인트**
    - 데스크탑 / 태블릿 / 모바일 별로 동일 계열 사이즈를 유지하여  
      크리에이티브 재사용성을 높이면서, 기기별 레이아웃 최적화
- **연결 로직**
  - View GPT에서는 `setupInArticleAds()`에서:
    - 기사 본문 텍스트 높이·시각 블록(이미지/Embed/데이터박스 등)을 기준으로  
      **MC in-article 슬롯 위치를 동적으로 재배치**
    - 사용되지 않는 슬롯은 DOM + GAM 슬롯에서 `destroySlots()`로 제거  
      → 실제 노출되는 MC in-article 슬롯만 위 sizeMapping을 사용해 요청  

> ⚠️ 사이즈 변경 시
> - GAM 인벤토리 정의와 반드시 맞춰야 함
> - `fluid` 사용 여부도 인벤토리 설정과 동기화 필요

---

### 4.6 슬롯 정의 패턴 및 슬롯 레벨 타게팅

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
  - DOM ID와 동일 (`<div id="TC_billboard_X">` ↔ `'TC_billboard_X'`) 관리 편의상 slot_id, div_id 동일하게 유지
- **position**
  - `TC(Top Center) / MC(Main Content) / MR(Right Rail) / BC(Below Content)` 광고 위치 카테고리라이징
- **ad_area_type**
  - 예: `ATF`, `in_article`, `below_byline`, `below_comments`, `below_popular_news` 등 광고의 세부 위치
- **default_exposure**
  - 향후 기본 노출/비노출 정책/사고방지 설정용
- **load_type**
  - `'eager'` / `'lazy'` 구분 → Lazy 인프라 로직에서 참조 가능 (필요 시 활용)

---

### 4.7 GAM 전역 설정 및 로딩전략

```js
googletag.pubads().enableSingleRequest();
googletag.pubads().disableInitialLoad();  
googletag.setConfig({
  centering: true,
  collapseDiv: 'BEFORE_FETCH'
});
googletag.enableServices();
```

- `collapseEmptyDivs(true)` 대신, 최신 기준에서는  
  `setConfig({ collapseDiv: 'BEFORE_FETCH' })`를 사용 (코드 기준)
- Lazy 슬롯은 반드시 IntersectionObserver에서 `display + refresh` 호출 필요

---

### 4.8 광고 로딩 방식: Eager vs Lazy

#### 4.8.1 Eager Load
- ATF, 화면상단 바로 광고 노출
- 사용 위치
  - View: `TC_billboard_X` (데스크탑 전용 상단 ATF)
  - HOME: `TC_billboard_leaderboard`
  - Section/List: `TC_billboard_leaderboard`

- 호출 패턴 예 (View):

```js
googletag.cmd.push(function () {
  if (TC_billboard_X_slot) {
    googletag.display('TC_billboard_X');
    googletag.pubads().refresh([TC_billboard_X_slot]);
  }
});
```

#### 4.8.2 Lazy Load
- BTF, 화면하단, 사용자 뷰포트가 광고에 가까워지는 지점에서 광고 호출 시작
- 사용 위치
  - View: `MR_side_rail_X`, `MC_article_rectangle_1~8`, `BC_byline_rectangle`, `BC_billboard_rectangle`, `BC_native_list_1/2`
  - HOME: `MC_billboard_rectangle`, `MR_halfpage_rectangle`
  - Section/List: `MR_halfpage_rectangle`

- 공통 변수 구조

```js
const LAZY_AD_SLOT_IDS = [
  // 페이지별로 필요한 슬롯 ID 세팅
];

let lazyAdSlotsHTML = [];
let lazyGamSlots = [];
let lazyObserver = null;
```

- Lazy 흐름 요약
  1. `.ad-slot` 및 `.native-ad-list-item` 중 Lazy 대상 ID 수집
  2. `googletag.pubads().getSlots()`로 GAM Slot 객체 필터링
  3. `IntersectionObserver`에서 **뷰포트 근접** 시점에 콜백 실행
  4. 콜백 내부에서 해당 슬롯에 대해 `googletag.display(id) + refresh([slot])`
  5. `observerInstance.unobserve(entry.target)`로 한 번만 실행

- 기본 옵션 예시

### 기존
```js
const options = {
  root: null,
  rootMargin: '0px 0px 100px 0px',
  threshold: 0.0
};
```
### 최적화 변경 (2026-01-05)
            /**
             * [객관적 최적화: 디바이스별 rootMargin 동적 할당]
             * - 한국의 안정적인 네트워크 인프라와 사용자 스크롤 속도를 고려한 '스윗 스팟' 수치 적용
             * - Smartphone: 600px (빠른 플릭 스크롤 대응)
             * - Tablet: 500px (넓은 화면비 고려)
             * - Desktop: 400px (글로벌 미디어 표준값)
             */
            // --- IntersectionObserver 옵션 설정 (디바이스별 rootMargin 최적화) ---

            const ua = navigator.userAgent.toLowerCase();
            const width = window.innerWidth;
            let marginValue = '400px'; // Desktop 기본값

            // 1. Smartphone 판별 로직 적용 (Mobile UA이면서 iPad가 아니거나, 폭이 768px 이하)
            if ((/mobile|android|iphone|ipod/.test(ua) && !/ipad/.test(ua)) || width <= 768) {
                marginValue = '600px'; // Smartphone: 모바일의 빠른 스크롤을 고려하여 가장 넓은 여백 설정
            } 
            // 2. Tablet 판별 로직 적용 (iPad/Tablet UA이거나, 폭이 1024px 이하)
            else if (/ipad|tablet/.test(ua) || width <= 1024) {
                marginValue = '500px'; // Tablet: 중간 정도의 여백 설정
            }
            // 3. 그 외: Desktop (기본값 400px 유지)
            // IntersectionObserver 생성
            /*
             * - root      : null → 브라우저 viewport 기준
             * - rootMargin: 동적으로 할당된 marginValue 적용
             * → 실제 뷰포트 하단보다 지정된 픽셀만큼 앞서 미리 로딩 시작(프리로딩 효과)
             * - threshold : 0.0
             * → 요소가 1px만 보이기 시작해도 isIntersecting === true
             */
            const options = {
                root: null,
                rootMargin: `0px 0px ${marginValue} 0px`, // 하단 여백을 디바이스별로 다르게 적용
                threshold: 0.0
            };

> 💡 **튜닝 포인트**
> - `rootMargin` / `threshold`는 **뷰어블 성과 & UX**를 보면서 조정 가능
> - 본 레이지로드 오프셋을 100px로 설정, 최적화에따라 변경 가능
---

### 4.9 Advertisement 라벨 구현 방식

#### 4.9.1 GAM 이벤트 리스너

```js
function addAdLabel(event) {
  const slotId = event.slot.getSlotElementId();
  const slotElement = document.getElementById(slotId);

  // View GPT 전용 예외: 상단 TC_billboard_leaderboard, BC_native_list_1~2 등 광고는 라벨 제외
  if (slotId === 'BC_native_list_1' || slotId === 'BC_native_list_2') {
    return;
  }

  if (event.isEmpty === false && slotElement) {
    const outerSlot = slotElement.closest('.ad-slot');
    if (!outerSlot) return;
    if (outerSlot.querySelector('.mk-ad-label')) return; // 중복 방지

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
- **상단 리더보드 (TC_billboard_leaderboard)**는 메뉴와의 공백 등 UI 고려하여   
   **별도 AD 라벨 없이** 간격/형태로만 광고임을 구분

- **Native 리스트(BC_native_list_1/2)**는 기사 리스트와 섞여 노출되는 특성상  
  리스트 내에서 **별도 AD 라벨 없이** 간격/형태로만 광고임을 구분

#### 4.9.2 공통 CSS (.mk-ad-label)

```css
.mk-ad-label {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
  padding: 0;
  background-color: transparent;
  color: #5A5A5A;
  font-size: 13px;
  font-weight: normal;
  text-transform: uppercase;
  white-space: nowrap;
}
```

- View 다크 모드에서는 아래처럼 변경:

```css
@media (prefers-color-scheme: dark) {
  .mk-ad-label {
    background-color: rgba(0, 0, 0, 0.7);
    color: #f5f5f5;
    box-shadow: 0 0 2px rgba(0, 0, 0, 0.8);
  }
}
```

---

### 4.10 .ad-slot 공통 CSS 정책

```css
.ad-slot {
  background-color: transparent;
  color: transparent;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  box-sizing: border-box;
  border: none;
  width: 100%;
  height: auto;
  position: relative;
  z-index: 10;
  padding-top: 10px; /* 라벨/상단 요소 간 최소 간격 */
}
```

- HOME/Section/List에서는 동일 구조에 **padding, max-width** 정도만 미세 조정

> 페이지별 TC/MR 등의 슬롯은 필요 시 ID 기반 CSS로 개별 오버라이드

---

## 5. View GPT 구현 가이드 (기사 View 페이지)

### 5.1 역할 개요

- 기사 본문 중심의 **가장 복잡한 템플릿**
- 포함 요소
  - 본문 in-article 광고(최대 8개)
  - 데스크탑 우측 MR 레일
  - Byline/댓글 하단 BC 광고
  - “이 시각 주요 뉴스” 영역 내 Native 리스트 광고

### 5.2 AdUnitPath 및 페이지 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/news/economy';

googletag.pubads().setTargeting('page_type', 'view');
googletag.pubads().setTargeting('section_front_nm', 'economy');
googletag.pubads().setTargeting('section_home_nm', '');

// section_list_nm 등은 '동적입력' placeholder
```

### 5.3 슬롯 구성 요약

- **Eager**
  - `TC_billboard_X` (데스크탑 전용 상단 ATF)
- **Lazy**
  - `MR_side_rail_X` (데스크탑 우측 레일)
  - `MC_article_rectangle_1` ~ `MC_article_rectangle_8` (본문 in-article)
  - `BC_byline_rectangle` (기자명 하단)
  - `BC_billboard_rectangle` (댓글 하단)
  - `BC_native_list_1`, `BC_native_list_2` (이 시각 주요 뉴스 내 네이티브, 모바일/태블릿 전용)

### 5.4 DOM 구조 및 ID 체계 (View GPT)

- 상위 레이아웃

```text
.container
 └─ .content-wrap (flex)
     ├─ .article-area
     │   └─ .article-body
     │       ├─ <p> / .article-image / .article-embed / ...
     │       └─ in-article slots (#MC_article_rectangle_1~8)
     └─ .related-area
         ├─ #MR_side_rail_X (우측 레일)
         └─ .related-list (이 시각 주요 뉴스)
 ├─ li.article-item (뉴스)
 ├─ li#BC_native_list_1.native-ad-list-item
 └─ li#BC_native_list_2.native-ad-list-item
```

- Byline/댓글 하단 BC

```html
<div id="BC_byline_rectangle" class="ad-slot ad-bc"></div>
<div id="BC_billboard_rectangle" class="ad-slot ad-bc"></div>
```

### 5.5 In-Article 동적 삽입 로직 (매우중요)

#### 5.5.1 실행 시점

```js
document.addEventListener('DOMContentLoaded', function () {
  setupInArticleAds();           // 본문 in-article 동적 삽입
  setupLazyLoading();// Lazy 인프라
  shufflePopularNewsNativeAds(); // 이 시각 주요 뉴스 내 네이티브 랜덤 배치
  startObservingLazySlots();     // IntersectionObserver 시작
});
```

#### 5.5.2 본문 블록 탐색

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

- 각 블록에 대해
  - 시각 블록 여부(`isVisual`)
  - 텍스트 높이(`textHeight`)
  - 누적 텍스트 높이(`cumulativeTextHeight`)를 계산하여 앵커 리스트 생성

#### 5.5.3 짧은 기사 처리

```js
const totalTextHeight = anchors.length > 0
  ? anchors[anchors.length - 1].cumulativeTextHeight
  : 0;

const isShortArticle = totalTextHeight < viewportHeight;

if (isShortArticle && lastContentBlock) {
  // MC_article_rectangle_1만 마지막 블록 뒤에 삽입
  // 나머지 MC 슬롯(2~8)은 DOM 제거 + googletag.destroySlots()
}
```

- 짧은 기사에서는 in-article 1개만 사용해 **과도한 광고 밀집 방지**

#### 5.5.4 일반 기사 삽입 규칙

- 파라미터
  - `VIEWPORT_GAP_BETWEEN_ADS = 1.0`
  - `VIEWPORT_AFTER_VISUAL_MIN = 0.5`

- 조건
  1. 광고 간 거리 ≥ `1.0 * viewportHeight`
  2. 마지막 시각 블록 이후 거리 ≥ `0.5 * viewportHeight`
  3. 인접 전/후 형제가 시각 블록이면 스킵

- 삽입 예시

```js
const slotId = MC_IN_ARTICLE_SLOT_IDS[slotIndex];
const slotEl = document.getElementById(slotId);

articleBody.insertBefore(slotEl, anchor.element);
placedSlotIds.add(slotId);
lastAdTextHeight = anchor.cumulativeTextHeight;
slotIndex++;
```

- 사용되지 않은 MC 슬롯은 DOM에서 제거 + `destroySlots` 호출

> ✅ 기사 구조/길이 변화에도 자동으로 in-article 수·위치가 조정되는 구조

### 5.6 “이 시각 주요 뉴스” 내 네이티브 랜덤 배치

```js
function shufflePopularNewsNativeAds() {
  const popularList = document.querySelector('.related-list');
  if (!popularList) return;

  const allItems = Array.from(popularList.children);
  const isAd = (item) => item.id && item.id.startsWith('BC_native');

  const articleItems = allItems.filter(item => !isAd(item));
  const adItems = allItems.filter(item => isAd(item));

  const firstArticle = articleItems.shift(); // 1위 뉴스는 고정
  const remainingItems = [...articleItems, ...adItems];

  // Fisher–Yates shuffle on remainingItems
  // ...
}
```

- 정책
  - 1위 뉴스는 항상 기사
  - 나머지 기사와 네이티브 광고(li.native-ad-list-item)는 섞어서 노출

- CSS

```css
.related-list {
  counter-reset: hotNewsRank;
}
.related-list li.article-item::before {
  counter-increment: hotNewsRank;
  content: counter(hotNewsRank);
}
.related-list .native-ad-list-item::before {
  counter-increment: none;
  content: "";
}
```

- Native 광고가 들어가도 기사 번호 1~N은 연속 유지

### 5.7 Lazy Load 구성 (View GPT)

```js
const LAZY_AD_SLOT_IDS = [
  'MR_side_rail_X',
  'MC_article_rectangle_1',
  'MC_article_rectangle_2',
  // ...
  'MC_article_rectangle_8',
  'BC_byline_rectangle',
  'BC_billboard_rectangle',
  'BC_native_list_1',
  'BC_native_list_2'
];
```

- `setupLazyLoading()`에서 위 ID와 DOM/GAM Slot을 매핑
- `startObservingLazySlots()`로 IntersectionObserver 시작

---

## 6. HOME GPT 구현 가이드 (HOME 페이지)

### 6.1 역할 개요

- 매경 **메인 홈(포털)** 역할
- 헤드라인/섹션 요약 + 대표 광고 슬롯 구성

### 6.2 AdUnitPath 및 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/HOME';

googletag.pubads().setTargeting('page_type', 'HOME');
googletag.pubads().setTargeting('section_home_nm', 'MK');
googletag.pubads().setTargeting('section_front_nm', '동적입력');
```

### 6.3 슬롯 구성

- **Eager**
  - `TC_billboard_leaderboard`
- **Lazy**
  - `MC_billboard_rectangle`
  - `MR_halfpage_rectangle`

### 6.4 특징

- in-article 로직 없음
- Lazy 스크롤 트리거 지점이 비교적 상단에 있어 **ATF/1스크롤 내 주요 인벤토리** 로딩 완료
- View와 동일한 Lazy 인프라를 재사용(슬롯 ID만 다름)

---

## 7. Section GPT 구현 가이드 (섹션 프론트)

### 7.1 AdUnitPath 및 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/news/economy/section_front';

googletag.pubads().setTargeting('page_type', 'section_front');
googletag.pubads().setTargeting('section_front_nm', 'economy');
googletag.pubads().setTargeting('section_list_nm', '동적입력');
```

### 7.2 슬롯 구성

- **Eager**
  - `TC_billboard_leaderboard`
- **Lazy**
  - `MR_halfpage_rectangle`

### 7.3 특징

- HOME GPT 레이아웃을 기반으로 섹션 콘텐츠만 교체한 구조
- Lazy 인프라 동일 (`LAZY_AD_SLOT_IDS = ['MR_halfpage_rectangle']`)
- 향후 MC/BC 슬롯 추가 시,
  - DOM에 ID 추가 → `defineSlot` → `LAZY_AD_SLOT_IDS`에 ID 추가하면 재사용 가능

---

## 8. List GPT 구현 가이드 (섹션 리스트)

### 8.1 AdUnitPath 및 타게팅

```js
const AdUnitPath = '/7450/www.mk.co.kr/news/economy/section_list';

googletag.pubads().setTargeting('page_type', 'section_list');
googletag.pubads().setTargeting('section_front_nm', '');
googletag.pubads().setTargeting('section_list_nm', 'economic-policy');
```

- Section GPT와 동일한 구조이나, 리스트 페이지에 맞게 `page_type`과 `section_list_nm` 중심 타게팅

### 8.2 슬롯 구성 및 Lazy 구조

- Section GPT와 동일
  - **Eager**: `TC_billboard_leaderboard`
  - **Lazy**: `MR_halfpage_rectangle`
- Lazy 인프라 로직은 완전히 동일

> 사실상 “Section GPT + 리스트용 page_type/타게팅” 버전으로 이해 가능

---

## 9. 확장 및 커스터마이징 가이드

### 9.1 새 페이지 타입 추가 체크리스트

1. **AdUnitPath 정의**
   - `/7450/www.mk.co.kr/...` 뒤 경로 설계
2. **page_type · section_*_nm 설계**
   - 리포트/타게팅에서 어떤 축으로 보고 싶은지 먼저 정의
3. **공통 페이지 타게팅 주입**
   - `device_category, web_type, gpt_version, domain, url, referer` 등
4. **슬롯 구조 설계**
   - TC / MC / MR / BC 구분, in-article 여부
   - Eager vs Lazy 구분
5. **사이즈 매핑 최종 확인**
  - 가로형 : `sizeMapping_horizontal` : 'TC_billboard_leaderboard', 'TC_billboard_X', 'BC_byline_rectangle' 등 수평 가로형 광고 슬롯 정의
  - 세로 박스형 / 사이드 레일 :  `sizeMapping_siderail_square` : 'MR_halfpage_rectangle' 우측 사이드레일 세로 박스형/세로형 슬롯 정의
  - 본문 박스형 : `sizeMapping_article_horizontal` : 'MC_article_rectangle_1~8' 본문 박스형 슬롯

6. **Lazy 인프라 적용**
   - `LAZY_AD_SLOT_IDS`에 Lazy 대상 ID 추가
   - 별도 로직이 없다면 `setupLazyLoading()` / `startObservingLazySlots()` 그대로 재사용

### 9.2 신규 슬롯 추가 시 주의사항

- DOM
  - `<div id="NEW_SLOT_ID" class="ad-slot ..."></div>`
- GPT 정의
  - `googletag.defineSlot(AdUnitPath, [...sizes...], 'NEW_SLOT_ID')...`
- 타게팅
  - `div_id`, `position`, `ad_area_type`, `load_type` 설정
- Lazy 여부
  - Lazy 대상이면 `LAZY_AD_SLOT_IDS` 배열에 `'NEW_SLOT_ID'` 추가 필수

---

## 10. QA 및 검증 가이드

### 10.1 Google Publisher Console

- 테스트 URL 뒤에 `?google_console=1` 또는 `#google_console` 추가
- 확인 항목
  - 각 슬롯의 **AdUnitPath, 사이즈, key-values**
  - `device_category`, `page_type`, `section_*` 값이 의도대로 세팅되었는지
  - Eager/Lazy 슬롯이 모두 단일 SRA 요청에 올바르게 포함되는지

### 10.2 Network 패널

- `securepubads.g.doubleclick.net` 요청 타이밍 점검
  - 초기 로딩 시 Eager 슬롯만 요청되는지
  - 스크롤 후 Lazy 슬롯이 순서대로 요청되는지
- `[Lazy Load SUCCESS] Ad ID: ...` 로그와 Network 요청이 매칭되는지 확인

### 10.3 브레이크포인트 테스트

- DevTools 디바이스 모드에서 320 / 768 / 1024 px 기준 테스트
  - `sizeMapping`에 따라 어떤 사이즈가 선택되는지 확인
  - 사이드바/본문 레이아웃이 깨지지 않는지 점검

### 10.4 레이아웃 검증

- `collapseDiv: 'BEFORE_FETCH'` 적용 상태에서
  - 빈 슬롯이 레이아웃을 깨지 않는지 확인
- `.ad-slot`의 `padding`, 주변 요소와의 간격, CLS(레이아웃 이동) 체크

### 10.5 View in-article 및 Native 검증

- 다양한 길이의 기사로 테스트
  - 짧은 기사: MC in-article 1개만 사용되고 나머지는 destroy 되는지
  - 긴 기사: 광고 간 간격이 대략 1 viewport 이상인지
  - 이미지/YouTube/데이터박스 바로 위·아래에 광고가 붙지 않는지
- Native 리스트
  - 데스크탑: `BC_native_list_1/2`는 DOM에는 있으나 `display:none`인지
  - 모바일/태블릿: Lazy Load 후 정상 노출되는지, 라벨이 붙지 않는지 확인

### 10.6 Dark Mode & PPID

- OS 다크 모드 전환 후
  - `is_dark_mode` 타게팅 값이 `true`로 들어오는지
  - View 다크 모드에서 `.mk-ad-label` 스타일이 변경되는지 확인
- PPID
  - 실제 운영 환경에서는 `setPublisherProvidedId`에  
    서버에서 해시한 1st-party ID가 세팅되는지 QA 필요

---

## 11. 향후 개선 아이디어 (Developer Notes)

이 섹션은 현재 코드에는 없거나 제한적으로만 구현된, **향후 개선 아이디어**입니다.

1. **Slot-level visibility 기반 추가 Lazy 제어**
   - 현재는 뷰포트 근접 기준(rootMargin)만 사용
   - 향후 viewability 데이터 기반으로 **성과 낮은 하단 슬롯**은 더 늦게 로딩 또는 비활성화하는 전략 가능

2. **data-googletag-enabled 속성 활용**
   - View GPT 일부 슬롯에 `data-googletag-enabled="false"` 속성이 있으나 현재 JS에서 사용하지 않음
   - 템플릿 단에서 이 속성으로 **슬롯 활성/비활성**을 제어하는 확장 가능

3. **공통 유틸 JS 분리**
   - `setupLazyLoading`, `startObservingLazySlots`, `addAdLabel` 등은 4개 템플릿에서 공통
   - `/js/gam-common.js` 형태로 모듈화하여 **중복 제거 + 유지보수성 향상**

4. **로그 레벨 관리**
   - 현재 `console.log`가 풍부하게 들어가 있으므로
   - 운영 배포 시에는 `DEBUG` 플래그 기반 on/off 구조 도입 권장

5. **뷰포트 기준 값 튜닝**
   - in-article:
     - `VIEWPORT_GAP_BETWEEN_ADS = 1.0`
     - `VIEWPORT_AFTER_VISUAL_MIN = 0.5`
   - Lazy Load:
     - `rootMargin: '0px 0px 100px 0px'`
   - 실제 스크롤 깊이/뷰어블 데이터를 바탕으로 **실험·튜닝 가능**

---

## 12. 마무리

향후 다음과 같은 방향으로 추가 개발 및 협의, 업데이트 예정입니다.

- 효율적 유지보수를 위해 **공통 광고 모듈(.js), 스타일(.css), HTML 템플릿** 분리
- 1st-party 데이터 강화 및 신규 타겟팅 발굴
- PPID 고도화 및 Audience 전략 정교화
- Prebid/오픈빗 통합, **헤더비딩 연동 규칙** 추가를 통한 수익화 극대화

실제 구현/개발 과정에서 막히는 부분,  
혹은 기획 단계에서 추가 정의가 필요한 부분이 있을 경우에는

**매일경제 광고마케팅국 디지털마케팅부 황재연**에게  
언제든 편하게 논의 요청 부탁드립니다.
