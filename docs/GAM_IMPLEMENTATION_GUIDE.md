0. 작업 목적·의의·구현 방식 요약
0-1. 작업 목적

페이지 유형별(GPT 4종)로 표준화된 GAM 템플릿을 구축

View(기사뷰), HOME(홈), Section(섹션 메인), List(섹션 리스트) 페이지에서
동일한 설계 철학으로 광고를 운용할 수 있도록 구조를 맞추는 것이 1차 목표입니다.

반응형·Lazy Loading·동적 in-article 삽입 등의 고급 기능을 공통 프레임으로 제공

디바이스별(Desktop/Tablet/Mobile) 대응

Eager/Lazy 로딩 전략

기사 본문 내 in-article 동적 삽입(특히 View GPT)
→ 이 로직을 재사용 가능한 형태로 정리해, 향후 다른 페이지(예: 특집, 이벤트 페이지)에도 쉽게 확장할 수 있게 하는 목적입니다.

광고/콘텐츠/UX 간 밸런스 최적화

무리한 광고 삽입으로 이탈을 유발하지 않으면서도,

Viewability·Coverage·Request 효율을 최대화하는 방향으로 설계되어 있습니다.

0-2. 구현 방식 개요

모든 템플릿에서 동일한 기본 패턴을 사용합니다.

gpt.js 비동기 로더 + window.googletag = { cmd: [] } 패턴

googletag.cmd.push(function(){ ... }); 안에서

페이지 레벨 타게팅 설정

googletag.defineSlot(...) + defineSizeMapping(...)

공통 GAM 옵션(disableInitialLoad, enableSingleRequest, collapseEmptyDivs 등)

이벤트 리스너(slotRenderEnded → ‘Advertisement’ 라벨 삽입)

로딩 전략

공통적으로 disableInitialLoad()를 사용하여
Eager 슬롯은 명시적 display + refresh, Lazy 슬롯은 IntersectionObserver에서 display + refresh 하는 구조입니다.

반응형 구조

googletag.sizeMapping()을 사용하여

Desktop(≥1024px) / Tablet(≥768px) / Mobile(<768px) 별로 허용 광고 사이즈를 정의.

동적 삽입 (View GPT)

DOMContentLoaded 시점에 본문 DOM을 스캔 →
사전에 정의한 “문단/영역 기준” 규칙에 따라 in-article 광고용 컨테이너를 동적으로 삽입 →
해당 컨테이너를 Lazy 슬롯으로 등록 후 IntersectionObserver로 노출 시점에 로드.

1. 이번 작업의 수익화 전략 요약
1-1. 반응형(Responsive) 전략의 수익 영향

수익 측면 포인트

동일 인벤토리가 다양한 사이즈를 허용함으로써

단일 페이지에서도 리치 미디어·대형 배너·스탠다드 배너 등 여러 상품군의 입찰 기회를 확보.

모바일/태블릿/데스크톱별로 알맞은 사이즈를 노출하여 Viewability·CTR 개선 → eCPM 상승 기대.

구현 관점

sizeMapping_horizontal, sizeMapping_square 등 공통 정의 사용.

다양한 크기([1200x300], [970x250], [728x90], [336x280], [300x250], [300x600], 'fluid', [1,1] 등)를 묶어 운영.

1-2. Lazy Loading 전략의 수익 영향

수익 측면 포인트

*아래쪽 인벤토리(특히 MR, in-article)*에 대해,

실제 사용자가 스크롤하여 뷰포트 근접 시점에만 노출 요청 →
불필요한 비가시 영역 노출/요청 감소 →
Viewable Impressions 비율 상승 → 뷰어빌리티 기준을 중시하는 브랜드/프로그래머틱 딜에서 경쟁력 증가.

페이지당 호출 수는 약간 줄더라도,
“요청 대비 수익(eCPM)”과 “브랜드 호감도(UX)” 관점에서 유리한 구조.

구현 관점

googletag.pubads().disableInitialLoad() + IntersectionObserver 사용.

Lazy 대상 ID를 LAZY_AD_SLOT_IDS 배열로 관리 → 공통 로직으로 처리.

1-3. View 페이지 in-article 동적 삽입의 수익 영향

수익 측면 포인트

기사 본문 중간 중간에 in-article 광고를 동적으로 삽입함으로써:

본문 체류 시간이 길고 스크롤이 발생하는 View 페이지에서

추가 인벤토리 확보 (기사 1건당 1~N개 in-article 가능)

본문 맥락과 가까운 지점에 광고 노출 → CTR·Viewability 높음.

단순히 상단/우측만 판매하는 구조보다,
**기사당 평균 광고 노출 수(Inventory Depth)**를 크게 높일 수 있습니다.

구현 관점

DOM 구조(문단/블럭)를 기반으로 스크롤/뷰포트 노출 패턴을 고려해 “삽입 위치 규칙”을 정의.

IntersectionObserver로 실제 읽는 위치에 도달했을 때만 광고 요청.

2. 공통 GAM 구현 가이드 (4종 GPT 공통)
2-1. AdUnitPath 설계 가이드

기본 개념

AdUnitPath는 GAM 콘솔의 인벤토리 구조와 1:1 매핑되는 경로입니다.

예시

HOME : /7450/www.mk.co.kr/HOME

Section Front : /7450/www.mk.co.kr/news/economy/section_front

Section List : /7450/www.mk.co.kr/news/economy/section_list

View : /7450/www.mk.co.kr/news/economy/view (또는 /article/... 구조 등)

동적 생성 전략(실서비스)

공통 규칙

/{networkCode}/{domain}/{service}/{section}/{pageType}

CMS/백엔드에서

service = news / premium / sports …

section = economy / realestate / stock …

pageType = HOME / section_front / section_list / view 등
으로 조합해 AdUnitPath를 서버에서 주입해주시면 됩니다.

권장:

각 GPT 템플릿에서는 상수처럼 보이지만,
실제 배포 시에는 서버 사이드에서 AdUnitPath와 고정 타게팅 값을 주입받도록 설계(템플릿화).

2-2. GPT 로더 & googletag.cmd 패턴

사용 스크립트

https://securepubads.g.doubleclick.net/tag/js/gpt.js

기본 패턴

<script async src=".../gpt.js"></script>
<script>
  window.googletag = window.googletag || { cmd: [] };

  googletag.cmd.push(function() {
    // 1) 페이지 레벨 타게팅
    // 2) defineSlot / sizeMapping
    // 3) pubads 옵션 설정
    // 4) enableServices
    // 5) eager 슬롯 display + refresh
  });
</script>


핵심 메소드 설명

googletag.defineSlot(adUnitPath, sizeArray, divId)

특정 DOM ID(divId)에 연결된 광고 슬롯을 정의합니다.

.addService(googletag.pubads())

해당 슬롯을 Publisher Ads 서비스에 등록.

googletag.pubads()

페이지 전체의 광고 설정(타게팅, 옵션 등)을 관리하는 객체입니다.

googletag.cmd.push(function(){...});

gpt.js가 로드된 이후에 실행되어야 하는 모든 로직을 큐에 쌓는 표준 방식입니다.

googletag.display(divId)

해당 div에 연결된 슬롯을 활성화(요청 준비).

googletag.pubads().refresh([slot])

실제 광고 요청을 트리거합니다.

disableInitialLoad() 사용 시 반드시 display + refresh를 명시적으로 호출해야 요청이 발생합니다.

2-3. GAM 핵심 기능 옵션

googletag.pubads().disableInitialLoad();

초기 자동 로드 비활성화

장점

Eager / Lazy / 동적 삽입 등 광고 요청 타이밍을 전적으로 우리가 통제할 수 있습니다.

스크롤 기반 Lazy 로딩 구현에 필수.

googletag.pubads().enableSingleRequest();

SRA(Single Request Architecture) 활성화

여러 슬롯에 대한 광고 요청을 한 번의 네트워크 콜로 묶어

레이턴시를 줄이고,

전체 페이지에서 라인아이템 경쟁을 최적화합니다.

googletag.pubads().collapseEmptyDivs(true);

광고가 매칭되지 않은 슬롯의 div를 자동으로 접어서 레이아웃 공백을 줄입니다.

단, 레이아웃 안정성을 위해

CSS min-height를 활용해 “자리만 잡아두고 광고 없을 때는 실제 높이는 줄이는” 식으로 조정 가능합니다.

(HOME에서 사용) googletag.pubads().setCentering(true);

크기가 다양한 크리에이티브를 슬롯 중앙에 정렬하여 UX를 개선합니다.

2-4. 디바이스 카테고리 & 반응형 사이즈 매핑
(1) 디바이스 카테고리 결정 로직
const userAgent = navigator.userAgent.toLowerCase();
let deviceCategory = 'desktop';

if (/mobile|android|iphone|ipad|ipod/.test(userAgent) || window.innerWidth <= 768) {
    deviceCategory = 'mobile';
} else if (window.innerWidth <= 1024) {
    deviceCategory = 'tablet';
}

googletag.pubads().setTargeting('device_category', deviceCategory);


역할

타게팅·리포팅 축으로 사용 (예: 디바이스별 eCPM, 뷰어빌리티, CTR 비교).

구현 포인트

엄밀한 디바이스 구분은 아니지만,

media query와 sizeMapping 브레이크포인트(768, 1024)와 동일 기준을 사용해
분석/운영에서 헷갈리지 않도록 맞춰두었습니다.

(2) 반응형 sizeMapping 공통 패턴

가로형 배너 (TC/MC 등):

const sizeMapping_horizontal = googletag.sizeMapping()
  .addSize([1024, 0], [[1200, 300], [1200, 100], [970, 250], [980, 120], [970, 90], [930, 180], [728, 90], [2, 1], [1, 1], 'fluid'])
  .addSize([768, 0], [[750, 300], [750, 200], [728, 90], [480, 320], [336, 280], [300, 250], [2, 1], [1, 1], 'fluid'])
  .addSize([0, 0],   [[336, 280], [300, 250], [320, 480], [320, 100], [2, 1], [1, 1], 'fluid'])
  .build();


박스형 배너 (MR 등):

const sizeMapping_square = googletag.sizeMapping()
  .addSize([1024, 0], [[300, 600], [336, 280], [2, 1], [1, 1], 'fluid'])
  .addSize([768, 0],  [[336, 280], [300, 250], [2, 1], [1, 1], 'fluid'])
  .addSize([0, 0],    [[336, 280], [300, 250], [320, 480], [320, 100], [2, 1], [1, 1], 'fluid'])
  .build();


슬롯 적용 예시

googletag.defineSlot(AdUnitPath, [...], 'TC_billboard_leaderboard')
  .defineSizeMapping(sizeMapping_horizontal)
  .addService(googletag.pubads());

2-5. 광고 로딩 방식: Eager vs Lazy

공통 개념

load_type 슬롯 타게팅 값으로 관리:

'eager': 페이지 진입 시 바로 display + refresh

'lazy' : IntersectionObserver 등으로 뷰포트 근접 시에만 display + refresh

실제 로딩 로직은:

Eager: googletag.display(...) + googletag.pubads().refresh([...])

Lazy: Observer 안에서 동일 호출

Eager 로딩 사용 위치

대부분 TC_billboard_leaderboard (Above The Fold 핵심 인벤토리)

이유

페이지 브랜드/프리미엄 인벤토리로 가장 높은 단가 상품 유치.

페이지 로딩과 함께 자연스럽게 노출되어도 UX 리스크가 크지 않음.

Lazy 로딩 사용 위치

HOME: MC_billboard_rectangle, MR_halfpage_rectangle

Section/List: MR_halfpage_rectangle

View: in-article 슬롯, 일부 MR 등

이유

화면 하단 인벤토리는 실제 스크롤이 발생했을 때만 요청하여

불필요한 호출 감소

Viewability 개선

페이지 초기 로딩 속도 향상

2-6. Lazy Load 구현 로직 (공통 패턴)
const LAZY_AD_SLOT_IDS = ['MC_billboard_rectangle', 'MR_halfpage_rectangle']; // 예시

let lazyAdSlotsHTML = [];
let lazyGamSlots = [];
let lazyObserver = null;

function setupLazyLoading() {
  const allAdSlots = document.querySelectorAll('.ad-slot');
  lazyAdSlotsHTML = [];

  allAdSlots.forEach(slot => {
    if (LAZY_AD_SLOT_IDS.includes(slot.id)) {
      lazyAdSlotsHTML.push(slot);
    }
  });

  lazyGamSlots = [];
  if (window.googletag && googletag.cmd) {
    googletag.cmd.push(function() {
      const definedSlots = googletag.pubads().getSlots();
      lazyGamSlots = definedSlots.filter(slot =>
        LAZY_AD_SLOT_IDS.includes(slot.getSlotElementId())
      );
    });
  }

  const options = {
    root: null,
    rootMargin: '0px 0px 100px 0px',
    threshold: 0.0
  };

  lazyObserver = new IntersectionObserver(function(entries, observerInstance) {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const slotId = entry.target.id;

      if (window.googletag && googletag.cmd) {
        googletag.cmd.push(function() {
          const slotToRefresh = lazyGamSlots.find(
            s => s.getSlotElementId() === slotId
          );
          if (slotToRefresh) {
            googletag.display(slotId);
            googletag.pubads().refresh([slotToRefresh]);
            console.log(`✅ [Lazy Load] ${slotId} DISPLAY + REFRESH`);
          }
        });
      }
      observerInstance.unobserve(entry.target);
    });
  }, options);
}

function startObservingLazySlots() {
  if (!lazyObserver || !lazyAdSlotsHTML || lazyAdSlotsHTML.length === 0) return;
  lazyAdSlotsHTML.forEach(slot => lazyObserver.observe(slot));
}

document.addEventListener('DOMContentLoaded', function() {
  setupLazyLoading();
  startObservingLazySlots();
});


중요 포인트

disableInitialLoad()와 항상 세트로 생각해 주세요.

Lazy 대상 슬롯을 추가하려면

① LAZY_AD_SLOT_IDS 배열에 ID 추가

② 해당 ID로 defineSlot 정의

③ CSS에서 .ad-slot 기본 스타일만 맞춰주면 공통 로직 그대로 재사용 가능합니다.

2-7. View 페이지 in-article 동적 삽입 규칙 (요약)

자세한 구현은 View GPT 템플릿 가이드에서 다시 설명합니다. 여기서는 공통 개념만 요약합니다.

DOMContentLoaded 시점 사용 이유

본문 DOM 구조(문단, 이미지 블록 등)가 완전히 렌더링된 이후에
삽입 위치를 계산해야 하기 때문입니다.

기본 규칙 예시 (컨셉)

article 또는 .mk-body 내의 자식 요소 중,

텍스트 문단(P) 또는 특정 콘텐츠 블록 기준으로
“n번째 이후, 최소 y px 이상 스크롤 거리 확보” 등 규칙 작성

예)

3번째 문단 뒤에 첫 번째 in-article 광고 삽입

이후 화면 높이의 1~1.5배 이상 간격을 유지하며 추가 삽입

한 기사당 최대 2~3개까지 등.

광고 컨테이너 구조

삽입 시, 다음 형태의 DIV를 생성:

<div id="IN_ARTICLE_1" class="ad-slot ad-in-article">
  <div class="gam-container"></div>
</div>


그리고 JS에서

googletag.defineSlot(AdUnitPath, [...], 'IN_ARTICLE_1')

setTargeting('position', 'IN'), setTargeting('load_type', 'lazy') 등 설정 후

LAZY_AD_SLOT_IDS에 IN_ARTICLE_1을 추가하여 Lazy 로딩 대상에 포함.

2-8. ‘Advertisement’ 라벨 구현 방식 (공통)

이벤트 리스너 등록

googletag.pubads().addEventListener('slotRenderEnded', addAdLabel);


핵심 함수

function addAdLabel(event) {
  const slotId = event.slot.getSlotElementId();
  const slotElement = document.getElementById(slotId); 
  
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


CSS (공통)

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


주의사항

라벨은 광고가 실제로 집행된 슬롯에만 붙습니다 (event.isEmpty === false).

.ad-slot의 border/background는 JS에서 제거하여,
실제 서비스에서는 광고 크리에이티브만 보이도록 합니다.

2-9. 페이지 레벨 vs 슬롯 레벨 타게팅
(1) 페이지 레벨 타게팅 (공통)

공통적으로 googletag.pubads().setTargeting(key, value) 사용.

주요 키(페이지 유형별 상이하나 공통적으로 사용):

카테고리	예시 키	설명
디바이스/환경	device_category	desktop / tablet / mobile
	is_dark_mode	'true' / 'false'
페이지 기본 정보	page_type	'HOME' / 'section_front' / 'view' 등
	web_type	'responsive'
섹션 정보	section_home_nm	'MK' 등
	section_front_nm	'economy' (섹션 메인)
	section_list_nm	'economic-policy' (리스트 카테고리)
URL/도메인	domain, url, referer	실제 로케이션 기반
기사/콘텐츠 정보	SOURCE_ID, CODE_ID, article_num, title 등	View/일부 페이지에서 CMS 연동 시 실제 값 주입
유료/성인/민감	is_paid_content, has_adult_content, brandsensitive	딜/제한 세그먼트용
로그인/회원 정보	login_status, membership_type 등	퍼스트파티 세그먼트 확장용
(2) 슬롯 레벨 타게팅

SlotObject.setTargeting(key, value) 형태.

주로 사용하는 키:

키	예시 값	설명
div_id	'TC_billboard_leaderboard'	디버깅/리포팅용 슬롯 ID
position	'TC' / 'MC' / 'MR' / 'IN'	레이아웃 내 위치
ad_area_type	'ATF' / ''	Above The Fold 여부 등
load_type	'eager' / 'lazy'	로딩 전략 구분용
default_exposure	'hidden' / ''	기본 노출 정책·테스트용
2-10. 광고 관련 CSS 공통 규칙

.ad-slot

광고 컨테이너의 공통 클래스

역할:

광고가 없을 때도 레이아웃 유지

라벨 영역(top -15px) 확보를 위해 padding-top: 10px

min-height로 레이아웃 붕괴 방지

.gam-container

GPT iframe을 수평/수직 중앙 정렬하기 위한 래퍼

다크모드

광고 슬롯 및 라벨은 투명 배경 + 중립 색상 사용
→ 콘텐츠/광고 모두 디자인 통일성 유지

3. View GPT 구현 가이드 (기사 뷰 페이지)

가장 복잡한 페이지라서, 광고 삽입 규칙·동적 로딩·in-article 관련 내용을 자세히 정리합니다.

3-1. 페이지 특성

사용자가 기사 내용을 깊이 읽는 페이지입니다.

수익화 전략 상,

TC 상단 빌보드(프리미엄),

우측 MR(사이드바),

본문 내 in-article 여러 개
등 다양한 인벤토리를 활용하는 것이 핵심입니다.

3-2. AdUnitPath & 타게팅 특이점

AdUnitPath 예시(컨셉):

const AdUnitPath = '/7450/www.mk.co.kr/news/economy/view';


View 페이지에서는 특히 기사 정보 기반 타게팅이 중요합니다.

SOURCE_ID, CODE_ID, MIDDLE_CODE_ENG_NM

article_num, title, title_slug, url_slug

publish_date, journalist, keywords, ticker_name

is_paid_content, has_adult_content, brandsensitive

CMS에서 View 페이지 로딩 시 해당 값들을 JS에 주입해 주시면,
GAM 측에서 콘텐츠 타게팅·브랜드 세이프티·세그먼트 분석에 활용 가능합니다.

3-3. 슬롯 구성 (예시 구조)

Eager

TC_billboard_leaderboard

Lazy

MR_halfpage_rectangle

IN_ARTICLE_1, IN_ARTICLE_2, ... (동적 생성 in-article)

필요 시

본문 하단 or 추천 기사 상단 등에도 추가 슬롯을 설계할 수 있습니다.

3-4. in-article 동적 삽입 로직 (상세)

DOM 분석 단계 (DOMContentLoaded)

document.addEventListener('DOMContentLoaded', function() {
  const articleBody = document.querySelector('.article-body'); // 실제 클래스명 예시
  const blocks = Array.from(articleBody.children);
  
  // 1) 문단/이미지/헤더 등 블록 요소 목록 확보
  // 2) 규칙에 맞는 삽입 포인트 계산
  // 3) 해당 위치에 in-article ad-slot DIV 삽입
  // 4) LAZY_AD_SLOT_IDS에 동적 ID 추가
  // 5) googletag.defineSlot(...) 으로 슬롯 정의
  // 6) Lazy 인프라(setupLazyLoading/startObservingLazySlots)에서 자동 처리
});


삽입 규칙(예시 방향)
(실제 규칙은 추후 기획 시점에 수치 조정 가능합니다)

기본 원칙

첫 광고는 스크롤 1~2 화면 이후 위치에 삽입

광고 간 최소 간격: 뷰포트 높이의 1.5배 이상 또는 문단 수 기준 5~7개 간격

한 기사당 최대 2~3개의 in-article만 허용 (UX 고려)

구현 방식

각 블록의 offsetTop 또는 getBoundingClientRect().top + window.scrollY 값 계산

이전 삽입 위치와의 거리 비교 → 일정 이상의 간격일 때만 삽입.

슬롯 정의/타게팅

const inArticleSlotId = 'IN_ARTICLE_1';

googletag.cmd.push(function() {
  const inArticleSlot = googletag.defineSlot(
    AdUnitPath,
    [[336, 280], [300, 250], [1, 1], 'fluid'],
    inArticleSlotId
  )
    .defineSizeMapping(sizeMapping_square)
    .setTargeting('div_id', inArticleSlotId)
    .setTargeting('position', 'IN')
    .setTargeting('ad_area_type', '')
    .setTargeting('load_type', 'lazy')
    .addService(googletag.pubads());

  LAZY_AD_SLOT_IDS.push(inArticleSlotId);
});


Lazy 로딩과의 연계

LAZY_AD_SLOT_IDS 배열에 in-article ID를 추가해 두면,
기존 Lazy 인프라(setupLazyLoading, startObservingLazySlots)가 자동으로 처리합니다.

3-5. View 페이지 수익화 포인트

TC + MR + in-article 조합으로:

페이지당 총 인벤토리 수(Inventory Depth) 증가

본문 내 광고는 머무는 시간·뷰어빌리티가 높아 eCPM이 높은 편

특히 브랜디드 콘텐츠·네이티브·프리미엄 딜에서
in-article 영역은 중요한 슬롯이 될 수 있습니다.

4. HOME GPT 구현 가이드
4-1. 페이지 특성

사이트 최상단 허브 페이지

다양한 섹션으로 진입하는 게이트 역할.

방문자 수는 많지만 체류시간/스크롤은 비교적 짧을 수 있음.

전략

상단 TC에서 프리미엄/브랜딩 인벤토리 확보.

MC/MR는 Lazy 로딩으로, 실제 스크롤하는 사용자에게만 광고 요청.

4-2. AdUnitPath & 타게팅
const AdUnitPath = '/7450/www.mk.co.kr/HOME';

googletag.pubads().setTargeting('page_type', 'HOME');
googletag.pubads().setTargeting('section_home_nm', 'MK');
googletag.pubads().setTargeting('section_front_nm', '동적입력'); // 필요 시
// 기타 View와 동일한 공통 키들 세팅

4-3. 슬롯 구성

TC_billboard_leaderboard

Eager

상단 메인 빌보드

ad_area_type: 'ATF', load_type: 'eager'

MC_billboard_rectangle

Lazy (LAZY_AD_SLOT_IDS에 포함)

메인 콘텐츠 중간 위치, 가로형/네이티브 배너용

MR_halfpage_rectangle

Lazy

우측 사이드 레일, 300x600/336x280 등 고단가 인벤토리

4-4. HOME 전용 CSS/레이아웃 포인트

.ad-slot에 max-width: 1210px;로 메인 그리드에 맞춤.

.gam-container로 광고 중앙 정렬.

다크모드에서도 광고 슬롯은 투명하게 유지하여 브랜드 레이아웃을 해치지 않도록 설계.

4-5. 수익화 관점

HOME은 **노출량(volume)**이 가장 큰 페이지이므로,

프리미엄 로드블럭, 스폰서십, 특집 패키지를 운영하기에 적합.

Lazy MC/MR를 활용해

초기 로딩 부담은 줄이고,

충분히 스크롤한 사용자의 세션에서 추가 인벤토리로 수익 확대.

5. Section GPT 구현 가이드 (섹션 메인)
5-1. 페이지 특성

특정 섹션(예: 경제)에 대한 허브 페이지.

리스트와 달리, 섹션 전체의 대표성이 있는 페이지입니다.

주요 전략

섹션 내 프리미엄 인벤토리를 여기 집중 배치.

경제·증권·부동산 등 섹션 성격을 section_front_nm 타게팅으로 구분.

5-2. AdUnitPath & 타게팅
const AdUnitPath = '/7450/www.mk.co.kr/news/economy/section_front';

googletag.pubads().setTargeting('page_type', 'section_front');
googletag.pubads().setTargeting('section_home_nm', ''); 
googletag.pubads().setTargeting('section_front_nm', 'economy');


추가로 View와 동일한 동적 키들(SOURCE_ID 등)을 필요 시 사용할 수 있으나,
섹션 메인은 보통 기사 단위보다는 섹션 레벨 분석·딜에 활용하는 편입니다.

5-3. 슬롯 구성

TC_billboard_leaderboard

Eager

섹션 메인 상단 빌보드

MR_halfpage_rectangle

Lazy

우측 상단 MR, ad_area_type: 'ATF', load_type: 'lazy'

5-4. Section 전용 수익화 포인트

섹션별 패키지 판매 시,

해당 섹션의 HOME/Section/List/View를 묶은 패키지에서도
섹션 메인 TC/MR는 프리미엄 단가로 책정할 수 있습니다.

section_front_nm = 'economy'와 같은 타게팅을 활용하면:

경제 섹션만 별도 딜(예: 금융/증권 광고주 전용)을 운영하기 쉬워집니다.

6. List GPT 구현 가이드 (섹션 리스트)
6-1. 페이지 특성

경제 > 정책, 경제 > 산업 등 보다 세분화된 리스트 페이지를 가정.

예시:

section_list_nm = 'economic-policy'

6-2. AdUnitPath & 타게팅
const AdUnitPath = '/7450/www.mk.co.kr/news/economy/section_list';

googletag.pubads().setTargeting('page_type', 'section_front');
googletag.pubads().setTargeting('section_home_nm', ''); 
googletag.pubads().setTargeting('section_front_nm', '');   // List에서는 없음
googletag.pubads().setTargeting('section_list_nm', 'economic-policy');


View/Section과 동일하게 나머지 동적 키들도 주입 가능.

6-3. 슬롯 구성

TC_billboard_leaderboard

Eager

리스트 상단 빌보드

MR_halfpage_rectangle

Lazy

우측 MR, load_type: 'lazy'

LAZY_AD_SLOT_IDS = ['MR_halfpage_rectangle']

6-4. List 전용 수익화 포인트

리스트 페이지는

기사 뷰보다는 체류시간이 짧지만

섹션 내 깊은 관심 영역으로 진입한 유저이기 때문에
타겟팅 관점에서 가치가 높습니다.

section_list_nm를 이용해

예: economic-policy 전용 딜, 리포트, A/B 테스트 등 운영 가능.

7. 추가 개발 유의사항 및 확장 가이드

PPID(퍼스트파티 ID) 연동

현재 템플릿에서는

googletag.pubads().setPublisherProvidedId('PPID_DYNAMIC_HASHED_ID (SERVER_HASHED_ID)');


실제 서비스에서는

서버에서 해시 처리된 유저 ID(또는 로그인 ID 기반)을 주입해 주세요.

1st-party 기반 Audience 세그먼트·리타게팅에 활용 가능합니다.

코드 재사용/모듈화

현재는 각 템플릿에 JS가 포함되어 있으나,

공통 모듈로 분리 가능:

gpt-common.js

setupLazyLoading, startObservingLazySlots

addAdLabel

공통 sizeMapping 정의

페이지별로:

AdUnitPath, 페이지 타게팅 값, 슬롯 정의만 별도 스크립트에서 호출.

디버깅 팁

console.log:

[Lazy Load] SLOT_ID DISPLAY + REFRESH

[Ad Label] Label added for SLOT_ID

[Eager] TC_billboard_leaderboard DISPLAY + REFRESH

GAM 콘솔의 **“View in Browser” + “Developer Console”**를 함께 사용하면
로딩 흐름을 이해하기 쉽습니다.

버전 관리

gpt_version = '2025' 타게팅을 페이지 레벨에 항상 세팅하고 있으므로,

추후 2026 버전 템플릿을 만들 경우

gpt_version = '2026'으로 변경하여

버전별 성과 비교·AB 테스트에 활용할 수 있습니다.