# [DSP 실습 06] DTFS, DTFT, 주파수 영역 특성 이해

## 1. 실습 목적
- 주기 이산시간 신호를 주파수 영역으로 표현하는 **DTFS(Discrete-Time Fourier Series)**의 개념을 이해한다.
- 비주기 이산시간 신호의 주파수 영역 표현인 **DTFT(Discrete-Time Fourier Transform)**의 개념을 학습한다.
- 시간 영역 신호와 주파수 영역 계수 사이의 관계를 이해하고, Parseval's theorem을 통해 시간 영역 power와 주파수 영역 power를 비교한다.
- MATLAB에서 DTFT를 직접 계산하기 위해 matrix-vector product 방식을 사용하는 방법을 익힌다.
- DTFT 결과의 magnitude, angle, real part, imaginary part를 시각화한다.
- DTFT의 주기성, conjugate-symmetry property, frequency shifting property를 그래프를 통해 확인한다.

---

## 2. 실습 예제 및 결과

### Q1. DTFS 개념

DTFS는 주기 이산시간 신호를 복소 지수 함수들의 합으로 표현하는 방법이다.  
주기 신호 x[n]이 주기 N을 가지면 다음과 같이 나타낼 수 있다.

x[n] = x[n + N]

DTFS 표현은 시간 영역의 주기 신호를 주파수 영역 계수 a<sub>k</sub>로 변환하는 과정이다.

x[n] = Σ a<sub>k</sub> exp(j2πkn / N)

여기서 a<sub>k</sub>는 DTFS coefficient이며, 각 주파수 성분이 신호에 얼마나 포함되어 있는지를 나타낸다.  
실습에서는 예제 신호 x[n] = cos(2πn / 6)에 대해 DTFS 계수를 구하고, 시간 영역 power와 주파수 영역 power가 같은지 확인하였다.

```matlab
n = 0:5;
x = cos(2*n*pi/6);

N = 6;
k = 0:5;

Pt = 1/N * sum((abs(x)).^2);

W = exp(-1j*2*pi/N.*k);

ak = zeros(1,6);

for i = 1:6
    ak(i) = 1/N * sum(x.*W(i).^n);
end

Pw = sum(abs(ak).^2);
```

이 예제를 통해 시간 영역에서 계산한 power와 주파수 영역에서 DTFS coefficient를 이용해 계산한 power가 같음을 확인할 수 있다.  
이는 Parseval's theorem을 의미하며, 시간 영역과 주파수 영역이 같은 에너지 또는 power 정보를 담고 있음을 보여 준다.

---

### Q2. DTFT 개념

DTFT는 비주기 이산시간 신호를 연속적인 주파수 변수 Ω에 대한 함수로 표현하는 방법이다.  
DTFT는 다음과 같이 정의된다.

X(e<sup>jΩ</sup>) = Σ x[n]e<sup>-jΩn</sup>

DTFT를 사용하면 이산시간 신호가 각 디지털 주파수 성분을 얼마나 포함하는지 분석할 수 있다.  
inverse DTFT를 사용하면 주파수 영역의 X(e<sup>jΩ</sup>)로부터 다시 시간 영역 신호 x[n]을 복원할 수 있다.

x[n] = (1 / 2π) ∫ X(e<sup>jΩ</sup>)e<sup>jΩn</sup>dΩ

DTFT는 Ω에 대해 2π 주기를 가지므로, 일반적으로 한 주기 구간인 -π ≤ Ω ≤ π 또는 0 ≤ Ω ≤ π 범위에서 분석한다.  
특히 x[n]이 real-valued signal인 경우 magnitude는 even symmetry를 가지고, phase는 odd symmetry를 가지는 conjugate-symmetry property가 나타난다.

---

### Q3. MATLAB에서 DTFT 계산

유한 길이 이산 신호는 MATLAB에서 DTFT 정의식을 이용하여 수치적으로 계산할 수 있다.  
신호 x[n]이 특정 n 범위에서만 존재한다고 하면, DTFT는 다음과 같이 행렬 곱 형태로 계산할 수 있다.

X = x * exp(-jΩn)

실습 자료에서는 Ω를 0부터 π까지 균일하게 나누고, 다음과 같은 matrix-vector product 형태로 DTFT를 계산하였다.

```matlab
k = [0:M];
n = [n1:n2];

X = x * (exp(-j*pi/M)).^(n'*k);
```

이 방식은 각 주파수 샘플에 대해 DTFT 합을 반복문으로 계산하지 않고, 행렬 연산으로 한 번에 계산할 수 있다는 장점이 있다.  
이후 `abs(X)`, `angle(X)`, `real(X)`, `imag(X)`를 이용하여 magnitude, angle, real part, imaginary part를 각각 분석할 수 있다.

---

## 3. 실습 문제 및 결과

### Practice 1. DTFT of x[n] = (0.5)<sup>n</sup>u[n]

Practice 1에서는 다음 신호의 DTFT를 직접 유도하고, 0 ≤ Ω ≤ π 구간에서 magnitude, angle, real part, imaginary part를 그래프로 나타내었다.

x[n] = (0.5)<sup>n</sup>u[n]

이 신호는 n ≥ 0에서만 존재하므로 DTFT는 다음과 같이 계산된다.

X(e<sup>jΩ</sup>) = Σ (0.5)<sup>n</sup>e<sup>-jΩn</sup>

등비급수 형태로 정리하면 다음과 같다.

X(e<sup>jΩ</sup>) = 1 / (1 - 0.5e<sup>-jΩ</sup>)

아래 코드는 이 식을 직접 사용하여 501개의 주파수 샘플에서 DTFT를 계산하고 네 가지 성분을 그린 것이다.

```matlab
w = linspace(0, pi, 501);
X = 1 ./ (1 - 0.5*exp(-1j*w));

figure;

subplot(2,2,1);
plot(w, abs(X), 'LineWidth', 1.5);
grid on;
title('Magnitude Part');
xlabel('Angular Frequency (\omega)');
ylabel('Magnitude');

subplot(2,2,2);
plot(w, angle(X), 'LineWidth', 1.5);
grid on;
title('Angle Part');
xlabel('Angular Frequency (\omega)');
ylabel('Radians');

subplot(2,2,3);
plot(w, real(X), 'LineWidth', 1.5);
grid on;
title('Real Part');
xlabel('Angular Frequency (\omega)');
ylabel('Real');

subplot(2,2,4);
plot(w, imag(X), 'LineWidth', 1.5);
grid on;
title('Imaginary Part');
xlabel('Angular Frequency (\omega)');
ylabel('Imaginary');
```

#### Practice 1 결과 그래프

<img width="1012" height="676" alt="lab06_P1_capture" src="https://github.com/user-attachments/assets/ce627ec4-4500-453e-a115-b007f51567fd" />

Practice 1 결과에서 magnitude는 Ω = 0 근처에서 가장 크고, Ω가 π로 증가할수록 감소하는 형태를 보인다.  
이는 x[n] = (0.5)<sup>n</sup>u[n]이 저주파 성분을 상대적으로 많이 포함하고 있음을 의미한다.  
또한 angle, real part, imaginary part 그래프를 통해 X(e<sup>jΩ</sup>)가 복소수 값을 가지며 주파수에 따라 위상과 실수부, 허수부가 변화함을 확인할 수 있다.

---

### Practice 2. Numerical DTFT using Matrix-Vector Product

Practice 2에서는 Practice 1과 같은 신호 x[n] = (0.5)<sup>n</sup>u[n]에 대해 DTFT를 matrix-vector product 방식으로 수치 계산하였다.  
무한 길이 신호를 실제 MATLAB에서 직접 계산할 수는 없기 때문에, n = 0부터 N = 50까지로 신호를 truncate하여 근사하였다.

```matlab
clear; clc; close all;

% x[n] = (0.5)^n u[n]
N = 50;                 % truncation length
n = 0:N;
x = (0.5).^n;           % u[n] included since n >= 0 only

% 501 equispaced points in [0, pi]
k = 0:500;
w = pi*k/500;

% Numerical DTFT using matrix-vector product
E = exp(-1j*pi/500).^(n.' * k);   % (N+1) x 501
X = x * E;                        % 1 x 501

% Plot
figure;

subplot(2,2,1);
plot(w, abs(X), 'LineWidth', 1.5);
grid on;
title('Magnitude');
xlabel('\omega');
ylabel('|X(e^{j\omega})|');

subplot(2,2,2);
plot(w, angle(X), 'LineWidth', 1.5);
grid on;
title('Angle');
xlabel('\omega');
ylabel('Phase (rad)');

subplot(2,2,3);
plot(w, real(X), 'LineWidth', 1.5);
grid on;
title('Real Part');
xlabel('\omega');
ylabel('Re\{X(e^{j\omega})\}');

subplot(2,2,4);
plot(w, imag(X), 'LineWidth', 1.5);
grid on;
title('Imaginary Part');
xlabel('\omega');
ylabel('Im\{X(e^{j\omega})\}');
```

#### Practice 2 결과 그래프

<img width="1007" height="677" alt="lab06_P2_capture" src="https://github.com/user-attachments/assets/8d70bc3a-f4f9-43b4-8767-8f61a594f5e6" />

Practice 2 결과는 Practice 1에서 직접 구한 DTFT 식의 결과와 거의 같은 형태를 보인다.  
이는 유한 길이로 truncate한 신호에 대해 matrix-vector product 방식으로 DTFT를 계산해도 충분히 원래 DTFT의 형태를 근사할 수 있음을 보여 준다.  
또한 행렬 연산을 사용하면 여러 주파수 지점에 대한 DTFT 값을 효율적으로 한 번에 계산할 수 있다.

---

### Q1-A. Periodicity of DTFT

Q1-A에서는 다음 신호에 대해 -2π ≤ Ω ≤ 2π 범위에서 DTFT의 amplitude와 angle을 구하고, DTFT의 주기성을 확인하였다.

x[n] = (0.9e<sup>jπ/3</sup>)<sup>n</sup>, 0 ≤ n ≤ 10

DTFT는 2π 주기를 가지므로, magnitude와 angle 그래프는 Ω축에서 2π 간격으로 반복되는 형태를 보여야 한다.

```matlab
clear; clc; close all;

n = 0:10;
x = (0.9*exp(1j*pi/3)).^n;

% omega from -2pi to 2pi
k = -200:200;                 % 401 points
w = (pi/100)*k;               % -2pi ~ 2pi

% DTFT using matrix-vector product
X = x * (exp(-1j*pi/100)).^(n.'*k);

figure;

subplot(2,1,1);
plot(w/pi, abs(X), 'LineWidth', 1.5);
grid on;
title('Magnitude Part');
xlabel('frequency in units of \pi');
ylabel('|X|');

subplot(2,1,2);
plot(w/pi, angle(X)/pi, 'LineWidth', 1.5);
grid on;
title('Angle Part');
xlabel('frequency in units of \pi');
ylabel('radians/\pi');
```

#### Q1-A 결과 그래프

<img width="1052" height="685" alt="lab06_Q1A_capture" src="https://github.com/user-attachments/assets/6ecb57c0-a52a-4e17-ab30-7545b088244b" />

Q1-A 결과에서 magnitude response는 특정 주파수 부근에서 큰 값을 가지며, -2π부터 2π까지의 범위에서 같은 형태가 반복된다.  
이는 DTFT가 주파수 Ω에 대해 2π-periodic하다는 성질을 확인시켜 준다.  
Angle response 역시 주기적으로 반복되는 경향을 보이며, phase wrapping으로 인해 불연속적으로 보이는 구간이 나타날 수 있다.

---

### Q1-B. Conjugate-Symmetry Property

Q1-B에서는 다음 real-valued signal에 대해 DTFT를 계산하고 conjugate-symmetry property를 확인하였다.

x[n] = (0.9)<sup>n</sup>, -5 ≤ n ≤ 5

실수 신호 x[n]에 대해 DTFT는 다음 성질을 가진다.

X(Ω) = X<sup>*</sup>(-Ω)

따라서 magnitude는 even symmetry를 가지고, phase는 odd symmetry를 갖는다.  
또한 real part는 even symmetry, imaginary part는 odd symmetry를 보인다.

```matlab
clear; clc; close all;

n = -5:5;
x = (0.9).^n;

k = -200:200;              % gives omega from -2pi to 2pi
w = (pi/100)*k;

% DTFT using matrix-vector product
X = x * (exp(-1j*pi/100)).^(n.'*k);

figure;

subplot(2,2,1);
plot(w/pi, abs(X), 'LineWidth', 1.5);
grid on;
title('Magnitude Part');
xlabel('frequency in units of \pi');
ylabel('|X|');

subplot(2,2,2);
plot(w/pi, angle(X)/pi, 'LineWidth', 1.5);
grid on;
title('Angle Part');
xlabel('frequency in units of \pi');
ylabel('radians/\pi');

subplot(2,2,3);
plot(w/pi, real(X), 'LineWidth', 1.5);
grid on;
title('Real Part');
xlabel('frequency in units of \pi');
ylabel('Real');

subplot(2,2,4);
plot(w/pi, imag(X), 'LineWidth', 1.5);
grid on;
title('Imaginary Part');
xlabel('frequency in units of \pi');
ylabel('Imaginary');
```

#### Q1-B 결과 그래프

<img width="1025" height="682" alt="lab06_Q1B_capture" src="https://github.com/user-attachments/assets/422553ed-497a-4165-a920-13ce64b3f6db" />

Q1-B 결과에서 magnitude plot은 Ω = 0을 기준으로 좌우가 대칭인 even symmetry를 보인다.  
또한 real part도 좌우 대칭 형태를 가지며, imaginary part는 원점을 기준으로 부호가 반대가 되는 odd symmetry를 보인다.  
이는 x[n]이 real-valued signal일 때 DTFT가 conjugate-symmetry property를 만족한다는 것을 의미한다.

---

### Q2. Frequency Shifting Property

Q2에서는 다음 두 신호를 이용하여 DTFT의 frequency shifting property를 확인하였다.

x[n] = cos(πn / 2), 0 ≤ n ≤ 100

y[n] = e<sup>jπn/4</sup>x[n]

DTFT의 frequency shifting property는 다음과 같다.

e<sup>jω<sub>0</sub>n</sup>x[n] ⇔ X(e<sup>j(ω-ω<sub>0</sub>)</sup>)

즉, 시간 영역에서 x[n]에 복소 지수 e<sup>jω<sub>0</sub>n</sup>을 곱하면, 주파수 영역에서는 원래 X(e<sup>jω</sup>)가 ω<sub>0</sub>만큼 이동한다.  
이 문제에서는 ω<sub>0</sub> = π/4이므로, Y(e<sup>jω</sup>)는 X(e<sup>jω</sup>)를 π/4만큼 shift한 결과와 같아야 한다.

```matlab
clear; clc; close all;

% sequence definition
n = 0:100;
x = cos(pi*n/2);
y = exp(1j*pi*n/4).*x;

% frequency samples: -pi to pi
k = -200:200;
w = (pi/200)*k;

% DTFT by matrix-vector product
X = x * exp(-1j*(n.'*w));
Y = y * exp(-1j*(n.'*w));

% shifted version of X : X(e^{j(w-w0)})
w0 = pi/4;
Xshift = x * exp(-1j*(n.'*(w - w0)));

% plots
figure;

subplot(2,2,1);
plot(w/pi, abs(X), 'LineWidth', 1.2);
grid on;
title('Magnitude of X');
xlabel('frequency in \pi units');
ylabel('|X|');

subplot(2,2,2);
plot(w/pi, angle(X)/pi, 'LineWidth', 1.2);
grid on;
title('Angle of X');
xlabel('frequency in \pi units');
ylabel('radians/\pi');

subplot(2,2,3);
plot(w/pi, abs(Y), 'LineWidth', 1.2);
grid on;
title('Magnitude of Y');
xlabel('frequency in \pi units');
ylabel('|Y|');

subplot(2,2,4);
plot(w/pi, angle(Y)/pi, 'LineWidth', 1.2);
grid on;
title('Angle of Y');
xlabel('frequency in \pi units');
ylabel('radians/\pi');
```

#### Q2 결과 그래프

<img width="1012" height="681" alt="lab06_Q2_capture" src="https://github.com/user-attachments/assets/3024827d-33d2-4915-b418-0e0cec7a5f4b" />

Q2 결과에서 x[n] = cos(πn / 2)는 주파수 영역에서 ±π/2 부근에 큰 magnitude peak를 갖는다.  
여기에 e<sup>jπn/4</sup>를 곱해 y[n]을 만들면, Y(e<sup>jω</sup>)의 magnitude peak는 원래 X(e<sup>jω</sup>)의 peak보다 π/4만큼 이동한 위치에 나타난다.  
즉, 시간 영역에서 복소 지수 신호를 곱하는 것은 주파수 영역에서 스펙트럼을 shift시키는 효과를 가지며, 이를 통해 frequency shifting property를 확인할 수 있다.

---

## 4. 실습 내용 정리

이번 실습에서는 DTFS와 DTFT의 기본 개념을 학습하고, MATLAB을 이용하여 이산시간 신호의 주파수 영역 특성을 분석하였다.  
먼저 DTFS를 통해 주기 신호를 복소 지수 함수의 합으로 표현할 수 있음을 확인하였고, 시간 영역 power와 주파수 영역 power가 같다는 Parseval's theorem의 의미를 이해하였다.

이후 DTFT를 이용하여 비주기 이산시간 신호를 연속적인 주파수 함수 X(e<sup>jΩ</sup>)로 표현하는 방법을 학습하였다.  
Practice 1에서는 x[n] = (0.5)<sup>n</sup>u[n]의 DTFT를 직접 유도하고 magnitude, angle, real part, imaginary part를 그래프로 나타내었다.  
Practice 2에서는 같은 신호를 유한 길이로 truncate한 뒤 matrix-vector product 방식으로 DTFT를 수치 계산하였다.

Q1-A에서는 x[n] = (0.9e<sup>jπ/3</sup>)<sup>n</sup>에 대해 -2π부터 2π까지 DTFT를 계산하여 2π 주기성을 확인하였다.  
Q1-B에서는 real-valued signal x[n] = (0.9)<sup>n</sup>에 대해 magnitude와 real part는 even symmetry, imaginary part는 odd symmetry를 보이며 conjugate-symmetry property를 만족함을 확인하였다.  
마지막으로 Q2에서는 x[n]에 e<sup>jπn/4</sup>를 곱했을 때 주파수 영역에서 스펙트럼이 π/4만큼 이동하는 것을 확인하여 frequency shifting property를 검증하였다.
