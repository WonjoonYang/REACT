# CallKit으로 "뮤즈와 전화하기" 구현 검토

작성일: 2026-11-13 · 상위 문서: [iphone-muse-voice-call-feasibility.md](./iphone-muse-voice-call-feasibility.md)

## 1. 전제: Meta Muse 앱에 CallKit을 붙일 수는 없다

- CallKit은 **내 앱**의 통화를 iOS 전화 UI에 연결하는 프레임워크다.
- Meta Muse 앱 안에 CallKit을 끼워 넣는 것은 불가능하다.
- 따라서 이 방식은 **자체 "Muse" 앱 + 자체 음성 AI 백엔드**를 만드는 것을 전제로 한다.
- 백엔드로 Meta Muse Spark를 쓰고 싶다면 Meta가 외부에 API를 제공하는지 먼저 확인해야 한다(미확인).

## 2. CallKit으로 얻는 것

| 기능 | 효과 |
|---|---|
| 발신 (`CXStartCallAction`) | 앱 버튼, Siri, 최근 통화 기록, 연락처에서 "뮤즈에게 전화"를 걸 수 있다 |
| 수신 (PushKit VoIP 푸시 → `reportNewIncomingCall`) | **뮤즈가 먼저 전화를 건다.** 잠금 화면에 전체 화면 착신벨이 울리고, 앱이 꺼져 있어도 동작한다 |
| 시스템 통화 UI | 잠금 화면 통화 화면, Dynamic Island, 상단 초록 표시 |
| 오디오 라우팅 | 에어팟, 블루투스, **CarPlay**, 스피커 전환을 시스템이 처리한다 |
| 최근 통화 기록 | 전화 앱의 최근 통화에 "Muse"가 남고, 탭하면 다시 건다 |
| 기본 통화 앱 (iOS 18.2+) | Default Calling App 엔타이틀먼트를 받으면 사용자가 기본 통화 앱으로 지정할 수 있다 |

가장 차별화되는 기능은 **수신**이다. 이런 시나리오가 가능해진다.
- 아침 기상 콜
- 일정 리마인드 전화
- "30분 뒤에 다시 전화해 줘"

Meta Muse 앱에도 없는 경험이다.

## 3. 아키텍처

```
[iPhone 앱]
  CXProvider / CXCallController ── 시스템 통화 UI
  PKPushRegistry (VoIP 토큰) ──┐
  AVAudioEngine (voice processing = 에코 제거)
        │  WebRTC / WebSocket (오디오 스트리밍)
        ▼
[백엔드]
  세션 서버 ── 실시간 음성 AI (speech-to-speech, 또는 STT → LLM → TTS 스트리밍)
  스케줄러 ── APNs VoIP 푸시 ("뮤즈가 전화 걸기")
```

### 발신 흐름
1. 사용자가 앱 버튼이나 Siri로 통화를 시작한다.
2. 앱이 `CXStartCallAction`을 요청한다.
3. `provider(_:perform: CXStartCallAction)`가 호출되면 백엔드 세션을 연다.
4. `provider(_:didActivate: AVAudioSession)`가 호출되면 그때 오디오 엔진을 시작한다.
   - **반드시 이 콜백에서 시작해야 한다.** 그 전에 시작하면 소리가 나지 않는다.

### 수신 흐름
1. 백엔드가 APNs로 VoIP 푸시를 보낸다.
2. iOS가 앱을 깨운다.
3. 앱은 **즉시** `reportNewIncomingCall`을 호출해야 한다.
4. 사용자가 받으면 `CXAnswerCallAction` → 세션을 연결한다.

## 4. 함정 / 제약

- **VoIP 푸시 규칙**
  - iOS 13부터는 VoIP 푸시를 받을 때마다 반드시 CallKit에 착신을 보고해야 한다.
  - 보고하지 않으면 앱이 강제 종료되고, 반복되면 VoIP 푸시 자체가 차단된다.
  - "조용한 알림" 용도로는 쓸 수 없다.
- **App Store 심사**
  - CallKit과 VoIP는 "실제 통화 기능"을 위한 것이다.
  - AI와의 음성 대화가 이 용도로 인정되는지 명확한 기준을 찾지 못했다(위험 요소).
  - 사용자가 직접 예약한 콜백처럼 **사용자 의도가 분명한 수신만** 보내도록 설계해야 안전하다.
- **중국 본토 스토어**에서는 CallKit 사용이 금지되어 있다.
- **시뮬레이터 불가**
  - PushKit과 CallKit 착신은 실기기에서 테스트해야 한다.
  - 유료 Apple Developer 계정이 필요하다.
- **지연시간:** 전화처럼 느끼려면 응답 지연이 1초 이내여야 한다.
  - speech-to-speech 실시간 API를 쓰거나, STT·LLM·TTS를 모두 스트리밍으로 연결해야 한다.
  - 끼어들기(barge-in) 처리도 필요하다.
- **에코:** 스피커폰 모드에서 AI가 자기 목소리를 다시 듣지 않도록 `setVoiceProcessingEnabled(true)`로 에코를 제거해야 한다.
- **LiveCommunicationKit** (iOS 17.4+)
  - Apple이 CallKit의 후속으로 내놓은 VoIP 대화 프레임워크다.
  - 최신 iOS만 지원한다면 함께 검토할 가치가 있다.

## 5. Siri 연동

- `INStartCallIntent`를 처리하면 "Siri야, *앱이름*으로 뮤즈한테 전화해"가 동작한다.
- 기본 통화 앱으로 지정된 경우에는 앱 이름 없이 "뮤즈한테 전화해"만으로 동작할 가능성이 있다.
  - 실기기에서 확인이 필요하다.
- App Intents의 `AppShortcutsProvider`로 "뮤즈 불러" 같은 고정 문구도 추가할 수 있다.

## 6. 구현 스택 선택

| 선택지 | 장점 | 단점 |
|---|---|---|
| Swift 네이티브 | CallKit, PushKit, App Intents를 직접 다뤄 가장 안정적 | — |
| React Native + `react-native-callkeep` + `react-native-voip-push-notification` | 저장소 이름(REACT)과 맞고 UI 개발이 빠름 | 오디오·푸시 경계에서 디버깅이 까다로움. 오디오 엔진은 결국 네이티브 모듈로 작성해야 함 |

## 7. PoC 단계 제안

1. **실기기 발신 PoC**
   - 버튼을 누르면 CallKit 통화 UI가 뜨고, 실시간 음성 AI와 대화한다.
   - 지연시간과 에코를 확인하는 단계다.
2. **Siri 발신:** `INStartCallIntent`와 App Shortcuts 문구를 연결한다.
3. **수신 PoC**
   - 백엔드에서 APNs VoIP 푸시를 보내 착신벨을 울리고, 받으면 대화를 연결한다.
   - "5분 뒤 전화해 줘" 시나리오로 검증한다.
4. (선택) Default Calling App 엔타이틀먼트를 신청하고, CarPlay에서 테스트한다.

## 참고
- [Apple: CallKit](https://developer.apple.com/documentation/callkit)
- [Apple: Preparing your app to be the default calling app](https://developer.apple.com/documentation/callkit/preparing-your-app-to-be-the-default-calling-app)
- [Apple: Initiating VoIP conversations with LiveCommunicationKit](https://developer.apple.com/documentation/livecommunicationkit/initiating-voip-conversations-with-livecommunicationkit)
- [Apple Developer Forums: default calling app on iOS 18.2](https://developer.apple.com/forums/thread/772066)
- [Stream: Integrate CallKit on iOS](https://getstream.io/blog/integrate-callkit-ios/)
