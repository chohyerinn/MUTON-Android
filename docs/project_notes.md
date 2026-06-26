# Project Notes

## Why I built this

MUTON-Android는 MUTON 백엔드가 만든 자막, 표정 정보, 멀티모달 요약을 실제 사용자가 보는 모바일 화면으로 연결하기 위해 만든 클라이언트다. 백엔드 모델이 있어도 사용자가 편하게 볼 수 있는 앱이 없으면 실시간 대화 보조 서비스로 설명하기 어렵다.

그래서 이 앱은 카메라와 마이크 입력을 받고, 백엔드로 전송하고, 서버가 반환한 자막/감정/요약 결과를 대화 흐름에 맞게 보여주는 데 집중했다.

## What was difficult

가장 어려웠던 점은 백엔드 주소가 자주 바뀌는 문제였다. 데모에서는 Cloudflare Tunnel을 사용했기 때문에 서버를 다시 열 때 URL이 바뀔 수 있었다. 앱 안에 URL을 고정하면 매번 APK를 다시 빌드해야 했다.

또 하나 어려웠던 부분은 API 키를 앱에 넣지 않는 것이었다. conversation record summary 같은 기능을 클라이언트에서 직접 OpenAI API로 호출하면 APK 안에 키가 들어갈 위험이 있다. 그래서 요약 요청은 서버 endpoint를 통해 처리하도록 바꿨다.

## Issues I ran into

### 1. 백엔드 URL 변경 때문에 앱이 자주 깨졌다

초기에는 서버 URL을 앱 코드에 직접 넣는 방식이 단순해 보였다. 하지만 데모 환경에서는 tunnel 주소가 바뀌기 때문에 앱을 다시 빌드해야 했다. 이 문제를 줄이기 위해 GitHub의 `backend_url.json`에서 현재 서버 주소를 읽는 방식으로 바꿨다.

### 2. 카메라/마이크 권한과 네트워크 요청 흐름이 동시에 필요했다

앱은 카메라 frame과 audio chunk를 모두 다룬다. 권한 처리, capture timing, network request가 엮이기 때문에 단순 화면 앱보다 상태 관리가 복잡했다.

### 3. 서버 모델이 바뀌어도 UI가 유지되어야 했다

백엔드가 legacy fusion에서 Qwen server path로 바뀌면서 응답 구조와 설명 방식이 달라질 수 있었다. 앱은 가능한 한 결과를 표시하는 역할에 집중하고, 모델 세부 로직은 서버 쪽에 두는 편이 안정적이었다.

### 4. API 키를 클라이언트에 넣으면 안 됐다

초기에는 앱에서 직접 요약 API를 호출하는 방식도 생각할 수 있었지만, 모바일 앱에 API 키를 포함하는 것은 위험하다. 그래서 conversation record summary는 서버 endpoint로 요청하고, 키는 서버 환경변수에서 관리하도록 했다.

## How I fixed them

- backend URL을 GitHub의 `backend_url.json`에서 동적으로 읽도록 했다.
- 카메라, 마이크, 네트워크 권한을 앱 실행 흐름에 맞게 분리했다.
- Android는 capture와 display에 집중하고, STT/summary/model logic은 백엔드에 두었다.
- conversation record summary는 서버 endpoint를 호출하도록 바꿔 API 키를 APK에 넣지 않게 했다.
- 실제 데모에서는 emulator보다 물리 Android 기기를 우선 사용하도록 README에 적었다.

## What I learned

AI 서비스에서 모바일 클라이언트는 단순히 “결과를 보여주는 화면”이 아니라, 실시간 입력과 서버 상태를 이어주는 중요한 부분이라는 걸 배웠다. 특히 데모에서는 서버 URL, 권한, 네트워크 상태 같은 작은 운영 문제가 전체 경험을 망칠 수 있다.

또 민감한 API 키를 클라이언트에 두지 않는 구조가 중요하다는 것도 배웠다. 기능을 빨리 붙이는 것보다, 어디에서 어떤 요청을 처리해야 안전한지 나누는 것이 필요했다.

## What I would improve next

- 서버 연결 실패 시 사용자에게 더 명확한 상태 메시지를 보여주고 싶다.
- 오디오/비디오 전송 주기를 조절할 수 있는 설정 화면을 추가하고 싶다.
- 실제 기기에서 latency를 기록해 UI 지연을 줄이고 싶다.
- Android repo에도 formal license와 CI build check를 추가하고 싶다.
