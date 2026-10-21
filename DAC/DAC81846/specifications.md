# Specifications
1.  주요사양

|  항목	 |  DAC81416  | 
|  --------  |  --------  | 
| 제조사     | Texas Instruments  |
| 해상도	| 16bit (65,536 Steps) |
| 채널 수	| 16채널 |
| DAC 구조	| R-2R |
| 출력 형식	| Buffered Voltage Output |
| 양극성 출력	| ±2.5V / ±5V / ±10V / ±20V |
| 단극성 출력 | 0~5V / 0~10V / 0~20V / 0~40V |
| INL	| ±1LSB Max (일부 범위 ±2LSB) |
| DNL	| ±1LSB Max |
| Monotonicity	| 16bit 보장 |
| TUE	| ±0.1% FSR Max (일부 범위 ±0.2%) |
| Settling Time |  12µs Typ |
| 출력 구동	| 최대 ±25mA (전원 레일 1.5V 여유 조건) |
| 내부 기준 전압	| 2.5V |
| 기준 전압 정확도 | ±2.5mV Max |
| 기준 전압 드리프트 | 5ppm/°C Typ |
| 통신	| SPI, 최대 50MHz |
| 소비 전력	| 530mW Typ |
| 동작 온도	| -40~125°C |
| 패키지	| 40-pin VQFN, 6×6mm |

3.  출력전압범위 및 분해능

| 출력 모드 | 전압 범위 | 1LSB | 
| -------- | -------- | -------- |
| Bipolar	| -2.5~+2.5V	| 76.29µV |
| Bipolar	| -5~+5V |	152.59µV |
| Bipolar | -10~+10V	| 305.18µV |
| Bipolar | -20~+20V	| 610.35µV |
| Unipolar |	0~5V	| 76.29µV |
| Unipolar | 0~10V |	152.59µV |
| Unipolar | 0~20V |	305.18µV |
| Unipolar	|0~40V |	610.35µV |

계산식 : ![ㅁㅁ](https://static.wikidocs.net/images/page/456141/image_202610191712.png)

4.  전원사양

| 전원	| 동작 범위	| 역할 |
| -------- | -------- | -------- |
| VDD	| 4.5~5.5V	| 디지털 회로 | 
| VAA	| 4.5~5.5V	| 아날로그 회로 | 
| VIO	| 1.7~5.5V	| SPI 디지털 I/O | 
| VCC	| 9~41.5V	| 고전압 출력 양전원 | 
| VSS	| -21.5~0V	| 고전압 출력 음전원 |  

6.  인터페이스와 제어기능

 | 기능	 | 설명 | 
| -------- | -------- |
 | SPI	 | 최대 50MHz, 4-wire | 
 | LDAC	 | 다채널 동시 업데이트 | 
 | RESET	 | 하드웨어 초기화 | 
 | CLR	출력  | Clear | 
 | A/B Toggle	 | 두 출력 코드 간 빠른 전환 | 
 | CRC	 | SPI 통신 오류 검출 | 
 | Daisy Chain	 | 여러 DAC 연결 | 
 | Differential	 | 채널 쌍을 차동 출력으로 설정 | 
 | ALMOUT	 | 알람 출력 | 
 | TEMPOUT	 | 칩 내부 온도 모니터링 | 

 '전원이 인가되면 DAC 출력은 GND에 연결된 상태로 유지되며, 레지스터를 올바르게 설정한 이후 정상 출력을 시작합니다.'
