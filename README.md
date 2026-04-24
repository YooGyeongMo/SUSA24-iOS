# 수사24<img src="Assets/logo.png" align=left width=120>

> 경찰의 추적수사 효율을 높이는 지리적 프로파일링 앱  
> **팀 DreamWorms(왕꿈틀이)** · iOS Developer 4 · Designer 1 · PM 1 (2025.09 ~ 2026.03)  
> 🏆 Apple Developer Academy 2025 **Showcase Spotlight Team 선정** · 🤝 경북지방경찰청 협업  
> [📰 개발팀 외부언론 소개](https://www.notion.so/demn/24-310ca74acc7b8155964fe2f077483ec8) · [✍️ 팀 테크 블로그 (Medium)](https://medium.com/%EC%99%95%EA%BF%88%ED%8B%80%EC%9D%B4-dreamworms)

<br />

## 💭 소개

> 형사들의 신속하고 정확한 추적수사를 위해 피의자 통신 기록과 활동 패턴을 실시간 분석하고 수사 전략을 제안하는 **'지리적 프로파일링'** 앱

<img width="100%" alt="경북지방경찰청 형사기동대 협업" src="Assets/hero.jpeg" />

<br>

[![Swift](https://img.shields.io/badge/Swift-6.0-F05138?style=flat&logo=swift&logoColor=white&labelColor=F05138&color=F05138)](https://swift.org)
[![iOS](https://img.shields.io/badge/iOS-18.0+-000000?style=flat&logo=apple&logoColor=white&labelColor=000000&color=000000)](https://developer.apple.com/ios/)
[![Xcode](https://img.shields.io/badge/Xcode-16.0+-007ACC?style=flat&logo=xcode&logoColor=white&labelColor=007ACC&color=007ACC)](https://developer.apple.com/xcode/)

<br>

## 💡 기능

### 0. 사건 분류
> 배정받은 사건을 사건번호 기준으로 등록하고, 피의자 통신추적 기록을 사건별로 자동 구분·관리합니다.

<img width="100%" alt="사건 분류" src="https://github.com/user-attachments/assets/9676cceb-d035-4bd2-8cdd-c77214283823" />

----

### 1. 메시지에서 위치 데이터 자동 추출
> **App Intent**로 피의자의 통신 기록 메시지를 선택하면, 백그라운드에서 피의자 위치 데이터를 **자동으로 파싱**하여 사건에 즉시 등록합니다.

<img width="100%" alt="App Intent" src="https://github.com/user-attachments/assets/194b83d0-88ae-4e44-a020-9761e32581af" />

----

### 2. 피의자의 현재위치 확인 & 생활 패턴 분석
> 수사24 **`지도 탭`**에서 피의자의 행적과 현재 위치를 확인해 돌발 행동은 없는지 살핍니다.

<img width="100%" alt="지도" src="https://github.com/user-attachments/assets/54d1c455-9228-4b22-b612-62007c0bd1ca" />

----

### 3. 피의자의 수상한 행동 패턴 파악
> 누적 빈도를 통해 피의자가 피해자의 생활 반경에 자주 접근하는 등 특이 위험 패턴을 파악합니다.

<img width="100%" alt="수상한 패턴" src="https://github.com/user-attachments/assets/e99274ca-e893-4637-9fe8-fa6c7f1b6a45" />

----

### 4. 추가 증거 활용
> 피의자 지인으로부터 `제보받은 증거`를 스캔해서 등록하고, 지도에서 `장소 별 연관성을 파악`하고 스토킹 정황을 포착합니다.

<img width="100%" alt="증거" src="https://github.com/user-attachments/assets/a1bb1e40-b305-462b-824d-c4012083139d" />

----

### 5. CCTV 수사
> `추적 탭`에서 피해자 생활반경 내의 `CCTV 리스트`를 확인하고 탐문지역을 명확히 좁힙니다.

<img width="100%" alt="CCTV" src="https://github.com/user-attachments/assets/71135810-55d5-4db0-8982-33daa215d6f5" />

----

### 6. 주요 거점 분석
> `애플의 AI모델(파운데이션 모델)`이 한 줄로 요약해주는 `피의자 주요 거점` 분석 결과를 통해 잠복해야할 곳의 `장소와 시간`을 신속 정확하게 확인해서 `검거 전략`을 세울 수 있습니다.

<img width="100%" alt="AI 분석" src="https://github.com/user-attachments/assets/55d2bb40-666b-41aa-8e5c-21ee065a9725" />

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
