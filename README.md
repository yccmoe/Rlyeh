# ⚓ Project R'lyeh (르리에)
> **Real-Time Beat-Synced Acoustic Submarine Filter Pedal**  
> 기타 연주의 리듬을 실시간으로 추적하여 마디(Bar) 단위로 주파수 심해를 유영하는 EDM 필터 페달

---

## 📖 1. 프로젝트 개요 (Overview)

**R'lyeh(르리에)** 는 1970년대 냉전 시대 핵잠수함의 조종석 계기판 감성과 현대 EDM(일렉트로닉 댄스 뮤직)의 빌드업/드롭(Build-up & Drop) 오디오 필터링을 결합한 부티크 디지털-아날로그 하이브리드 기타 이펙터입니다.

* **핵심 컨셉:** 연주자의 피킹 리듬(BPM)을 실시간으로 분석하여 그리드(Grid)를 생성하고, 원버튼 풋스위치 조작 시 설정된 마디 수만큼 로우패스 필터(LPF) 컷오프 주파수를 하강(잠수) 및 상승(부상)시킵니다.
* **시각적 아이덴티티:** 1590BB 알루미늄 인클로저 최상단에 배치된 빈티지 적색 7-Segment LED 디스플레이를 통해 현재 차단 주파수($\text{Hz}$ / $\text{kHz}$)와 잠수 진행 상태를 실시간으로 계측합니다.

---

## 🧠 2. 핵심 작동 논리 (Core Logic & DSP)

### 2.1 실시간 비트 트래킹 및 퀀타이즈 스위칭 (Quantize Switching)
* **그리드 재단(Grid Slicing):** 외부 탭 템포(Tap Tempo) 입력 없이, 입력되는 기타 오디오 신호의 어택(피킹) 간격을 실시간으로 계산하여 내부 마스터 BPM 및 16분 음표 단위 블록($t_1 \sim t_n$)을 생성합니다.
* **동적 시간 신축(Dynamic Time Warping):** 연주 중 속도가 빨라지거나 느려지면 다음 블록의 길이를 실시간으로 재조정하여 필터 스윕 곡선의 박자 싱크를 유지합니다.
* **비상 부상 로직(Auto-Surface on Silence):** 잠수 중 연주가 갑자기 멈춰 1박자 이상 오디오 입력이 감지되지 않을 경우, 소리가 먹먹하게 갇히는 현상(Filter Freeze)을 방지하기 위해 즉시 수면 위($5.00\text{kHz}$)로 강제 복귀합니다.

### 2.2 원버튼 잠수 시퀀스 (1-Button Toggle & Reset Sequence)
직관적인 원버튼 풋스위치 하나만으로 잠수, 유지, 부상, 비상 리셋을 모두 제어하는 상태 머신(State Machine)으로 작동합니다.

1. **짧게 클릭 (Short Press - 토글 작동):**
   * **수면 위($5.00\text{kHz}$) 상태일 때:** 설정된 하강 속도(`DOWN SPEED`)에 따라 목표 수심(`DEPTH`)까지 주파수를 하강시킨 후, 해당 깊이를 **자동 유지(Hold)**합니다.
   * **잠수 중 또는 바닥 유지 상태일 때:** 설정된 상승 속도(`UP SPEED`)에 따라 초기 수면 상태($5.00\text{kHz}$)로 복귀합니다.
2. **길게 누르기 (Long Press - $0.8\text{s}$ 이상):**
   * **강제 비상 부상(Instant Reset):** 현재 어느 수심에 있든 즉시 계기판을 $5.00\text{kHz}$로 튕겨 올리며 초기 상태로 리셋합니다.

### 2.3 주파수 로그 매핑 (Logarithmic Frequency Mapping)
인간의 청각 인지 특성에 맞추어 선형(Linear)이 아닌 **로그($\log$) 스케일**로 컷오프 주파수를 제어합니다.
* **작동 범위:** $5.00\text{kHz}$ (수면 위, 클린 톤) $\leftrightarrow$ $0.40\text{kHz}$ (심해 바닥)
* **공명 보정(Resonance Drift):** 주파수가 하강함에 따라 `RESO` 노브로 설정된 값만큼 컷오프 지점의 피크(Q)를 강조하여 일렉트로닉 신시사이저 특유의 웅웅거리는 질감을 구현합니다.

---

## 🛠️ 3. 하드웨어 사양 및 부품 리스트 (BOM)

| 카테고리 | 부품명 | 스펙 및 수량 | 역할 |
| :--- | :--- | :--- | :--- |
| **MCU (뇌)** | Teensy 4.0 | ARM Cortex-M7 (600MHz) x 1 | 실시간 오디오 DSP 및 제어 로직 연산 |
| **Audio Engine** | Teensy Audio Shield | Rev D (SGTL5000) x 1 | $44.1\text{kHz}$ 16-bit 오디오 입출력 및 LPF 처리 |
| **Display** | 7-Segment LED Module | 0.36" Red, TM1637 드라이버 내장 (소수점 지원) x 1 | 실시간 차단 주파수($5.00 \sim 0.40$) 표시 |
| **Speed Select** | Rotary Switch | 6-Pin 5-Position (5단) x 2 | 하강(`DOWN`) 및 상승(`UP`) 마디 속도 선택 |
| **Potentiometers** | B10K 가변저항 | 선형(Linear) x 3 | `DEPTH`(목표 수심), `RESO`(공명), `VOLUME`(출력) |
| **Foot Switch** | Momentary Foot Switch | SPST (논래칭) x 1 | 잠수/부상 토글 및 롱클릭 비상 리셋 트리거 |
| **I/O & Case** | 6.35mm Jack / 1590BB | 모노 잭 x 2 / 알루미늄 케이스 x 1 | 기타 신호 입출력 및 외장 인클로저 |

---

## 🎛️ 4. 조종석 레이아웃 및 컨트롤 (Control Layout)

최상단 4층 계기판부터 최하단 1층 풋스위치까지 직관적인 수직 계층 구조로 배치합니다.

```text
      ┌──────────────────────────────────────────┐
      │             [ 5 . 0 0 ] kHz              │  <-- [4층] 7-Segment Display
      │                                          │
      │        ( ◎ )              ( ◎ )        │
      │      DOWN SPEED          UP SPEED        │  <-- [3층] 5-Way Rotary x2
      │                                          │
      │     ( ○ )         ( ○ )         ( ○ )    │
      │    VOLUME         RESO          DEPTH    │  <-- [2층] B10K Potentiometer x3
      │                                          │
      │                  [ ⚓ ]                 │
      │              DIVE / RISE (Hold)          │  <-- [1층] 1-Button Foot Switch
      └──────────────────────────────────────────┘
```

### 층별 작동부 상세 설명
* **[4층] 7-Segment Display (계기판):** 현재 필터가 깎아내고 있는 실시간 차단 주파수($5.00 \sim 0.40\text{kHz}$)를 적색 LED로 표시하여 현재 잠수 진행 상태를 직관적으로 보여줍니다.
* **[3층] DOWN / UP SPEED (속도 로터리 스위치 2개):** 각각 하강(잠수)과 상승(부상)에 걸리는 시간을 마디 단위로 선택합니다.
  * `1단 (00)`: 즉시 실행 (Instant / 약 1초 내 스윕 완료)
  * `2단 (04)`: 4마디 ($64\text{ blocks}$) 동안 스윕
  * `3단 (08)`: 8마디 ($128\text{ blocks}$) 동안 스윕 (표준 빌드업)
  * `4단 (16)`: 16마디 ($256\text{ blocks}$) 동안 스윕
  * `5단 (32)`: 32마디 ($512\text{ blocks}$) 동안 스윕
* **[2층] VOLUME / RESO / DEPTH (메인 노브 3개):**
  * `VOLUME`: 이펙터의 최종 출력 음량 조절
  * `RESO`: 컷오프 주파수 경계면의 공명(Resonance, Q) 피크 강도 조절 (웅웅거리는 신스 질감)
  * `DEPTH`: 하강 버튼 입력 시 도달할 목표 최대 잠수 수심(최저 차단 주파수) 설정
* **[1층] 1-Button Foot Switch (메인 트리거):**
  * **짧게 누르기 (Toggle):** 잠수 시작(목표 수심 도달 후 자동 유지) $\leftrightarrow$ 수면 복귀 전환
  * **길게 누르기 (Long Press):** 어느 위치에서든 즉시 수면($5.00\text{kHz}$)으로 비상 부상(리셋)

---

## 🔌 5. 배선 및 핀맵 가이드 (Wiring & Pinout)

*Teensy 4.0과 Audio Shield 결합 후 남는 디지털/아날로그 핀을 아래와 같이 할당합니다.*

### 5.1 디스플레이 (TM1637 - 4층)
* `VCC` $\rightarrow$ Teensy `5V` (또는 `3.3V`)
* `GND` $\rightarrow$ Teensy `GND`
* `CLK` $\rightarrow$ Digital Pin `2`
* `DIO` $\rightarrow$ Digital Pin `3`

### 5.2 아날로그 노브 (B10K Potentiometers - 2층)
* 양 끝 다리는 각각 `3.3V`와 `GND`에 공통 연결, 가운데(Wiper) 다리를 아래 핀에 연결합니다.
* `VOLUME` $\rightarrow$ Analog Pin `A0` (Pin 14)
* `RESO` $\rightarrow$ Analog Pin `A1` (Pin 15)
* `DEPTH` $\rightarrow$ Analog Pin `A2` (Pin 16)

### 5.3 스위치 및 로터리 (Digital Inputs - `INPUT_PULLUP` 사용)
* 모든 스위치의 공통(Common) 단자는 `GND`에 연결합니다.
* **[1층] Main Foot Switch:** Digital Pin `4` (원버튼 통합으로 Pin `5` 여유 확보)
* **[3층] DOWN Rotary (5단):** Digital Pins `6, 9, 10, 11, 12` *(주의: 7, 8번 핀은 Audio Shield 전용이므로 제외)*
* **[3층] UP Rotary (5단):** Digital Pins `17, 22, 24, 25, 26`

---

## 🚀 6. 빌드 및 펌웨어 로드 순서 (Build Steps)

1. **오디오 쉴드 결합:** Teensy 4.0과 Audio Shield 사이에 핀 헤더를 납땜하여 적층합니다. (Audio Shield 전용 핀인 7, 8, 18, 19, 20, 21, 23번의 간섭 주의)
2. **만능기판 배선:** `3.3V` 전원 레일과 `GND` 접지 레일을 만능기판에 구성하고, 각 노브와 스위치의 공통선을 모읍니다.
3. **케이스 타공 및 조립:** 1590BB 케이스 상판 도면(4층~1층 배치)에 맞춰 드릴링 후 부품을 체결합니다. 7-Segment 모듈 뒷면이 케이스와 닿아 쇼트(Short)가 나지 않도록 절연 테이프로 마감합니다.
4. **펌웨어 업로드:** Arduino IDE에서 `Teensyduino` 보드 매니저와 `TM1637Display` 라이브러리를 설치한 후 소스코드를 컴파일하여 업로드합니다.
