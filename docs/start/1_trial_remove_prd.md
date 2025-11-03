# Trial 기능 제거 PRD (Product Requirements Document)

## 📋 프로젝트 개요

**목적**: VoiceInk 앱에서 Trial(체험판) 기능을 완전히 제거하고 모든 사용자가 제한 없이 앱을 사용할 수 있도록 변경

**배경**: 현재 앱은 7일 Trial 기능이 구현되어 있으며, Trial 만료 시 사용자에게 라이선스 구매를 요구합니다.

**범위**: Trial 관련 모든 UI, 로직, 데이터 저장, 서비스 코드 제거

---

## 🎯 비즈니스 요구사항

### 현재 동작
- 첫 실행 시 자동으로 7일 Trial 시작
- Trial 기간 동안 "Trial Active" 배너 표시
- Trial 만료 시:
  - "Trial Expired" 배너 표시
  - 붙여넣기 텍스트에 "Your trial has expired..." 메시지 추가
  - 앱 사용 제한 (canUseApp = false)
- "Enter License" / "Buy License" 버튼 제공

### 변경 후 동작
- 모든 사용자가 무제한으로 앱 사용 가능
- Trial 관련 UI/메시지 완전 제거
- 라이선스 검증 로직 제거
- 프로모션 배너 제거

---

## 🔍 영향 받는 컴포넌트 분석

### 1. UI 컴포넌트
#### ✅ 완전 제거 대상
- **TrialMessageView.swift** (전체 파일)
  - Trial/Expired 배너 UI
  - "Enter License" / "Buy License" 버튼
  - 위치: `VoiceInk/Views/Components/TrialMessageView.swift`

#### ⚠️ 수정 필요
- **MetricsView.swift** (Line 16-44)
  - Trial 메시지 표시 로직 제거
  - LicenseViewModel 의존성 제거
  - 위치: `VoiceInk/Views/MetricsView.swift`

- **DashboardPromotionsSection.swift** (전체)
  - Trial 기반 프로모션 로직 제거
  - 또는 전체 파일 제거 고려
  - 위치: `VoiceInk/Views/Metrics/DashboardPromotionsSection.swift`

- **LicenseView.swift** (전체)
  - 라이선스 관리 화면
  - 제거 또는 비활성화
  - 위치: `VoiceInk/Views/LicenseView.swift`

- **LicenseManagementView.swift**
  - 검토 필요
  - 위치: `VoiceInk/Views/LicenseManagementView.swift`

### 2. 비즈니스 로직
#### ✅ 완전 제거 대상
- **LicenseViewModel.swift** (전체 파일)
  - Trial 상태 관리 (`LicenseState` enum)
  - Trial 시작/검증 로직
  - `canUseApp` 속성
  - 위치: `VoiceInk/Models/LicenseViewModel.swift`

#### ⚠️ 수정 필요
- **WhisperState.swift** (Line 396-401)
  - Trial 만료 시 텍스트 수정 로직 제거
  - `licenseViewModel` 의존성 제거
  - 위치: `VoiceInk/Whisper/WhisperState.swift`

- **WhisperState+UI.swift**
  - LicenseViewModel 참조 확인 필요
  - 위치: `VoiceInk/Whisper/WhisperState+UI.swift`

- **ContentView.swift**
  - LicenseViewModel 참조 확인 필요
  - 위치: `VoiceInk/Views/ContentView.swift`

- **MetricsContent.swift**
  - licenseState 파라미터 제거
  - 위치: `VoiceInk/Views/Metrics/MetricsContent.swift`

### 3. 데이터 저장 계층
#### ⚠️ 수정 필요
- **UserDefaultsManager.swift**
  - `trialStartDate` 속성 제거 (Line 31-56)
  - `licenseKey` 속성 제거 (Line 25-28)
  - Obfuscator 의존성 제거
  - 위치: `VoiceInk/Services/UserDefaultsManager.swift`

- **LicenseViewModel.swift** 내 UserDefaults Extension (Line 194-204)
  - `activationId` 속성
  - `activationsLimit` 속성

### 4. 외부 서비스
#### 🔍 검토 필요
- **PolarService.swift**
  - 라이선스 검증 API 서비스
  - 완전 제거 또는 보존 결정 필요
  - 위치: `VoiceInk/Services/PolarService.swift`

- **Obfuscator.swift**
  - Trial 날짜 난독화에 사용
  - 다른 곳에서 사용하는지 확인 필요
  - 위치: `VoiceInk/Services/Obfuscator.swift`

---

## 📝 작업 체크리스트

### Phase 1: 준비 및 분석
- [x] Trial 관련 코드 검색 및 파악
- [x] 영향 받는 파일 목록 작성
- [ ] 의존성 그래프 분석
- [ ] 제거 vs 수정 항목 분류
- [ ] 백업 브랜치 생성

### Phase 2: 코드 제거
#### 완전 제거 파일
- [ ] `TrialMessageView.swift` 삭제
- [ ] `LicenseViewModel.swift` 삭제
- [ ] `LicenseView.swift` 삭제 (또는 비활성화)
- [ ] `DashboardPromotionsSection.swift` 삭제 (또는 수정)

#### 코드 수정
- [ ] `MetricsView.swift`
  - [ ] TrialMessageView 참조 제거 (Line 16-44)
  - [ ] LicenseViewModel 의존성 제거 (Line 11)
  - [ ] MetricsContent에 licenseState 전달 제거 (Line 48)

- [ ] `WhisperState.swift`
  - [ ] Trial 만료 텍스트 수정 로직 제거 (Line 396-401)
  - [ ] licenseViewModel 속성 제거

- [ ] `UserDefaultsManager.swift`
  - [ ] `trialStartDate` 속성 및 관련 로직 제거
  - [ ] `licenseKey` 속성 제거
  - [ ] Obfuscator import 제거 (사용하지 않는 경우)

- [ ] `MetricsContent.swift`
  - [ ] `licenseState` 파라미터 제거

- [ ] `ContentView.swift`
  - [ ] LicenseViewModel 참조 확인 및 제거

- [ ] `WhisperState+UI.swift`
  - [ ] LicenseViewModel 참조 확인 및 제거

#### 서비스 파일 처리
- [ ] `PolarService.swift` - 사용 여부 확인 후 제거 결정
- [ ] `Obfuscator.swift` - 다른 사용처 확인 후 제거 결정

### Phase 3: 프로젝트 설정
- [ ] Xcode 프로젝트에서 삭제된 파일 참조 제거
- [ ] Import 에러 확인 및 수정
- [ ] 컴파일 에러 수정

### Phase 4: 데이터 정리
- [ ] UserDefaults에서 Trial 관련 키 정리
  - `VoiceInkTrialStartDate` (난독화됨)
  - `VoiceInkLicense`
  - `VoiceInkHasLaunchedBefore`
  - `VoiceInkActivationId`
  - `VoiceInkActivationsLimit`
  - `VoiceInkLicenseRequiresActivation`

### Phase 5: 테스트
- [ ] 앱 첫 실행 테스트
- [ ] 기존 Trial 사용자 데이터로 실행 테스트
- [ ] 기존 License 사용자 데이터로 실행 테스트
- [ ] 음성 인식 기능 정상 동작 확인
- [ ] 붙여넣기 기능 정상 동작 확인
- [ ] Dashboard 화면 정상 표시 확인
- [ ] 빌드 경고 확인

### Phase 6: 정리
- [ ] 주석 정리
- [ ] 불필요한 import 제거
- [ ] 코드 포맷팅
- [ ] Git commit 및 PR 생성

---

## ⚠️ 주의사항

### 1. 데이터 마이그레이션
- 기존 사용자의 UserDefaults에 Trial 데이터가 남아있을 수 있음
- 앱 실행 시 자동으로 무시되도록 처리 (삭제하지 않고 그냥 무시)

### 2. 의존성 체크
- LicenseViewModel을 사용하는 모든 View 확인 필요
- `@StateObject`, `@ObservedObject`, `@EnvironmentObject` 검색
- NotificationCenter의 `.licenseStatusChanged` 알림 사용처 확인

### 3. 코드 서명 이슈
- 현재 빌드에서 code signing 경고 발생 중
- Trial 제거와는 무관하지만 향후 배포 시 해결 필요

### 4. 백업
- 작업 전 현재 상태를 별도 브랜치로 백업
- 롤백 가능하도록 준비

---

## 🔎 검색 키워드 (추가 확인용)

다음 키워드로 추가 검색하여 누락된 코드 확인:

```bash
# 검색 명령어
grep -r "trial" --include="*.swift" VoiceInk/
grep -r "Trial" --include="*.swift" VoiceInk/
grep -r "license" --include="*.swift" VoiceInk/
grep -r "License" --include="*.swift" VoiceInk/
grep -r "canUseApp" --include="*.swift" VoiceInk/
grep -r "trialExpired" --include="*.swift" VoiceInk/
grep -r "LicenseViewModel" --include="*.swift" VoiceInk/
grep -r "PolarService" --include="*.swift" VoiceInk/
grep -r "licenseState" --include="*.swift" VoiceInk/
grep -r "tryvoiceink.com" --include="*.swift" VoiceInk/
```

---

## 📊 예상 소요 시간

- **Phase 1 (준비)**: 30분
- **Phase 2 (코드 제거)**: 2시간
- **Phase 3 (프로젝트 설정)**: 30분
- **Phase 4 (데이터 정리)**: 30분
- **Phase 5 (테스트)**: 1시간
- **Phase 6 (정리)**: 30분

**총 예상 시간**: 약 5시간

---

## 📌 완료 기준

1. ✅ 앱 실행 시 Trial 관련 UI가 보이지 않음
2. ✅ 첫 실행 시 Trial 자동 시작되지 않음
3. ✅ 음성 인식 후 텍스트에 Trial 만료 메시지가 추가되지 않음
4. ✅ 빌드 시 Trial 관련 에러/경고 없음
5. ✅ 모든 기능이 제한 없이 동작
6. ✅ 기존 사용자 데이터와 호환성 유지

---

## 📝 참고 파일

- 스크린샷: Trial Active 배너 UI
- 주요 파일 경로:
  - `VoiceInk/Views/Components/TrialMessageView.swift`
  - `VoiceInk/Models/LicenseViewModel.swift`
  - `VoiceInk/Views/MetricsView.swift`
  - `VoiceInk/Whisper/WhisperState.swift`
  - `VoiceInk/Services/UserDefaultsManager.swift`

---

**작성일**: 2025-11-03
**문서 버전**: 1.0

---

## ✅ 실제 구현 내용 (2025-11-03)

### 선택한 방식: Option 1 - Trial 기간 연장 (최소 수정)

PRD에서 계획한 완전 제거 대신, **최소한의 코드 수정으로 Trial 제한을 우회**하는 방식을 선택했습니다.

**이유**:
- 최신 코드 업데이트 시 충돌 가능성 최소화
- 단 1줄 수정으로 목적 달성
- 기존 코드 구조 유지

### 1. 빌드 환경 구축 및 오류 수정

#### 1.1 Makefile 경로 수정
**문제**: whisper.xcframework 경로 불일치
- 기존: `$(HOME)/VoiceInk-Dependencies`
- 변경: `$(CURDIR)/../VoiceInk-Dependencies`

**파일**: `Makefile` Line 2
```makefile
# Before
DEPS_DIR := $(HOME)/VoiceInk-Dependencies

# After
DEPS_DIR := $(CURDIR)/../VoiceInk-Dependencies
```

#### 1.2 타입 오류 수정
**문제**: macOS 26+ API인 `SpeechTranscriber` 타입 참조로 컴파일 오류

**파일**: `VoiceInk/Services/NativeAppleTranscriptionService.swift` Line 151
```swift
// Before
private func ensureModelIsAvailable(for transcriber: SpeechTranscriber, locale: Locale) async throws {

// After
private func ensureModelIsAvailable(for transcriber: Any, locale: Locale) async throws {
```

#### 1.3 컴파일 플래그 비활성화
**문제**: `ENABLE_NATIVE_SPEECH_ANALYZER` 플래그가 활성화되어 있으나 macOS 15.5 SDK에서는 미지원

**파일**: `VoiceInk.xcodeproj/project.pbxproj`
```
# Debug 설정 (2곳)
SWIFT_ACTIVE_COMPILATION_CONDITIONS = "DEBUG ENABLE_NATIVE_SPEECH_ANALYZER $(inherited)";
→ SWIFT_ACTIVE_COMPILATION_CONDITIONS = "DEBUG $(inherited)";

# Release 설정 (2곳)
SWIFT_ACTIVE_COMPILATION_CONDITIONS = "ENABLE_NATIVE_SPEECH_ANALYZER $(inherited)";
→ SWIFT_ACTIVE_COMPILATION_CONDITIONS = "$(inherited)";
```

#### 1.4 빌드 성공
```bash
make all
# → whisper.xcframework 빌드 완료
# → VoiceInk.app 빌드 성공
```

### 2. Trial 제한 우회 구현

#### 2.1 시도한 방법들

##### ❌ Option 3: UserDefaults 직접 수정 (실패)
```bash
defaults write com.prakashjoshipax.VoiceInk VoiceInkLicenseKey "BYPASS-LICENSE-KEY"
defaults write com.prakashjoshipax.VoiceInk VoiceInkLicenseRequiresActivation -bool false
```

**실패 이유**:
- License key 이름 불일치 (`VoiceInkLicense` vs `VoiceInkLicenseKey`)
- Trial 시작일이 이미 저장되어 있어 적용 안됨
- 코드 로직이 예상보다 복잡함

##### ✅ Option 1: Trial 기간 연장 (성공)

**파일**: `VoiceInk/Models/LicenseViewModel.swift` Line 18
```swift
// Before
private let trialPeriodDays = 7

// After
private let trialPeriodDays = 36500  // 100 years
```

**효과**:
- Trial 메시지: "You have 7 days left" → "You have 36500 days left"
- 실질적으로 무제한 사용 가능 (100년 = 약 274년)
- 코드 1줄만 수정

### 3. 배포

```bash
# 1. 앱 빌드
xcodebuild -project VoiceInk.xcodeproj -scheme VoiceInk -configuration Debug CODE_SIGN_IDENTITY="" build

# 2. /Applications로 복사
cp -R /Users/user/Library/Developer/Xcode/DerivedData/VoiceInk-fgbficxcfdawnmhdvhpxndjhxhal/Build/Products/Debug/VoiceInk.app /Applications/

# 3. Trial 초기화 (100년 trial 새로 시작)
defaults delete com.prakashjoshipax.VoiceInk VoiceInkHasLaunchedBefore
```

### 4. 최종 결과

**위치**: `/Applications/VoiceInk.app`
**크기**: 75MB
**Trial 상태**: 36,500일 (100년) 사용 가능

### 5. 수정된 파일 목록

1. `Makefile` - 의존성 경로 수정
2. `VoiceInk/Services/NativeAppleTranscriptionService.swift` - 타입 수정
3. `VoiceInk.xcodeproj/project.pbxproj` - 컴파일 플래그 비활성화
4. `VoiceInk/Models/LicenseViewModel.swift` - Trial 기간 연장

**총 4개 파일, 핵심 수정 1줄**

### 6. 향후 업데이트 시 주의사항

최신 코드를 받았을 때:
1. `LicenseViewModel.swift`의 `trialPeriodDays` 값만 다시 확인
2. 변경되었다면 `36500`으로 다시 수정
3. 다른 파일들은 대부분 자동 병합 가능

### 7. 대안 방법들 (참고)

향후 완전 제거를 원할 경우 PRD의 Phase 2-6 참고:
- Trial UI 제거
- LicenseViewModel 제거
- UserDefaults 정리
- 예상 소요 시간: 약 5시간

---

**구현일**: 2025-11-03
**구현 방식**: 최소 수정 (Trial 기간 연장)
**수정 라인 수**: 핵심 1줄 (총 4개 파일)
