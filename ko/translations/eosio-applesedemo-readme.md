# EOSIO Apple Secure Enclave 데모

EOSIO는 타원 곡선 전자 서명 알고리즘(ECDSA)에 대해 두 가지 타원 곡선을
지원합니다. 각각 secp256k1과 (prime256v1으로 불리기도 하는) secp256r1입니다.
r1 곡선의 장점은 아이폰이나 안드로이드 디바이스와 같은 많은 모바일 기기에서
하드웨어 지원이 가능하다는 점입니다. 이 어플리케이션은 터치바가 장착된
최근의 맥북 프로 노트북에 있는 Secure Enclave를 통해서 하드웨어 장치 내에서
r1 곡선을 사용하는 데모입니다. r1 비밀키는 secure enclave에 저장되고,
호스트 시스템에 보이지 않습니다. 이 비밀키는 (EOSIO 트랜잭션의 해시로
사용될 수 있는) SHA256 다이제스트를 secure enclave를 거쳐서 서명하는데
사용할 수 있습니다. 적절한 작동인지 검증하기 위해서, 어플리케이션은
EOSIO 코드 베이스에서 공개키 복구의 검증을 가능케 합니다.
 
# 빌드하고 사인하기
`applesedemo` 어플리케이션은 EOSIO 소스 트리에서 macOS 빌드로 컴파일됩니다.
사용하는 API는 macOS 10.12 이상이 필요합니다(iOS 10 이상에서도 역시 작동할
것입니다). 그러나 Secure Enclave를 사용하기 위해선 어플리케이션은
Mac App Store (MAS) 스타일의 서명으로 사인되어야 합니다 -- 과거의 다윈
스타일 서명으로는 충분하지 않습니다. 이런 강한 서명 체계로,
특정 비밀키를 생성한  어플리케이션이 해당 비밀키를 사용할 수 있는 유일한
어플리케이션임을 보장합니다.

`applesedemo`를 MAS 스타일 어플리케이션으로 서명하는 작업은 두 가지 면에서
다루기 힘듭니다. 1) 이 서명은 커맨드 라인 어플리케이션으로 썩 어울리지
않는 "app bundle" 형식으로 패키징해야 합니다. 2) 모든 빌드 과정을
XCode 사용 없이 유지해야 합니다.

`sign.sh` 스크립트는 app bundle `applesedemp.app`를 생성하고 서명합니다.
이 작업은 서명되지 않는 `applesedemo` 가 현재 작업 디렉토리에 있는 상황에서
진행되어야만 합니다. 세 가지 파라미터가 필요합니다.

1. 서명에 사용할 인증서의 SHA1 thumbprint
2. 서명된 어플리케이션의 완전한 AppID (teamID + bundleID)
3. 서명에 사용할 프로비저닝 프로파일 경로

CMakeLists.txt에 주석 처리된 예제가 하나 있는데 이 예제는 빌드 이후에
자동으로 실행될 수 있는 방법을 보여줍니다.

서명한 후, `applesedemo`를 app build 안에서 번들된 형태인
`applesedemo.app/Contents/MacOS/applesedemo`로 실행해야 합니다.
서명하지 않은 `applesedemo`를 실행할 경우 에러를 출력하는 
대충 만든 체크가 하나 있지만, 유효하지 않게 서명된 어플리케이션의
경우를 잡아내지 못할지도 모릅니다. 유효하지 않게 서명된
어플리케이션이 Secure Enclave를 사용하려고 시도하면 
도움이 되지 않는 에러 메시지를 만나게 될 것입니다.

## Personal Signing

Apple Developer Program에 계정이 있다면 개발 포탈에 들어가서
프로비저닝 프로파일을 다운로드하고/거나 아직 인증서가 없다면
개발자 인증서를 생성할 수 있습니다.

그러나 Developer 계정이 없는 경우라도 아무 Apple ID를 이용해서
로컬 개발 환경에서 사용할 수 있는 개발용 인증서와 프로비저닝
프로파일을 생성할 수 있습니다. 불행하게도, 이 과정은 
XCode가 자동으로 "관리하는" 서명 메커니즘으로 진행됩니다.
XCode 안에서 Cocoa 앱을 만들고, 적절한 번들ID를 할당하고,
사용할 Team에 대응하는 "Personal Team"을 선택해야 합니다.
XCode는 애플 서버에 접속해서 아직 없다면 적절한 프로비저닝
프로파일과 인증서를 생성할 것입니다.
`~/Library/MobileDevice/Provisioning\ Profiles/` 디렉토리를
들여다보면서 어떤 프로파일이 사용하기에 적절한지 
손으로 찾아보아야 할지도 모릅니다.

XCode가 Secure Enclave에 접근하기 위해 사용하는 TeamID와 Bundle ID를
기억해두세요. 둘 중 어느 하나를 값을 변경하면, 변경 전의 값으로
생성한 키를 볼 수 없게 됩니다.

# 사용법
## 비밀키 생성
먼저, Secure Enclave 내부에서 비밀키를 생성합니다. 이 어플리케이션은
키를 보호할 보안 레벨을 제어할 수 있는 몇 가지 다른 방법으로
키를 생성하는 기능을 지원합니다 (`--help` 도움말을 확인하세요).
이 예제에서는, 다이제스트를 서명할 때(트랜잭션에 서명하는 것과 같이)
지문 사용이 필요한 방식으로 키를 생성할 것입니다.
```
$ applesedemo.app/Contents/MacOS/applesedemo --create-se-touch-only
Successfully created
public_key(PUB_R1_7iKCZrb1JqSjUncCbDGQenzSqyuNmPU8iUA15efNW5KD1iHd9x)
```
비밀키에 대응하는 공개키가 출력됩니다. 이 값을 어딘가 적어두세요.
Secure Enclave를 이용해서 비밀키 여러 개를 생성하고 각각에 이름을
붙일 수 있습니다만 이 어플리케이션은 오직 한 번에 한 개의 키만 
지원합니다.
## 다이제스트 생성 및 서명
EOSIO는 SHA256 다이제스트를 이용해서 트랜잭션을 서명하고, 짧은 문자열로부터
단순한 SHA256 다이제스트를 만듭니다.

```
$ echo 'Hello world!' | shasum -a 256
0ba904eae8773b70c75333db4de2f3ac45a8ad4ddba1b242f0b3cfc199391dd8  -
```
이제 Secure Enclave만 알고 있는 비밀키를 이용해서 다이제스트를
서명해봅시다. 유효한 지문을 사용하기 전에는, 서명이 생성되지 않을 것입니다.
```
$ applesedemo.app/Contents/MacOS/applesedemo --sign 0ba904eae8773b70c75333db4de2f3ac45a8ad4ddba1b242f0b3cfc199391dd8
signature(SIG_R1_Jx4sBidhFV6PSvS8hWbG5oh77HKud8xpkoHLvWaZVaBeWttRpyEjaGbPRVEKu3JePTyVjANmP4GKFtG2DAuB4MTVqsdC9W)
```
## 키 복구
서명과 다이제스트가 주어지면, 이 내용을 다이제스트를 서명하는 데 
사용한 공개키와의 연관성을 보여줄 수 있어야 합니다.
```
$ applesedemo.app/Contents/MacOS/applesedemo --recover 0ba904eae8773b70c75333db4de2f3ac45a8ad4ddba1b242f0b3cfc199391dd8 \
     SIG_R1_Jx4sBidhFV6PSvS8hWbG5oh77HKud8xpkoHLvWaZVaBeWttRpyEjaGbPRVEKu3JePTyVjANmP4GKFtG2DAuB4MTVqsdC9W
public_key(PUB_R1_7iKCZrb1JqSjUncCbDGQenzSqyuNmPU8iUA15efNW5KD1iHd9x)
```
실제로 이 다이제스트와 서명을 통해 복구한 공개키는 Secure Enclave에 
저장된 공개키와 동일합니다.

## 번역 정보

* 원문 : https://github.com/EOSIO/eos/blob/master/programs/eosio-applesedemo/README.md
* 번역 기준 리비전 : f55f431f8fb4d91e8d872d281ffd3a1cd9de3758
