# [DSP 실습 04] MATLAB 함수, 파일 저장/불러오기, 컨볼루션 이해

## 1. 실습 목적
- MATLAB에서 **함수 파일(Function File)**을 작성하고 호출하는 방법을 익힌다.
- `save`, `load` 명령어를 사용하여 데이터를 저장하고 불러오는 방법을 학습한다.
- `impseq`, `stepseq` 함수를 직접 작성하여 **unit impulse sequence**와 **unit step sequence**를 생성하는 방법을 이해한다.
- `conv()` 함수를 사용하여 두 이산 신호의 **컨볼루션(Convolution)** 결과를 구하고 시각화하는 방법을 익힌다.
- `census.mat` 데이터를 불러와 그래프를 그리고, `subplot`, `stem`, `sinc` 등을 이용하여 여러 이산 신호를 한 figure에 표현하는 방법을 학습한다.

---

## 2. 실습 예제 및 결과

### Q1. 함수 파일 작성 및 호출

Q1에서는 MATLAB에서 함수 파일을 작성하는 방법을 실습하였다.  
실습 자료에서는 `function` 키워드로 시작하는 파일을 작성하고, 이를 `sample_signal_1.m`이라는 이름으로 저장한 뒤 호출하는 예제를 제시하였다. 이 함수는 연속시간 코사인 신호와 이산시간 샘플 신호를 함께 그려 주는 역할을 한다.

```matlab
function sample_signal_1(ti,tf,dt,fs,A,f0,rs,cs,r)
    t = ti:dt:tf;
    xc = A*cos(2*pi*f0*t);

    n = ti:1/fs:tf;
    xd = A*cos(2*pi*f0*n);

    subplot(rs,cs,r);
    plot(t,xc,':r');
    axis([ti tf -(1.2*A) 1.2*A]);
    hold on;
    stem(n,xd);
end
```

이 함수를 아래와 같이 호출하면 샘플링 주파수에 따라 서로 다른 이산 신호 표현을 비교할 수 있다.

```matlab
close all;
clear all;
clc;

Ts = [0.05 0.1 0.2];
fs = 1./Ts;

sample_signal_1(-1,3,0.01,fs(1),1,1,3,1,1);
sample_signal_1(-1,3,0.01,fs(2),1,1,3,1,2);
sample_signal_1(-1,3,0.01,fs(3),1,1,3,1,3);
```
Q1 결과에서는 같은 연속 신호를 서로 다른 샘플링 주파수로 샘플링했을 때, 이산 샘플의 간격이 달라지는 것을 확인할 수 있다.  
샘플링 주파수가 높을수록 원래의 연속 신호를 더 촘촘하게 표현함을 알 수 있다.

---

### Q2. 파일 저장과 불러오기: `save`, `load`

Q2에서는 MATLAB의 `save`와 `load` 명령어를 사용하여 데이터를 파일로 저장하고 다시 불러오는 방법을 실습하였다.  
먼저 임의의 벡터와 행렬을 생성한 뒤 `save` 명령어를 사용해 `.mat` 파일로 저장하고, 이후 `load` 명령어를 사용해 다시 workspace로 불러오는 과정을 수행하였다.

```matlab
% 데이터 생성
p = rand(1,10);
q = ones(10);

% 파일 저장
save pqfile p q

% 파일 불러오기
load pqfile p q

% 변수 확인
p
q
```
Q2 결과를 통해 MATLAB에서는 변수 데이터를 파일에 저장한 뒤, 필요할 때 다시 불러와 재사용할 수 있음을 확인하였다.  
이 기능은 실험 데이터를 보관하거나 반복 분석할 때 매우 유용하다.

---

### Q3. `impseq`와 `stepseq` 함수 작성

이번 실습에서는 unit sample sequence와 unit step sequence를 각각 함수 파일로 작성하였다.  
`impseq`는 특정 시점 `n0`에서만 값이 1이고 나머지는 0인 시퀀스를 생성하며, `stepseq`는 `n0` 이상에서 1, 그 이전에서는 0인 시퀀스를 생성한다.

#### `impseq.m`
```matlab
function [x,n] = impseq(n0,lb,ub)
% Generates x(n) = delta(n-n0)
% lb <= n <= ub

n = lb:ub;
x = (n-n0) == 0;
end
```

#### `stepseq.m`
```matlab
function [x,n] = stepseq(n0,lb,ub)
% Generates x(n) = u(n-n0)
% lb <= n <= ub

n = lb:ub;
x = (n-n0) >= 0;
end
```

이 두 함수는 이후 practice question과 제출 문제에서도 직접 사용되므로, 이산 신호를 다루는 기본적인 도구라고 볼 수 있다.

---

## 3. 실습 문제 및 결과

### Practice Question 1. Impulse & Step

Practice Question 1에서는 두 개의 이산 시퀀스를 주어진 범위에서 `stem` 함수로 그리는 실습을 수행하였다.  
첫 번째 문제는 impulse sequence의 조합이고, 두 번째 문제는 step sequence를 이용해 구간별로 정의된 신호이다.

\[
x(n)=2\delta(n+2)-\delta(n-4), \quad -5 \le n \le 5
\]

\[
x(n)=n[u(n)-u(n-10)] + 10e^{-0.3(n-10)}[u(n-10)-u(n-20)], \quad 0 \le n \le 20
\]

```matlab
clc; clear; close all;

%% Problem (1)
n1 = -5:5;
x1 = 2*(n1 == -2) - (n1 == 4);

figure;

subplot(1,2,1);
stem(n1, x1, 'filled', 'LineWidth', 1.2);
grid on;
xlabel('n');
ylabel('x(n)');
title('Problem (1)');
xlim([-5 5]);

%% Problem (2)
n2 = 0:20;
part1 = n2 .* ((n2 >= 0) & (n2 < 10));
part2 = 10 * exp(-0.3*(n2 - 10)) .* ((n2 >= 10) & (n2 < 20));
x2 = part1 + part2;

subplot(1,2,2);
stem(n2, x2, 'filled', 'LineWidth', 1.2);
grid on;
xlabel('n');
ylabel('x(n)');
title('Problem (2)');
xlim([0 20]);
```

#### Problem 1 결과 그래프
<img width="506" height="677" alt="그림1" src="https://github.com/user-attachments/assets/13742c98-621a-4471-8dca-8086f2cd78b7" />

#### Problem 2 결과 그래프
<img width="492" height="670" alt="그림2" src="https://github.com/user-attachments/assets/beda97d5-03c9-4272-a185-f9e4d640d0e6" />

첫 번째 문제에서는 \(n=-2\)에서 값이 2, \(n=4\)에서 값이 -1이고 나머지는 모두 0인 impulse sequence를 확인할 수 있다.  
두 번째 문제에서는 \(0 \le n \le 9\) 구간에서는 선형적으로 증가하고, \(10 \le n \le 19\) 구간에서는 지수적으로 감소하는 형태가 나타난다.

---

### Practice Question 2. Convolution

Practice Question 2에서는 두 개의 시퀀스 `u`, `v`를 정의하고, `conv()` 함수를 사용하여 컨볼루션 결과 `w`를 구하는 실습을 수행하였다.  
이후 `subplot`을 사용하여 `u`, `v`, `w`를 각각 한 figure 안에 나타내었다.

```matlab
clc; clear; close all;

u = [1 0 2 3 1 1 1];
v = [2 1 2 1 3];

w = conv(u, v);

figure;

subplot(3,1,1);
stem(u, 'filled');
axis([1 12 0 5]);
grid on;
title('u');
xlabel('n');
ylabel('u[n]');

subplot(3,1,2);
stem(v, 'filled');
axis([1 12 0 10]);
grid on;
title('v');
xlabel('n');
ylabel('v[n]');

subplot(3,1,3);
stem(w, 'filled');
axis([1 12 0 20]);
grid on;
title('w = conv(u,v)');
xlabel('n');
ylabel('w[n]');
```

컨볼루션 결과는 다음과 같다.

```matlab
w = [2 1 6 9 12 11 14 13 6 4 3];
```

#### Practice Question 2 결과 그래프
<img width="1016" height="666" alt="그림3" src="https://github.com/user-attachments/assets/7b133b4b-e118-4e7f-84b0-9c7f607f33f4" />

이 결과를 통해 두 유한 길이 시퀀스의 컨볼루션 결과 길이는 `7 + 5 - 1 = 11`이 됨을 확인할 수 있었다.  
또한 `conv()` 함수가 두 시퀀스의 겹침 정도에 따라 출력값을 계산한다는 점을 시각적으로 이해할 수 있었다.

---

### Q4. `census.mat` 불러오기 및 plot

이번 문제에서는 MATLAB의 내장 데이터셋 `census.mat`를 불러와 그래프를 그리는 실습을 수행하였다.  
`cdate`는 연도 데이터를, `pop`은 해당 연도의 미국 인구 데이터를 나타낸다. 그래프는 marker only 형식으로 표현하고, 제목과 축 라벨을 추가하였다.

```matlab
clc; clear; close all;

load census.mat

figure;
plot(cdate, pop, 'o', ...
    'LineStyle', 'none', ...
    'MarkerSize', 6, ...
    'LineWidth', 1.2);

grid on;
title('US Census 1790-1990');
xlabel('Year');
ylabel('Population (Millions)');

xlim([1780 2000]);
ylim([0 260]);
```

#### Q4 결과 그래프
<img width="1015" height="675" alt="그림4" src="https://github.com/user-attachments/assets/b4e5502b-f80a-4303-9b56-b170fef44ed7" />

이 그래프에서는 시간이 지남에 따라 미국 인구가 꾸준히 증가하는 경향을 확인할 수 있다.  
또한 선을 그리지 않고 marker만 표시하여 각 데이터가 개별 관측값이라는 점을 더 명확히 나타낼 수 있었다.

---

### Q5. Function + Plot

마지막 문제에서는 \(-10 \le n \le 10\) 범위에서 하나의 figure 안에 네 개의 subplot을 만들고, impulse sequence, step sequence, rectangle sequence, sinc function을 각각 `stem`으로 표현하였다.  
이를 통해 여러 기본 이산 신호의 형태를 한 화면에서 비교할 수 있었다.

```matlab
clc; clear; close all;

n = -10:10;

[x1, ~] = impseq(0, -10, 10);
[x2, ~] = stepseq(-5, -10, 10);
x3 = double(abs(n) <= 4);
x4 = sinc(n/3);

figure;

subplot(2,2,1);
stem(n, x1, 'o', 'LineWidth', 1.2);
grid on;
title('Unit sample sequence \delta[n]');
xlabel('n');
ylabel('x[n]');
axis([-10 10 -0.5 1.5]);

subplot(2,2,2);
stem(n, x2, 'o', 'LineWidth', 1.2);
grid on;
title('Unit step sequence u[n+5]');
xlabel('n');
ylabel('x[n]');
axis([-10 10 -0.5 1.5]);

subplot(2,2,3);
stem(n, x3, 'o', 'LineWidth', 1.2);
grid on;
title('Rectangle sequence rect[n/8]');
xlabel('n');
ylabel('x[n]');
axis([-10 10 -0.5 1.5]);

subplot(2,2,4);
stem(n, x4, 'o', 'LineWidth', 1.2);
grid on;
title('Sinc function sinc(n/3)');
xlabel('n');
ylabel('x[n]');
axis([-10 10 -0.5 1.5]);
```

#### Q5 결과 그래프
<img width="1028" height="682" alt="그림5" src="https://github.com/user-attachments/assets/a4bf6589-95fc-4f84-85d5-70f28c359beb" />

이 결과를 통해 impulse sequence는 한 시점에서만 1의 값을 가지고, step sequence는 특정 시점 이후 1을 유지함을 확인할 수 있었다.  
또한 rectangle sequence는 제한된 구간에서만 1의 값을 가지며, sinc 함수는 중심에서 최대값을 갖고 양옆으로 진동하면서 점차 감소하는 형태를 보였다.

---

## 4. 실습 내용 정리

이번 실습에서는 MATLAB에서 함수 파일을 작성하고 이를 호출하는 방법을 학습하였다. 또한 `save`와 `load` 명령어를 사용하여 데이터를 파일에 저장하고 다시 불러오는 과정도 실습하였다.

이후 `impseq`와 `stepseq` 함수를 직접 작성하여 이산 신호를 생성하는 방법을 익혔고, 이를 바탕으로 impulse sequence와 step sequence를 포함한 여러 시퀀스를 `stem`으로 시각화하였다.

또한 `conv()` 함수를 사용하여 두 이산 신호의 컨볼루션 결과를 구하고, 입력 시퀀스와 출력 시퀀스를 `subplot`으로 비교하였다. 마지막으로 `census.mat` 데이터를 이용한 그래프 작성과, impulse·step·rectangle·sinc 함수를 한 figure에 정리하는 과제를 수행하면서 MATLAB의 기본적인 데이터 처리와 시각화 기능을 종합적으로 연습할 수 있었다.
