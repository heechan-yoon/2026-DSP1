# [DSP 실습 07+08] DTFT, Sampling, Reconstruction 이해

## 1. 실습 목적

- 비주기 이산시간 신호의 주파수 영역 표현인 **DTFT(Discrete-Time Fourier Transform)**의 개념을 복습한다.
- MATLAB에서 DTFT를 직접 계산하기 위해 **matrix-vector product** 방식을 사용한다.
- DTFT 결과의 magnitude, angle, real part, imaginary part를 시각화한다.
- 시간 영역에서의 shift가 주파수 영역에서 phase shift로 나타나는 **time shifting property**를 확인한다.
- 신호를 even part와 odd part로 분해하고, 각각의 DTFT 특성을 비교한다.
- Sampling theorem과 aliasing의 개념을 이해한다.
- 연속시간 신호를 서로 다른 sampling frequency로 샘플링하고, sinc 함수를 이용하여 reconstruction을 수행한다.
- Sampling frequency가 충분한 경우와 부족한 경우의 복원 결과를 비교한다.

---

## 2. Lab 07: DTFT

### Review. Matrix-Vector Product를 이용한 DTFT 계산

DTFT는 이산시간 신호 `x[n]`을 연속적인 주파수 변수 `ω`에 대한 함수로 표현하는 방법이다.

DTFT의 정의는 다음과 같다.

X(e<sup>jω</sup>) = Σ x[n]e<sup>-jωn</sup>

MATLAB에서는 주파수 샘플을 여러 개 설정한 뒤, 다음과 같은 matrix-vector product 방식으로 DTFT를 한 번에 계산할 수 있다.

```matlab
X = x * exp(-1j*n'*w);
```

또는 주파수를 `0 ≤ ω ≤ π` 구간에서 501개 점으로 나누는 경우 다음과 같이 표현할 수 있다.

```matlab
X = x * (exp(-1j*pi/500)).^(n'*k);
```

아래 예제는 `x[n] = {1, 2, 3, 4, 5}`, `-1 ≤ n ≤ 3`인 신호에 대해 DTFT를 계산하고 magnitude, angle, real part, imaginary part를 나타낸 코드이다.

```matlab
clear; clc; close all;

% Sequence definition
n = -1:3;
x = 1:5;

% Frequency samples: 501 equispaced points in [0, pi]
k = 0:500;
w = pi*k/500;

% DTFT using matrix-vector product
X = x * (exp(-1j*pi/500)).^(n.'*k);

% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,2,1);
plot(w/pi, abs(X), 'LineWidth', 1.5);
grid on;
title('Magnitude Part');
xlabel('frequency in \pi units');
ylabel('|X(e^{j\omega})|');

subplot(2,2,2);
plot(w/pi, angle(X), 'LineWidth', 1.5);
grid on;
title('Angle Part');
xlabel('frequency in \pi units');
ylabel('Radians');

subplot(2,2,3);
plot(w/pi, real(X), 'LineWidth', 1.5);
grid on;
title('Real Part');
xlabel('frequency in \pi units');
ylabel('Real');

subplot(2,2,4);
plot(w/pi, imag(X), 'LineWidth', 1.5);
grid on;
title('Imaginary Part');
xlabel('frequency in \pi units');
ylabel('Imaginary');
```

#### Review 결과 그래프

<img width="1563" height="992" alt="lab07_review" src="https://github.com/user-attachments/assets/cef33128-3e1f-4f3f-b6af-7bb07937a214" />

이 예제에서는 유한 길이 이산 신호의 DTFT를 직접 계산하고, 주파수에 따라 magnitude와 phase가 어떻게 변화하는지 확인하였다.  
`abs(X)`는 각 주파수 성분의 크기를 나타내고, `angle(X)`는 위상 정보를 나타낸다.  
또한 `real(X)`와 `imag(X)`를 통해 DTFT가 일반적으로 복소수 값을 갖는다는 점을 확인할 수 있다.

---

## 3. Lab 07 Practice 1. Time Shifting Property 확인

Practice 1에서는 `0 ≤ n ≤ 10` 범위에서 `[0, π]` 사이에 균일 분포하는 random sequence `x[n]`을 생성한 뒤, 다음 신호를 정의한다.

y[n] = x[n - 2]

DTFT의 time shifting property는 다음과 같다.

x[n - n<sub>d</sub>] ⇔ e<sup>-jωn<sub>d</sub></sup>X(e<sup>jω</sup>)

즉, 시간 영역에서 신호가 오른쪽으로 `n_d`만큼 이동하면, 주파수 영역에서는 magnitude는 변하지 않고 phase만 `-ωn_d`만큼 변한다.

```matlab
clear; clc; close all;

% Random sequence x[n], 0 <= n <= 10
rng(0);                     % for reproducible result
nx = 0:10;
x = pi*rand(1, length(nx)); % uniformly distributed between [0, pi]

% Time shifted signal y[n] = x[n - 2]
nd = 2;
ny = nx + nd;
y = x;

% Frequency samples
k = 0:500;
w = pi*k/500;

% DTFT of x[n] and y[n]
X = x * exp(-1j*(nx.'*w));
Y = y * exp(-1j*(ny.'*w));

% Theoretical shifted version
Y_theory = exp(-1j*w*nd).*X;

% Plot comparison
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,2,1);
plot(w/pi, abs(Y), 'LineWidth', 1.5);
grid on;
title('|Y(e^{j\omega})|');
xlabel('frequency in \pi units');
ylabel('Magnitude');

subplot(2,2,2);
plot(w/pi, abs(Y_theory), '--', 'LineWidth', 1.5);
grid on;
title('|e^{-j\omega n_d}X(e^{j\omega})|');
xlabel('frequency in \pi units');
ylabel('Magnitude');

subplot(2,2,3);
plot(w/pi, unwrap(angle(Y)), 'LineWidth', 1.5);
grid on;
title('Phase of Y(e^{j\omega})');
xlabel('frequency in \pi units');
ylabel('Phase');

subplot(2,2,4);
plot(w/pi, unwrap(angle(Y_theory)), '--', 'LineWidth', 1.5);
grid on;
title('Phase of e^{-j\omega n_d}X(e^{j\omega})');
xlabel('frequency in \pi units');
ylabel('Phase');
```

#### Practice 1 결과 그래프

<img width="1587" height="1021" alt="lab07_P1_capture" src="https://github.com/user-attachments/assets/99b0662b-5b4d-4aa1-ac82-196d265a2627" />

Practice 1 결과에서 `Y(e^{jω})`의 magnitude와 `e^{-jωn_d}X(e^{jω})`의 magnitude는 동일한 형태를 보인다.  
이는 시간 영역에서 신호를 shift해도 주파수 성분의 크기는 변하지 않는다는 것을 의미한다.

반면 phase plot에서는 시간 shift에 의한 선형 위상 변화가 나타난다.  
따라서 시간 영역의 delay는 주파수 영역에서 magnitude 변화가 아니라 phase 변화로 나타난다는 time shifting property를 확인할 수 있다.

---

## 4. Lab 07 Practice 2. Even/Odd Decomposition과 DTFT

### 4.1 evenodd.m 함수

임의의 real sequence `x[n]`은 even part와 odd part로 분해할 수 있다.

x<sub>e</sub>[n] = 1/2 {x[n] + x[-n]}

x<sub>o</sub>[n] = 1/2 {x[n] - x[-n]}

이를 MATLAB에서 계산하기 위해 `evenodd.m` 함수를 사용한다.

```matlab
function [xe, xo, m] = evenodd(x, n)
% Real signal decomposition into even and odd parts
% [xe, xo, m] = evenodd(x, n)

    if any(imag(x) ~= 0)
        error('x is not a real sequence');
    end

    m = -fliplr(n);
    m1 = min([m, n]);
    m2 = max([m, n]);
    m = m1:m2;

    nm = n(1) - m(1);
    n1 = 1:length(n);

    x1 = zeros(1, length(m));
    x1(n1 + nm) = x;

    x = x1;
    xe = 0.5*(x + fliplr(x));
    xo = 0.5*(x - fliplr(x));
end
```

---

### 4.2 Practice 2 코드

Practice 2에서는 다음 신호를 even part와 odd part로 분해한 뒤, 각각의 DTFT를 구한다.

x[n] = sin(πn / 2), -5 ≤ n ≤ 10

```matlab
clear; clc; close all;

% Sequence definition
n = -5:10;
x = sin(pi*n/2);

% Even/Odd decomposition
[xe, xo, m] = evenodd(x, n);

% Frequency samples
k = -100:100;
w = pi*k/100;

% DTFT
X  = x  * exp(-1j*(n.'*w));
XE = xe * exp(-1j*(m.'*w));
XO = xo * exp(-1j*(m.'*w));

% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,2,1);
plot(w/pi, real(X), 'LineWidth', 1.5);
grid on;
title('Real Part of X');
xlabel('frequency in \pi units');
ylabel('Real');

subplot(3,2,2);
plot(w/pi, imag(X), 'LineWidth', 1.5);
grid on;
title('Imaginary Part of X');
xlabel('frequency in \pi units');
ylabel('Imaginary');

subplot(3,2,3);
plot(w/pi, real(XE), 'LineWidth', 1.5);
grid on;
title('Real Part of Even Part');
xlabel('frequency in \pi units');
ylabel('Real');

subplot(3,2,4);
plot(w/pi, imag(XE), 'LineWidth', 1.5);
grid on;
title('Imaginary Part of Even Part');
xlabel('frequency in \pi units');
ylabel('Imaginary');

subplot(3,2,5);
plot(w/pi, real(XO), 'LineWidth', 1.5);
grid on;
title('Real Part of Odd Part');
xlabel('frequency in \pi units');
ylabel('Real');

subplot(3,2,6);
plot(w/pi, imag(XO), 'LineWidth', 1.5);
grid on;
title('Imaginary Part of Odd Part');
xlabel('frequency in \pi units');
ylabel('Imaginary');
```

#### Practice 2 결과 그래프

<img width="998" height="677" alt="lab07_P2_capture" src="https://github.com/user-attachments/assets/572087ab-7afe-46e6-b232-bfcee32aab77" />

Practice 2에서는 원래 신호 `x[n]`을 even part와 odd part로 나누고, 각각의 DTFT를 비교하였다.  
일반적으로 real even signal의 DTFT는 real-valued 특성을 강하게 보이고, real odd signal의 DTFT는 imaginary-valued 특성을 강하게 보인다.  
따라서 even/odd decomposition을 이용하면 시간 영역의 대칭성이 주파수 영역의 real part와 imaginary part에 어떻게 반영되는지 확인할 수 있다.

---

## 5. Lab 07 Question 1. Sampling and Sinc Reconstruction

Question 1에서는 5Hz cosine wave를 생성하고, 이를 8Hz와 12Hz로 각각 sampling한 뒤 sinc function을 이용하여 reconstruction을 수행한다.

원래 신호는 다음과 같다.

x(t) = cos(2πf<sub>0</sub>t), f<sub>0</sub> = 5Hz

Sampling theorem에 따르면 sampling frequency는 신호 주파수의 2배 이상이어야 한다.

F<sub>s</sub> ≥ 2F<sub>signal</sub>

따라서 5Hz 신호를 완전히 복원하려면 최소 10Hz 이상의 sampling frequency가 필요하다.  
8Hz sampling은 Nyquist rate보다 낮으므로 aliasing이 발생하고, 12Hz sampling은 Nyquist rate보다 높으므로 상대적으로 원 신호에 가깝게 복원된다.

```matlab
clear; clc; close all;

% Original analog-like signal
f0 = 5;
t = -0.5:0.001:0.5;
y = cos(2*pi*f0*t);

%% Sampling by 8 Hz
f1 = 8;
T1 = 1/f1;
t1 = -0.5:T1:0.5;
yn_1 = cos(2*pi*f0*t1);

% Reconstruction using sinc function
yr_1 = zeros(1, length(t));
for i = 1:length(t1)
    yr_1 = yr_1 + yn_1(i)*sinc((t - t1(i))/T1);
end

%% Sampling by 12 Hz
f2 = 12;
T2 = 1/f2;
t2 = -0.5:T2:0.5;
yn_2 = cos(2*pi*f0*t2);

% Reconstruction using sinc function
yr_2 = zeros(1, length(t));
for i = 1:length(t2)
    yr_2 = yr_2 + yn_2(i)*sinc((t - t2(i))/T2);
end

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);
tiledlayout(3, 2, 'TileSpacing', 'compact', 'Padding', 'compact');

nexttile([1 2]);
plot(t, y, 'LineWidth', 1.5);
grid on;
title('Original 5Hz Cosine Signal');
xlabel('Time (sec)');
ylabel('Amplitude');

nexttile;
stem(t1, yn_1, 'filled');
grid on;
title('8Hz Sampling');
xlabel('Time (sec)');
ylabel('Amplitude');

nexttile;
plot(t, y, 'LineWidth', 1.0); hold on;
plot(t, yr_1, '--', 'LineWidth', 1.5);
stem(t1, yn_1, 'filled');
grid on;
title('8Hz Reconstruction');
xlabel('Time (sec)');
ylabel('Amplitude');
legend('Original', 'Reconstructed', 'Samples');

nexttile;
stem(t2, yn_2, 'filled');
grid on;
title('12Hz Sampling');
xlabel('Time (sec)');
ylabel('Amplitude');

nexttile;
plot(t, y, 'LineWidth', 1.0); hold on;
plot(t, yr_2, '--', 'LineWidth', 1.5);
stem(t2, yn_2, 'filled');
grid on;
title('12Hz Reconstruction');
xlabel('Time (sec)');
ylabel('Amplitude');
legend('Original', 'Reconstructed', 'Samples');
```

#### Question 1 결과 그래프

<img width="1033" height="678" alt="lab07_Q_capture" src="https://github.com/user-attachments/assets/4a287f8e-02b9-4109-9061-bff6a95c3698" />

Question 1 결과에서 8Hz sampling은 5Hz cosine wave를 충분히 표현하지 못한다.  
5Hz 신호의 Nyquist rate는 10Hz이므로, 8Hz로 sampling하면 aliasing이 발생한다.  
따라서 sinc reconstruction을 수행해도 원래의 5Hz cosine wave와 다른 형태의 신호가 복원된다.

반면 12Hz sampling은 Nyquist rate보다 크기 때문에 원래 신호의 변화 양상을 더 잘 반영한다.  
복원 결과 역시 8Hz sampling보다 원 신호에 더 가까운 형태를 보인다.  
이를 통해 sampling frequency가 충분하지 않으면 aliasing 때문에 원 신호를 정확히 복원할 수 없다는 점을 확인할 수 있다.

---

## 6. Lab 08 Part 1: Sampling

Lab 08에서는 다음 quasi-analog signal을 사용한다.

x<sub>a</sub>(t) = e<sup>-1000|t|</sup>

이 신호를 서로 다른 sampling frequency로 sampling한 뒤, sampled signal의 DTFT를 계산하고 sinc reconstruction 결과를 비교한다.

---

## 7. Lab 08 Practice 1. Sampling at 5kHz

Practice 1에서는 `x_a(t) = e^{-1000|t|}`를 sampling frequency `f_s = 5kHz`로 sampling하여 `x1[n]`을 얻고, 원래 신호와 비교한다.  
또한 sampled sequence `x1[n]`에 대해 DTFT를 수행하여 `X(e^{jω})`를 plot한다.

```matlab
clear; clc; close all;

%% Analog-like signal
Dt = 0.00005;
t = -0.005:Dt:0.005;
xa = exp(-1000*abs(t));

%% Discrete-time Signal: fs = 5kHz
fs = 5000;
Ts = 1/fs;
n = -25:25;
x1 = exp(-1000*abs(n*Ts));

%% Discrete-time Fourier Transform
K = 500;
k = -K:1:K;
w = pi*k/K;

X1 = x1 * exp(-1j*(n.'*w));
X1 = real(X1);

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
plot(t*1000, xa, 'LineWidth', 1.5); hold on;
stem(n*Ts*1000, x1, 'filled');
grid on;
title('Discrete Signal sampled at 5kHz');
xlabel('t in msec.');
ylabel('x_1[n]');
legend('Analog-like signal', 'Sampled signal');

subplot(2,1,2);
plot(w/pi, X1, 'LineWidth', 1.5);
grid on;
title('Discrete-time Fourier Transform of x_1[n]');
xlabel('Frequency in \pi units');
ylabel('X_1(e^{j\omega})');
```

#### Practice 1 결과 그래프

<img width="1582" height="1020" alt="lab08_P1" src="https://github.com/user-attachments/assets/a9f797aa-2378-468b-b4ca-e707b28af6a8" />

Practice 1에서는 5kHz의 높은 sampling frequency를 사용했기 때문에 `x_a(t)`의 급격한 변화가 비교적 촘촘하게 sampling된다.  
따라서 sampled signal `x1[n]`은 원래 analog-like signal의 형태를 잘 따라간다.  
DTFT 결과에서는 `ω = 0` 근처에서 큰 값을 가지며, 주파수가 증가할수록 감소하는 형태를 확인할 수 있다.

---

## 8. Lab 08 Practice 2. Sampling at 1kHz

Practice 2에서는 같은 신호 `x_a(t) = e^{-1000|t|}`를 sampling frequency `f_s = 1kHz`로 sampling한다.

```matlab
clear; clc; close all;

%% Analog-like signal
Dt = 0.00005;
t = -0.005:Dt:0.005;
xa = exp(-1000*abs(t));

%% Discrete-time Signal: fs = 1kHz
fs = 1000;
Ts = 1/fs;
n = -5:5;
x2 = exp(-1000*abs(n*Ts));

%% Discrete-time Fourier Transform
K = 500;
k = -K:1:K;
w = pi*k/K;

X2 = x2 * exp(-1j*(n.'*w));
X2 = real(X2);

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
plot(t*1000, xa, 'LineWidth', 1.5); hold on;
stem(n*Ts*1000, x2, 'filled');
grid on;
title('Discrete Signal sampled at 1kHz');
xlabel('t in msec.');
ylabel('x_2[n]');
legend('Analog-like signal', 'Sampled signal');

subplot(2,1,2);
plot(w/pi, X2, 'LineWidth', 1.5);
grid on;
title('Discrete-time Fourier Transform of x_2[n]');
xlabel('Frequency in \pi units');
ylabel('X_2(e^{j\omega})');
```

#### Practice 2 결과 그래프

<img width="1552" height="987" alt="lab08_P2" src="https://github.com/user-attachments/assets/cba4426b-c898-4f9d-981d-00a94551bb68" />

Practice 2에서는 sampling frequency가 1kHz로 낮아졌기 때문에, 같은 시간 구간에서 얻는 sample 개수가 크게 줄어든다.  
따라서 sampled signal `x2[n]`은 5kHz sampling 결과보다 원래 신호의 세부적인 변화를 덜 정확하게 표현한다.  
특히 `t = 0` 주변에서 급격히 변하는 신호의 형태를 충분히 촘촘하게 잡지 못하므로, 이후 reconstruction 결과에서도 오차가 커질 수 있다.

---

## 9. Lab 08 Practice 3. Reconstruction from 5kHz Samples

Practice 3에서는 Practice 1에서 얻은 `x1[n]`을 이용하여 원래 신호 `x_a(t)`를 sinc function으로 복원한다.

Sinc reconstruction 식은 다음과 같다.

x<sub>r</sub>(t) = Σ x[n]sinc((t - nT) / T)

```matlab
clear; clc; close all;

%% Analog-like signal
Dt = 0.00005;
t = -0.005:Dt:0.005;
xa = exp(-1000*abs(t));

%% Reuse x1[n] from Practice 1
fs1 = 5000;
Ts1 = 1/fs1;
n1 = -25:25;
x1 = exp(-1000*abs(n1*Ts1));

%% Analog signal reconstruction
sinc_basis1 = sinc((t - n1'*Ts1) / Ts1);
reconstructed_xa1 = x1 * sinc_basis1;

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

plot(t*1000, xa, 'LineWidth', 1.5); hold on;
plot(t*1000, reconstructed_xa1, '--', 'LineWidth', 1.5);
stem(n1*Ts1*1000, x1, 'filled');
grid on;
title('Reconstructed Signal from x_1[n] using sinc function');
xlabel('t in msec.');
ylabel('x_r(t)');
legend('Original signal', 'Reconstructed signal', 'Samples');
```

#### Practice 3 결과 그래프

<img width="1562" height="1013" alt="lab08_P3" src="https://github.com/user-attachments/assets/86b3b729-b649-4168-bc03-dcf95c30469a" />

Practice 3 결과에서 5kHz로 sampling한 `x1[n]`을 sinc function으로 복원하면 원래의 `x_a(t)`와 비교적 비슷한 형태가 나타난다.  
이는 sampling interval이 작아 충분히 많은 sample을 얻었기 때문에, 원 신호의 시간 영역 특성을 잘 반영할 수 있었기 때문이다.

다만 `x_a(t)=e^{-1000|t|}`는 완전히 band-limited signal은 아니기 때문에, 이론적으로 완벽한 복원은 어렵고 약간의 오차가 발생할 수 있다.

---

## 10. Lab 08 Practice 4. Reconstruction from 1kHz Samples

Practice 4에서는 Practice 2에서 얻은 `x2[n]`을 이용하여 원래 신호 `x_a(t)`를 sinc function으로 복원한다.

```matlab
clear; clc; close all;

%% Analog-like signal
Dt = 0.00005;
t = -0.005:Dt:0.005;
xa = exp(-1000*abs(t));

%% Reuse x2[n] from Practice 2
fs2 = 1000;
Ts2 = 1/fs2;
n2 = -5:5;
x2 = exp(-1000*abs(n2*Ts2));

%% Analog signal reconstruction
sinc_basis2 = sinc((t - n2'*Ts2) / Ts2);
reconstructed_xa2 = x2 * sinc_basis2;

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

plot(t*1000, xa, 'LineWidth', 1.5); hold on;
plot(t*1000, reconstructed_xa2, '--', 'LineWidth', 1.5);
stem(n2*Ts2*1000, x2, 'filled');
grid on;
title('Reconstructed Signal from x_2[n] using sinc function');
xlabel('t in msec.');
ylabel('x_r(t)');
legend('Original signal', 'Reconstructed signal', 'Samples');
```

#### Practice 4 결과 그래프

<img width="1555" height="1001" alt="lab08_P4" src="https://github.com/user-attachments/assets/ba45ddba-82ab-441f-a4a5-6c0f6b03c538" />

Practice 4 결과에서 1kHz로 sampling한 `x2[n]`을 이용한 reconstruction은 5kHz sampling 결과보다 원래 신호와의 차이가 더 크게 나타난다.  
sample 개수가 적기 때문에 `t = 0` 근처의 급격한 변화를 충분히 표현하지 못하며, sinc interpolation 과정에서도 원 신호와 다른 형태가 일부 나타날 수 있다.

따라서 sampling frequency가 높을수록 원 신호의 시간 영역 특성을 더 정확히 보존할 수 있고, reconstruction 결과도 더 안정적으로 나타난다는 점을 확인할 수 있다.

---

## 11. 실습 내용 정리

이번 실습에서는 Lab 07과 Lab 08 Part 1을 통해 DTFT, sampling, aliasing, sinc reconstruction의 개념을 MATLAB으로 확인하였다.

Lab 07에서는 먼저 matrix-vector product를 이용하여 유한 길이 이산 신호의 DTFT를 수치적으로 계산하였다.  
이 방식은 여러 주파수 점에서의 DTFT 값을 행렬 연산으로 한 번에 계산할 수 있기 때문에 반복문을 사용하는 방식보다 효율적이다.  
또한 magnitude, angle, real part, imaginary part를 각각 plot하여 DTFT가 복소수 주파수 응답이라는 점을 확인하였다.

Practice 1에서는 `y[n] = x[n-2]`를 이용하여 time shifting property를 확인하였다.  
시간 영역에서 신호가 오른쪽으로 이동하면 주파수 영역의 magnitude는 유지되고 phase만 선형적으로 변한다.  
이를 통해 delay가 주파수 영역에서 phase shift로 나타난다는 것을 이해할 수 있었다.

Practice 2에서는 `x[n] = sin(πn/2)`를 even part와 odd part로 분해하고, 각각의 DTFT를 비교하였다.  
시간 영역에서의 even/odd 대칭성이 주파수 영역의 real part와 imaginary part에 반영된다는 점을 확인하였다.

Question 1에서는 5Hz cosine wave를 8Hz와 12Hz로 sampling하고 sinc reconstruction을 수행하였다.  
8Hz는 5Hz 신호의 Nyquist rate인 10Hz보다 낮기 때문에 aliasing이 발생하여 원래 신호를 정확히 복원할 수 없었다.  
반면 12Hz는 Nyquist rate보다 높기 때문에 8Hz보다 원 신호에 가까운 reconstruction 결과를 얻을 수 있었다.

Lab 08에서는 `x_a(t)=e^{-1000|t|}` 신호를 5kHz와 1kHz로 각각 sampling하였다.  
5kHz sampling은 sample 간격이 작아 원래 신호를 더 촘촘하게 표현했으며, sinc reconstruction 결과도 원 신호와 비교적 유사하게 나타났다.  
반면 1kHz sampling은 sample 개수가 적어 신호의 급격한 변화를 충분히 표현하지 못했고, reconstruction 결과에서도 더 큰 오차가 나타났다.

결과적으로 이번 실습을 통해 DTFT를 이용한 이산시간 신호의 주파수 분석 방법과, sampling frequency가 신호 복원 품질에 미치는 영향을 이해할 수 있었다.  
또한 sinc function을 이용한 이상적인 reconstruction 과정을 직접 구현해 보면서, sampling theorem과 aliasing의 의미를 시간 영역과 주파수 영역에서 함께 확인할 수 있었다.
