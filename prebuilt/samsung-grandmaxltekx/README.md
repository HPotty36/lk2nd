# lk2nd for Galaxy Grand Max 한국판 (SM-G720N0 / grandmaxltekx)

[msm8916-mainline/lk2nd](https://github.com/msm8916-mainline/lk2nd) `main`
브랜치에 갤럭시 그랜드 맥스 한국판(SM-G720N0) 지원을 추가해 빌드한 이미지와,
이걸로 postmarketOS를 설치하는 방법입니다.

> [!WARNING]
> 실기기에서 아직 부팅 테스트를 하지 않은 빌드입니다. 하드웨어 정보는 SM-G720N0
> 순정 커널 소스에서 확인했고, 이미 지원되는 SM-G720AX/SM-G7200과 같은
> 설정을 씁니다. 순정이 아닌 boot 이미지를 플래싱하면 KNOX 워런티 비트가 바뀔 수
> 있고, lk2nd를 올린 뒤에는 안드로이드가 부팅되지 않습니다.

## 전체 흐름

1. 다운로드 모드에서 **lk2nd**를 boot 파티션에 올립니다 (Odin 또는 Heimdall).
2. lk2nd의 **fastboot 모드**가 PC에 잡히는지 확인합니다.
3. PC에서 `pmbootstrap`으로 **postmarketOS**를 만들어 fastboot로 올립니다.

삼성 순정 부트로더에는 fastboot가 없어서 lk2nd로 fastboot를 먼저 만드는 구조입니다.

## 받을 파일

이 폴더에서 파일을 누른 뒤 오른쪽 위 다운로드 버튼으로 받습니다.

| 파일 | 언제 쓰나 |
|------|-----------|
| [`lk2nd-grandmaxltekx-odin.tar.md5`](lk2nd-grandmaxltekx-odin.tar.md5) | Windows에서 **Odin**으로 올릴 때 |
| [`lk2nd.img`](lk2nd.img) | Linux/macOS에서 **Heimdall**로 올릴 때, 또는 나중에 lk2nd를 업데이트할 때 |
| [`SHA256SUMS`](SHA256SUMS) | 받은 파일이 깨지지 않았는지 확인할 때 |

체크섬 확인 (선택):

```
# Linux / macOS
sha256sum -c SHA256SUMS
# Windows (명령 프롬프트)
certutil -hashfile lk2nd-grandmaxltekx-odin.tar.md5 SHA256
```

출력된 값이 `SHA256SUMS` 안의 값과 같으면 됩니다.

## 0. 준비

- 휴대폰 데이터 백업 (postmarketOS 설치 시 내부 데이터가 지워집니다)
- 되돌릴 때 쓸 **SM-G720N0 순정 펌웨어** (Frija, Bifrost 같은 삼성 펌웨어 다운로드 도구로 받을 수 있습니다)
- Windows라면 삼성 USB 드라이버와 Odin, Linux라면 `heimdall`과 `fastboot`
  (`sudo apt install heimdall-flash fastboot`)
- postmarketOS 설치 단계(3번)는 **Linux PC**가 필요합니다

## 1. lk2nd 올리기

1. 휴대폰 전원을 끄고 **볼륨 아래 + 홈 + 전원**을 누른 채 기다렸다가, 경고 화면에서
   **볼륨 위**를 눌러 다운로드 모드로 들어갑니다.
2. USB로 PC에 연결하고 둘 중 하나로 올립니다.
   - **Odin:** `lk2nd-grandmaxltekx-odin.tar.md5`를 **AP** 슬롯에 넣고 Start
   - **Heimdall:**
     ```
     heimdall flash --BOOT lk2nd.img
     ```
3. 휴대폰이 재부팅되면서 lk2nd가 실행됩니다.

## 2. lk2nd 동작 확인 (가장 중요)

1. 재부팅 중 삼성 로고가 뜬 **다음에** **볼륨 아래만** 누르고 있으면 lk2nd fastboot
   모드로 들어갑니다. (로고 전에 누르면 순정 부트로더가 키를 가져갑니다. OS가 아직
   없으면 보통 그대로 lk2nd 화면에 머뭅니다.)
2. PC에서 확인합니다.
   ```
   fastboot devices
   ```
   기기 하나가 나오면 성공입니다. **여기서 안 잡히면 3번으로 넘어가도 안 됩니다.**
   이 경우 아래 [문제 해결](#문제-해결)을 보세요.
3. 한국판으로 제대로 인식됐는지 확인합니다. lk2nd 화면에 기기 이름이 표시되고,
   PC에서도 볼 수 있습니다.
   ```
   fastboot getvar lk2nd:model
   ```
   `Samsung Galaxy Grand Max (SM-G720N0)`가 나오면 성공입니다. 자세한 로그가 필요하면
   `fastboot oem log && fastboot get_staged lk2nd-log.txt`로 받을 수 있습니다.

## 3. postmarketOS 설치 (Linux PC)

```
pipx install pmbootstrap       # 또는 배포판의 pmbootstrap 패키지
pmbootstrap init               # vendor: samsung → device: grandmax, UI 선택
pmbootstrap install            # 사용자 비밀번호 설정
```

RAM이 1.5GB라 처음에는 `console`이나 가벼운 UI로 부팅부터 확인하는 걸 권합니다.
그다음 휴대폰을 lk2nd fastboot 모드(2번)로 두고:

```
pmbootstrap flasher flash_rootfs   # userdata 파티션에 기록 (기존 데이터 삭제)
pmbootstrap flasher flash_kernel   # boot 파티션의 512 KiB 뒤에 기록, lk2nd는 그대로 유지
fastboot reboot
```

> [!IMPORTANT]
> - `pmbootstrap flasher flash_lk2nd`는 **쓰지 마세요.** postmarketOS에 들어 있는
>   업스트림 lk2nd로 덮어써서 한국판 지원이 사라집니다. lk2nd를 다시 올려야 하면
>   이 폴더의 `lk2nd.img`를 쓰세요 (`fastboot flash lk2nd lk2nd.img`).
> - 그랜드 맥스 펌웨어는 32비트 전용이라 arm64 커널은 부팅되지 않습니다.
>   `pmbootstrap init`에서 기기를 고르면 자동으로 armv7이 되니 직접 바꾸지 마세요.

## lk2nd 키 조작

- 부팅 중 **볼륨 아래**: lk2nd fastboot 모드
- 부팅 중 **볼륨 위**: 리커버리 모드

## 문제 해결

- **`fastboot devices`에 아무것도 안 나옴:** USB 케이블과 포트를 바꿔 보고, Linux라면
  `sudo fastboot devices`로도 시도해 보세요. 그래도 안 되면 lk2nd 화면을 사진으로
  남겨 두세요. 화면에 `SM-G720N0`이 보이는데 USB만 안 되는 경우와, 기기 이름이
  안 보이는 경우(한국판으로 인식 실패)는 원인이 다릅니다.
- **lk2nd 화면이 아예 안 뜸:** 다운로드 모드로 들어가 순정 펌웨어를 Odin으로 다시 올리면
  원래대로 돌아갑니다.
- **순정으로 되돌리기:** SM-G720N0 순정 펌웨어를 Odin으로 플래싱합니다.

## 빌드 정보

- lk2nd 버전: `23.1-next-18c40ad19-20261015`
- 소스 커밋: `18c40ad1` (`dts: msm8916: msm8916-samsung: add grandmaxltekx`)
- 컴파일러: arm-none-eabi-gcc 13.2.1, `make TOOLCHAIN_PREFIX=arm-none-eabi- lk2nd-msm8916`
- 대상: 순정 부트로더 버전이 `G720N0`으로 시작하는 기기, 보드 리비전 4 (Rev00)와 5 (Rev01)
- Linux용 DTB: `msm8916-samsung-grandmax`

## 직접 빌드하기

```
sudo apt install gcc-arm-none-eabi device-tree-compiler python3
make TOOLCHAIN_PREFIX=arm-none-eabi- lk2nd-msm8916
# 결과물: build-lk2nd-msm8916/lk2nd.img
```

이 브랜치에 push하면 GitHub Actions가 자동으로 빌드하며, 결과물은 Actions 실행
페이지의 Artifacts(`lk2nd-msm8916`)에서 받을 수 있습니다.
