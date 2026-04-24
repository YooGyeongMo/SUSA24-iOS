<div align="center">

<!-- 🔲 앱 로고 이미지 (여기에 앱 아이콘 이미지 넣기) -->
<img width="120" src="https://github.com/user-attachments/assets/e795f92e-08e8-4128-aaf9-9f7750a1f7c1" alt="수사24 로고" />

# 수사24

**경찰의 추적수사 효율을 높이는 지리적 프로파일링 앱**

[![Swift](https://img.shields.io/badge/Swift-6.0-F05138?style=flat&logo=swift&logoColor=white)](https://swift.org)
[![iOS](https://img.shields.io/badge/iOS-26.0+-000000?style=flat&logo=apple&logoColor=white)](https://developer.apple.com/ios/)
[![Xcode](https://img.shields.io/badge/Xcode-26.0+-007ACC?style=flat&logo=xcode&logoColor=white)](https://developer.apple.com/xcode/)
[![License](https://img.shields.io/badge/License-Apache_2.0-0A84FF?style=flat)](LICENSE)

🏆 **Apple Developer Academy 2025 Showcase Spotlight Team 선정**<br>
🤝 경북지방경찰청 · 포항북부경찰서 협업

**개발 기간** : 2025.09 ~ 2026.03 (7개월)<br>
**팀 구성** : iOS Developer 4 · Designer 1 · PM 1

</div>

---

## 💭 소개

통신사 기지국 문자를 **자동으로 수집**하여, 사건별 피의자 동선을 **지도와 타임라인으로 시각화**하는 수사 지원 앱입니다.

형사님들이 5분마다 수신되는 기지국 SMS를 일일이 확인하고 엑셀에 정리하던 수작업을, **AppIntent 기반 End-to-End 자동화 파이프라인**으로 대체하여 문자 확인·문서작업 시간을 **0시간으로 단축**했습니다.

> 현재 **경상북도경찰청 테스트베드 투입 준비 중**

---

## ✨ 주요 기능

### 1. App Intent 기반 위치 데이터 자동 수집

통신사 기지국 문자를 **iOS 단축어 + AppIntent**로 자동 수집합니다. SMS 수신 → 주소 파싱 → 좌표 변환 → CoreData 저장 → 지도/타임라인 실시간 반영까지 **7단계 자동화 파이프라인**.

<!-- 📸 스크린샷: App Intent 동작 화면 (SMS → 지도에 자동 표시) -->
<img width="100%" alt="App Intent" src="https://github.com/user-attachments/assets/194b83d0-88ae-4e44-a020-9761e32581af" />

---

### 2. 사건별 지도 시각화 & 기지국 중첩 영역

Naver Maps SDK 기반 위치 핀 표시, **기지국 500m 반경 바운딩 박스 오버레이**로 피의자가 머문 영역을 시각화합니다. 여러 시점의 기지국 반경이 **겹치는 영역**을 한눈에 파악하여 출동 판단을 지원합니다.

<!-- 📸 스크린샷: 지도 탭 (기지국 범위 + 누적 빈도) -->
<img width="100%" alt="지도" src="https://github.com/user-attachments/assets/54d1c455-9228-4b22-b612-62007c0bd1ca" />

---

### 3. Timeline BottomSheet (시·공간 통합 탐색)

날짜별 그룹화된 위치 데이터를 **3단 Detent 바텀시트 타임라인**으로 시각화합니다. 연속 방문 셀 그룹화, **Top3 방문 장소 ColorStick**, 실시간 검색(Debounce 250ms), 날짜 칩 탭→스크롤 이동을 지원합니다.

<!-- 📸 스크린샷: 바텀시트 타임라인 (날짜 칩 + 위치 셀 리스트) -->
<img width="100%" alt="수상한 패턴" src="https://github.com/user-attachments/assets/e99274ca-e893-4637-9fe8-fa6c7f1b6a45" />

---

### 4. 증거 사진 촬영 & OCR 인식

AVFoundation 기반 커스텀 카메라로 현장 증거를 촬영하고, **Vision 프레임워크로 문서를 자동 감지·텍스트 인식**하여 지도 위치와 연결합니다.

<!-- 📸 스크린샷: 증거 스캔 화면 -->
<img width="100%" alt="증거" src="https://github.com/user-attachments/assets/a1bb1e40-b305-462b-824d-c4012083139d" />

---

### 5. CCTV 수사 & AI 거점 분석

지도 위 **거점 3곳을 지정**하면 해당 지역 공공 CCTV 리스트를 자동 취합합니다. **Apple Foundation Models(온디바이스 AI)**가 피의자 주요 거점을 한 줄로 요약하여 수사 전략 수립을 지원합니다.

<!-- 📸 스크린샷: CCTV + AI 분석 -->
<img width="100%" alt="CCTV" src="https://github.com/user-attachments/assets/71135810-55d5-4db0-8982-33daa215d6f5" />
<img width="100%" alt="AI 분석" src="https://github.com/user-attachments/assets/55d2bb40-666b-41aa-8e5c-21ee065a9725" />

---

<!-- 🎬 데모 영상 GIF (여기에 앱 시연 GIF 또는 영상 링크) -->
https://github.com/user-attachments/assets/312d3e54-f710-4a5c-839a-2b13fb9869ab

---

## 🤔 트러블슈팅

### 1. Timeline 대량 데이터 렌더링 성능 최적화

5,000건 규모의 데이터를 VStack이 한 번에 렌더링하면서 **All Heap 메모리 653MB**, **Hang 평균 3.88초**가 발생. Xcode Instruments의 Allocations과 Time Profiler로 병목을 추적하여 **VStack(헤더) + LazyVStack(아이템) 하이브리드 전략**과 **DateFormatter `static let` 캐싱(수천 개→3개)**을 적용.

| 항목 | Before | After | 변화 |
|:---:|:---:|:---:|:---:|
| All Heap 메모리 | 653 MB | 143 MB | **4.6배 감소** |
| Hang | 평균 3.88s | 평균 729ms | **5.3배 개선** |
| Thermal State | Nominal → Fair | Nominal 유지 | **개선** |

### 2. 아키텍처 진화 — MVVM → Custom Redux (DWStore)

MVP에서 MVVM + Combine 사용 시 **ViewModel 간 상태 불일치, onChange 지옥, Side Effect 관리 부재** 문제 발생. TCA에서 영감을 받되 학습 곡선과 일정을 고려해 **State/Action/Reducer/Effect 4요소만 추출**한 경량 자체 프레임워크 DWStore를 설계. 팀원 4명 **1주일 내 숙달**.

### 3. App Intent — iOS Shortcuts Privacy 3단계 피벗

초기 전화번호 기반 매칭 → **Apple Privacy 정책으로 차단** → 사건번호 매칭으로 전환 → 현장 피드백 "매번 입력 어렵다" → **전화번호 앱 내 사전 등록 방식으로 재전환**. 수사관 입력 단계 **4단계 → 0단계**.

### 4. BottomSheet 스크롤/드래그 충돌

시트 mid 높이에서 스크롤 대신 드래그가 잡히는 문제를 `.presentationContentInteraction(.scrolls)`로 해결. 같은 날짜 칩 재탭 시 scrollTo 미동작을 **ScrollTarget UUID + Equatable 오버라이드**로 해결.

### 5. 검색 디바운스 — Combine 없이 UUID 기반

한글 조합형 입력 시 검색 호출 **8회→1회** 감소. 매 입력마다 UUID 생성 → 250ms 후 ID 일치 검증 → stale 결과 자동 무시. **Combine 없이 DWStore 아키텍처 일관성 유지**.

---

## 📚 아키텍처 & 기술 스택

| Category | Stack |
|:---:|:---|
| **Architecture** | Custom Redux (DWStore) · SSOT · Repository Pattern · Coordinator · DI |
| **UI** | SwiftUI · PresentationDetent · Custom BottomSheet |
| **Data** | CoreData · Repository Protocol · `@MainActor` isolation |
| **Automation** | AppIntents · iOS Shortcuts · MessageParser |
| **Concurrency** | Swift 6 Strict Concurrency · AsyncStream · `Sendable` |
| **Map** | Naver Maps SDK · Naver Geocoding API · Kakao Search API |
| **AI** | Apple Foundation Models (On-Device) |
| **Media** | AVFoundation · Vision (OCR) |
| **Network** | Alamofire · URLSession · Endpoint Pattern |
| **Documentation** | DocC (인수인계 문서 — SVG 다이어그램 10개 포함) |

---

## 🗂 프로젝트 구조

```
SUSA24-iOS/
├── Sources/
│   ├── Application/
│   │   ├── Coordinator/          # AppCoordinator · AppRoute
│   │   └── Factory/              # ModuleFactory (DI)
│   ├── Core/
│   │   ├── DWArchitecture/       # DWStore · DWReducer · DWEffect
│   │   └── Components/           # DWButton · DWTabBar · DWToast
│   ├── Data/
│   │   ├── Persistence/          # CoreData · PersistenceController
│   │   └── Repository/           # CaseRepository · LocationRepository
│   ├── Presentation/
│   │   ├── MainTabScene/         # MainTabView · MainTabFeature
│   │   ├── MapScene/             # MapView · MapFeature · MapDispatcher
│   │   ├── TimeLineScene/        # TimeLineView · TimeLineFeature
│   │   ├── CaseListScene/        # CaseListView
│   │   ├── SearchScene/          # SearchView · Kakao API
│   │   ├── CameraScene/          # Custom Camera · Vision OCR
│   │   └── DashboardScene/       # AI 거점 분석
│   └── Util/
│       ├── AppIntent/            # ReceiveMessageIntent · MessageParser
│       ├── Network/              # Alamofire · Endpoint · Geocoding
│       └── Camera/               # CameraModel · CaptureSession
├── Documentation.docc/           # DocC 인수인계 문서
└── Tests/
```

---

## 👥 Contributors

<div align="center">

| <a href="https://github.com/YooGyeongMo"><img src="https://github.com/YooGyeongMo.png" width="80"/><br/><b>Demian Yoo</b></a> | <a href="https://github.com/MuchanKim"><img src="https://github.com/MuchanKim.png" width="80"/><br/><b>Muchan Kim</b></a> | <a href="https://github.com/mini-min"><img src="https://github.com/mini-min.png" width="80"/><br/><b>mini-min</b></a> | <a href="https://github.com/delightPIP"><img src="https://github.com/delightPIP.png" width="80"/><br/><b>delightPIP</b></a> | <a href="https://github.com/Jikiim"><img src="https://github.com/Jikiim.png" width="80"/><br/><b>Jikiim</b></a> | <a href="https://github.com/youryeony"><img src="https://github.com/youryeony.png" width="80"/><br/><b>youryeony</b></a> |
|:---:|:---:|:---:|:---:|:---:|:---:|
| iOS Developer | iOS Developer | iOS Developer | iOS Developer | UI/UX Designer | Product Manager |

</div>

---

## 📎 Links

- 📰 [드림웜즈 언론소개](https://www.notion.so/24-310ca74acc7b8155964fe2f077483ec8)
- 📱 [수사24 앱 소개](https://www.notion.so/24-310ca74acc7b8142807bf02a3b47d928)
- ✍️ [팀 테크 블로그 (Medium)](https://medium.com/%EC%99%95%EA%BF%88%ED%8B%80%EC%9D%B4-dreamworms)

---

<div align="center">

**Apple Developer Academy @ POSTECH · Team DreamWorms(왕꿈틀이) · 2025–2026**

</div>
