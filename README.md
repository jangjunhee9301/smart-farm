# 📘 Jetson Nano + Arduino + DHT11 자동 이메일 시스템

Jetson Nano와 Arduino를 활용하여 실시간 온·습도(DHT11) 데이터를 측정하고,  
일정 주기마다 이메일로 자동 전송하는 시스템입니다.

---

## 📦 구성 요소

| 구성 | 설명 |
|------|------|
| Jetson Nano | Python을 이용해 시리얼 통신 및 이메일 전송 |
| Arduino Uno | DHT11 센서 값을 JSON 형식으로 시리얼 출력 |
| DHT11 센서 | 온도 및 습도 측정용 디지털 센서 |
| Gmail SMTP | 이메일 전송을 위한 구글 메일 서버 |

---

## 🛠️ 준비 과정

### ✅ 하드웨어 연결
- DHT11 → Arduino Uno  
  - VCC → 5V  
  - GND → GND  
  - DATA → D2  
- Arduino Uno → Jetson Nano  
  - USB 케이블로 연결

---

### ✅ Arduino 코드 업로드

```cpp
#include <DHT.h>

#define DHTPIN 2
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  if (isnan(h) || isnan(t)) return;

  Serial.print("{\"temperature\":");
  Serial.print(t);
  Serial.print(",\"humidity\":");
  Serial.print(h);
  Serial.println("}");
  
  delay(2000);
}
```

#### 📤 출력 예시
```json
{"temperature":24.5,"humidity":60.0}
```

---

### ✅ Jetson Nano 설정

1. **시리얼 포트 권한 설정**
```bash
sudo usermod -aG dialout $USER
# 적용을 위해 로그아웃 후 다시 로그인 필요
```

2. **포트 확인**
```bash
ls /dev/ttyACM*
# 또는
dmesg | grep tty
```

> 일반적으로 `/dev/ttyACM0` 또는 `/dev/ttyUSB0` 경로 사용

---

## 🐍 Python 코드 (`simpledht.py`)

Jetson Nano에서 실행되는 파이썬 코드로 다음을 수행합니다:

- 시리얼 포트로부터 JSON 데이터 수신  
- 데이터 로그 저장 (CSV 파일)  
- 주기적으로 이메일 전송 (1시간 간격)  
- 평균 온·습도 계산 + 그래프 이미지 생성 후 이메일에 첨부

---

## 🔧 주요 설정

| 항목 | 값 |
|------|----|
| 시리얼 포트 | `/dev/ttyACM0` |
| Baudrate | 115200 |
| 이메일 전송 간격 | 3600초 (1시간) |
| Gmail SMTP | `smtp.gmail.com:587` |
| 인증 방식 | Gmail 앱 비밀번호 사용 필요 |

---

## ✉️ 이메일 예시

**제목:**  
`DHT11 센서 데이터 리포트 - 2025-07-11 13:00`

**본문 예시:**
```
🌡️ 온도 평균: 24.5°C  
💧 습도 평균: 58.2%  
📈 첨부된 그래프 확인
```

**첨부파일:**  
- 온·습도 변화 추이 그래프 (PNG 이미지)

---

## 🧪 문제 해결 팁

| 증상 | 원인 | 해결 방법 |
|------|------|------------|
| "No data to send" | 센서 데이터 수신 실패 | 포트 및 권한 확인, in_waiting 제거 |
| "SerialException" | 시리얼 포트 없음 | 포트 경로 재확인 필요 |
| 그래프 없음 | 데이터 부족 (2개 미만) | 일정량의 로그가 쌓일 때까지 대기 필요 |

---

## ▶️ 실행 방법

Jetson Nano에서 다음 명령어 실행:

```bash
python3 simpledht.py
```

실행 결과 예시:
```
Email sent successfully to a64100761@gmail.com
```

---

## 📁 파일 구조 예시

```
Jetson-DHT11-Email/
├── simpledht.py
├── data_log.csv
├── graph.png
└── README.md
```
