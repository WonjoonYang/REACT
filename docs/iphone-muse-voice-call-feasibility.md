# 아이폰에서 "뮤즈(Muse)"를 불러 음성 통화로 호출하기: 가능성 탐구

작성일: 2026-11-13

## 0. 전제

- "뮤즈"는 **Meta의 Muse 앱**(구 Meta AI 앱, Muse Spark 모델 기반)을 가리킨다고 보았다.
  - Meta는 2026-11-05 Muse 앱에 **실시간 음성 통화(voice call)** 와 아바타 기능을 출시했다.
  - 현재는 **미국 한정**이며, 앱 대화 화면 오른쪽 위의 전화 아이콘을 눌러 시작한다.
- 목표: 앱을 직접 열고 버튼을 누르지 않고, **말 한마디나 버튼 한 번으로** 뮤즈와 통화를 시작하는 것.
- 직접 만든 자체 "Muse" 어시스턴트를 뜻하는 경우는 §4에서 따로 다룬다.

## 1. 결론 요약

| 방법 | 가능 여부 | 호출 방식 | 비고 |
|---|---|---|---|
| "Hey Muse" 상시 웨이크워드 | ❌ 불가 | — | iOS는 서드파티 앱의 백그라운드 상시 마이크 청취를 허용하지 않음 |
| Siri 경유 ("Hey Siri, 뮤즈한테 전화") | ✅ 가능 | 음성 | 단축어 이름을 Siri 문구로 사용. Siri를 한 번 거친다 |
| 액션 버튼 → 단축어 | ✅ 가능(가장 현실적) | 버튼 1회 | iPhone 15 Pro 이상. "Start a call with Muse" 단축어가 존재 |
| 뒷면 탭(Back Tap) → 단축어 | ✅ 가능 | 뒷면 두/세 번 탭 | 손쉬운 사용 › 터치 › 뒷면 탭. 모든 최근 기종 |
| 잠금 화면 / 제어 센터 컨트롤 | ✅ 가능 | 탭 | 단축어 실행 컨트롤 활용 |
| 측면 버튼을 Siri 대신 뮤즈로 지정 | ⚠️ 조건부 | 측면 버튼 길게 | iOS 26.2+, **일본 계정·일본 체류자만**. 앱 개발사(Meta)의 엔타이틀먼트 필요. EU 확대는 미확인 |
| 실제 전화번호로 뮤즈에게 전화 | ❓ 공식 경로 없음 | 전화 앱 | Meta가 Muse용 전화번호를 제공한다는 자료는 찾지 못함. 자체 구현 시 가능(§4) |

**한 줄 결론:** "Hey Muse"처럼 진짜로 이름을 불러 깨우는 기능은 iOS 정책상 불가능하다.
**"Hey Siri, 뮤즈" 또는 액션 버튼 한 번으로 통화를 시작하는 것**은 지금 바로 가능하고, 그게 실질적인 최선이다.

## 2. 왜 "Hey Muse" 웨이크워드는 안 되나

- iOS에서 상시 웨이크워드 감지는 Siri(전용 저전력 하드웨어 경로)만 쓸 수 있다.
- 서드파티 앱은 백그라운드에서 마이크를 계속 열어 둘 수 없다.
  - 오디오 세션을 유지하는 꼼수는 배터리 소모가 크고, 상단에 마이크 표시가 계속 뜨며, App Store 심사에서 거절될 위험이 크다.
- 일본에서 허용된 측면 버튼 대체 기능도 **"버튼 길게 누르기"에만** 적용된다. 음성으로 서드파티 어시스턴트를 깨우는 것은 여전히 불가능하다.

## 3. 지금 바로 할 수 있는 구성 (사용자 입장)

### A. 액션 버튼 (추천)
1. 단축어 앱에서 Muse 앱이 제공하는 액션이나 공개된 "Start a call with Muse" 단축어를 추가한다.
   - Matthew Cassinelli의 Shortcuts Library에 있다.
2. 설정 › 동작 버튼 › 단축어 › 위 단축어를 선택한다.
3. 버튼을 길게 누르면 Muse가 열리면서 바로 통화가 시작된다.

### B. Siri 음성 호출
- 위 단축어의 이름을 "뮤즈 전화"처럼 짧게 바꾼다.
- "Siri야, 뮤즈 전화"라고 말하면 실행된다. 잠금 상태에서는 Face ID 잠금 해제가 필요할 수 있다.

### C. 액션 버튼이 없는 기종
- 뒷면 탭에 같은 단축어를 지정하거나, 제어 센터/잠금 화면 컨트롤로 추가한다.

### 주의
- Muse 음성 통화는 **현재 미국 한정**이다. 한국 계정·지역에서는 통화 기능 자체가 보이지 않을 수 있다.
- 커뮤니티에서는 Muse의 음성 모드 딥링크가 Meta 측에서 막혀 있다는 보고가 있다.
  - 그래서 손쉬운 사용 탭으로 우회하는 실험적 프로젝트도 있지만 불안정하다.
  - 공식 단축어 액션이 없는 환경이라면 "앱 열기"까지만 자동화된다.
- 단축어 설정 화면에 앱 이름이 예전 이름("Hatch" 등)으로 보인다는 보고가 있다. 액션 목록이 앱 업데이트에 따라 바뀔 수 있다.

## 4. 자체 "Muse" 어시스턴트를 직접 만든다면 (개발자 입장)

이 저장소(REACT)에서 직접 구현할 경우 쓸 수 있는 iOS 기술 경로다.

1. **App Intents** (iOS 16+)
   - `StartCallWithMuseIntent`를 정의하고 `AppShortcutsProvider`에 Siri 문구를 등록한다.
     - 예: "Talk to Muse in ${applicationName}"
   - 사용자가 단축어를 따로 만들지 않아도 Siri, 액션 버튼, Spotlight에서 바로 호출할 수 있다.
2. **ControlWidget** (iOS 18+): 제어 센터, 잠금 화면, 액션 버튼에 "뮤즈 호출" 컨트롤을 제공한다.
3. **CallKit + VoIP 푸시(PushKit)**
   - 뮤즈와의 대화를 **실제 전화 UI**(잠금 화면 수신/발신, 최근 통화 기록)로 보여줄 수 있다.
   - 뮤즈가 사용자에게 **먼저 전화를 거는** 시나리오도 가능하다.
   - 단, VoIP 푸시는 수신하면 반드시 CallKit에 실제 통화를 보고해야 한다는 규칙이 있다.
4. **음성 파이프라인**
   - 실시간 음성 LLM API(WebRTC/WebSocket)에 연결한다.
   - 또는 STT → LLM → TTS로 구성한다. 지연을 줄이려면 스트리밍 방식이 필수다.
5. **전화번호 방식**
   - Twilio 같은 전화망 API로 번호를 받아 AI 음성 에이전트에 연결한다.
   - 그러면 "Siri야, 뮤즈한테 전화해"가 **일반 전화 연락처 호출**로 동작한다.
   - 앱 설치가 필요 없고 CarPlay·에어팟에서도 자연스럽다. 대신 통화 요금과 음질 제약이 있다.
6. **측면 버튼 대체** (일본 한정)
   - Side Button Access 엔타이틀먼트를 신청하고, activate 스키마를 따르는 App Intent를 구현한다.

React Native로 만든다면 위 기능은 모두 네이티브(Swift) 모듈로 작성해야 한다.
- App Intents와 위젯은 Swift 타깃이 따로 필요하다.
- CallKit은 `react-native-callkeep` 등의 라이브러리를 쓸 수 있다.

## 5. 다음 단계 제안

- **사용만 하면 되는 경우:** §3-A로 액션 버튼 + Siri 문구를 설정한다. 지역 제한 때문에 통화 기능이 보이지 않으면 현재로서는 방법이 없다.
- **직접 만드는 경우:** 아래 순서의 PoC를 권장한다.
  1. App Intents로 "Siri야, 뮤즈" 호출을 구현한다.
  2. 실시간 음성 API로 통화를 연결한다.
  3. CallKit으로 전화 UI를 입힌다.

## 출처
- [Meta: Introducing Muse Spark (2026-04)](https://about.fb.com/news/2026/04/introducing-muse-spark-meta-superintelligence-labs-first-model-built-to-prioritize-people/)
- [Meta: Muse AI voice & expressive avatars (2026-11)](https://about.fb.com/news/2026/11/introducing-muse-ai-voice-expressive-avatars/amp/)
- [Android Authority: Muse real-time voice chat mode](https://www.androidauthority.com/meta-muse-real-time-voice-chat-mode-3732321/)
- [The Next Web: Muse voice chat and avatars](https://thenextweb.com/news/meta-muse-ai-voice-chat-animated-avatars)
- [Matthew Cassinelli: Muse shortcuts in the Shortcuts Library](https://matthewcassinelli.com/new-shortcuts-library-muse-meta/)
- [GitHub: muse-voice-mode-shortcut (비공식 우회)](https://github.com/jonathancruzate/muse-voice-mode-shortcut)
- [Threads: 단축어에 "Hatch"로 표시된다는 보고](https://www.threads.com/@jasteinerman/post/DdH1U1cEeRq/looks-like-meta-muse-is-still-called-hatch-when-you-try-setting-up-a-shortcut/)
- [MacRumors: iOS 26.2 Japan side button assistant](https://www.macrumors.com/2025/11/17/ios-26-2-iphone-users-japan-siri/)
- [9to5Mac: Replace Siri side button in Japan](https://9to5mac.com/2025/11/17/iphone-replace-siri-side-button-japan-ios-26-2/)
- [TechCrunch: Action Button as ChatGPT voice assistant](https://techcrunch.com/2023/11/22/forget-siri-turn-your-iphones-action-button-into-a-chatgpt-voice-assistant-instead/)
