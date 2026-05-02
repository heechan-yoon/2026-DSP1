# [DSP 실습 05] Z-transform, Inverse Z-transform, 주파수 응답 이해

## 1. 실습 목적
- 이산시간 신호와 시스템을 해석하기 위한 **Z-transform**의 개념을 이해한다.
- 유리함수 형태의 H(z)를 z<sup>-1</sup>에 대한 다항식 형태로 정리하는 방법을 학습한다.
- `residue`, `roots`, `zplane` 함수를 사용하여 부분분수 전개, zero/pole 계산, pole-zero plot을 수행한다.
- 복소평면 위에서 H(z)의 크기, 실수부, 허수부를 3D 그래프로 시각화한다.
- `freqz()` 함수를 사용하여 시스템의 주파수 응답을 구하고, magnitude response와 phase response를 해석한다.
- 주파수 응답 그래프를 바탕으로 시스템이 low-pass, high-pass, band-pass 중 어떤 필터 특성을 갖는지 판단한다.

---

## 2. 실습 예제 및 결과

### Q1. Z-transform과 Partial Fraction Expansion

이번 실습에서는 Z-transform의 정의와 inverse Z-transform의 개념을 학습하였다.  
Z-transform은 이산시간 신호 x[n]을 복소 변수 z에 대한 함수 X(z)로 변환하는 방법이다.  
이를 통해 시간 영역의 이산 신호를 z-domain에서 해석할 수 있으며, 시스템의 pole, zero, ROC를 이용하여 시스템 특성을 분석할 수 있다.

X(z) = Σ x[n]z<sup>-n</sup>

또한 유리함수 형태의 H(z)는 부분분수 전개를 통해 더 단순한 항들의 합으로 표현할 수 있다.  
MATLAB에서는 `residuez` 또는 `residue` 함수를 사용하여 residue, pole, direct term을 구할 수 있다.

```matlab
num = [1 1 -2];
den = [1 -2 3/4];

[R, p, C] = residuez(num, den)
```

이 함수는 전달함수를 각 pole에 대한 부분분수 형태로 분해해 주며, inverse Z-transform을 계산할 때 유용하게 사용할 수 있다.

---

### Q2. `roots`, `poly`, `zplane` 함수

Z-transform으로 표현된 시스템은 zero와 pole을 통해 주파수 특성 및 안정성을 분석할 수 있다.  
MATLAB의 `roots()` 함수는 다항식 계수로부터 근을 계산하고, `poly()` 함수는 근으로부터 다시 다항식 계수를 구할 수 있다.

```matlab
num = [1 1 -2];
den = [1 -2 3/4];

zeros = roots(num)
poles = roots(den)

num_c = poly(zeros)
den_c = poly(poles)
```

또한 `zplane()` 함수를 사용하면 zero와 pole의 위치를 복소평면 위에 시각화할 수 있다.

```matlab
zplane(num, den)
grid on;
```

Pole-zero plot에서는 일반적으로 zero가 원형 표시, pole이 x 표시로 나타난다.  
이를 통해 시스템의 주파수 응답이 어느 주파수 성분을 통과시키거나 감쇠시키는지 직관적으로 확인할 수 있다.

---

### Q3. `freqz()`를 이용한 Frequency Response

`freqz()` 함수는 전달함수의 numerator, denominator 계수를 입력받아 시스템의 주파수 응답을 계산한다.  
즉, z = e<sup>jΩ</sup>를 대입했을 때의 H(e<sup>jΩ</sup>)를 구하는 함수이다.

```matlab
[H, w] = freqz(num, den, N);
```

여기서 `H`는 복소 주파수 응답이고, `w`는 주파수 벡터이다.  
`abs(H)`를 이용하면 magnitude response를 구할 수 있고, `angle(H)`를 이용하면 phase response를 구할 수 있다.

```matlab
magH = abs(H);
angH = angle(H);

subplot(2,1,1);
plot(w/pi, magH);
grid on;
xlabel('frequency in PI unit');
ylabel('Magnitude');
title('Magnitude Response');

subplot(2,1,2);
plot(w/pi, angH);
grid on;
xlabel('frequency in PI unit');
ylabel('Phase in PI units');
title('Phase Response');
```

이를 통해 시스템이 특정 주파수 영역에서 신호를 증폭시키는지, 감쇠시키는지 확인할 수 있다.

---

## 3. 실습 문제 및 결과

### Practice Question. Z-transform, Pole-Zero Plot, 3D Graph

Practice Question에서는 다음 형태의 시스템을 다루었다.

H(z) = 2 - 9 / (1 - 0.5z<sup>-1</sup>) + 8 / (1 - z<sup>-1</sup>)

이를 q = z<sup>-1</sup>로 두면 다음과 같이 정리할 수 있다.

H(q) = 2 + 18 / (q - 2) - 8 / (q - 1)

이 식에 대해 `residue`를 사용하여 z<sup>-1</sup>에 대한 전달함수 형태로 정리하고, `roots`를 이용해 zero와 pole을 계산하였다.  
이후 `zplane`으로 pole-zero plot을 그리고, 복소평면에서 H(z)의 크기, 실수부, 허수부를 각각 3D mesh plot으로 나타내었다.

```matlab
clc; clear; close all;

%% Q1-a) Rearrange H(z) using residue
% Let q = z^(-1)
% H(q) = 2 + 18/(q-2) - 8/(q-1)

r = [18 -8];     % residues
p = [2 1];       % poles in q-plane
k = 2;           % direct term

% residue returns numerator/denominator in descending powers of q
[b_desc, a_desc] = residue(r, p, k);

disp('--- Result from residue (descending powers of q = z^{-1}) ---');
disp('Numerator coefficients (descending in q):');
disp(b_desc);
disp('Denominator coefficients (descending in q):');
disp(a_desc);

% Convert to the form:
% (b0 + b1 z^-1 + b2 z^-2) / (a0 + a1 z^-1 + a2 z^-2)
b = fliplr(b_desc);
a = fliplr(a_desc);

% Normalize so that a0 = 1
b = b / a(1);
a = a / a(1);

disp('--- Rearranged H(z) in z^{-1} form ---');
disp('b = ');
disp(b);
disp('a = ');
disp(a);

fprintf('\nH(z) = (%.4f + %.4f z^-1 + %.4f z^-2) / (%.4f + %.4f z^-1 + %.4f z^-2)\n', ...
    b(1), b(2), b(3), a(1), a(2), a(3));

%% Q1-b) Calculate zeros and poles using roots
z_zero = roots(b);
z_pole = roots(a);

disp(' ');
disp('--- Zeros of H(z) ---');
disp(z_zero);

disp('--- Poles of H(z) ---');
disp(z_pole);

%% Q1-c) Pole-zero plot using zplane
figure;
zplane(b, a);
grid on;
xlabel('Real part');
ylabel('Imaginary part');
title('Pole-Zero Plot of H(z)');

%% Q1-d) 3D graph of H(z)
x = linspace(-2, 2, 51);
y = linspace(-2, 2, 51);
[X, Y] = meshgrid(x, y);

% Build complex plane: z = x + jy
Z = X + 1j*Y;

% Use positive-power form to avoid z=0 numerical issue:
% H(z) = (z^2 + 2z + 1) / (z^2 - 1.5z + 0.5)
Hz = (Z.^2 + 2*Z + 1) ./ (Z.^2 - 1.5*Z + 0.5);

figure;
mesh(X, Y, abs(Hz));
xlabel('Real axis');
ylabel('Imaginary axis');
zlabel('|H(z)|');
title('3D Magnitude Plot of H(z)');
grid on;

% Optional: Real and Imaginary parts
figure;
mesh(X, Y, real(Hz));
xlabel('Real axis');
ylabel('Imaginary axis');
zlabel('Re\{H(z)\}');
title('3D Plot of Real Part of H(z)');
grid on;

figure;
mesh(X, Y, imag(Hz));
xlabel('Real axis');
ylabel('Imaginary axis');
zlabel('Im\{H(z)\}');
title('3D Plot of Imaginary Part of H(z)');
grid on;
```

#### Practice Question 결과 그래프

<img width="1017" height="673" alt="그림1" src="https://github.com/user-attachments/assets/6904c1c4-e55c-403e-bf9b-62f063569441" />
<img width="1037" height="665" alt="그림2" src="https://github.com/user-attachments/assets/a6899b2c-25f7-47df-ac45-d3c47e2c6c7a" />

실행 결과, H(z)는 다음과 같이 정리된다.

H(z) = (1 + 2z<sup>-1</sup> + z<sup>-2</sup>) / (1 - 1.5z<sup>-1</sup> + 0.5z<sup>-2</sup>)

따라서 zero는 z = -1에서 중복으로 나타나고, pole은 z = 1, z = 0.5에 위치한다.  
Pole-zero plot을 통해 zero와 pole의 위치를 시각적으로 확인할 수 있었으며, 3D magnitude plot에서는 pole 근처에서 |H(z)| 값이 크게 증가하는 것을 확인할 수 있다.  
또한 real part와 imaginary part의 3D plot을 통해 복소평면에서 전달함수 값이 어떻게 변화하는지 확인할 수 있었다.

---

### Q1. Frequency Response using `freqz`

Q1에서는 다음 시스템의 주파수 응답을 구하였다.

H(z) = (1 + (1/3)z<sup>-1</sup>) / (1 - 1.85cos(π/18)z<sup>-1</sup> + 0.83z<sup>-2</sup>)

`freqz()` 함수를 사용하여 N = 512개의 점에서 0 ≤ Ω ≤ π 범위의 주파수 응답을 계산하였다.  
Magnitude response는 dB 단위로 표현하였고, phase response는 degree 단위로 변환하여 그래프로 나타내었다.

```matlab
clc; clear all; close all;

% Numerator and denominator coefficients
b = [1 1/3];
a = [1 -1.85*cos(pi/18) 0.83];

% Frequency response with N = 512
[h, w] = freqz(b, a, 512);

% Magnitude and phase
mag_h = abs(h);
mag_h_db = 20*log10(mag_h);
ang_h = angle(h);

% Plot
figure;

subplot(2,1,1);
plot(w/pi, mag_h_db, 'LineWidth', 1.5);
xlabel('frequency in \pi unit');
ylabel('Magnitude (dB)');
title('Magnitude Response');
grid on;

subplot(2,1,2);
plot(w/pi, ang_h*180/pi, 'LineWidth', 1.5);
xlabel('frequency in \pi unit');
ylabel('Phase (degrees)');
title('Phase Response');
grid on;

% Q2
% This system represents a low-pass filter.
% Reason:
% The magnitude response is largest near Omega = 0 (low frequency)
% and decreases as Omega approaches pi (high frequency).
% Therefore, low-frequency components pass more easily,
% while high-frequency components are attenuated.
```

#### Q1 결과 그래프

<img width="1028" height="678" alt="lab05_Q_capture" src="https://github.com/user-attachments/assets/97305d8a-c8bd-4d12-9ba7-5647f09bf793" />

Q1 결과 그래프에서 magnitude response는 낮은 주파수 영역, 즉 Ω = 0 근처에서 가장 큰 값을 갖고, 주파수가 π에 가까워질수록 감소하는 경향을 보인다.  
따라서 이 시스템은 저주파 성분은 상대적으로 잘 통과시키고, 고주파 성분은 감쇠시키는 특성을 가진다.  
Phase response는 주파수가 증가함에 따라 위상이 변화하는 모습을 보여 주며, 이는 시스템이 각 주파수 성분에 대해 서로 다른 위상 지연을 발생시킨다는 것을 의미한다.

---

### Q2. Filter Type Analysis

Q2에서는 Q1의 주파수 응답 그래프를 바탕으로 시스템의 필터 종류를 판단하였다.  
Magnitude response를 보면 Ω = 0 근처의 저주파 영역에서 이득이 가장 크고, Ω가 π에 가까워지는 고주파 영역에서는 magnitude가 감소한다.

따라서 Q1 시스템은 **low-pass filter**에 해당한다.

```matlab
% Q2
% This system represents a low-pass filter.
% Reason:
% The magnitude response is largest near Omega = 0 (low frequency)
% and decreases as Omega approaches pi (high frequency).
% Therefore, low-frequency components pass more easily,
% while high-frequency components are attenuated.
```

즉, 이 시스템은 저주파 성분을 통과시키고 고주파 성분을 억제하는 특성을 가진다.  
이러한 판단은 `freqz()`로 얻은 magnitude response가 주파수 증가에 따라 감소하는 형태를 보였기 때문이다.

---

## 4. 실습 내용 정리

이번 실습에서는 Z-transform과 inverse Z-transform의 기본 개념을 학습하고, MATLAB을 이용하여 전달함수의 zero와 pole을 분석하였다.  
먼저 부분분수 전개를 통해 주어진 H(z)를 z<sup>-1</sup>에 대한 다항식 비율 형태로 정리하였고, `roots()` 함수를 사용하여 zero와 pole을 계산하였다.

또한 `zplane()` 함수를 이용해 pole-zero plot을 그려 복소평면에서 시스템의 구조를 시각적으로 확인하였다.  
이 과정에서 zero는 z = -1, pole은 z = 1, z = 0.5에 위치함을 확인할 수 있었다.  
추가적으로 복소평면 z = x + jy를 구성한 뒤 H(z)의 magnitude, real part, imaginary part를 3D mesh plot으로 나타내어 전달함수가 복소평면에서 어떻게 변화하는지 확인하였다.

마지막으로 `freqz()` 함수를 사용하여 Q1 시스템의 주파수 응답을 계산하였다.  
Magnitude response와 phase response를 각각 그래프로 나타낸 결과, 낮은 주파수 영역에서 magnitude가 크고 높은 주파수 영역으로 갈수록 감소하는 특성을 보였다.  
이를 통해 해당 시스템이 low-pass filter로 동작한다는 것을 확인할 수 있었다.
