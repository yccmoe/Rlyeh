# ⚓ Project R'lyeh (르리에)
> **Real-Time Beat-Synced Acoustic Submarine Filter Pedal**  
> 기타 연주의 리듬을 실시간으로 추적하여 마디(Bar) 단위로 주파수 심해를 유영하는 듀얼 디스플레이 EDM 필터 페달

---

## 📖 1. 프로젝트 개요 (Overview)

**R'lyeh(르리에)** 는 1970년대 냉전 시대 핵잠수함의 조종석 계기판 감성과 현대 EDM(일렉트로닉 댄스 뮤직)의 빌드업/드롭(Build-up & Drop) 오디오 필터링을 결합한 부티크 디지털-아날로그 하이브리드 기타 이펙터입니다.

* **핵심 컨셉:** 연주자의 피킹 리듬(BPM)을 실시간으로 분석하여 그리드(Grid)를 생성하고, 원버튼 풋스위치 조작 시 설정된 마디 수만큼 로우패스 필터(LPF) 컷오프 주파수를 하강(잠수) 및 상승(부상)시킵니다.
* **듀얼 계측 디스플레이:**
  * **4층 소나 레이더 (SSD1309 2.42" OLED):** 2차원(X-Y) 원형 레이더 그래프 위에 `DEPTH`와 `RESO` 노브로 설정한 목표 좌표를 띄우고, 잠수 시 정중앙의 내 위치가 목표 지점으로 접근하는 궤적과 1마디 주기 BPM 싱크 소나 스캔을 시각화합니다.
  * **명치 주파수 계기판 (TM1637 7-Segment):** 빈티지 적색 LED를 통해 현재 차단 주파수($5.00 \sim 0.40\text{ kHz}$)를 실시간 수치로 계측합니다.

---

## 🧠 2. 핵심 작동 논리 (Core Logic & DSP)

### 2.1 실시간 비트 트래킹 및 퀀타이즈 스위칭 (Quantize Switching)
* **그리드 재단(Grid Slicing):** 외부 탭 템포(Tap Tempo) 입력 없이 입력 오디오의 어택(피킹) 간격을 실시간 분석하여 마스터 BPM 및 16분음표 단위 블록($t_1 \sim t_n$)을 생성합니다.
* **동적 시간 신축(Dynamic Time Warping):** 연주 속도가 변하면 다음 블록의 길이를 실시간 재조정하여 필터 스윕 곡선의 박자 싱크를 유지합니다.
* **비상 부상 로직(Auto-Surface on Silence):** 잠수 중 연주가 멈춰 일정 시간(1박자 이상) 입력 신호가 감지되지 않을 경우, 소리가 먹먹하게 갇히는 현상(Filter Freeze)을 방지하기 위해 즉시 수면 위($5.00\text{ kHz}$)로 자동 복귀합니다.

### 2.2 원버튼 풋스위치 상태 머신 (1-Button Toggle & Long-Click Reset)
* **짧은 클릭 (토글 - 잠수/부상 전환):**
  * **수면 위($5.00\text{ kHz}$) 상태:** `DOWN SPEED` 로터리로 설정된 마디 수만큼 목표 수심(`DEPTH`)까지 잠수한 뒤, 해당 깊이를 **자동 유지(Hold)**합니다.
  * **잠수 중 또는 심해 유지 상태:** `UP SPEED` 로터리로 설정된 마디 수만큼 초기 수면 위($5.00\text{ kHz}$)로 부상하여 복귀합니다.
* **롱클릭 ($0.8\text{초}$ 이상 꾹 누름 - 비상 부상 리셋):**
  * 현재 수심이나 진행 상태와 무관하게 즉시 수면 위($5.00\text{ kHz}$) 및 레이더 정중앙으로 강제 워프 복귀합니다.

### 2.3 주파수 로그 매핑 & 2D 소나 레이더 시각화
* **주파수 제어 범위:** $5.00\text{ kHz}$ (수면 위, 클린 톤) $\leftrightarrow$ $0.40\text{ kHz}$ (심해 바닥) 구간을 청각 특성에 맞춘 로그($\log$) 곡선으로 제어합니다.
* **2차원 레이더 매핑:**
  * **중앙 십자선(`+`):** 현재 수면 위 기준점
  * **목표 좌표(Target):** X축(`DEPTH`)과 Y축(`RESO`) 노브 값으로 지정된 심해 타겟 지점
  * **레이더 스캔 & 볼륨 링:** 탐지선은 실시간 BPM의 1마디(4박자) 주기에 맞춰 회전하며, 레이더 외경 원호(12시~270도)의 강조 굵기(Bold)는 현재 `VOLUME` 출력 레벨에 비례해 표시됩니다.

---

## 🛠️ 3. 하드웨어 사양 및 부품 리스트 (BOM)

| 카테고리 | 부품명 | 스펙 및 수량 | 역할 |
| :--- | :--- | :--- | :--- |
| **MCU (뇌)** | Teensy 4.0 | ARM Cortex-M7 (600MHz) x 1 | 실시간 오디오 DSP 및 듀얼 디스플레이 연산 |
| **Audio Engine** | Teensy Audio Shield | Rev D (SGTL5000) x 1 | $44.1\text{kHz}$ 16-bit 오디오 입출력 및 LPF 처리 |
| **Radar Display** | 2.42" OLED Module | SSD1309 (128x64, **7-Pin SPI**, Cyan) x 1 | 2D 소나 레이더, 목표 궤적, BPM 스캔 표시 |
| **Freq Display** | 7-Segment LED Module | 0.36" Red, TM1637 드라이버 (소수점 지원) x 1 | 실시간 차단 주파수($5.00 \sim 0.40\text{ kHz}$) 표시 |
| **Speed Select** | Rotary Switch | 6-Pin 5-Position (5단) x 2 | 하강(`DOWN`) 및 상승(`UP`) 마디 속도 선택 |
| **Potentiometers** | B10K 가변저항 | 선형(Linear) x 3 | `DEPTH`(목표 수심), `RESO`(공명), `VOLUME`(출력) |
| **Foot Switch** | Momentary Foot Switch | SPST (논래칭) x 1 | 짧은 클릭(잠수/부상 토글) / 롱클릭(비상 리셋) |
| **Power Module** | MP1584EN Step-Down | 고정 5V 출력 블랙 보드 (입력 7~26V) x 1 | 9V 센터 마이너스 입력을 Teensy용 5V로 강압 |
| **Power Filter** | 콘덴서 2종 (병렬) | 전해 `100uF 25V` (6x7mm) x 1 + 세라믹 `100nF (0.1uF, 104)` x 1 | 5V 출력단 스위칭 리플 및 고주파 노이즈 제거 |
| **I/O & Case** | 6.35mm Jack / DC Jack / Case | 모노 잭 x 2 / 2.1mm DC 잭 x 1 / 1590BB 케이스 ($94\times119\text{mm}$) | 기타 입출력, 표준 9V 전원 입력, 외장 인클로저 |

---

## 🎛️ 4. 조종석 레이아웃 및 컨트롤 (4층 계층 구조)

```text
      ┌─────────────────────── 70mm ───────────────────────┐
      │         ┌────────────────────────────────┐         │
      │         │    SSD1309 OLED (57 x 29 mm)   │         │  ◀ 4층 (최상단): 2D 소나 레이더 & BPM 스캔
      │         └────────────────────────────────┘         │
      │             ┌────────────────────────┐             │
      │             │  7-Segment (46x14 mm)  │             │  ◀ 명치: 실시간 차단 주파수 (5.00~0.40 kHz)
      │             └────────────────────────┘             │
      │          ( ◎ )                     ( ◎ )          │  ◀ 3층: 하강 속도(DOWN) / 상승 속도(UP) 로터리
      │                                                    │
      │        ( ○ )           ( ○ )          ( ○ )        │  ◀ 2층: VOLUME / RESO / DEPTH 노브
      │                                                    │
      │                        [ ⚓ ]                      │  ◀ 1층 (최하단): 원버튼 풋스위치 (토글/리셋)
      └────────────────────────────────────────────────────┘
        * 입출력 및 전원: 6.35mm INPUT x 1, OUTPUT x 1, DC 9V (Center Negative) x 1
```

### 속도 로터리 스위치 5단 매핑 (DOWN / UP 공통)
* **1단 (`00`):** 즉시 실행 (Instant / 약 1초 내 스윕 완료)
* **2단 (`04`):** 4마디 ($64\text{ blocks}$) 동안 스윕
* **3단 (`08`):** 8마디 ($128\text{ blocks}$) 동안 스윕 (표준 빌드업)
* **4단 (`16`):** 16마디 ($256\text{ blocks}$) 동안 스윕
* **5단 (`32`):** 32마디 ($512\text{ blocks}$) 동안 스윕

---

## 🔌 5. 배선 및 핀맵 가이드 (Wiring & Pinout)

*Teensy 4.0과 Audio Shield 결합 후 남는 디지털/아날로그 핀을 아래와 같이 할당합니다. (Audio Shield 전용 핀: `7, 8, 18, 19, 20, 21, 23`번 회피)*

### 5.1 전원부 및 노이즈 필터링 회로 (9V Center Negative $\rightarrow$ 5V)
1. **극성 변환:** 2.1mm DC 잭의 바깥쪽 슬리브(`+9V`)를 MP1584EN의 `IN+`에, 가운데 심(`GND`)을 `IN-`에 연결합니다.
2. **전압 강압:** MP1584EN 고정 5V 보드의 `OUT+`를 Teensy `VIN`에, `OUT-`를 `GND`에 연결합니다.
3. **출력단 디커플링(노이즈 필터):** MP1584EN의 `OUT+`와 `OUT-` 사이에 **`100uF 25V` 전해 콘덴서**(흰색 띠가 `OUT-` 방향)와 **`100nF (0.1uF, 104)` 세라믹 콘덴서**(극성 무관)를 나란히 병렬 연결하여 고주파 스위칭 노이즈를 차단합니다.

### 5.2 상단 소나 레이더 OLED (SSD1309 - 7핀 SPI 통신)
*오디오 노이즈 유입 방지 및 부드러운 레이더 애니메이션 구현을 위해 고속 SPI 통신을 사용합니다.*
* `GND` $\rightarrow$ Teensy `GND`
* `VCC` $\rightarrow$ Teensy `3.3V`
* `SCL (D0 / SCK)` $\rightarrow$ Digital Pin `13`
* `SDA (D1 / MOSI)` $\rightarrow$ Digital Pin `11`
* `RES (Reset)` $\rightarrow$ Digital Pin `9`
* `DC (Data/Command)` $\rightarrow$ Digital Pin `10`
* `CS (Chip Select)` $\rightarrow$ Digital Pin `5`

### 5.3 명치 주파수 계기판 (TM1637 7-Segment)
* `VCC` $\rightarrow$ Teensy `5V` (또는 `3.3V`)
* `GND` $\rightarrow$ Teensy `GND`
* `CLK` $\rightarrow$ Digital Pin `2`
* `DIO` $\rightarrow$ Digital Pin `3`

### 5.4 아날로그 노브 (B10K Potentiometers)
* 양 끝 다리는 각각 `3.3V`와 `GND`에 공통 연결하고, 가운데(Wiper) 다리를 아래 핀에 연결합니다.
* `VOLUME` $\rightarrow$ Analog Pin `A0` (Pin 14)
* `RESO` $\rightarrow$ Analog Pin `A1` (Pin 15)
* `DEPTH` $\rightarrow$ Analog Pin `A2` (Pin 16)

### 5.5 스위치 및 로터리 (Digital Inputs - `INPUT_PULLUP` 사용)
* 모든 스위치의 공통(Common) 단자는 `GND`에 연결합니다.
* **원버튼 풋스위치 (토글/리셋):** Digital Pin `4`
* **DOWN Speed 로터리 (5단):** Digital Pins `0, 1, 6, 12, 17`
* **UP Speed 로터리 (5단):** Digital Pins `22, 24, 25, 26, 27`

---

## 🚀 6. 빌드 및 펌웨어 로드 순서 (Build Steps)

1. **전원부 조립 및 검증:** MP1584EN 출력단에 `100uF 25V` 전해 콘덴서와 `100nF` 세라믹 콘덴서를 병렬 납땜한 뒤, 멀티미터로 `5.0V` 출력을 확인하고 Teensy `VIN`에 연결합니다.
2. **오디오 쉴드 및 디스플레이 배선:** Teensy 4.0과 Audio Shield를 핀 헤더로 적층한 뒤, 7핀 SPI OLED와 7-Segment 모듈, 아날로그 팟, 로터리 스위치를 핀맵에 맞춰 배선합니다.
3. **케이스 타공 및 절연:** 1590BB 케이스 상판 도면(4층~1층 계층 구조)에 맞춰 가공하고, 두 디스플레이 모듈 뒷면이 알루미늄 케이스와 닿아 쇼트 나지 않도록 캡톤 테이프로 절연합니다.
4. **펌웨어 업로드:** Arduino IDE에서 `Teensyduino` 보드 패키지와 `U8g2` (SSD1309 SPI 제어용), `TM1637Display` 라이브러리를 설치한 뒤 소스코드를 컴파일하여 업로드합니다.
