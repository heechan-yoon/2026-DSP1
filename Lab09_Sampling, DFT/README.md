# [DSP 실습 08+09] Downsampling, Upsampling, DFT 이해

## 1. 실습 목적

- MATLAB의 `downsample()` 함수를 이용하여 이산시간 신호의 sample rate를 감소시키는 방법을 이해한다.
- MATLAB의 `upsample()` 함수를 이용하여 이산시간 신호의 sample rate를 증가시키는 방법을 이해한다.
- Downsampling 과정에서 sequence의 길이와 sample index가 어떻게 변화하는지 확인한다.
- Upsampling 과정에서 원래 sample 사이에 zero가 삽입되는 구조를 이해한다.
- DFT(Discrete Fourier Transform)의 정의와 matrix form을 학습한다.
- 직접 작성한 `dft.m`, `idft.m` 함수와 MATLAB 내장 함수 `fft`, `ifft`의 결과를 비교한다.
- DTFT와 DFT의 관계를 이해하고, DFT가 DTFT를 일정한 frequency grid에서 sampling한 결과임을 확인한다.
- Zero-padding이 DFT의 frequency resolution에 미치는 영향을 분석한다.
- DFT point 수가 충분한 경우와 부족한 경우의 IDFT 복원 결과를 비교한다.

---

## 2. Lab 08 Part 2: Sample Rate Conversion

Lab 08 Part 2에서는 이산시간 신호의 sample rate를 변화시키는 두 가지 기본 연산인 downsampling과 upsampling을 다룬다.

### 2.1 Downsampling

Downsampling은 신호의 sample 중 일부만 선택하여 sample rate를 낮추는 과정이다.  
예를 들어 factor가 3이면 원래 sequence에서 3개마다 하나의 sample만 선택한다.

```matlab
x = [1 2 3 4 5 6 7 8 9 10];

y = downsample(x, 3)
```

결과는 다음과 같다.

```matlab
y = [1 4 7 10]
```

phase offset을 추가하면 시작 위치를 바꿔서 sample을 선택할 수 있다.

```matlab
y = downsample(x, 3, 2)
```

결과는 다음과 같다.

```matlab
y = [3 6 9]
```

즉, `downsample(x, M)`은 원래 신호에서 `M` 간격으로 sample을 선택하는 과정이며, sequence의 길이는 줄어든다.

---

### 2.2 Upsampling

Upsampling은 원래 sample 사이에 zero를 삽입하여 sample rate를 높이는 과정이다.  
factor가 3이면 각 sample 사이에 zero 2개가 추가된다.

```matlab
x = [1 2 3 4];

y = upsample(x, 3)
```

결과는 다음과 같다.

```matlab
y = [1 0 0 2 0 0 3 0 0 4 0 0]
```

phase offset을 추가하면 zero 삽입 위치가 달라진다.

```matlab
y = upsample(x, 3, 2)
```

결과는 다음과 같다.

```matlab
y = [0 0 1 0 0 2 0 0 3 0 0 4]
```

즉, `upsample(x, L)`은 sample 사이에 zero를 삽입하여 sequence의 길이를 증가시키는 과정이다.

---

## 3. Lab 08 Q1. Downsampling

Q1에서는 다음 세 가지 sequence를 생성한다.

1. Normalized frequency가 0.15인 sinusoidal sequence
2. Normalized frequency가 0.1과 0.3인 두 sinusoidal sequence의 합
3. Normalized frequency가 0.15인 sinusoidal sequence와 real exponential sequence `0.8^n`의 곱

각 sequence의 길이는 51로 설정한다.

```matlab
clear; clc; close all;

N = 51;                 % Length of the sequences
n = 0:N-1;              % Time index

% Original sequences
x1 = sin(2*pi*0.15*n);
x2 = sin(2*pi*0.1*n) + sin(2*pi*0.3*n);
x3 = x1 .* (0.8).^n;

% Downsampling factor
M = 2;

% Downsampled sequences
y1 = downsample(x1, M);
y2 = downsample(x2, M);
y3 = downsample(x3, M);

L1 = length(y1);
L2 = length(y2);
L3 = length(y3);

% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,2,1);
stem(0:N-1, x1, 'filled');
grid on;
title('Original Signal x_1[n]');
xlabel('Time index n');
ylabel('x_1[n]');

subplot(3,2,3);
stem(0:N-1, x2, 'filled');
grid on;
title('Original Signal x_2[n]');
xlabel('Time index n');
ylabel('x_2[n]');

subplot(3,2,5);
stem(0:N-1, x3, 'filled');
grid on;
title('Original Signal x_3[n]');
xlabel('Time index n');
ylabel('x_3[n]');

subplot(3,2,2);
stem(0:L1-1, y1, 'filled');
grid on;
title('Downsampled Signal y_1[m], M=2');
xlabel('Time index m');
ylabel('y_1[m]');
axis([0 26 -1.2 1.2]);

subplot(3,2,4);
stem(0:L2-1, y2, 'filled');
grid on;
title('Downsampled Signal y_2[m], M=2');
xlabel('Time index m');
ylabel('y_2[m]');
axis([0 26 -2.2 2.2]);

subplot(3,2,6);
stem(0:L3-1, y3, 'filled');
grid on;
title('Downsampled Signal y_3[m], M=2');
xlabel('Time index m');
ylabel('y_3[m]');
axis([0 26 -1.2 1.2]);
```

#### Q1 결과 그래프

<img width="1032" height="673" alt="lab08_Q1_capture" src="https://github.com/user-attachments/assets/dbfa43b5-bbaf-4767-912c-f7cde9d7ef64" />

Q1에서는 factor-of-2 downsampling을 수행하였다.  
원래 길이가 51인 sequence에서 2개마다 하나의 sample을 선택하므로 downsampled sequence의 길이는 약 절반으로 줄어든다.

`x1[n]`은 하나의 sinusoidal sequence이므로 downsampling 후에도 sinusoidal 형태를 유지하지만, sample 간격이 넓어져 원래 신호보다 더 성긴 형태로 나타난다.  
`x2[n]`은 두 sinusoidal sequence의 합이므로 downsampling 후에도 두 주파수 성분이 섞인 형태가 유지된다.  
`x3[n]`은 sinusoidal sequence에 exponential decay가 곱해진 신호이므로, downsampling 후에도 시간이 지남에 따라 amplitude가 감소하는 특성이 나타난다.

Downsampling은 sample 수를 줄이는 과정이므로 계산량을 줄일 수 있지만, 주파수 성분이 folding되어 aliasing이 발생할 수 있다.  
따라서 실제 시스템에서는 downsampling 전에 anti-aliasing filter를 적용하는 것이 중요하다.

---

## 4. Lab 08 Q2. Upsampling

Q2에서는 Q1과 같은 세 가지 sequence를 생성하고, factor-of-2와 factor-of-4 upsampling을 수행한다.

Upsampling은 원래 sample 사이에 zero를 삽입하여 sequence의 길이를 증가시키는 과정이다.

```matlab
clear; clc; close all;

N = 51;
n = 0:N-1;

% Original sequences
x1 = sin(2*pi*0.15*n);
x2 = sin(2*pi*0.1*n) + sin(2*pi*0.3*n);
x3 = x1 .* (0.8).^n;

%% Factor-of-2 upsampling
L = 2;

u1_2 = upsample(x1, L);
u2_2 = upsample(x2, L);
u3_2 = upsample(x3, L);

n_u2 = 0:length(u1_2)-1;

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,2,1);
stem(n, x1, 'filled');
grid on;
title('Original Signal x_1[n]');
xlabel('Time index n');
ylabel('x_1[n]');

subplot(3,2,3);
stem(n, x2, 'filled');
grid on;
title('Original Signal x_2[n]');
xlabel('Time index n');
ylabel('x_2[n]');

subplot(3,2,5);
stem(n, x3, 'filled');
grid on;
title('Original Signal x_3[n]');
xlabel('Time index n');
ylabel('x_3[n]');

subplot(3,2,2);
stem(n_u2, u1_2, 'filled');
grid on;
title('Upsampled Signal u_1[n], L=2');
xlabel('Time index n');
ylabel('u_1[n]');
axis([0 105 -1.2 1.2]);

subplot(3,2,4);
stem(n_u2, u2_2, 'filled');
grid on;
title('Upsampled Signal u_2[n], L=2');
xlabel('Time index n');
ylabel('u_2[n]');
axis([0 105 -2.2 2.2]);

subplot(3,2,6);
stem(n_u2, u3_2, 'filled');
grid on;
title('Upsampled Signal u_3[n], L=2');
xlabel('Time index n');
ylabel('u_3[n]');
axis([0 105 -1.2 1.2]);

%% Factor-of-4 upsampling
L = 4;

u1_4 = upsample(x1, L);
u2_4 = upsample(x2, L);
u3_4 = upsample(x3, L);

n_u4 = 0:length(u1_4)-1;

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,2,1);
stem(n, x1, 'filled');
grid on;
title('Original Signal x_1[n]');
xlabel('Time index n');
ylabel('x_1[n]');

subplot(3,2,3);
stem(n, x2, 'filled');
grid on;
title('Original Signal x_2[n]');
xlabel('Time index n');
ylabel('x_2[n]');

subplot(3,2,5);
stem(n, x3, 'filled');
grid on;
title('Original Signal x_3[n]');
xlabel('Time index n');
ylabel('x_3[n]');

subplot(3,2,2);
stem(n_u4, u1_4, 'filled');
grid on;
title('Upsampled Signal u_1[n], L=4');
xlabel('Time index n');
ylabel('u_1[n]');
axis([0 210 -1.2 1.2]);

subplot(3,2,4);
stem(n_u4, u2_4, 'filled');
grid on;
title('Upsampled Signal u_2[n], L=4');
xlabel('Time index n');
ylabel('u_2[n]');
axis([0 210 -2.2 2.2]);

subplot(3,2,6);
stem(n_u4, u3_4, 'filled');
grid on;
title('Upsampled Signal u_3[n], L=4');
xlabel('Time index n');
ylabel('u_3[n]');
axis([0 210 -1.2 1.2]);
```

#### Q2 결과 그래프

<img width="1012" height="667" alt="lab08_Q2_capture1" src="https://github.com/user-attachments/assets/dc95d8f5-de94-4e7b-acab-b23974f45eed" />

<img width="1002" height="663" alt="lab08_Q2_capture2" src="https://github.com/user-attachments/assets/7ce0b011-6ce8-4085-84fc-75de3a052132" />

Q2에서는 factor-of-2와 factor-of-4 upsampling을 수행하였다.  
Upsampling을 하면 원래 sample은 유지되고, sample 사이에 zero가 삽입된다.

Factor가 2인 경우 원래 sample 사이에 zero가 1개씩 삽입되고, factor가 4인 경우 원래 sample 사이에 zero가 3개씩 삽입된다.  
따라서 upsampling factor가 증가할수록 sequence의 길이는 더 길어지고, zero sample의 개수도 증가한다.

Upsampling 자체는 새로운 정보를 만들어내는 과정이 아니라 zero insertion 과정이다.  
따라서 실제 interpolation을 위해서는 upsampling 이후 low-pass filter를 적용하여 zero insertion으로 인해 생긴 spectral image를 제거해야 한다.

---

## 5. Lab 09: DFT

Lab 09에서는 DFT와 IDFT의 개념을 학습한다.

DFT는 유한 길이 sequence `x[n]`을 유한 개의 frequency coefficient `X[k]`로 변환하는 방법이다.

DFT 정의식은 다음과 같다.

X[k] = Σ x[n] exp(-j2πkn/N), 0 ≤ k ≤ N-1

IDFT 정의식은 다음과 같다.

x[n] = 1/N Σ X[k] exp(j2πkn/N), 0 ≤ n ≤ N-1

DFT는 DTFT를 `N`개의 균일한 주파수 지점에서 sampling한 결과로 해석할 수 있다.  
즉, DFT coefficient `X[k]`는 다음 주파수에서의 DTFT 값과 대응된다.

ω<sub>k</sub> = 2πk/N

---

## 6. DFT Matrix와 MATLAB 구현

DFT는 matrix multiplication으로 표현할 수 있다.

X = W<sub>N</sub>x

여기서 DFT matrix의 각 원소는 다음과 같다.

W<sub>N</sub>[n,k] = exp(-j2πnk/N)

MATLAB에서는 직접 matrix를 만들거나, 내장 함수 `fft()`를 이용하여 DFT를 계산할 수 있다.  
IDFT는 inverse DFT matrix를 사용하거나 `ifft()`를 이용하여 계산할 수 있다.

```matlab
clear; clc; close all;

x = [1 2 2 1];
n = 0:3;
N = 4;

% DTFT
M = 1600;
k = 0:M-1;
w1 = 2*pi*k/M;

X1 = x * exp(-1j*(n.'*w1));

% 4-point DFT
k_dft = 0:N-1;
w2 = 2*pi*k_dft/N;

W_N = exp(-1j*2*pi/N*(n.'*k_dft));
X2 = x * W_N;

% IDFT
x2 = (1/N) * X2 * conj(W_N).';

% Built-in functions
X2_fft = fft(x, N);
x2_ifft = ifft(X2_fft, N);

% Plot DTFT and DFT samples
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
plot(w1/pi, abs(X1), '--', 'LineWidth', 1.5); hold on;
stem(w2/pi, abs(X2), 'filled');
grid on;
title('Magnitude: DTFT and 4-point DFT');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Magnitude');
legend('DTFT', '4-point DFT');

subplot(2,1,2);
plot(w1/pi, angle(X1), '--', 'LineWidth', 1.5); hold on;
stem(w2/pi, angle(X2), 'filled');
grid on;
title('Phase: DTFT and 4-point DFT');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Phase');
legend('DTFT', '4-point DFT');
```

#### DFT 예제 결과 그래프

<img width="1548" height="1017" alt="lab09_example" src="https://github.com/user-attachments/assets/4cc6552c-3490-408b-91bb-eb527f565cc5" />

이 예제에서는 x[n] = [1, 2, 2, 1]의 DTFT와 4-point DFT를 비교하였다. 
DTFT는 연속적인 주파수 축에서 계산된 주파수 응답이고, 4-point DFT는 그 DTFT를 네 개의 주파수 지점에서 sampling한 결과이다.

그래프에서 plot으로 표시된 곡선은 DTFT이고, stem으로 표시된 점들은 4-point DFT 값이다. 
DFT의 주파수 샘플 위치는 ω = 2πk/4, k = 0, 1, 2, 3 이므로 normalized frequency 기준으로 ω/π = 0, 0.5, 1, 1.5에 해당한다. 
따라서 stem으로 표시된 DFT 값들이 DTFT 곡선 위의 특정 지점에 위치하는 것을 확인할 수 있다.

이를 통해 DFT는 DTFT를 일정한 주파수 간격으로 sampling한 결과라는 것을 확인할 수 있다. 
단, magnitude가 0인 지점에서는 phase가 이론적으로 정의되지 않으므로 해당 phase 값은 큰 의미를 갖지 않는다.

---

## 7. Lab 09 Practice 1. dft.m / idft.m 함수 작성

Practice 1에서는 직접 DFT와 IDFT를 계산하는 함수를 작성한다.

### 7.1 dft.m

```matlab
function [Xk] = dft(xn)
% Compute DFT
% Xk = DFT coefficient array over 0 <= k <= N-1
% xn = N-point finite-duration sequence
% N = length of DFT

    N = length(xn);
    n = 0:1:N-1;       % time index
    k = 0:1:N-1;       % frequency index

    W_N = exp(-1j*2*pi/N*(n.'*k));
    Xk = xn * W_N;
end
```

---

### 7.2 idft.m

```matlab
function [xn] = idft(Xk)
% Compute IDFT
% xn = N-point sequence over 0 <= n <= N-1
% Xk = DFT coefficient array over 0 <= k <= N-1
% N = length of IDFT

    N = length(Xk);
    k = 0:1:N-1;       % frequency index
    n = 0:1:N-1;       % time index

    W_N_inv = exp(1j*2*pi/N*(k.'*n));
    xn = (1/N) * Xk * W_N_inv;
end
```

---

### 7.3 DFT와 IDFT 검증

다음 sequence에 대해 `idft(dft(x))`를 계산하고, 원래 sequence와 복원된 sequence가 같은지 확인한다.

x[n] = [1, 1, 1, 1, 0, 0, 0, 0]

```matlab
clear; clc; close all;

x = [1 1 1 1 0 0 0 0];

X = dft(x);
y = idft(X);

error = abs(x - y);

disp('Original x[n] = ');
disp(x);

disp('Reconstructed y[n] = idft(dft(x)) = ');
disp(y);

disp('Absolute error abs(x-y) = ');
disp(error);
```

#### Practice 1 결과

```matlab
abs(x - y) ≈ 0
```

`idft(dft(x))`의 결과가 원래 sequence와 거의 같게 나타나므로, 직접 작성한 `dft.m`과 `idft.m` 함수가 정상적으로 동작함을 확인할 수 있다.

---

## 8. Lab 09 Q1. DTFT와 8-point DFT 비교

Q1에서는 다음 4-point sequence를 사용한다.

x[n] = 1, 0 ≤ n ≤ 3  
x[n] = 0, otherwise

즉,

x[n] = [1, 1, 1, 1]

이 sequence의 DTFT는 다음과 같다.

X(e<sup>jΩ</sup>) = 1 + e<sup>-jΩ</sup> + e<sup>-j2Ω</sup> + e<sup>-j3Ω</sup>

Q1의 목표는 이 신호의 DTFT와 8-point DFT `X1[k]`를 같은 평면에 나타내고, magnitude와 phase를 비교하는 것이다.

```matlab
clear; clc; close all;

% 4-point sequence
x = [1 1 1 1];
n = 0:3;

% DTFT
M = 1600;
k_dtft = 0:M-1;
w = 2*pi*k_dtft/M;

X = x * exp(-1j*(n.'*w));

% 8-point DFT
N = 8;
x8 = [x zeros(1, N-length(x))];
k_dft = 0:N-1;
w_dft = 2*pi*k_dft/N;

X1 = dft(x8);

% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
plot(w, abs(X), 'LineWidth', 1.5); hold on;
stem(w_dft, abs(X1), 'filled');
grid on;
title('Magnitude of DTFT and 8-point DFT');
xlabel('Angular frequency \omega');
ylabel('Magnitude');
legend('DTFT X(e^{j\omega})', '8-point DFT X_1[k]');

subplot(2,1,2);
plot(w, angle(X), 'LineWidth', 1.5); hold on;
stem(w_dft, angle(X1), 'filled');
grid on;
title('Phase of DTFT and 8-point DFT');
xlabel('Angular frequency \omega');
ylabel('Phase');
legend('DTFT X(e^{j\omega})', '8-point DFT X_1[k]');
```

#### Q1 결과 그래프

<img width="1001" height="681" alt="lab09_Q1_capture" src="https://github.com/user-attachments/assets/b8b82f5a-d362-4a00-a61d-3556e028a6cb" />

Q1 결과에서 8-point DFT 값은 DTFT curve 위의 특정 frequency sample 지점에 위치한다.  
이는 DFT가 DTFT를 일정한 간격으로 sampling한 결과임을 보여준다.

4-point sequence `[1, 1, 1, 1]`는 rectangular sequence이므로 magnitude response는 low-frequency 영역에서 큰 값을 가지고, 특정 주파수에서 null이 발생한다.  
8-point DFT는 이러한 DTFT의 전체 곡선을 모두 표현하는 것이 아니라 8개의 discrete frequency point에서의 값만 나타낸다.

---

## 9. Lab 09 Practice 2. DFT, Zero Padding, FFT 비교

Practice 2에서는 다음 sequence를 생성한다.

x[n] = (0.95)<sup>n</sup> cos(πn/20), 0 ≤ n ≤ 63

먼저 64-point sequence를 생성하고, DFT를 계산한다.  
이후 오른쪽에 64개의 zero를 추가하여 128-point sequence를 만든 뒤 128-point DFT를 계산한다.  
마지막으로 `fft()` 함수를 이용해 계산한 결과와 직접 작성한 `dft()` 함수 결과를 비교한다.

```matlab
clear; clc; close all;

%% Generate sequence
n = 0:63;
x = (0.95).^n .* cos(pi*n/20);

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);
stem(n, x, 'filled');
grid on;
title('Sequence x[n] = (0.95)^n cos(\pi n / 20)');
xlabel('n');
ylabel('x[n]');

%% 64-point DFT
X1 = dft(x);
k1 = 0:length(X1)-1;

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
stem(k1, abs(X1), 'filled');
grid on;
title('Magnitude of 64-point DFT X_1[k]');
xlabel('k');
ylabel('|X_1[k]|');

subplot(2,1,2);
stem(k1, angle(X1), 'filled');
grid on;
title('Phase of 64-point DFT X_1[k]');
xlabel('k');
ylabel('Phase');

%% 128-point DFT by zero-padding
x_pad = [x zeros(1, 64)];
X2 = dft(x_pad);
k2 = 0:length(X2)-1;

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
stem(k2, abs(X2), 'filled');
grid on;
title('Magnitude of 128-point DFT X_2[k]');
xlabel('k');
ylabel('|X_2[k]|');

subplot(2,1,2);
stem(k2, angle(X2), 'filled');
grid on;
title('Phase of 128-point DFT X_2[k]');
xlabel('k');
ylabel('Phase');

%% FFT comparison
Y = fft(x);

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
stem(k1, abs(X1), 'filled');
grid on;
title('Magnitude of DFT using dft.m');
xlabel('k');
ylabel('|X_1[k]|');

subplot(2,1,2);
stem(k1, abs(Y), 'filled');
grid on;
title('Magnitude of DFT using fft()');
xlabel('k');
ylabel('|Y[k]|');
```

#### Practice 2 결과 그래프

<img width="1007" height="667" alt="lab09_P2_capture_a" src="https://github.com/user-attachments/assets/bb00fbe8-0954-4454-9c67-f736485797e3" />

<img width="1010" height="671" alt="lab09_P2_capture_b" src="https://github.com/user-attachments/assets/30677f97-525c-436b-923f-0534be0af7d5" />

<img width="1002" height="681" alt="lab09_P2_capture_c" src="https://github.com/user-attachments/assets/4935eaa8-e395-45e9-b8b1-be7377e95001" />

<img width="998" height="668" alt="lab09_P2_capture_d" src="https://github.com/user-attachments/assets/b08ba2bd-e742-45ec-939a-749d8d12099a" />

Practice 2 결과에서 64-point DFT는 64개의 frequency bin에서 spectrum을 나타낸다.  
128-point DFT는 원래 sequence 뒤에 zero를 추가한 후 DFT를 계산한 것이므로 frequency sample point 수가 증가한다.

Zero-padding은 원래 신호에 새로운 정보를 추가하는 것이 아니라, DTFT를 더 촘촘한 frequency grid에서 sampling하는 효과를 준다.  
따라서 magnitude spectrum의 전체적인 모양은 같지만, 128-point DFT에서는 더 부드럽고 촘촘한 frequency response를 확인할 수 있다.

또한 직접 작성한 `dft()` 함수와 MATLAB 내장 함수 `fft()`의 결과는 거의 동일하게 나타난다.  
이를 통해 `fft()`는 DFT를 빠르게 계산하는 알고리즘이며, 수학적으로는 DFT와 같은 결과를 제공한다는 것을 확인할 수 있다.

---

## 10. Lab 09 Q2. DTFT Sampling과 IDFT 복원

Q2에서는 다음 sequence를 사용한다.

x[n] = (0.95)<sup>n</sup> cos(πn/20), 0 ≤ n ≤ 63

먼저 DTFT를 서로 다른 개수의 frequency sample로 sampling한다.

- M1 = 100
- M2 = 40

이후 각각의 sampled DTFT sequence에 대해 직접 작성한 `idft()` 함수를 적용하여 시간 영역 sequence `x1[n]`, `x2[n]`를 복원하고 원래 `x[n]`과 비교한다.

```matlab
clear; clc; close all;

%% Original sequence
n = 0:63;
x = (0.95).^n .* cos(pi*n/20);

%% DTFT sampled with M1 = 100
M1 = 100;
k1 = 0:M1-1;
w1 = 2*pi*k1/M1;

X1 = x * exp(-1j*(n.'*w1));

%% DTFT sampled with M2 = 40
M2 = 40;
k2 = 0:M2-1;
w2 = 2*pi*k2/M2;

X2 = x * exp(-1j*(n.'*w2));

%% Plot magnitudes
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
stem(w1, abs(X1), 'filled');
grid on;
title('Magnitude of DTFT Samples, M_1 = 100');
xlabel('Angular frequency \omega');
ylabel('|X_1[k]|');

subplot(2,1,2);
stem(w2, abs(X2), 'filled');
grid on;
title('Magnitude of DTFT Samples, M_2 = 40');
xlabel('Angular frequency \omega');
ylabel('|X_2[k]|');

%% IDFT reconstruction
x1 = idft(X1);
x2 = idft(X2);

n1 = 0:length(x1)-1;
n2 = 0:length(x2)-1;

%% Plot comparison
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,1,1);
stem(n, x, 'filled');
grid on;
title('Original Sequence x[n]');
xlabel('n');
ylabel('x[n]');

subplot(3,1,2);
stem(n1, real(x1), 'filled');
grid on;
title('Reconstructed Sequence x_1[n] from M_1 = 100');
xlabel('n');
ylabel('x_1[n]');

subplot(3,1,3);
stem(n2, real(x2), 'filled');
grid on;
title('Reconstructed Sequence x_2[n] from M_2 = 40');
xlabel('n');
ylabel('x_2[n]');
```

#### Q2 결과 그래프

<img width="1018" height="677" alt="lab09_Q2_capture_a" src="https://github.com/user-attachments/assets/6d892d15-d2dc-4755-8ad6-5652593db33e" />

<img width="1005" height="668" alt="lab09_Q2_capture_b" src="https://github.com/user-attachments/assets/913c3e5a-1b2f-412c-a44b-1af8cda48f24" />

Q2 결과에서 M1 = 100인 경우, frequency sample 개수가 원래 sequence 길이 64보다 크다.  
따라서 IDFT를 수행하면 원래 신호가 충분히 보존되며, `x1[n]`은 원래 sequence `x[n]`와 유사하게 나타난다.

반면 M2 = 40인 경우, frequency sample 개수가 원래 sequence 길이보다 작다.  
이 경우 주파수 영역 sample 수가 부족하므로 시간 영역에서 aliasing이 발생한다.  
따라서 `x2[n]`는 원래 sequence `x[n]`와 정확히 일치하지 않으며, 일부 sample이 겹쳐진 형태로 나타날 수 있다.

즉, DFT point 수가 원래 sequence 길이보다 충분히 크거나 같아야 시간 영역 sequence를 올바르게 복원할 수 있다.

---

## 11. 실습 내용 정리

이번 실습에서는 Lab 08 Part 2와 Lab 09를 통해 sample rate conversion과 DFT/IDFT의 개념을 MATLAB으로 확인하였다.

Lab 08 Part 2에서는 먼저 `downsample()` 함수를 이용하여 factor-of-2 downsampling을 수행하였다.  
Downsampling은 원래 sequence에서 일정 간격의 sample만 선택하여 sequence의 길이를 줄이는 과정이다.  
이 과정에서 sample 수가 줄어들기 때문에 계산량은 감소하지만, anti-aliasing filter 없이 수행하면 주파수 성분이 겹치는 aliasing이 발생할 수 있다.

또한 `upsample()` 함수를 이용하여 factor-of-2와 factor-of-4 upsampling을 수행하였다.  
Upsampling은 원래 sample 사이에 zero를 삽입하여 sequence 길이를 증가시키는 과정이다.  
Factor가 커질수록 zero가 더 많이 삽입되며, 실제 interpolation을 위해서는 upsampling 이후 low-pass filter가 필요하다.

Lab 09에서는 DFT와 IDFT의 정의를 학습하였다.  
DFT는 유한 길이 sequence를 유한 개의 frequency coefficient로 변환하는 방법이며, DTFT를 균일한 frequency grid에서 sampling한 결과로 해석할 수 있다.  
직접 작성한 `dft.m`과 `idft.m` 함수를 통해 DFT matrix를 이용한 변환과 복원을 구현하였고, `idft(dft(x))`가 원래 sequence를 복원함을 확인하였다.

Q1에서는 4-point rectangular sequence `[1, 1, 1, 1]`의 DTFT와 8-point DFT를 비교하였다.  
DFT sample들이 DTFT curve 위에 위치하는 것을 통해 DFT가 DTFT의 discrete frequency sample이라는 사실을 확인할 수 있었다.

Practice 2에서는 `x[n]=(0.95)^n cos(πn/20)`에 대해 64-point DFT, zero-padding을 적용한 128-point DFT, 그리고 MATLAB `fft()` 결과를 비교하였다.  
Zero-padding은 frequency sample point를 증가시켜 spectrum을 더 촘촘하게 보여 주지만, 원래 신호에 새로운 정보를 추가하는 것은 아니다.  
또한 `fft()` 결과와 직접 작성한 `dft()` 결과가 거의 같으므로, FFT는 DFT를 빠르게 계산하는 알고리즘임을 확인하였다.

마지막으로 Q2에서는 DTFT를 M1 = 100과 M2 = 40개의 frequency sample로 sampling한 뒤 IDFT를 수행하였다.  
M1 = 100은 원래 sequence 길이보다 충분히 크기 때문에 원 신호를 잘 복원할 수 있었지만, M2 = 40은 sample 수가 부족하여 시간 영역 aliasing이 발생하고 원래 sequence와 차이가 나타났다.

결과적으로 이번 실습을 통해 downsampling, upsampling, DFT, IDFT, zero-padding, FFT, DTFT sampling의 관계를 종합적으로 이해할 수 있었다.
