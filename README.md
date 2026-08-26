
# WhereYou - iOS App
> 약속 시간 전후에만 위치를 공유하는 지각 방지 iOS 앱

<a href="https://apps.apple.com/kr/app/id6745590209">
    <img src="https://developer.apple.com/assets/elements/badges/download-on-the-app-store.svg" alt="App Store에서 다운로드" height="40" />
  </a>

<p align="center">
  <img src="https://github.com/user-attachments/assets/092f01ee-d4ac-44e6-a59d-be4ed9fb5114" width="100%" alt="웨어유 썸네일"/>
</p>

### 📱 프로젝트 소개

**웨어유(WhereYou)** 는 약속한 모임 시간 전후에 멤버들의 위치가 지도에 표시되는 iOS 앱입니다.

정해진 시간에만 위치를 공유하고 모임이 끝나면 **자동 삭제**되는 구조로 설계했습니다.

---

### ✨ 주요 기능

- **모임 생성·초대** - 시간과 장소를 정한 모임을 만든 후, 멤버를 초대합니다.
  > `Firestore` 실시간 수신
- **실시간 위치 공유** - 모임 시간 3시간 전 · 1시간 후 동안에만 멤버들의 현재 위치가 지도에 함께 표시됩니다.
  > `Core Location` 백그라운드 갱신 · `Realtime Database` 좌표 기록 · `Naver Maps SDK` 지도 표시
- **위치 공유 자동 종료** - 모임 시각이 지나거나 모임에서 나가면 위치 기록이 자동으로 멈추고 올라간 좌표가 정리됩니다.
  > `Cloud Functions` 스케줄러 만료 정리 · 업로드 직전 자격 검증
- **알림으로 바로 진입** - 친구 요청·모임 초대 알림을 누르면 해당 화면으로 바로 들어갑니다.
  > `FCM` + 딥링크 라우팅

---

### 🖥️ 화면 구성

<table align="center">
  <tr align="center">
    <td width="33.33%"><img src="https://github.com/user-attachments/assets/a0e47a9c-07df-4079-b207-06560186d50e" width="200" alt="모임 목록"></td>
    <td width="33.33%"><img src="https://github.com/user-attachments/assets/95d9eaf1-9521-4960-ba73-49b746c2b6c9" width="200" alt="모임 상세"></td>
    <td width="33.33%"><img src="https://github.com/user-attachments/assets/4a11a317-8d2f-454c-8485-61fb22e37657" width="200" alt="친구 추가"></td>
  </tr>
  <tr align="center">
    <td width="33.33%"><img src="https://github.com/user-attachments/assets/7c952cdd-20b8-4f58-815c-6f70227fe0e8" width="200" alt="모임 추가"></td>
    <td width="33.33%"><img src="https://github.com/user-attachments/assets/d2478d75-8076-4c6f-bee4-138ab34ab52a" width="200" alt="멤버 추가"></td>
    <td width="33.33%"><img src="https://github.com/user-attachments/assets/a01ce603-9e44-4255-afb7-eeef36d882b7" width="200" alt="모임 참여"></td>
  </tr>
</table>

---

### 🏗️ 시스템 아키텍처

<p align="center">
  <img src="https://github.com/user-attachments/assets/7639181b-eaed-4a2b-aca1-6e74c47abd62" width="100%" alt="웨어유 시스템 아키텍처"/>
</p>

---

### ⚒️ 기술 스택

| 분류 | 기술 |
|------|------|
| **Language** | Swift 5 |
| **UI Framework** | SwiftUI (UIKit 혼용 - Representable · UIHostingController) |
| **Architecture** | MVVM |
| **Reactive** | Combine (상태 바인딩 · 타이머 구독) |
| **Location** | Core Location (백그라운드 위치 갱신) |
| **Map** | Naver Maps SDK |
| **Notification** | UserNotifications (탭 시 딥링크 라우팅) |
| **Backend** | Firebase (Auth · Firestore · Realtime Database · Cloud Messaging · Remote Config) |
| **Serverless Functions** | Firebase Cloud Functions (Node.js) |
| **Dependency** | Swift Package Manager |
| **CI/CD** | Xcode Cloud |
| **Deployment Target** | iOS 16.6+ |

---

### 🗂️ 프로젝트 구조

```
WhereYou-iOS/
├── iOS_Project.xcworkspace           # SPM 패키지 의존성 관리 워크스페이스
├── functions/                        # Firebase Cloud Functions
│
└── iOS_Project/                      # 앱 타겟
    ├── Deep-Link/                    # 알림 payload 파싱 · 라우팅
    ├── LocationManagers/             # 위치 추적 · 업로드 가드
    ├── Map/                          # 네이버 지도 연동
    ├── Login/ · SignUp/ · FindAuth/  # 인증 - 로그인 / 회원가입 / 계정 찾기
    ├── Meeting/ · MeetingList/       # 모임 상세 · 목록
    ├── AddMeeting/ · EditMeeting/    # 모임 생성 · 수정
    ├── Friend/ · Request/            # 친구 목록 · 친구/모임 요청
    ├── KickOut/ · LeaderSelction/    # 멤버 강퇴 · 모임장 위임
    ├── Profile/                      # 프로필 · 신고
    └── Extension/                    # 키보드 회피 등 공통 처리
```

---

## 🔍 문제 해결

### 1. 로그인에서 한 계정이 여러 기기에 동시 접속되던 문제, 기기 식별자 기반 단일 세션 정책으로 해결

<img width="90%" alt="기기 전환과 유예 구간" src="https://github.com/user-attachments/assets/ad3cf3d5-0706-4ce6-b467-d146e042abae"/> <br>

**📍 제약 상황**

- 인증 세션은 기기마다 독립 유지 → **같은 계정으로 다른 기기에서 로그인해도 앱 쪽에 어떤 신호도 오지 않음.** 위치를 공유하는 앱인데 한 계정이 여러 기기에서 동시에 좌표를 올릴 수 있음
- 단일 세션을 강제하자 이번에는 기기를 교체한 직후 방금 로그인한 기기가 자기 갱신 신호를 보고 스스로 로그아웃

**⚖️ 선택한 구조와 이유**

새 기기를 아예 막으면 기기를 바꾼 사용자가 들어올 수 없고, 기존 기기를 무조건 세션 종료 처리하면 방금 로그인한 기기가 자기 갱신 신호에 반응해 접속이 해제됩니다. 두 경우 모두 도착한 신호가 자기 것인지 남의 것인지 구분할 수단이 없기 때문입니다. **기기를 식별자로 구분하고, 방금 바꾼 기기에는 유예를 두기로 했습니다.**

**✅ 결과**

- 로그인 시 `identifierForVendor` 기반 식별자를 사용자 문서에 기록하고 전 기기는 스냅샷 리스너로 불일치를 감지해 스스로 로그아웃 - 같은 계정이 여러 기기에서 동시에 좌표를 올리던 경로 차단
- 새 기기는 `유예 구간` 동안 도착하는 스냅샷을 무시 - 기기 교체 직후 새 기기가 스스로 로그아웃되던 충돌 해소
- 강제 로그아웃 알림과 기기 전환 확인 알림을 하나의 알림 타입으로 통합 - 두 알림이 서로를 덮어쓰던 문제 정리
- 로그인 상태·기기 식별자·알림 토큰이 사용자 문서 한 곳에서 함께 갱신·삭제됨

<details>
<summary>근거</summary>

- 코드 - [`LoginViewModel.swift:16-25`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Login/LoginViewModel.swift#L16-L25)(알림 타입 정의) · [`:39`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Login/LoginViewModel.swift#L39)(기기 식별자 획득) · [`:42`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Login/LoginViewModel.swift#L42)(유예 구간 상수) · [`:64-113`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Login/LoginViewModel.swift#L64-L113)(로그인·기기 식별자 기록) · [`:147-192`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Login/LoginViewModel.swift#L147-L192)(기기 전환 확정 · `:151-152` 유예 시작) · [`:222-249`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Login/LoginViewModel.swift#L222-L249)(강제 로그아웃 스냅샷 리스너 · `:236-238` 유예 중 스냅샷 무시)
- 커밋 - [`0979b4a`](https://github.com/ChanSolShin/WhereYou-iOS/commit/0979b4a) [`f1371b1`](https://github.com/ChanSolShin/WhereYou-iOS/commit/f1371b1) [`977d012`](https://github.com/ChanSolShin/WhereYou-iOS/commit/977d012) [`93f7b7d`](https://github.com/ChanSolShin/WhereYou-iOS/commit/93f7b7d) [`2a7c236`](https://github.com/ChanSolShin/WhereYou-iOS/commit/2a7c236) [`fe28f9a`](https://github.com/ChanSolShin/WhereYou-iOS/commit/fe28f9a) [`d7abf3e`](https://github.com/ChanSolShin/WhereYou-iOS/commit/d7abf3e) / [PR #44](https://github.com/ChanSolShin/WhereYou-iOS/pull/44)
- 검증 - 실기기 수동 재현

</details>

---

### 2. 푸시 알림 진입에서 탭·네비게이션 스택 상태가 어긋나던 문제, 1회성 소비형 전역 라우터로 해결

<img width="90%" alt="딥링크 라우팅 단일 진입점" src="https://github.com/user-attachments/assets/3b6802ff-54d8-4fd3-abef-2f73e45c8fef"/> <br>

**📍 제약 상황**

- 알림을 받는 곳은 `AppDelegate`, 탭 컨테이너는 `UITabBarController`를 감싼 UIKit 뷰, 화면 전환 권한은 그 안의 각 SwiftUI 뷰가 나눠 보유 → **수신 지점과 전환 지점 사이에 상태를 옮길 공용 통로 없음**
- 다른 탭으로 넘어가면 떠나온 탭이 이전 상세 화면을 붙들고 있고, 같은 알림을 다시 누르면 동일 화면이 겹쳐 쌓임

**⚖️ 선택한 구조와 이유**

각 탭이 자기 알림을 직접 처리하게 하면 탭마다 같은 분기가 복제되고, 반대로 알림 수신부에서 화면을 직접 밀어 올리면 그 시점에 어떤 탭이 떠 있는지 알 수 없습니다. 두 방식 모두 알림을 받는 곳과 화면을 바꾸는 곳이 서로의 상태를 모른다는 점은 그대로입니다. **알림을 상태 신호로 바꿔 한 곳에서 발행하고 각 탭이 한 번만 소비하게 했습니다.**

**✅ 결과**

- 서버 payload의 `route` 필드를 파서가 목적지 열거형으로 변환, 전역 라우터가 `선택 탭`·`대기 경로`·`루트 복귀` 세 신호를 발행 - 알림 수신부가 화면을 직접 밀어 올리던 경로 제거
- 각 탭 루트 뷰가 신호를 한 번만 가져간 뒤 비움 - 같은 알림을 반복해 눌러도 동일 화면이 겹쳐 쌓이지 않음
- 목적지를 모델이 아니라 식별자로 넘겨 로컬에 없으면 단건 조회로 채움 - 앱 종료 상태에서 알림으로 실행된 경우도 같은 경로를 탐
- 직전 목적지를 기억해 같은 식별자면 중복 push를 건너뜀

<details>
<summary>근거</summary>

- 코드 - [`Deep-Link/AppRoute.swift:12-23`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Deep-Link/AppRoute.swift#L12-L23)(목적지 열거형 정의) · [`AppRouter.swift:19-49`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Deep-Link/AppRouter.swift#L19-L49)(선택 탭·대기 경로·루트 복귀 신호 발행) · [`NotificationPayloadParser.swift:13-33`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/Deep-Link/NotificationPayloadParser.swift#L13-L33)(payload `route` 필드 → 목적지 변환)
- 연결부 - [`AppDelegate.swift:33-38`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/AppDelegate.swift#L33-L38)(알림 수신 등록) · [`:111-128`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/AppDelegate.swift#L111-L128)(알림 탭 → 라우터로 전달) · [`MainTapView.swift:55-103`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/MainTapView.swift#L55-L103)(선택 탭 신호 수신·탭 전환) · [`MeetingListView.swift:155-179`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/MeetingList/MeetingListView.swift#L155-L179)(식별자 단건 조회 후 상세 진입) · [`:185-258`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/MeetingList/MeetingListView.swift#L185-L258)(대기 경로 1회 소비 후 비움)
- 서버 - [`functions/index.js:37-39`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/functions/index.js#L37-L39)(친구 요청 도착 알림) · [`:131-134`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/functions/index.js#L131-L134)(모임 초대 도착 알림) · [`:227-231`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/functions/index.js#L227-L231)(위치 조회 개시 알림 - 1분 주기 스케줄러) · [`:331-334`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/functions/index.js#L331-L334)(새 멤버 합류 알림) · [`:412-415`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/functions/index.js#L412-L415)(모임 정보 변경 알림) - 다섯 지점 모두 payload에 `route` 부여
- 커밋 - [`0e28096`](https://github.com/ChanSolShin/WhereYou-iOS/commit/0e28096) [`6e80257`](https://github.com/ChanSolShin/WhereYou-iOS/commit/6e80257) [`2d99e9e`](https://github.com/ChanSolShin/WhereYou-iOS/commit/2d99e9e) [`dab796e`](https://github.com/ChanSolShin/WhereYou-iOS/commit/dab796e) [`9a0fea8`](https://github.com/ChanSolShin/WhereYou-iOS/commit/9a0fea8) [`badbc57`](https://github.com/ChanSolShin/WhereYou-iOS/commit/badbc57) [`6450ec3`](https://github.com/ChanSolShin/WhereYou-iOS/commit/6450ec3) / [PR #86](https://github.com/ChanSolShin/WhereYou-iOS/pull/86)
- 검증 - 실기기 수동 재현

</details>

---

### 3. 모임 위치 공유에서 만료·비멤버 상태의 좌표가 계속 올라가던 문제, 시간창·멤버십 이중 가드와 서버 트리거 정리로 해결

<img width="90%" alt="좌표 업로드 검증 게이트" src="https://github.com/user-attachments/assets/ec5a002d-91d4-41f5-9e56-2405edf1daf6"/> <br>

**📍 제약 상황**

- 업로드 타이머가 한 번 켜지면 활성 모임 배열이 빌 때까지 계속 도는데, **이 배열은 예약된 타이머로만 정리되도록 설계** → 백그라운드 전환이나 종료 후 복귀 시 예약이 유실
- 모임 시각이 지난 뒤에도, 모임장에게 강퇴당한 뒤에도 해당 모임 경로로 자기 좌표가 계속 기록됨

**⚖️ 선택한 구조와 이유**

타이머를 예약해 두고 시간이 되면 끄는 방식은 앱이 백그라운드로 내려가거나 종료됐다 돌아오면 예약 자체가 사라집니다. 클라이언트에서만 막으면 이미 올라간 좌표는 그대로 남습니다. 두 방식 모두 업로드가 시작된 뒤의 상태 변화를 따라가지 못합니다. **업로드 직전마다 자격을 다시 확인하고, 이미 올라간 것은 서버가 지우게 했습니다.**

**✅ 결과**

- 업로드 루프 진입 직전·포그라운드 복귀 직후·스냅샷 변경 시 시간창을 다시 훑음 - 백그라운드 복귀로 예약이 유실돼도 업로드가 계속되던 경로 제거
- 멤버십이 확인된 모임 식별자만 담는 `화이트리스트`를 두고 시간창과 동시에 만족할 때만 기록 - 모임 시각이 지난 뒤와 강퇴 후 좌표가 올라가던 경로 차단
- 멤버 배열에서 사라진 사용자의 좌표 노드는 서버 트리거가 삭제 - 클라이언트 차단만으로 남던 잔여 제거
- 자격 확인 지점이 세 곳으로 고정 - 진입·복귀·변경 감지 어느 경로로 와도 같은 검사를 거침

<details>
<summary>근거</summary>

- 코드 - [`AppLocationCoordinator.swift:19`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L19)(멤버십 확인된 모임 식별자 화이트리스트) · [`:34-43`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L34-L43)(포그라운드 복귀 직후 시간창 재확인) · [`:85-106`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L85-L106)(모임 시간창 검증) · [`:108-136`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L108-L136)(업로드 직전 자격 가드) · [`:161-187`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L161-L187)(리스너 등록 시 초기 검증) · [`:196-202`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L196-L202)(모임 문서 삭제 감지) · [`:203-218`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L203-L218)(멤버 목록에서 본인 제거 감지) · [`:224-267`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/iOS_Project/LocationManagers/AppLocationCoordinator.swift#L224-L267)(업로드 시작·종료 스케줄링)
- 서버 - [`functions/index.js:533-572`](https://github.com/ChanSolShin/WhereYou-iOS/blob/c4685c7328c21fbf4eb40d42cdc23b7489fb21ea/functions/index.js#L533-L572)(강퇴·탈퇴 멤버의 좌표 노드 삭제 - 알림 발송 없는 데이터 정리 트리거)
- 커밋 - [`215a5dc`](https://github.com/ChanSolShin/WhereYou-iOS/commit/215a5dc) [`4789c0b`](https://github.com/ChanSolShin/WhereYou-iOS/commit/4789c0b) [`36fce56`](https://github.com/ChanSolShin/WhereYou-iOS/commit/36fce56) [`8417e10`](https://github.com/ChanSolShin/WhereYou-iOS/commit/8417e10) [`9f4cc2d`](https://github.com/ChanSolShin/WhereYou-iOS/commit/9f4cc2d) [`335c5e1`](https://github.com/ChanSolShin/WhereYou-iOS/commit/335c5e1) / [PR #70](https://github.com/ChanSolShin/WhereYou-iOS/pull/70) [#72](https://github.com/ChanSolShin/WhereYou-iOS/pull/72) [#84](https://github.com/ChanSolShin/WhereYou-iOS/pull/84)
- 검증 - 실기기 수동 재현

</details>

---

## 👥 팀원 소개

| **iOS Developer** | **iOS Developer** |
| :---: | :---: |
| <a href="https://github.com/HunCY5"><img src="https://github.com/HunCY5.png" width="200px" alt="HunCY5"/></a> | <a href="https://github.com/ChanSolShin"><img src="https://github.com/ChanSolShin.png" width="200px" alt="ChanSolShin"/></a> |
| [**HunCY5**](https://github.com/HunCY5) | [**ChanSolShin**](https://github.com/ChanSolShin) |

---

## 📌 브랜치 전략

| 브랜치 | 용도 | 병합 대상 |
|--------|------|------|
| `main` | 앱스토어 출시용 | - |
| `release` | 출시 준비 | `main`, `develop` |
| `hotfix` | 배포 버전 버그 수정 | `main`, `develop` |
| `develop` | 개발 완료 | `release` |
| `feature` | 기능 개발 | `develop` |

## 📌 커밋 컨벤션

| 태그 | 설명 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 추가 | feat.로그인 화면 구현 |
| `fix` | 버그 수정 | fix.로그인 시도 시 오류 해결 |
| `chore` | 빌드, 패키지 등 기타 작업 | chore.Tuist 설정 수정 |
| `refactor` | 코드 리팩토링 | refactor.뷰 레이아웃 정리 |
| `docs` | 문서 수정 | docs.Tuist 사용법 작성 |
| `style` | 포맷팅·주석 등 | style.코드 정렬 및 주석 수정 |
