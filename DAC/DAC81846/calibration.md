## 정밀 계측장비에서 주의할 점 (오차)
예를 들어 ±10V 출력일 때:
- 분해능: 약 0.305mV/LSB
- INL 최대 오차: ±0.305mV (명시된 조건)
INL (Integral Non-Linearity): DAC 출력 특성이 이상적인 직선에서 얼마나 벗어나는지 나타내는 비선형 오차
![](https://static.wikidocs.net/images/page/456141/image_202610191720.png)
Gain과 Offset 등 선형 오차를 제외한 기준 직선 대비 비선형성을 의미
- TUE 최대 오차: ±20mV (±0.1% FSR)
TUE (Total Unadjusted Error): Offset, Gain, INL 등 여러 오차를 포함한 보정 전 전체 출력 오차입니다.
DAC81416의 TUE가 최대 ±0.1% FSR인 조건이라면:
![](https://static.wikidocs.net/images/page/456141/image_202610191723.png)

## Offset, Gain 보정방법
정확도를 올리려면, 2점 보정방식을 해야한다.
각 채널별 보정에 필요한 Offset, Gain 을 저장한다.
* Offset 과 Gain 의 이해
![](https://static.wikidocs.net/images/page/456141/image_202610191728.png)
실제 DAC 출력은 다음처럼 근사할 수 있습니다.
![](https://static.wikidocs.net/images/page/456141/image_202610191728_0onRx53.png)
여기서
- \(a\): 실제 Gain (이상적일 때 1)
- \(b\): Offset 오차 (이상적일 때 0V)
보정 출력 명령은 역으로 계산합니다.
![](https://static.wikidocs.net/images/page/456141/image_202610191730.png)
* 2점 보정 절차
DAC81416을 ±10V 출력 모드로 설정했다고 가정하고 DAC 출력에 정밀 DMM(디지털 멀티미터)을 연결하고 다음 두 전압을 출력한다.

 | 항목	 | DAC 명령 전압	 | DMM 측정 전압  | 
| -------- | -------- | -------- |
 | Point 1	 | -8.000V	 | -7.987V  | 
 | Point 2 | 	+8.000V	 | +8.021V  | 
 
Step 1. Gain 계산
![](https://static.wikidocs.net/images/page/456141/image_202610191732.png)
Step 2. Offset 계산
![](https://static.wikidocs.net/images/page/456141/image_202610191732_9lDmqlQ.png)
따라서 측정된 오차는 다음과 같습니다.

 | 항목	 | 계산 결과 | 
| -------- | -------- | 
 | Gain	 | 1.0005 | 
 | Gain Error	 | +0.05% | 
 | Offset	 | +17mV | 
 
Step 3. 목표 전압에 보정 적용
실제로 +5.000V를 출력하고 싶다면:
![](https://static.wikidocs.net/images/page/456141/image_202610191733.png)

* DAC81416 보정 코드 구현
```

#include <stdint.h>
#include <math.h>

#define DAC_CHANNELS 16
#define DAC_MAX_CODE 65535U

typedef struct {
    double gain;
    double offset;
} DAC_Calibration;

static DAC_Calibration cal[DAC_CHANNELS];

// 두 점의 명령 전압과 측정 전압으로 보정 계수 계산
int dac_calibrate(uint8_t ch,
                  double cmd1, double meas1,
                  double cmd2, double meas2)
{
    if (ch >= DAC_CHANNELS || cmd1 == cmd2)
        return -1;

    double gain = (meas2 - meas1) / (cmd2 - cmd1);

    if (!isfinite(gain) || gain <= 0.0)
        return -1;

    cal[ch].gain = gain;
    cal[ch].offset = meas1 - gain * cmd1;

    return 0;
}

// 보정 계수를 반영한 16bit DAC 코드
uint16_t dac_voltage_to_code(uint8_t ch,
                             double target,
                             double vmin,
                             double vmax)
{
    if (ch >= DAC_CHANNELS ||
        !(vmax > vmin) ||
        !isfinite(target) ||
        !isfinite(cal[ch].gain) ||
        cal[ch].gain <= 0.0)
        return 0; // 실제 제품에서는 오류를 별도 보고

    double corrected =
        (target - cal[ch].offset) / cal[ch].gain;

    if (corrected < vmin) corrected = vmin;
    if (corrected > vmax) corrected = vmax;

    double code = (corrected - vmin) /
                  (vmax - vmin) * 65536.0;

    if (code > DAC_MAX_CODE)
        code = DAC_MAX_CODE;

    return (uint16_t)lround(code);
}
```

사용예시
```

/* 측정된 두 점으로 CH0 보정 */
dac_calibrate(0,
              -8.0, -7.987,
               8.0,  8.021);

/* CH0에서 실제 5.000V 목표 */
uint16_t code = dac_voltage_to_code(
    0, 5.0, -10.0, 10.0);

/* DAC0 Data Register: 0x10 */
dac81416_write(0x10, code);

```


## 측정방법
다회 측정 후 평균을 내서 Gain, Offset 값을 정한다.
이때, 평균만 측정하지 않고 표준편차까지 측정을 해야 출력 전압이 불안정하거나 측정한 환경이 노이즈가 있는 환경인지 확인할 수 있다.

## 측정절차
1) DAC와 DMM을 충분히 워밍업
2) -9V 출력 설정
3) 출력 및 DMM 안정화 대기
4) 20회 측정 후 평균·표준편차 계산
5) +9V 출력 설정
6) 같은 방식으로 20회 측정
7) Offset/Gain 보정 계수 계산
8) 0V, ±5V 등에서 보정 결과 검증

