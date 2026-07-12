# VISA를 이용한 오실로스코프 연동 및 FFT 실습

|  |  |
|---|---|
|**실험날짜:**|2026년 7월 12일|
|**작성자:**|박현|
|  |  |

## 개요

오실로스코프에서 측정한 값을 외부의 방대한 라이브러리들을 이용해 분석하기 위함으로
본 실험에서는 PyVISA와 SciPy를 이용해 오실로스코프에서 측정한 톱니파의 주파수를 계산한다.


## 이론적 배경

### VISA

VISA는 Virtual instrument software architecture의 약자로 시험 측정 분야에 있어
게측 장치와 통신하기 위한 API로 오실로스코프와 PC 간의 통신에 필요하다.

### 푸리에 변환

푸리에 변환은 시간에 따라 변화하는 신호로부터 주기를 얻어낼 수 있는 기법으로 아래와 같은 수학적 변환을 통해 얻어내진다.

$ F(u) = cal(F)[f(x)] = integral_(-oo)^(oo) f(x) e^(-2pi i u t) dif t $

## 실험 과정

+ HMO1002에서 SETUP > INTERFACE를 USB TMC로 설정한다.
+ HMO1002와 컴퓨터를 USB로 연결한다.
+ HMO1002에서 SETUP > DEVICE INFORMATION에서 VISA를 확인한다.
+ 아래 코드를 이용해 접속이 잘 되었는지 확인한다.
  ```py
  import pyvisa
  
  rm = pyvisa.ResourceManager()
  inst = rm.open_resource("USB::0x0AAD::0x0119::029913419::INSTR")
  print(inst.query("*IDN?"), end="")
  ```
+ HMO1002의 함수 생성 기능을 이용해 50Hz의 톱니파를 생성한다.
+ AUX OUT과 CH1을 연결한다.
+ 아래 코드를 이용해 CH1에서 측정한 값을 가져온다.
  ```py
  import matplotlib.pyplot as plt
  
  inst.write(":STOP")
  inst.write(":FORM REAL")
  cmd = ":CHANnel1:DATA?"
  signal = inst.query_binary_values(cmd, datatype="f", is_big_endian=True)
  plt.plot(range(len(signal)), signal)
  plt.show()
  ```
+ 아래 코드를 이용해 각 샘플 사이의 시간 간격을 구한다.
  ```py
  start, end, N, _ = eval(inst.query(":CHANnel1:DATA:HEADer?"))
  dt = (end - start) / N
  ```
+ 위에서 구한 시간 간격을 이용해 푸리에 변환을 수행한다.
  ```py
  import numpy as np
  from scipy.fft import rfft, rfftfreq
  
  signal_centered = signal - np.mean(signal)
  window = np.hanning(len(signal_centered))
  signal_windowed = signal_centered * window
  
  yf = rfft(signal_windowed)
  xf = rfftfreq(len(signal_windowed), dt)
  
  plt.plot(xf, np.abs(yf) * 2 / N)
  plt.xlabel("Frequency (Hz)")
  plt.ylabel("Amplitude")
  plt.xlim(0, 200)
  plt.grid(True)
  plt.show()
  ```

== 실험 결과

#grid(
  columns: 2,
  figure(caption: [오실로스코프의 값을 가져온 모습])[
    #image("images/Figure_1.png", width: 8cm)
  ],
  figure(caption: [푸리에 변환 결과])[
    #image("images/Figure_2.png", width: 8cm)
  ],
)
