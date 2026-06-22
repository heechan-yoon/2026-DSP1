# [DSP 실습 11] FFT 및 Downsampling 후 Spectrum 변화 이해

## 1. 실습 목적

- FFT(Fast Fourier Transform)의 기본 개념과 Cooley-Tukey algorithm의 구조를 이해한다.
- DFT를 직접 계산하는 방식과 FFT를 이용하는 방식의 연산량 차이를 비교한다.
- 입력 길이가 2의 거듭제곱인 sequence에 대해 재귀적으로 FFT를 수행하는 `my_fft.m` 함수를 작성한다.
- `my_fft()` 함수와 MATLAB 내장 함수 `fft()`의 결과를 비교하여 FFT 구현이 올바르게 동작하는지 확인한다.
- 256-point DFT를 이용하여 이산시간 신호의 magnitude spectrum을 분석한다.
- Downsampling factor `M = 2`를 적용한 뒤, spectrum peak 위치가 어떻게 변하는지 확인한다.
- Downsampling 후 frequency component가 compression 및 folding되면서 aliasing이 발생할 수 있음을 이해한다.
- Original signal과 downsampled signal의 spectrum을 normalized frequency axis에서 비교한다.

---

## 2. Lab 11: FFT

Lab 11에서는 DFT를 빠르게 계산하는 알고리즘인 FFT를 다룬다.

DFT는 다음과 같이 정의된다.

X[k] = Σ x[n]W<sub>N</sub><sup>kn</sup>

여기서,

W<sub>N</sub> = e<sup>-j2π/N</sup>

이다.

일반적인 DFT 계산은 모든 `k`와 `n`에 대해 곱셈을 수행해야 하므로 연산량이 많다.  
길이가 `N`인 sequence에 대해 DFT를 직접 계산하면 대략 `N^2`번의 complex multiplication이 필요하다.

반면 FFT는 DFT 계산을 even-indexed part와 odd-indexed part로 나누어 재귀적으로 계산한다.  
이를 통해 연산량을 크게 줄일 수 있다.

FFT의 대표적인 알고리즘이 **Cooley-Tukey algorithm**이다.

---

## 3. Cooley-Tukey FFT Algorithm

길이 `N`인 sequence `x[n]`에 대해 DFT를 계산할 때, sequence를 even sample과 odd sample로 나눌 수 있다.

```text
Even part: x[0], x[2], x[4], ...
Odd part : x[1], x[3], x[5], ...
```

이를 이용하면 DFT는 다음과 같이 나눌 수 있다.

X[k] = E[k] + W<sub>N</sub><sup>k</sup>O[k]

X[k + N/2] = E[k] - W<sub>N</sub><sup>k</sup>O[k]

여기서 `E[k]`는 even-indexed sequence의 `N/2`-point DFT이고,  
`O[k]`는 odd-indexed sequence의 `N/2`-point DFT이다.

즉, 하나의 `N`-point DFT를 두 개의 `N/2`-point DFT로 나누고, 이를 반복적으로 수행하면 전체 DFT를 빠르게 계산할 수 있다.

```text
N-point DFT
 ├─ N/2-point DFT of even-indexed samples
 ├─ N/2-point DFT of odd-indexed samples
 └─ Butterfly combination using twiddle factors
```

이때 필요한 multiplication count는 대략 다음과 같다.

N/2 log<sub>2</sub>N

따라서 FFT는 직접 DFT 계산보다 훨씬 효율적이다.

---

## 4. Practice 1: `my_fft.m` 함수 작성

Practice 1에서는 입력 길이가 2의 거듭제곱인 sequence에 대해 FFT를 계산하는 `my_fft.m` 함수를 작성한다.

### 4.1 `my_fft.m`

```matlab
function [X, count] = my_fft(x)
% Compute Fast Fourier Transform using Cooley-Tukey algorithm
% X     = FFT result
% count = number of complex multiplications for twiddle factor multiplication

    if iscolumn(x)
        x = x.';
    end

    N = length(x);

    if N == 1
        X = x;
        count = 0;
        return;
    end

    [E, count_E] = my_fft(x(1:2:end));
    [O, count_O] = my_fft(x(2:2:end));

    k = 0:(N/2 - 1);                 % index for butterfly combination
    W = exp(-1j * 2 * pi / N * k);   % twiddle factor

    T = W .* O;

    X = [E + T, E - T];

    count = count_E + count_O + N/2;
end
```

---

### 4.2 코드 동작 설명

`my_fft()` 함수는 입력 sequence를 even-indexed samples와 odd-indexed samples로 나눈다.

```matlab
E input = x(1:2:end)
O input = x(2:2:end)
```

MATLAB index는 1부터 시작하므로,

```text
x(1:2:end) → x[0], x[2], x[4], ...
x(2:2:end) → x[1], x[3], x[5], ...
```

에 해당한다.

각 부분에 대해 다시 `my_fft()`를 호출하여 recursive하게 FFT를 계산한다.

이후 twiddle factor `W`를 이용하여 even part와 odd part를 결합한다.

```matlab
T = W .* O;

X = [E + T, E - T];
```

이는 다음 식에 대응된다.

```text
X[k]       = E[k] + W_N^k O[k]
X[k+N/2]   = E[k] - W_N^k O[k]
```

---

### 4.3 Multiplication Count 확인

길이가 64인 sequence `ones(64,1)`을 생성하고, `my_fft()` 함수가 계산한 multiplication count를 확인한다.

```matlab
clear; clc; close all;

x = ones(64, 1);

[X, count] = my_fft(x);

N = length(x);

theoretical_fft_count = (N/2) * log2(N);
direct_dft_count = N^2;

fprintf('N = %d\n', N);
fprintf('Multiplication count from my_fft() = %d\n', count);
fprintf('Theoretical FFT multiplication count = %.0f\n', theoretical_fft_count);
fprintf('Direct DFT multiplication count = %d\n', direct_dft_count);
```

#### Practice 1 결과

```matlab
N = 64
Multiplication count from my_fft() = 192
Theoretical FFT multiplication count = 192
Direct DFT multiplication count = 4096
```

길이 `N = 64`인 경우 FFT의 multiplication count는 다음과 같다.

```text
N/2 log2(N) = 64/2 × log2(64)
            = 32 × 6
            = 192
```

직접 DFT를 계산하는 경우 필요한 multiplication count는 다음과 같다.

```text
N^2 = 64^2 = 4096
```

따라서 FFT는 직접 DFT 계산보다 훨씬 적은 연산량으로 동일한 DFT 결과를 얻을 수 있다.

---

## 5. Practice 2: `my_fft()`와 `fft()` 비교

Practice 2에서는 다음 sequence를 사용한다.

x[n] = cos(πn/20) + 3cos(πn/12), 0 ≤ n ≤ 511

이 신호에 대해 직접 작성한 `my_fft()` 함수와 MATLAB 내장 함수 `fft()`를 이용하여 512-point DFT를 계산하고, magnitude와 phase를 비교한다.

---

### 5.1 MATLAB 코드

```matlab
clear; clc; close all;

%% Generate sequence
n = 0:511;
x = cos(pi*n/20) + 3*cos(pi*n/12);

%% 512-point DFT using my_fft()
[X1, count] = my_fft(x);

%% 512-point DFT using MATLAB built-in fft()
X2 = fft(x);

%% Frequency index
N = length(x);
k = 0:N-1;

%% Error check
mag_error = max(abs(abs(X1) - abs(X2)));
complex_error = max(abs(X1 - X2));

fprintf('Multiplication count from my_fft() = %d\n', count);
fprintf('Maximum magnitude error = %.12f\n', mag_error);
fprintf('Maximum complex error = %.12f\n', complex_error);

%% Plot
figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(2,2,1);
plot(k, abs(X1), 'LineWidth', 1.2);
grid on;
title('Magnitude of X_1[k] using my\_fft()');
xlabel('Index k');
ylabel('|X_1[k]|');

subplot(2,2,2);
plot(k, abs(X2), 'LineWidth', 1.2);
grid on;
title('Magnitude of X_2[k] using fft()');
xlabel('Index k');
ylabel('|X_2[k]|');

subplot(2,2,3);
plot(k, angle(X1), 'LineWidth', 1.2);
grid on;
title('Phase of X_1[k] using my\_fft()');
xlabel('Index k');
ylabel('Angle of X_1[k]');

subplot(2,2,4);
plot(k, angle(X2), 'LineWidth', 1.2);
grid on;
title('Phase of X_2[k] using fft()');
xlabel('Index k');
ylabel('Angle of X_2[k]');
```

#### Practice 2 결과 그래프

<img width="1062" height="677" alt="lab11_P2_capture1" src="https://github.com/user-attachments/assets/baff5da7-55e9-47b2-a55b-6806c8ee584c" />

`my_fft()` 함수와 MATLAB 내장 함수 `fft()`를 비교한 결과, magnitude spectrum과 phase spectrum이 거의 동일하게 나타난다.

이는 직접 작성한 recursive FFT 함수가 MATLAB의 built-in FFT와 같은 DFT 결과를 계산한다는 것을 의미한다.

차이가 존재하더라도 이는 floating-point calculation 과정에서 발생하는 매우 작은 numerical error로 볼 수 있다.

---

## 6. Assignment: 256-point DFT before and after Downsampling

Assignment에서는 세 개의 signal `x1[n]`, `x2[n]`, `x3[n]`을 생성하고, 각각에 대해 256-point DFT를 수행한다.

이후 각 signal을 factor `M = 2`로 downsampling한 뒤 다시 256-point DFT를 수행하고, original signal과 downsampled signal의 magnitude spectrum을 비교한다.

---

## 7. Assignment Signal Definition

이번 assignment에서 사용하는 signal은 다음과 같다.

### 7.1 Signal 1

Normalized frequency가 0.15인 sinusoidal signal이다.

```matlab
x1[n] = sin(2π × 0.15 × n)
```

### 7.2 Signal 2

Normalized frequency가 0.1과 0.3인 두 sinusoidal signal의 합이다.

```matlab
x2[n] = sin(2π × 0.1 × n) + sin(2π × 0.3 × n)
```

### 7.3 Signal 3

`x1[n]`에 exponential decay `0.8^n`이 곱해진 signal이다.

```matlab
x3[n] = x1[n] × (0.8)^n
```

---

## 8. Assignment MATLAB 코드

```matlab
clear; clc; close all;

%% ---------------------------------------------------------
% Assignment
% 256-point DFT before and after downsampling
% ---------------------------------------------------------

N_fft = 256;      % DFT point
M = 2;            % Downsampling factor
n = 0:50;         % Original time index

%% ---------------------------------------------------------
% Original signals
% ---------------------------------------------------------

x1 = sin(2*pi*0.15*n);
x2 = sin(2*pi*0.1*n) + sin(2*pi*0.3*n);
x3 = x1 .* (0.8).^n;

%% ---------------------------------------------------------
% Downsampling by a factor of M = 2
%
% MATLAB index starts from 1.
% x(1), x(3), x(5), ... correspond to x[0], x[2], x[4], ...
% Therefore, y[m] = x[2m].
% ---------------------------------------------------------

y1 = x1(1:M:end);
y2 = x2(1:M:end);
y3 = x3(1:M:end);

%% ---------------------------------------------------------
% Zero-padding for 256-point DFT
% ---------------------------------------------------------

x1_pad = [x1 zeros(1, N_fft - length(x1))];
x2_pad = [x2 zeros(1, N_fft - length(x2))];
x3_pad = [x3 zeros(1, N_fft - length(x3))];

y1_pad = [y1 zeros(1, N_fft - length(y1))];
y2_pad = [y2 zeros(1, N_fft - length(y2))];
y3_pad = [y3 zeros(1, N_fft - length(y3))];

%% ---------------------------------------------------------
% 256-point DFT using my_fft()
% ---------------------------------------------------------

[X1, count_X1] = my_fft(x1_pad);
[X2, count_X2] = my_fft(x2_pad);
[X3, count_X3] = my_fft(x3_pad);

[Y1, count_Y1] = my_fft(y1_pad);
[Y2, count_Y2] = my_fft(y2_pad);
[Y3, count_Y3] = my_fft(y3_pad);

%% ---------------------------------------------------------
% Normalized frequency axis
% 0 <= normalized frequency <= 1 corresponds to 0 <= omega <= pi
% Use first half of DFT result.
% ---------------------------------------------------------

f = (0:N_fft/2) / (N_fft/2);

X1_mag = abs(X1(1:N_fft/2+1));
X2_mag = abs(X2(1:N_fft/2+1));
X3_mag = abs(X3(1:N_fft/2+1));

Y1_mag = abs(Y1(1:N_fft/2+1));
Y2_mag = abs(Y2(1:N_fft/2+1));
Y3_mag = abs(Y3(1:N_fft/2+1));

%% ---------------------------------------------------------
% Plot magnitude spectra
% ---------------------------------------------------------

figure('Units', 'normalized', 'OuterPosition', [0 0 1 1]);

subplot(3,2,1);
plot(f, X1_mag, 'LineWidth', 1.2);
grid on;
title('|X_1[k]|: Original x_1[n]');
xlabel('Normalized frequency');
ylabel('|X_1[k]|');

subplot(3,2,2);
plot(f, Y1_mag, 'LineWidth', 1.2);
grid on;
title('|Y_1[k]|: Downsampled y_1[n]');
xlabel('Normalized frequency');
ylabel('|Y_1[k]|');

subplot(3,2,3);
plot(f, X2_mag, 'LineWidth', 1.2);
grid on;
title('|X_2[k]|: Original x_2[n]');
xlabel('Normalized frequency');
ylabel('|X_2[k]|');

subplot(3,2,4);
plot(f, Y2_mag, 'LineWidth', 1.2);
grid on;
title('|Y_2[k]|: Downsampled y_2[n]');
xlabel('Normalized frequency');
ylabel('|Y_2[k]|');

subplot(3,2,5);
plot(f, X3_mag, 'LineWidth', 1.2);
grid on;
title('|X_3[k]|: Original x_3[n]');
xlabel('Normalized frequency');
ylabel('|X_3[k]|');

subplot(3,2,6);
plot(f, Y3_mag, 'LineWidth', 1.2);
grid on;
title('|Y_3[k]|: Downsampled y_3[n]');
xlabel('Normalized frequency');
ylabel('|Y_3[k]|');

%% ---------------------------------------------------------
% Print multiplication count
% ---------------------------------------------------------

fprintf('FFT multiplication count for 256-point DFT = %d\n', count_X1);
```

---

## 9. Assignment 결과 그래프

<img width="1002" height="667" alt="lab11_Q_capture" src="https://github.com/user-attachments/assets/4d5ef88b-2223-4583-a515-e1c67e588d4e" />

---

## 10. Assignment 결과 분석

### 10.1 Original Signal Spectrum

Original signal의 spectrum은 각각 신호에 포함된 normalized frequency component에 대응되는 peak를 가진다.

| Signal | 구성 성분 | Original spectrum 특징 |
| :--- | :--- | :--- |
| `x1[n]` | `0.15` normalized frequency sinusoid | 약 0.15와 대칭 위치 부근에 peak 발생 |
| `x2[n]` | `0.1`, `0.3` normalized frequency sinusoid 합 | 약 0.1, 0.3 및 대칭 위치 부근에 peak 발생 |
| `x3[n]` | sinusoid × exponential decay | peak가 넓게 퍼진 형태로 나타남 |

`x1[n]`은 하나의 sinusoidal component를 가지므로 spectrum에서 하나의 주요 peak pair가 나타난다.

`x2[n]`는 두 개의 sinusoidal component를 포함하므로 spectrum에서도 두 개의 주요 peak pair가 나타난다.

`x3[n]`는 sinusoidal signal에 exponential decay가 곱해진 형태이다.  
시간 영역에서 exponential window가 곱해지면 frequency 영역에서는 spectrum이 넓게 퍼지는 효과가 나타난다.  
따라서 `x3[n]`의 magnitude spectrum은 `x1[n]`보다 더 broad한 형태를 가진다.

---

### 10.2 Downsampling 후 Spectrum 변화

Downsampling factor `M = 2`를 적용하면 시간 영역에서는 sample을 2개마다 하나씩 선택하게 된다.

```text
y[m] = x[2m]
```

시간 영역에서 downsampling을 하면 frequency 영역에서는 spectrum이 compression되고 folding된다.

Downsampling 후 normalized frequency는 다음과 같이 변한다.

```text
f_new = M × f_original
```

여기서 `M = 2`이므로,

```text
f_new = 2f_original
```

이다.

따라서 original spectrum에서 normalized frequency `0.15`에 있던 component는 downsampled signal에서는 약 `0.30` 위치로 이동한다.

마찬가지로 `0.10` component는 약 `0.20`으로 이동하고, `0.30` component는 약 `0.60`으로 이동한다.

---

### 10.3 Spectral Peak 이동 원인

Downsampling 후 spectral peak가 이동하는 이유는 sample 간격이 커지면서 discrete-time frequency가 상대적으로 증가하기 때문이다.

원래 sequence에서 normalized frequency가 `f`인 sinusoid는 다음과 같이 표현된다.

```text
x[n] = sin(2πfn)
```

Downsampling factor `M = 2`를 적용하면,

```text
y[m] = x[2m]
```

이므로,

```text
y[m] = sin(2πf(2m))
     = sin(2π(2f)m)
```

따라서 downsampled signal의 normalized frequency는 `2f`가 된다.

즉, downsampling factor가 2이면 discrete-time frequency가 2배가 되는 것처럼 나타난다.

---

### 10.4 Aliasing 발생 가능성

Downsampling 후 frequency가 Nyquist range를 벗어나면 aliasing이 발생한다.

Discrete-time normalized frequency에서 `0.5`를 초과하는 성분은 folding되어 낮은 frequency 위치로 겹쳐질 수 있다.

이번 signal에서는 다음과 같은 변화가 나타난다.

| Original frequency | Downsampled frequency before folding | 해석 |
| :--- | :--- | :--- |
| 0.10 | 0.20 | aliasing 없이 이동 |
| 0.15 | 0.30 | aliasing 없이 이동 |
| 0.30 | 0.60 | Nyquist 기준을 넘어 folding 가능 |

`x2[n]`에 포함된 `0.30` component는 downsampling 후 `0.60`으로 이동한다.  
이 값은 normalized frequency 기준으로 Nyquist frequency인 `0.5`보다 크다.  
따라서 spectrum에서 folding되어 aliasing이 발생할 수 있다.

이 때문에 downsampling 후 `x2[n]`의 spectrum은 단순히 peak가 2배 위치로 이동하는 것뿐만 아니라, 일부 component가 접혀서 다른 위치에 나타날 수 있다.

---

## 11. Assignment 신호별 해석

### 11.1 `x1[n]`과 `y1[n]`

`x1[n]`은 normalized frequency `0.15`를 가지는 sinusoidal signal이다.

Downsampling 후에는 다음과 같이 frequency가 변한다.

```text
0.15 × 2 = 0.30
```

따라서 `y1[n]`의 spectrum에서는 주요 peak가 약 `0.30` 부근으로 이동한다.

---

### 11.2 `x2[n]`과 `y2[n]`

`x2[n]`은 normalized frequency `0.1`과 `0.3`을 가지는 두 sinusoidal signal의 합이다.

Downsampling 후에는 각각 다음과 같이 변한다.

```text
0.1 × 2 = 0.2
0.3 × 2 = 0.6
```

`0.2` component는 Nyquist range 안에 있으므로 그대로 나타난다.  
그러나 `0.6` component는 Nyquist range를 넘어 folding이 발생할 수 있다.

따라서 `y2[n]`의 spectrum에서는 peak 위치가 이동하며, aliasing에 의해 일부 frequency component가 접힌 위치에 나타난다.

---

### 11.3 `x3[n]`과 `y3[n]`

`x3[n]`은 `x1[n]`에 exponential decay `0.8^n`이 곱해진 signal이다.

시간 영역에서 exponential decay가 곱해지면 finite-duration windowing 효과가 발생하여 spectrum이 넓게 퍼진다.

Downsampling 후에는 중심 frequency가 약 `0.30` 부근으로 이동하지만, spectrum이 넓기 때문에 peak가 sharp하게 나타나기보다는 broad한 형태를 유지한다.

또한 downsampling으로 인해 frequency-domain folding이 발생할 수 있어 spectrum shape이 original signal과 다르게 나타난다.

---

## 12. 실습 내용 정리

이번 실습에서는 FFT의 구조와 downsampling이 magnitude spectrum에 미치는 영향을 MATLAB으로 확인하였다.

먼저 Cooley-Tukey FFT algorithm을 기반으로 `my_fft.m` 함수를 작성하였다.  
이 함수는 입력 sequence를 even-indexed samples와 odd-indexed samples로 나눈 뒤, 각각에 대해 recursive하게 FFT를 계산한다.  
이후 twiddle factor를 이용하여 두 결과를 butterfly structure로 결합한다.

Practice 1에서는 `ones(64,1)` sequence를 이용하여 `my_fft()` 함수의 multiplication count를 확인하였다.  
그 결과 `N = 64`일 때 FFT multiplication count는 `N/2 log2(N) = 192`로 나타났다.  
반면 direct DFT의 multiplication count는 `N^2 = 4096`이므로, FFT가 DFT보다 훨씬 적은 연산량으로 같은 결과를 얻을 수 있음을 확인하였다.

Practice 2에서는 `x[n] = cos(πn/20) + 3cos(πn/12)` sequence에 대해 `my_fft()`와 MATLAB built-in `fft()`를 비교하였다.  
두 방식으로 얻은 magnitude와 phase graph가 거의 동일하게 나타나 직접 구현한 FFT 함수가 정상적으로 동작함을 확인하였다.

Assignment에서는 `x1[n]`, `x2[n]`, `x3[n]`에 대해 256-point DFT를 수행하고, downsampling factor `M = 2`를 적용한 뒤 다시 256-point DFT를 계산하였다.

Downsampling은 시간 영역에서 sample을 줄이는 과정이지만, frequency 영역에서는 spectrum compression과 folding을 유발한다.  
특히 normalized frequency는 downsampling factor만큼 증가하는 것처럼 나타나므로, `M = 2`일 때 original frequency `f`는 `2f` 위치로 이동한다.

이로 인해 `x1[n]`의 0.15 성분은 downsampling 후 약 0.30 위치로 이동하였고, `x2[n]`의 0.1 성분은 0.2로 이동하였다.  
반면 `x2[n]`의 0.3 성분은 0.6으로 이동하여 Nyquist range를 넘기 때문에 aliasing이 발생할 수 있다.

결과적으로 이번 실습을 통해 FFT가 DFT 계산을 효율적으로 수행하는 방법이라는 점과, downsampling이 frequency spectrum의 peak 위치와 aliasing에 직접적인 영향을 준다는 점을 이해할 수 있었다.

---
