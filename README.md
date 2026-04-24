# 수사24<img src="https://github.com/user-attachments/assets/e795f92e-08e8-4128-aaf9-9f7750a1f7c1" align=left width=120>

> 경찰의 추적수사 효율을 높이는 지리적 프로파일링 앱  
> **팀 DreamWorms(왕꿈틀이)** · iOS Developer 4 · Designer 1 · PM 1 (2025.09 ~ 2026.03)  
> 🏆 Apple Developer Academy 2025 **Showcase Spotlight Team 선정** · 🤝 경북지방경찰청 협업

<br />

## 💭 소개

> 통신사 기지국 문자를 **자동으로 수집**하여, 사건별 피의자 동선을 **지도와 타임라인으로 시각화**하는 수사 지원 앱입니다.
>
> 형사님들이 5분마다 수신되는 기지국 SMS를 일일이 확인하고 엑셀에 정리하던 수작업을, **AppIntent 기반 End-to-End 자동화 파이프라인**으로 대체하여 문자 확인·문서작업 시간을 **0시간으로 단축**했습니다.
>
> 현재 **경상북도경찰청 테스트베드 투입 준비 중**

---

## ✨ 기능과 구현 사항

### 기술 개요

- **Custom Redux Architecture** (DWStore) 기반 단방향 상태 관리
- **AppIntent + AsyncStream** 기반 실시간 데이터 자동 수집
- **SSOT (Single Source of Truth)** 패턴으로 데이터 일관성 보장
- Swift 6 **Strict Concurrency** 완전 대응

<!-- 📸 전체 화면 스크린샷 그리드 (여기에 앱 전체 화면 캡처 이미지 넣기) -->
<!-- 예시: Kompass처럼 2행 5열 그리드 -->
<!-- <img width="100%" alt="앱 전체 화면" src="스크린샷_그리드_이미지_URL" /> -->

### 1. 사건 분류

배정받은 사건을 사건번호 기준으로 등록하고, 피의자 통신추적 기록을 사건별로 자동 구분·관리합니다.

<!-- 📸 사건 목록 화면 -->
<img width="100%" alt="사건 분류" src="https://github.com/user-attachments/assets/9676cceb-d035-4bd2-8cdd-c77214283823" />

### 2. App Intent 기반 위치 데이터 자동 수집

통신사 기지국 문자를 **iOS 단축어 + AppIntent**로 자동 수집합니다. SMS 수신 → 주소 파싱 → 좌표 변환 → CoreData 저장 → 지도/타임라인 실시간 반영까지 **7단계 자동화 파이프라인**.

```swift
struct ReceiveMessageIntent: AppIntent {
    static var title: LocalizedStringResource = "기지국 위치정보 저장하기"

    @Parameter(title: "메시지 내용") var messageBody: String
    @Parameter(title: "발신자 번호") var senderPhoneNumber: String

    func perform() async throws -> some IntentResult & ProvidesDialog {
        let normalized = senderPhoneNumber
            .replacingOccurrences(of: "-", with: "")
            .replacingOccurrences(of: "+82", with: "0")

        guard let caseID = try await caseRepo.findCaseTest(byCasePhoneNumber: normalized) else {
            return .result(dialog: "해당 전화번호의 사건을 찾을 수 없습니다")
        }

        guard let address = MessageParser.extractAddress(from: messageBody) else {
            return .result(dialog: "주소를 추출할 수 없습니다")
        }

        let geocodeResult = try await NaverGeocodeAPIService.shared.geocode(address: address)

        try await locationRepo.createLocationFromMessage(
            caseID: caseID, address: address,
            latitude: geocodeResult.latitude, longitude: geocodeResult.longitude
        )
        return .result(dialog: "위치가 저장되었습니다: \(address)")
    }
}
```

<!-- 📸 App Intent 동작 화면 -->
<img width="100%" alt="App Intent" src="https://github.com/user-attachments/assets/194b83d0-88ae-4e44-a020-9761e32581af" />

### 3. 피의자 현재위치 확인 & 생활 패턴 분석

Naver Maps SDK 기반 위치 핀 표시, **기지국 500m 반경 바운딩 박스 오버레이**로 피의자가 머문 영역을 시각화합니다. 여러 시점의 기지국 반경이 **겹치는 영역**을 한눈에 파악하여 출동 판단을 지원합니다.

<!-- 📸 지도 탭 화면 -->
<img width="100%" alt="지도" src="https://github.com/user-attachments/assets/54d1c455-9228-4b22-b612-62007c0bd1ca" />

### 4. Timeline BottomSheet (시·공간 통합 탐색)

날짜별 그룹화된 위치 데이터를 **3단 Detent 바텀시트 타임라인**으로 시각화합니다. 연속 방문 셀 그룹화, **Top3 방문 장소 ColorStick**, 실시간 검색(Debounce 250ms), 날짜 칩 탭→스크롤 이동을 지원합니다.

<!-- 📸 타임라인 바텀시트 화면 -->
<img width="100%" alt="수상한 패턴" src="https://github.com/user-attachments/assets/e99274ca-e893-4637-9fe8-fa6c7f1b6a45" />

### 5. 증거 사진 촬영 & OCR 인식

AVFoundation 기반 커스텀 카메라로 현장 증거를 촬영하고, **Vision 프레임워크로 문서를 자동 감지·텍스트 인식**하여 지도 위치와 연결합니다.

<!-- 📸 증거 스캔 화면 -->
<img width="100%" alt="증거" src="https://github.com/user-attachments/assets/a1bb1e40-b305-462b-824d-c4012083139d" />

### 6. CCTV 수사 & AI 거점 분석

지도 위 **거점 3곳을 지정**하면 해당 지역 공공 CCTV 리스트를 자동 취합합니다. **Apple Foundation Models(온디바이스 AI)**가 피의자 주요 거점을 한 줄로 요약하여 수사 전략 수립을 지원합니다.

<!-- 📸 CCTV + AI 분석 화면 -->
<img width="100%" alt="CCTV" src="https://github.com/user-attachments/assets/71135810-55d5-4db0-8982-33daa215d6f5" />
<img width="100%" alt="AI 분석" src="https://github.com/user-attachments/assets/55d2bb40-666b-41aa-8e5c-21ee065a9725" />

<!-- 🎬 데모 영상 -->
https://github.com/user-attachments/assets/312d3e54-f710-4a5c-839a-2b13fb9869ab

---

## 🤔 개발 과정에서의 고민과 배운 점

### Custom Redux Architecture — DWStore

MVVM + Combine(`ObservableObject` + `@Published`)으로 MVP를 구현했으나, ViewModel 간 상태 불일치와 Side Effect 관리 부재가 심각해져 **단방향 아키텍처로 전면 전환**했습니다. TCA의 학습 곡선과 일정을 고려해 **Redux/MVI에서 핵심 4요소만 추출**한 경량 자체 프레임워크를 설계했고, 팀원 4명이 **1주일 내 숙달**했습니다.

```swift
// DWStore — @Observable + @MainActor로 Swift 6 완전 호환
@MainActor @Observable
public final class DWStore<R: DWReducer> {
    public private(set) var state: R.State
    private let reducer: R

    public func send(_ action: R.Action) {
        let effect = reducer.reduce(into: &state, action: action)
        Task { [weak self] in
            await effect.run { [weak self] next in
                Task { @MainActor [weak self] in self?.send(next) }
            }
        }
    }
}

// Effect — 비동기 부수효과 모델링
public struct DWEffect<Action: DWAction>: Sendable {
    public static var none: Self { .init { _ in } }

    public static func task(_ work: @escaping @Sendable () async -> Action?) -> Self {
        .init { downstream in if let a = await work() { downstream(a) } }
    }

    public static func merge(_ effects: DWEffect<Action>...) -> DWEffect<Action> {
        .init { downstream in
            await withTaskGroup(of: Void.self) { group in
                for effect in effects { group.addTask { await effect.run(downstream) } }
            }
        }
    }
}
```

### SSOT Observer — AsyncStream 실시간 동기화

App Intent가 백그라운드에서 CoreData에 저장한 데이터가 자동으로 UI에 반영되는 핵심 메커니즘입니다. `MainTabFeature`가 **유일한 데이터 소유자(SSOT)**로서 `AsyncStream`을 구독하고, 하위 Feature에 단방향으로 전파합니다.

```swift
// CoreData 변경 감지 AsyncStream
func watchLocations(caseId: UUID) -> AsyncStream<[Location]> {
    AsyncStream { continuation in
        Task {
            if let initial = try? await fetchLocations(caseId: caseId) {
                continuation.yield(initial)
            }
            for await _ in NotificationCenter.default.notifications(
                named: .NSManagedObjectContextObjectsDidChange, object: context
            ) {
                if let locations = try? await fetchLocations(caseId: caseId) {
                    continuation.yield(locations)
                }
            }
        }
    }
}
```

### iOS Shortcuts Privacy 제한 우회 — 3단계 접근 전환

초기 전화번호 기반 매칭 → **Apple Privacy 정책으로 차단** → 사건번호 매칭으로 전환 → 현장 피드백 "매번 입력 어렵다" → **전화번호 앱 내 사전 등록으로 재전환**. 수사관 입력 단계 **4단계 → 0단계**.

```swift
// [2차] 사건번호 기반 매칭
guard let caseID = try await caseRepo.findCase(byCaseNumber: caseNumber) else {
    return .result(dialog: "해당 사건번호를 찾을 수 없습니다")
}

// [3차] 전화번호 앱 내부 사전 등록 → 자동 매칭
let normalized = senderPhoneNumber
    .replacingOccurrences(of: "-", with: "")
    .replacingOccurrences(of: "+82", with: "0")

guard let caseID = try await caseRepo.findCaseTest(byCasePhoneNumber: normalized) else {
    return .result(dialog: "해당 전화번호의 사건을 찾을 수 없습니다")
}
```

### Timeline 대량 데이터 렌더링 최적화

5,000건 데이터에서 **All Heap 메모리 653MB, Hang 평균 3.88초** 발생. Instruments(Allocations + Time Profiler)로 병목을 추적하여 **하이브리드 렌더링 + DateFormatter 캐싱**을 적용.

```swift
// [Before] VStack 전체 렌더링 — DateFormatter 매번 생성
VStack(spacing: 0) {
    ForEach(groupedLocations) { group in
        VStack(spacing: 0) {
            Color.clear.id(group.dateID)
            TimeLineDateSectionHeader(...)
            VStack(spacing: 0) {                  // ← 전체 렌더링
                ForEach(group.consecutiveGroups) { ... }
            }
        }
    }
}

// [After] 하이브리드 — 헤더 VStack(앵커 보장) + 셀 LazyVStack(화면 근처만)
VStack(spacing: 0) {
    ForEach(groupedLocations) { group in
        VStack(spacing: 0) {
            Color.clear.id(group.dateID)           // Anchor — 즉시 생성
            TimeLineDateSectionHeader(...)
            LazyVStack(spacing: 0) {               // ← 화면 근처만 렌더링
                ForEach(group.consecutiveGroups) { ... }
            }
        }
    }
}
```

| 항목 | Before | After | 변화 |
|:---:|:---:|:---:|:---:|
| All Heap 메모리 | 653 MB | 143 MB | **4.6배 감소** |
| Hang | 평균 3.88s | 평균 729ms | **5.3배 개선** |
| Thermal State | Nominal → Fair | Nominal 유지 | **개선** |

### BottomSheet 스크롤/드래그 충돌 + ScrollTarget UUID

시트 mid 높이에서 스크롤 대신 드래그가 잡히는 문제를 `.presentationContentInteraction(.scrolls)`로 해결. 같은 날짜 칩 재탭 시 scrollTo 미동작을 **UUID 기반 Equatable 오버라이드**로 해결.

```swift
struct ScrollTarget: Equatable {
    let dateID: String
    let triggerID: UUID    // 매 탭마다 새 UUID 생성

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.triggerID == rhs.triggerID  // 같은 dateID여도 "변경됨"으로 감지
    }
}
```

### 검색 디바운스 — Combine 없이 UUID 기반 무효화

한글 조합형 입력 시 검색 호출 **8회→1회** 감소. Combine의 `.debounce()` 대신 **Task + UUID 검증**으로 아키텍처 일관성을 유지.

```swift
case .searchTextChanged(let text):
    state.searchText = text
    let taskID = UUID()
    state.searchDebounceTaskID = taskID
    return .task {
        try? await Task.sleep(for: .milliseconds(250))
        return .performSearch(text, taskID: taskID)
    }

case .performSearch(let text, let taskID):
    guard taskID == state.searchDebounceTaskID else { return .none }
    // stale 결과 자동 무시 → 최신 검색만 실행
```

### MapDispatcher — 모듈 간 느슨한 결합 통신

Timeline 셀 탭 → Map 카메라 이동 등, Feature 간 직접 의존 없이 명령을 전달하는 Dispatcher 패턴.

```swift
@Observable
final class MapDispatcher {
    private(set) var request: RequestType?

    func send(_ request: RequestType) { self.request = request }
    func consume() { request = nil }

    enum RequestType: Equatable {
        case moveToLocation(coordinate: MapCoordinate)
        case focusCellTimeline(cellKey: String, title: String?)
    }
}
```

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

## 🗂 폴더 구조

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
│   │   ├── MainTabScene/         # MainTabView · MainTabFeature (SSOT)
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
├── Documentation.docc/           # DocC 인수인계 문서 (SVG 10개)
└── Tests/
```

---

## 📺 앱 구동 화면

<!-- 🎬 GIF 또는 스크린샷 (여기에 앱 주요 화면 GIF 넣기) -->
<!-- 
| 사건 등록 | 지도 탭 | 타임라인 | 증거 스캔 | AI 분석 |
|:---:|:---:|:---:|:---:|:---:|
| <img src="GIF_URL" width="180"/> | <img src="GIF_URL" width="180"/> | <img src="GIF_URL" width="180"/> | <img src="GIF_URL" width="180"/> | <img src="GIF_URL" width="180"/> |
-->

---

## 👥 Contributors

| <a href="https://github.com/YooGyeongMo"><img src="https://github.com/YooGyeongMo.png" width="80"/><br/><b>Demian Yoo</b></a> | <a href="https://github.com/MuchanKim"><img src="https://github.com/MuchanKim.png" width="80"/><br/><b>Muchan Kim</b></a> | <a href="https://github.com/mini-min"><img src="https://github.com/mini-min.png" width="80"/><br/><b>mini-min</b></a> | <a href="https://github.com/delightPIP"><img src="https://github.com/delightPIP.png" width="80"/><br/><b>delightPIP</b></a> | <a href="https://github.com/Jikiim"><img src="https://github.com/Jikiim.png" width="80"/><br/><b>Jikiim</b></a> | <a href="https://github.com/youryeony"><img src="https://github.com/youryeony.png" width="80"/><br/><b>youryeony</b></a> |
|:---:|:---:|:---:|:---:|:---:|:---:|
| iOS Developer | iOS Developer | iOS Developer | iOS Developer | UI/UX Designer | Product Manager |

---

## 📎 Links

- 📰 [드림웜즈 언론소개](https://www.notion.so/24-310ca74acc7b8155964fe2f077483ec8)
- 📱 [수사24 앱 소개](https://www.notion.so/24-310ca74acc7b8142807bf02a3b47d928)
- ✍️ [팀 테크 블로그 (Medium)](https://medium.com/%EC%99%95%EA%BF%88%ED%8B%80%EC%9D%B4-dreamworms)

---

<div align="center">

**Apple Developer Academy @ POSTECH · Team DreamWorms(왕꿈틀이) · 2025–2026**

</div>
