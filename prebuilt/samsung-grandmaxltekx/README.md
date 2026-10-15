# lk2nd for Galaxy Grand Max 한국판 (SM-G720N0 / grandmaxltekx)

[msm8916-mainline/lk2nd](https://github.com/msm8916-mainline/lk2nd) `main`
브랜치에 갤럭시 그랜드 맥스 한국판(SM-G720N0) 지원을 추가해 빌드한 이미지입니다.

> [!WARNING]
> 실기기에서 아직 부팅 테스트를 하지 않은 빌드입니다. 하드웨어 정보는 SM-G720N0
> 순정 커널 소스에서 확인했고, 이미 지원되는 SM-G720AX/SM-G7200과 같은
> 설정을 씁니다. 순정이 아닌 boot 이미지를 플래싱하면 KNOX 워런티 비트가 바뀔 수
> 있습니다. 진행하기 전에 데이터를 백업하고 순정 펌웨어를 준비해 두세요.

## 파일

| 파일 | 용도 |
|------|------|
| `lk2nd.img` | Heimdall 또는 lk2nd fastboot로 플래싱할 원본 이미지 |
| `lk2nd-grandmaxltekx-odin.tar.md5` | Odin의 **AP** 슬롯에 넣는 파일 (`boot.img` 하나만 들어 있음) |
| `SHA256SUMS` | 위 두 파일의 SHA-256 체크섬 |

- lk2nd 버전: `23.1-next-18c40ad19-20261015`
- 소스 커밋: `18c40ad1` (`dts: msm8916: msm8916-samsung: add grandmaxltekx`)
- 컴파일러: arm-none-eabi-gcc 13.2.1, `make TOOLCHAIN_PREFIX=arm-none-eabi- lk2nd-msm8916`

## 지원 내용

- 대상 기기: SM-G720N0 (순정 부트로더 버전이 `G720N0`으로 시작하는 기기)
- 보드 리비전: 4 (Rev00), 5 (Rev01) 둘 다 지원
- MUIC(SM5504) 리셋으로 USB 문제를 우회하며, Linux용 DTB 이름은
  `msm8916-samsung-grandmax`입니다.

## 설치

1. 기기를 다운로드 모드로 부팅합니다: 전원을 끈 상태에서
   **볼륨 아래 + 홈 + 전원**을 누르고, 경고 화면에서 **볼륨 위**를 누릅니다.
2. 둘 중 하나로 플래싱합니다.
   - **Odin (Windows):** `lk2nd-grandmaxltekx-odin.tar.md5`를 **AP** 슬롯에
     넣고 Start를 누릅니다.
   - **Heimdall (Linux/macOS):**
     ```
     heimdall flash --BOOT lk2nd.img
     ```
3. 재부팅하면 lk2nd 화면이 나타납니다.

## 사용법

- 부팅 중 **볼륨 아래**: lk2nd fastboot 모드
- 부팅 중 **볼륨 위**: 리커버리 모드

순정 부트로더도 같은 키 조합을 쓰므로, 화면이 켜지거나 진동이 온 다음에 볼륨
키를 누르세요.

fastboot 모드에서 쓸 수 있는 명령:

```
fastboot flash boot boot.img     # OS boot 이미지를 boot 파티션 512 KiB 오프셋에 기록 (lk2nd는 유지됨)
fastboot flash lk2nd lk2nd.img   # lk2nd 자체 업데이트
fastboot oem log && fastboot get_staged log.txt   # 문제 해결용 로그 받기
```

> [!IMPORTANT]
> 그랜드 맥스의 TrustZone 펌웨어는 32비트 전용이라서 arm64 커널은 부팅되지 않습니다.
> 메인라인 Linux는 ARM32(armv7) 커널로 빌드해야 합니다. postmarketOS의
> `samsung-grandmax` 디바이스 패키지가 armv7입니다.

## 순정으로 되돌리기

SM-G720N0 순정 펌웨어를 Odin으로 다시 플래싱하거나, 미리 백업한 순정 `boot.img`를
다운로드 모드에서 플래싱합니다.

## 직접 빌드하기

```
sudo apt install gcc-arm-none-eabi device-tree-compiler python3
make TOOLCHAIN_PREFIX=arm-none-eabi- lk2nd-msm8916
# 결과물: build-lk2nd-msm8916/lk2nd.img
```
