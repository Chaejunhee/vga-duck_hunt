# 🦆 FPGA Duck Hunt Game with OV7670 Camera

## 📋 프로젝트 소개 (Project Overview)
이 프로젝트는 **SystemVerilog**와 **FPGA (Digilent Basys 3)**를 이용하여 고전 게임 'Duck Hunt'를 재구현한 하드웨어 설계 프로젝트입니다.

기존의 광선총(Light Gun) 대신 **OV7670 카메라 모듈**을 사용하여, 카메라가 화면상의 **붉은색 물체(Red Object)를 인식하고 이를 조준점(Crosshair)으로 변환**하여 게임을 플레이하는 인터랙티브 시스템입니다.

### 🎥 데모 (Demo)
*(여기에 게임 동작 GIF나 스크린샷을 추가하세요)*

---

## 🎯 주요 기능 (Key Features)

* **비전 기반 컨트롤러:** OV7670 카메라로 입력된 영상에서 특정 색상(Red)의 좌표를 실시간으로 추출하여 마우스 커서처럼 활용.
* **VGA 디스플레이:** 640x480 @ 60Hz VGA 출력을 통해 게임 배경, 오리 애니메이션, UI(점수, 시간, 총알) 표시.
* **게임 로직 구현:**
    * 오리의 랜덤 비행 패턴 및 속도 조절.
    * 충돌 판정(Hit detection) 및 폭발 애니메이션.
    * 점수 계산 및 제한 시간 타이머.
    * 총알 개수 제한 및 재장전 메커니즘.
* **메모리 최적화:** BRAM 용량 한계를 극복하기 위해 이미지 리소스를 경량화하고 효율적인 ROM 구조 설계.

---

## 🛠 개발 환경 (Environment)

* **Hardware:**
    * FPGA Board: Digilent Basys 3 (Artix-7 XC7A35T)
    * Camera: OV7670 CMOS Sensor
    * Display: VGA Monitor
* **Software:** Xilinx Vivado Design Suite
* **Language:** SystemVerilog, Verilog HDL

---

## 🏗 시스템 구조 (System Architecture)

### 1. 전체 블록 다이어그램 (System Block Diagram)
시스템은 크게 **카메라 입력부**, **영상 처리부**, **게임 로직부**, **디스플레이 출력부**로 구성됩니다.

| 모듈 (Module) | 역할 (Description) |
| :--- | :--- |
| **OV7670 Controller** | SCCB 인터페이스를 통해 카메라 레지스터 설정 및 Pixel Data(RGB565) 수신. |
| **Red Detector** | 입력된 픽셀 데이터에서 Red 성분 임계값을 검사하여 붉은 물체의 X, Y 좌표 추출. |
| **Game Controller** | 오리의 생성, 이동, 피격 판정, 점수 관리 등 핵심 게임 플레이 로직 수행. |
| **Display Controller** | 배경 이미지, 스프라이트(오리, 조준점), 텍스트(점수)를 합성하여 VGA 신호 생성. |

### 2. 영상 처리 알고리즘 (Red Detection)
* **Thresholding:** 입력되는 RGB565 데이터에서 R 성분이 G, B 성분보다 현저히 높은 픽셀을 필터링합니다.
* **Coordinate Calculation:** 한 프레임 내에서 검출된 붉은 픽셀들의 평균 위치를 계산하여 조준점의 중심 좌표로 사용합니다.

### 3. 그래픽 및 메모리 (Graphics & Memory)
* FPGA 내부 BRAM 자원을 효율적으로 사용하기 위해 배경 및 스프라이트 이미지를 최적화된 해상도로 저장합니다.
* `duck_move.mem`, `bomb.mem` 등의 메모리 파일을 ROM으로 인스턴스화하여 실시간으로 렌더링합니다.

---

## 🎮 게임 방법 (How to Play)

1.  **시작 (Start):** FPGA 보드의 `Start` 버튼을 눌러 게임을 시작합니다.
2.  **조준 (Aim):** 붉은색 물체(레이저 포인터, 색종이 등)를 카메라에 비추면 화면의 크로스헤어가 움직입니다.
3.  **발사 (Shoot):** 크로스헤어를 오리에 맞춘 상태에서 `Shoot` 버튼을 누릅니다.
4.  **규칙 (Rules):**
    * 제한 시간 내에 최대한 많은 오리를 잡아야 합니다.
    * 총알은 제한되어 있으며, 다 쓰면 오리를 잡을 수 없습니다.
    * 오리를 놓치면 오리가 도망갑니다.

---

## 📂 파일 구조 (File Structure)

```text
├── src/
│   ├── DuckHunt_System.sv         # [Top Module] 전체 시스템 통합
│   ├── OV7670_config_rom.sv       # 카메라 설정 레지스터 값 (SCCB)
│   ├── OV7670_Mem_Controller.sv   # 카메라 데이터 메모리 제어
│   ├── SCCB_controller.sv         # I2C 호환 SCCB 프로토콜 제어
│   ├── TOP_SCCB.sv                # SCCB 탑 모듈
│   ├── Red_Detector_Threshold.sv  # 붉은색 좌표 검출 로직
│   ├── Duck_Controller.sv         # 오리 이동 및 상태 FSM
│   ├── Game_Display_Controller.sv # 화면 렌더링 및 레이어 합성
│   ├── VGA_Decoder.sv             # VGA 타이밍 생성 (H-Sync, V-Sync)
│   ├── frame_buffer.sv            # 영상 버퍼 메모리
│   ├── ImgROM.sv                  # 이미지 데이터 ROM
│   ├── Start_Screen.sv            # 시작 화면 텍스트 생성
│   ├── button_debouncer.sv        # 버튼 입력 디바운싱
│   └── pclk_gen.sv                # 픽셀 클럭 생성
├── assets/
│   ├── duck_move.mem              # 오리 스프라이트 데이터
│   ├── bomb.mem                   # 폭발 이펙트 데이터
│   └── duck_hunt_background.mem   # 배경 이미지 데이터
└── docs/
    └── presentation.pptx          # 프로젝트 발표 자료
