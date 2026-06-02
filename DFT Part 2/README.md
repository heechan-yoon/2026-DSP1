# [DSP 실습 10] DFT Part 2 및 Linearity Property 이해

## 1. 실습 목적

- DFT(Discrete Fourier Transform)와 IDFT(Inverse Discrete Fourier Transform)의 정의를 복습한다.
- DFT matrix를 이용하여 이산시간 신호를 주파수 영역으로 변환하는 방법을 이해한다.
- 동일한 신호에 대해 4-point DFT와 16-point DFT를 비교하고, DFT point 수에 따른 frequency sampling 간격의 차이를 확인한다.
- DTFT를 서로 다른 개수의 frequency point에서 sampling한 뒤 IDFT를 적용하여 시간 영역 신호를 복원한다.
- Frequency sample 수가 원래 sequence 길이보다 부족할 때 발생하는 time-domain aliasing을 분석한다.
- IDFS(Inverse Discrete Fourier Series)를 이용하여 복원된 sequence의 periodic extension을 확인한다.
- DFT의 linearity property를 MATLAB로 검증한다.
- 직접 계산한 `X3[k]`와 `3X1[k] + 2X2[k]`가 동일한지 비교한다.

---

## 2. Lab 10: DFT Part 2

Lab 10에서는 지난 실습에서 학습한 DFT와 IDFT를 복습하고, DTFT sampling과 DFT linearity property를 추가로 확인한다.

DFT는 길이가 `N`인 이산시간 sequence `x[n]`을 `N`개의 주파수 계수 `X[k]`로 변환하는 방법이다.

DFT 정의식은 다음과 같다.

X[k] = Σ x[n] exp(-j2πkn/N), 0 ≤ k ≤ N-1

IDFT 정의식은 다음과 같다.

x[n] = 1/N Σ X[k] exp(j2πkn/N), 0 ≤ n ≤ N-1

DFT는 DTFT를 일정한 간격으로 sampling한 결과로 해석할 수 있다.

ω<sub>k</sub> = 2πk/N

따라서 DFT point 수 `N`이 증가하면 동일한 DTFT를 더 촘촘한 frequency grid에서 sampling할 수 있다.

---

## 3. DFT Matrix 복습

DFT는 matrix multiplication으로 표현할 수 있다.

X = W<sub>N</sub>x

여기서 DFT matrix의 각 원소는 다음과 같다.

W<sub>N</sub>[n,k] = exp(-j2πnk/N)

IDFT는 inverse DFT matrix를 이용하여 다음과 같이 계산한다.

x = (1/N)W<sub>N</sub><sup>*</sup>X

MATLAB에서는 다음과 같이 DFT matrix를 직접 생성할 수 있다.

```matlab
N = length(x);

n = 0:N-1;
k = 0:N-1;

W_N = exp(-1j*2*pi/N*(n.'*k));

X = x * W_N;

x_reconstructed = (1/N) * X * conj(W_N).';
```

---

## 4. Example: 4-point DFT와 16-point DFT 비교

다음 sequence를 사용한다.

x[n] = [1, 2, 2, 1]

먼저 4-point DFT를 계산하고, 이후 zero-padding을 적용하여 16-point DFT를 계산한다.

```matlab
clear; clc; close all;

%% Original sequence
x = [1 2 2 1];
n = 0:3;

%% DTFT
M = 1600;
w = 2*pi*(0:M-1)/M;

X = x * exp(-1j*(n.'*w));

%% 4-point DFT
N1 = 4;

x_pad1 = [x zeros(1, N1-length(x))];

n1 = 0:N1-1;
k1 = 0:N1-1;
w1 = 2*pi*k1/N1;

W_N1 = exp(-1j*2*pi/N1*(n1.'*k1));
X1 = x_pad1 * W_N1;

%% 16-point DFT
N2 = 16;

x_pad2 = [x zeros(1, N2-length(x))];

n2 = 0:N2-1;
k2 = 0:N2-1;
w2 = 2*pi*k2/N2;

W_N2 = exp(-1j*2*pi/N2*(n2.'*k2));
X2 = x_pad2 * W_N2;

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,2,1);
plot(w/pi, abs(X), '--', 'LineWidth', 1.5); hold on;
stem(w1/pi, abs(X1), 'filled');
grid on;
xlim([0 2]);
title('Magnitude: DTFT and 4-point DFT');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Magnitude');
legend('DTFT', '4-point DFT');

subplot(2,2,2);
plot(w/pi, abs(X), '--', 'LineWidth', 1.5); hold on;
stem(w2/pi, abs(X2), 'filled');
grid on;
xlim([0 2]);
title('Magnitude: DTFT and 16-point DFT');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Magnitude');
legend('DTFT', '16-point DFT');

subplot(2,2,3);
plot(w/pi, angle(X), '--', 'LineWidth', 1.5); hold on;
stem(w1/pi, angle(X1), 'filled');
grid on;
xlim([0 2]);
title('Phase: DTFT and 4-point DFT');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Phase');
legend('DTFT', '4-point DFT');

subplot(2,2,4);
plot(w/pi, angle(X), '--', 'LineWidth', 1.5); hold on;
stem(w2/pi, angle(X2), 'filled');
grid on;
xlim([0 2]);
title('Phase: DTFT and 16-point DFT');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Phase');
legend('DTFT', '16-point DFT');
```

#### Example 결과 그래프

<img width="1587" height="1015" alt="example" src="https://github.com/user-attachments/assets/2453bae8-00d0-4598-9bb0-06a581d09d59" />

4-point DFT와 16-point DFT는 동일한 DTFT를 서로 다른 개수의 주파수 지점에서 sampling한 결과이다.

4-point DFT는 다음 네 개의 normalized frequency에서 값을 가진다.

ω/π = 0, 0.5, 1, 1.5

반면 16-point DFT는 더 많은 frequency sample을 가지므로 DTFT curve의 형태를 더 촘촘하게 나타낸다.

Zero-padding은 원래 신호에 새로운 정보를 추가하지 않는다.  
단지 DTFT를 더 세밀한 frequency grid에서 sampling하는 효과를 만든다.

---

## 5. Practice 1: DTFT Sampling과 IDFT 복원

Practice 1에서는 다음 sequence를 사용한다.

x[n] = (0.95)<sup>n</sup> cos(πn/20), 0 ≤ n ≤ 63

DTFT를 서로 다른 개수의 frequency point에서 sampling한다.

- M<sub>1</sub> = 100
- M<sub>2</sub> = 40

이후 각각의 DTFT sample에 대해 IDFT와 IDFS를 적용하고, 원래 sequence와 비교한다.

---

### 5.1 Practice 1-(a): DTFT Sampling

```matlab
clear; clc; close all;

%% Original sequence
n = 0:63;
x = (0.95).^n .* cos(pi*n/20);

%% Frequency sample sizes
M1 = 100;
M2 = 40;

k1 = 0:M1-1;
k2 = 0:M2-1;

w1 = 2*pi*k1/M1;
w2 = 2*pi*k2/M2;

%% DTFT sampling
X1 = x * exp(-1j*(n.'*w1));
X2 = x * exp(-1j*(n.'*w2));

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,1,1);
stem(w1, abs(X1), 'filled');
grid on;
xlim([0 2*pi]);
title('Magnitude of DTFT Samples, M_1 = 100');
xlabel('Angular Frequency \omega');
ylabel('|X_1[k]|');

subplot(2,1,2);
stem(w2, abs(X2), 'filled');
grid on;
xlim([0 2*pi]);
title('Magnitude of DTFT Samples, M_2 = 40');
xlabel('Angular Frequency \omega');
ylabel('|X_2[k]|');
```

#### Practice 1-(a) 결과 그래프

<img width="1023" height="682" alt="lab10_P1_capture_a" src="https://github.com/user-attachments/assets/a07e9e5c-16bd-44be-921e-c41d2b06f583" />

M<sub>1</sub> = 100인 경우에는 DTFT를 100개의 frequency point에서 sampling한다.  
M<sub>2</sub> = 40인 경우에는 DTFT를 40개의 frequency point에서 sampling한다.

따라서 두 결과의 전체적인 spectrum 형태는 유사하지만, M<sub>1</sub> = 100인 경우가 더 촘촘한 frequency grid를 제공한다.

---

### 5.2 Practice 1-(b): IDFT를 이용한 시간 영역 복원

Practice 1-(b)에서는 `X1`, `X2`에 대해 지난 실습에서 작성한 `idft.m` 함수를 적용한다.

```matlab
clear; clc; close all;

%% Original sequence
n = 0:63;
x = (0.95).^n .* cos(pi*n/20);

%% Frequency sample sizes
M1 = 100;
M2 = 40;

k1 = 0:M1-1;
k2 = 0:M2-1;

w1 = 2*pi*k1/M1;
w2 = 2*pi*k2/M2;

%% DTFT sampling
X1 = x * exp(-1j*(n.'*w1));
X2 = x * exp(-1j*(n.'*w2));

%% IDFT reconstruction
x1 = idft(X1);
x2 = idft(X2);

n1 = 0:length(x1)-1;
n2 = 0:length(x2)-1;

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,1,1);
stem(n, x, 'filled');
grid on;
xlim([0 63]);
title('Original Sequence x[n]');
xlabel('n');
ylabel('x[n]');

subplot(3,1,2);
stem(n1, real(x1), 'filled');
grid on;
xlim([0 63]);
title('Reconstructed Sequence x_1[n], M_1 = 100');
xlabel('n');
ylabel('x_1[n]');

subplot(3,1,3);
stem(n2, real(x2), 'filled');
grid on;
xlim([0 63]);
title('Reconstructed Sequence x_2[n], M_2 = 40');
xlabel('n');
ylabel('x_2[n]');
```

#### Practice 1-(b) 결과 그래프

<img width="1022" height="656" alt="lab10_P1_capture_b" src="https://github.com/user-attachments/assets/3986bc5d-a323-43e5-93bb-cc008c7698ff" />

M<sub>1</sub> = 100인 경우, frequency sample 수가 원래 sequence 길이 64보다 크다.

M<sub>1</sub> = 100 ≥ 64

따라서 IDFT를 수행하면 원래 sequence `x[n]`이 보존된다.  
복원된 `x1[n]`의 앞부분은 원래 sequence와 일치하고, 나머지 구간은 zero에 가까운 값으로 나타난다.

반면 M<sub>2</sub> = 40인 경우, frequency sample 수가 원래 sequence 길이보다 작다.

M<sub>2</sub> = 40 < 64

이 경우 time-domain aliasing이 발생한다.  
길이가 40인 sequence로 복원되는 과정에서 원래 신호의 일부 sample이 겹쳐진다.

구체적으로 `0 ≤ n ≤ 23` 구간에서는 다음과 같은 중첩이 발생한다.

x<sub>2</sub>[n] = x[n] + x[n+40]

따라서 `x2[n]`은 원래 sequence `x[n]`과 완전히 동일하지 않다.

---

### 5.3 Practice 1-(c): IDFS를 이용한 Periodic Sequence 복원

IDFS는 주파수 계수로부터 periodic sequence를 복원한다.

이번 실습에서는 `idfs.m` 함수를 이용하여 복원한 sequence의 periodic extension을 확인한다.

#### `idfs.m`

```matlab
function [xn, n] = idfs(Xk)
% Compute IDFS
% xn = Periodic sequence reconstructed from Xk
% Xk = DFS coefficient array
% N  = Period of sequence

    N = length(Xk);

    n = -2*N:2*N-1;
    k = 0:N-1;

    W_N = exp(-1j*2*pi/N);
    nk = k.'*n;

    W_N_nk = W_N.^(-nk);

    xn = (Xk * W_N_nk) / N;
end
```

#### Practice 1-(c) MATLAB 코드

```matlab
clear; clc; close all;

%% Original sequence
n = 0:63;
x = (0.95).^n .* cos(pi*n/20);

%% Frequency sample sizes
M1 = 100;
M2 = 40;

k1 = 0:M1-1;
k2 = 0:M2-1;

w1 = 2*pi*k1/M1;
w2 = 2*pi*k2/M2;

%% DTFT sampling
X1 = x * exp(-1j*(n.'*w1));
X2 = x * exp(-1j*(n.'*w2));

%% IDFS reconstruction
[x_tilde1, n_tilde1] = idfs(X1);
[x_tilde2, n_tilde2] = idfs(X2);

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,1,1);
stem(n, x, 'filled');
grid on;
xlim([0 80]);
title('Original Sequence x[n]');
xlabel('n');
ylabel('x[n]');

subplot(3,1,2);
stem(n_tilde1, real(x_tilde1), 'filled');
grid on;
xlim([0 80]);
title('Periodic Sequence \tilde{x}_1[n], M_1 = 100');
xlabel('n');
ylabel('\tilde{x}_1[n]');

subplot(3,1,3);
stem(n_tilde2, real(x_tilde2), 'filled');
grid on;
xlim([0 80]);
title('Periodic Sequence \tilde{x}_2[n], M_2 = 40');
xlabel('n');
ylabel('\tilde{x}_2[n]');
```

#### Practice 1-(c) 결과 그래프

<img width="1007" height="661" alt="lab10_P1_capture_c" src="https://github.com/user-attachments/assets/665113a7-7f21-427e-8918-907f3118e4ed" />

IDFS를 적용하면 복원된 sequence는 주기적으로 반복된다.

`X1`의 길이는 100이므로 `x_tilde1[n]`은 주기 100을 가진다.

`X2`의 길이는 40이므로 `x_tilde2[n]`은 주기 40을 가진다.

M<sub>1</sub> = 100인 경우에는 원래 sequence 길이 64보다 period가 크므로 원래 sequence가 충분히 보존된다.

반면 M<sub>2</sub> = 40인 경우에는 원래 sequence 길이보다 period가 짧으므로 time-domain aliasing이 발생한다.  
또한 주기 40의 sequence가 반복되므로 `0 ≤ n ≤ 80` 범위에서 동일한 waveform이 반복되어 나타난다.

---

## 6. Helper Functions

### 6.1 `dft.m`

```matlab
function [Xk] = dft(xn)
% Compute DFT
% Xk = DFT coefficient array over 0 <= k <= N-1
% xn = N-point finite-duration sequence

    N = length(xn);

    n = 0:N-1;
    k = 0:N-1;

    W_N = exp(-1j*2*pi/N*(n.'*k));

    Xk = xn * W_N;
end
```

---

### 6.2 `idft.m`

```matlab
function [xn] = idft(Xk)
% Compute IDFT
% xn = Reconstructed N-point sequence
% Xk = DFT coefficient array

    N = length(Xk);

    k = 0:N-1;
    n = 0:N-1;

    W_N_inv = exp(1j*2*pi/N*(k.'*n));

    xn = (1/N) * Xk * W_N_inv;
end
```

---

### 6.3 `idfs.m`

```matlab
function [xn, n] = idfs(Xk)
% Compute IDFS
% xn = Periodic sequence reconstructed from Xk
% Xk = DFS coefficient array
% N  = Period of sequence

    N = length(Xk);

    n = -2*N:2*N-1;
    k = 0:N-1;

    W_N = exp(-1j*2*pi/N);
    nk = k.'*n;

    W_N_nk = W_N.^(-nk);

    xn = (Xk * W_N_nk) / N;
end
```

---

## 7. DFT Property: Linearity

DFT는 linear transformation이므로 다음 property가 성립한다.

x<sub>1</sub>[n] ⇔ X<sub>1</sub>[k]

x<sub>2</sub>[n] ⇔ X<sub>2</sub>[k]

일 때,

a x<sub>1</sub>[n] + b x<sub>2</sub>[n]  
⇔  
a X<sub>1</sub>[k] + b X<sub>2</sub>[k]

즉, 시간 영역에서 두 신호를 선형 결합한 뒤 DFT를 수행한 결과는 각 신호의 DFT 결과를 같은 비율로 선형 결합한 값과 같다.

---

## 8. Question 1: DFT Linearity Property 검증

Q1에서는 다음 두 개의 4-point sequence를 사용한다.

x<sub>1</sub>[n] = [1, 2, 2, 1]

x<sub>2</sub>[n] = [2, 3, 4, 1]

다음과 같이 새로운 sequence를 정의한다.

x<sub>3</sub>[n] = 3x<sub>1</sub>[n] + 2x<sub>2</sub>[n]

따라서,

x<sub>3</sub>[n] = [7, 12, 14, 5]

Q1의 목표는 다음 두 결과가 동일한지 확인하는 것이다.

X<sub>3</sub>[k] = DFT{x<sub>3</sub>[n]}

X[k] = 3X<sub>1</sub>[k] + 2X<sub>2</sub>[k]

---

### 8.1 Question 1 MATLAB 코드

```matlab
clear; clc; close all;

%% Original sequences
x1 = [1 2 2 1];
x2 = [2 3 4 1];

%% Linear combination in time domain
x3 = 3*x1 + 2*x2;

%% DFT calculation
X1 = dft(x1);
X2 = dft(x2);

% Method 1: DFT of x3[n]
X3 = dft(x3);

% Method 2: Linear combination of X1[k] and X2[k]
X = 3*X1 + 2*X2;

%% Frequency axis
N = length(x1);
k = 0:N-1;
w = 2*pi*k/N;

%% Error calculation
complex_error = abs(X3 - X);
max_error = max(complex_error);

%% Display results
disp('x1[n] = ');
disp(x1);

disp('x2[n] = ');
disp(x2);

disp('x3[n] = 3*x1[n] + 2*x2[n] = ');
disp(x3);

disp('X3[k] = DFT{x3[n]} = ');
disp(X3);

disp('X[k] = 3*X1[k] + 2*X2[k] = ');
disp(X);

disp('Absolute error |X3[k] - X[k]| = ');
disp(complex_error);

fprintf('Maximum complex error = %.12f\n', max_error);

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,2,1);
stem(w/pi, abs(X), 'filled');
grid on;
xlim([0 2]);
title('Magnitude of X[k] = 3X_1[k] + 2X_2[k]');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Magnitude');

subplot(2,2,2);
stem(w/pi, abs(X3), 'filled');
grid on;
xlim([0 2]);
title('Magnitude of X_3[k] = DFT\{x_3[n]\}');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Magnitude');

subplot(2,2,3);
stem(w/pi, angle(X), 'filled');
grid on;
xlim([0 2]);
title('Phase of X[k] = 3X_1[k] + 2X_2[k]');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Phase');

subplot(2,2,4);
stem(w/pi, angle(X3), 'filled');
grid on;
xlim([0 2]);
title('Phase of X_3[k] = DFT\{x_3[n]\}');
xlabel('Normalized Frequency \omega / \pi');
ylabel('Phase');
```

---

### 8.2 Question 1 계산 결과

시간 영역의 선형 결합 결과는 다음과 같다.

```matlab
x3[n] = [7 12 14 5]
```

각 sequence의 DFT 결과는 다음과 같다.

```matlab
X1[k] = [6, -1-j, 0, -1+j]

X2[k] = [10, -2-2j, 2, -2+2j]
```

따라서 주파수 영역에서 선형 결합을 수행하면 다음과 같다.

```matlab
X[k] = 3X1[k] + 2X2[k]
     = [38, -7-7j, 4, -7+7j]
```

한편 `x3[n]`의 DFT를 직접 계산하면 다음 결과를 얻는다.

```matlab
X3[k] = DFT{x3[n]}
      = [38, -7-7j, 4, -7+7j]
```

즉,

```matlab
X3[k] = X[k]
```

가 성립한다.

---

### 8.3 Magnitude 및 Phase 결과

| k | X<sub>3</sub>[k] | Magnitude | Phase (rad) |
| :--- | :--- | :--- | :--- |
| 0 | 38 | 38.0000 | 0 |
| 1 | -7 - 7j | 9.8995 | -2.3562 |
| 2 | 4 | 4.0000 | 0 |
| 3 | -7 + 7j | 9.8995 | 2.3562 |

#### Question 1 결과 그래프

<img width="1605" height="868" alt="lab10_Q1_capture" src="https://github.com/user-attachments/assets/b49eb901-ad70-4c14-8674-cc0134dca93a" />

두 방식으로 구한 결과의 magnitude와 phase graph가 동일하게 나타난다.

또한 MATLAB에서 출력된 maximum error는 floating-point calculation 범위에서 0에 가까운 값을 가진다.

```matlab
Maximum complex error ≈ 0
```

따라서 다음 DFT linearity property가 성립함을 확인할 수 있다.

```text
DFT{3x1[n] + 2x2[n]} = 3DFT{x1[n]} + 2DFT{x2[n]}
```

---

## 9. Question 1 결과 분석

Q1에서는 두 개의 4-point sequence `x1[n]`, `x2[n]`를 이용하여 DFT linearity property를 확인하였다.

먼저 시간 영역에서 다음과 같이 선형 결합한 sequence를 생성하였다.

```text
x3[n] = 3x1[n] + 2x2[n]
```

이후 `x3[n]`의 DFT인 `X3[k]`를 직접 계산하였다.

동시에 `x1[n]`, `x2[n]`의 DFT인 `X1[k]`, `X2[k]`를 각각 계산하고, 주파수 영역에서 다음 선형 결합을 수행하였다.

```text
X[k] = 3X1[k] + 2X2[k]
```

비교 결과 `X3[k]`와 `X[k]`는 모든 frequency index에서 동일하게 나타났다.

Magnitude graph와 phase graph 역시 서로 완전히 겹쳤으며, numerical error도 0에 가까운 값으로 나타났다.

이를 통해 DFT가 linear transformation이라는 사실을 MATLAB 결과로 검증할 수 있었다.

---

## 10. 실습 내용 정리

이번 실습에서는 DFT Part 2를 통해 DTFT sampling, IDFT reconstruction, IDFS periodic extension, DFT linearity property를 MATLAB으로 확인하였다.

먼저 `[1, 2, 2, 1]` sequence에 대해 4-point DFT와 16-point DFT를 비교하였다.  
두 DFT는 동일한 DTFT를 서로 다른 개수의 frequency point에서 sampling한 결과이다.  
16-point DFT에서는 frequency grid가 더 촘촘해지므로 spectrum의 형태를 더욱 세밀하게 관찰할 수 있었다.  
다만 zero-padding은 새로운 정보를 추가하는 것이 아니라 DTFT sampling point를 증가시키는 과정임을 확인하였다.

Practice 1에서는 `x[n]=(0.95)^n cos(πn/20)` sequence의 DTFT를 M<sub>1</sub> = 100과 M<sub>2</sub> = 40개의 frequency point에서 sampling하였다.

M<sub>1</sub> = 100인 경우에는 frequency sample 수가 원래 sequence 길이 64보다 크므로 IDFT 이후 원래 sequence가 충분히 보존되었다.

반면 M<sub>2</sub> = 40인 경우에는 frequency sample 수가 원래 sequence 길이보다 작기 때문에 time-domain aliasing이 발생하였다.  
이 경우 원래 sequence의 일부 sample이 서로 겹쳐져 복원 결과가 달라졌다.

또한 IDFS를 적용하여 복원된 sequence가 주기적으로 반복되는 것을 확인하였다.  
M<sub>1</sub> = 100인 sequence는 주기 100을 가지며, M<sub>2</sub> = 40인 sequence는 주기 40을 가진다.

마지막으로 Question 1에서는 다음 DFT linearity property를 확인하였다.

```text
DFT{3x1[n] + 2x2[n]} = 3DFT{x1[n]} + 2DFT{x2[n]}
```

직접 계산한 `X3[k]`와 주파수 영역에서 선형 결합한 `X[k]`의 magnitude 및 phase graph가 동일하게 나타났다.

결과적으로 이번 실습을 통해 DFT가 DTFT의 discrete frequency sampling 결과라는 점, frequency sample 수에 따라 time-domain aliasing이 발생할 수 있다는 점, IDFS를 통해 periodic sequence를 복원할 수 있다는 점, 그리고 DFT가 linear transformation이라는 점을 종합적으로 이해할 수 있었다.

---
