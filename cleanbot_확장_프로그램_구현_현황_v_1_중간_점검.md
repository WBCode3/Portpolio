# 🧹 Cleanbot 확장 프로그램 – 구현 현황 (v1 중간 점검)

## ✅ 완료된 핵심 기능들

### 1) 텍스트 블러링 시스템
- **단어별 정밀 블러**: 욕설 단어만 정확히 가림
- **컨테이너 블러**: 짧은 제목/링크는 전체 블러 처리
- **클릭 해제**: 블러된 요소 클릭 시 즉시 해제

### 2) 입력 순화 제안 시스템
- **실시간 감지**: `textarea`, `input`, `contenteditable`에서 욕설 감지
- **원클릭 치환**: 제안 칩 클릭 시 즉시 텍스트 교체

### 3) 고급 텍스트 정규화
- **초성 복원**: "ㅆㅂ" → "씨발" 등 복원
- **반복 문자 축약**: "좆좆좆" → "좆"
- **동음/음차 매핑**: "ssibal" → "씨발" 등

### 4) 다중 감지 엔진
- **korcen.ts-stable**: 키워드 기반 1차 필터
- **rules.json**: 사전/패턴/페널티 기반 2차 필터
- **별칭 매핑**: "씨*발", "c발" 등 변형 감지

### 5) DOM 변화 감지
- **MutationObserver**: 새 댓글/피드 자동 감지
- **IntersectionObserver**: 화면에 보이는 요소만 처리
- **requestIdleCallback**: 배치 처리로 성능 최적화

### 6) 팝업 인터페이스
- **전체 해제 토글**: "블러 전체 해제"
- **실시간 피드백**: 해제된 요소 개수 표시
- **화이트 톤 모던 UI**

---

## 🎨 UI/UX 특징

### 텍스트 블러링
- 파란 배지 + 블러 효과
- 클릭 즉시 해제
- 단어별/컨테이너별 선택 적용

### 입력 순화 제안
- 다크 그라데이션 배경 오버레이
- 컬러 칩(파랑/초록/주황) + 호버/포커스 애니메이션
- ✨ 아이콘 + 헤더 라벨

---

## 🔧 기술 스택

### 핵심
- **Chrome Extension MV3**
- **TypeScript**, **Vite**
- **Web Components(Shadow DOM)**

### 감지 엔진
- **korcen.ts-stable** (키워드 필터)
- **정규식 패턴** (교란/변형 대응)

## 📁 프로젝트 구조

```
cleanbot-ext/
├── 📦 Build System
│   ├── vite.config.ts          # Vite 빌드 설정
│   ├── package.json            # 의존성 관리
│   └── dist/                   # 빌드 결과물
│
├── 🎯 Core Extension
│   ├── manifest.json           # Chrome MV3 매니페스트
│   ├── popup.html/js           # 확장 프로그램 팝업
│   └── bg.ts                   # 백그라운드 서비스 워커
│
├── 🧠 Content Scripts
│   ├── bootstrap.ts            # 메인 진입점
│   ├── observe.ts              # DOM 변화 감지
│   ├── detect.ts               # 텍스트 블록 감지
│   ├── rules.ts                # 욕설 분류 엔진
│   ├── normalize.ts            # 텍스트 정규화
│   ├── action.ts               # 블러/언블러 액션
│   ├── inputSanitizer.ts       # 입력 순화 제안
│   ├── extract.ts              # 이미지 추출
│   ├── ocr.worker.ts           # OCR 워커
│   ├── ocrBridge.ts            # OCR 브리지
│   └── ui/                     # UI 컴포넌트
│       ├── Badge.ts            # "정화됨" 배지
│       └── ImageOverlay.ts     # 이미지 오버레이
│
├── 📊 Assets & Data
│   ├── assets/rules.json       # 욕설 규칙 데이터
│   ├── assets/styles.css       # 블러 스타일
│   └── tessdata/kor.traineddata # OCR 언어 데이터
│
└── 🔗 External Dependencies
    └── korcen.ts-stable/       # 한국어 욕설 필터 라이브러리
```


## 🎯 핵심 유스케이스

### UC-1: 웹페이지 텍스트 블러링
```
Actor: 사용자
Goal: 악플이 포함된 웹페이지를 안전하게 탐색
Flow:
1. 사용자가 웹페이지 방문
2. Cleanbot이 DOM 변화 감지
3. 텍스트 블록에서 욕설 감지
4. 해당 텍스트를 블러 처리
5. "정화됨" 배지 표시
6. 사용자가 필요시 클릭으로 원문 확인
```

### UC-2: 입력 순화 제안
```
Actor: 사용자
Goal: 댓글 작성 시 욕설을 순화된 표현으로 교체
Flow:
1. 사용자가 댓글창에 텍스트 입력
2. Cleanbot이 실시간으로 욕설 감지
3. 순화 제안 칩 표시 (3가지 옵션)
4. 사용자가 원하는 제안 클릭
5. 텍스트 자동 교체
6. 사용자 선택 기억 (다음번 우선 표시)
```

### UC-3: 이미지 콘텐츠 필터링
```
Actor: 사용자
Goal: 욕설이 포함된 이미지를 안전하게 처리
Flow:
1. 웹페이지에 이미지 로드
2. Cleanbot이 이미지 감지
3. OCR로 이미지 내 텍스트 추출
4. 추출된 텍스트에서 욕설 감지
5. 이미지에 블러 오버레이 적용
6. 사용자가 필요시 클릭으로 원본 확인
```

### UC-4: 전체 블러 해제
```
Actor: 사용자
Goal: 모든 블러된 콘텐츠를 한 번에 해제
Flow:
1. 사용자가 확장 프로그램 팝업 클릭
2. "블러 전체 해제" 버튼 클릭
3. 모든 블러된 텍스트/이미지 해제
4. 해제된 요소 개수 표시
```

## 🔧 기술 스택

### Frontend
- **TypeScript**, **Vite**, **Web Components(Shadow DOM)**, **CSS Modules**

### Detection Engine
- **korcen.ts-stable**(키워드 필터), **정규식 패턴**, **텍스트 정규화**(초성/반복/유니코드)

### Performance
- **MutationObserver**, **IntersectionObserver**, **requestIdleCallback**, **Web Worker(OCR)**

### Storage
- **chrome.storage.sync**(설정), **chrome.storage.local**(캐시), **WeakSet**(중복 방지)

## 🎨 사용자 경험
- **비침습**: 필요한 순간에만 UI 표시, 웹 탐색 방해 최소화
- **사용자 제어**: 클릭 해제, 팝업 전체 제어, 설정 기억
- **피드백**: "정화됨" 배지, 블러 효과, 순화 제안 칩



---

## 🚀 고도화 로드맵 (v2 제안)

다음 항목들을 **단순·명확하게 추가**합니다:

1) **데이터셋 심화**: 욕설/비하 문장 라벨 데이터 확장, 규칙 파일(lexicon/patterns/penalties/replacements) 모듈화.
2) **다중 라벨 분류**: 욕설 외 조롱·선동·혐오(인종/성별/성적지향 등) 라벨 추가. 초기엔 규칙 기반, 필요 시 온디바이스 경량 분류기.
3) **경량 API 서버**: 규칙/사전 OTA 제공, (옵션) 익명 집계 메트릭, (옵션) AI 순화 문장 API.
4) **이미지 블러 개선**: `img.src` 교체 없이 **오버레이 방식**으로 블러 적용. CORS 이미지는 ALT/캡션 등 텍스트로 판단.
5) **AI 순화 문장**: 키워드 치환에서 **문장 단위 톤 다운**으로 확장. 온디바이스 우선, 필요 시 서버 폴백.
6) **운영/UX 다듬기**: 배지에 간단 사유 태그(예: "혐오(성별)") 노출, 옵션 최소화 유지.



> 📷 **이미지 예정 안내**: 아키텍처 다이어그램과 데이터 플로우 이미지는 추후에 문서에 직접 삽입 예정입니다.



---

## 📷 아키텍처 & 플로우 이미지

### 시스템 아키텍처
![시스템 아키텍처](sandbox:/mnt/data/시스템 아키텍쳐.png)

### 데이터 플로우
![데이터 플로우](sandbox:/mnt/data/데이터 플로우.png)

### 입력 순화 시퀀스
![입력 순화](sandbox:/mnt/data/입력순화.png)

