# [DSP 실습 03] MATLAB Plotting Graphs: 다중 곡선 및 판넬 제어

---

## 1. 실습 목적
- `plot()`과 `stem()` 함수를 이용한 연속 시간 및 이산 시간 신호 시각화 습득
- `hold on` 및 `subplot` 명령어를 이용한 그래프 배치 및 제어 방법 학습

---

## 2. 주요 명령어 요약
| 명령어 | 기능 설명 |
| :--- | :--- |
| **`plot(t, x)`** | 연속 시간(Continuous-time) 그래프 생성 |
| **`stem(k, x)`** | 이산 시간(Discrete-time) 그래프 생성 |
| **`hold on / off`** | 현재 창에 그래프를 유지하며 겹쳐 그리기 제어 |
| **`subplot(m, n, p)`** | 하나의 창을 m x n 구역으로 나누고 p번째 구역 선택 |

---

## 3. 실습 과제 상세

### **3-1. Q1: DRAW MULTIPLE CURVES (hold on)**
**[과제 내용]** 하나의 그래프 창에 `y1 = 2x^2 + 3*x - 5` (Red Solid)와 `y2 = 1.2x^2 + 4*x - 3` (Blue Dashed) 수식을 중첩하여 출력합니다. `hold on`을 사용하여 그래프가 겹쳐지도록 설정합니다.

**[실습 결과]** ![Q1 결과 이미지](./q1_plot.png)

---

### **3-2. Q2: PLOT WITH MULTIPLE PANELS (subplot)**
**[과제 내용]** `subplot(1, 2, p)` 명령어를 사용하여 창을 좌우로 분할하고, 각각의 판넬에 독립적인 그래프를 배치합니다. 왼쪽에는 `y1`, 오른쪽에는 `y2`를 배치하며 각각의 축 범위(`axis`)를 다르게 설정합니다.

**[실습 결과]** ![Q2 결과 이미지](./q2_plot.png)

---

## 4. [전체 복사] 실습 통합 실행 코드
> **아래 코드 블록을 한 번만 복사하여 MATLAB에 붙여넣으세요.** > Q1과 Q2 결과가 각각 독립된 Figure 창으로 출력됩니다.

```matlab
%% [Lab 03] Plotting Graphs 실습 통합 스크립트
clear; clc; close all;

% [0. 공통 데이터 설정]
x = 0:0.5:5;
y1 = 2*x.^2 + 3*x - 5;
y2 = 1.2*x.^2 + 4*x - 3;

%% ----------------------------------------------------
%  3-1. Q1 - DRAW MULTIPLE CURVES (hold on 활용)
% -----------------------------------------------------
figure('Name', 'Q1: Multiple Curves');

% Curve 1: 빨간색 실선, 원형 마커, 두께 1.5, 내부 초록색
plot(x, y1, '-or', 'MarkerFaceColor', 'g', 'LineWidth', 1.5); 
hold on; % ★기존 그래프 유지

% Curve 2: 파란색 점선, 원형 마커, 두께 1.5, 내부 초록색
plot(x, y2, '--ob', 'MarkerFaceColor', 'g', 'LineWidth', 1.5); 

xlabel('X'); ylabel('Y');
legend('Curve 1', 'Curve 2', 'Location', 'NorthWest');
axis([0 5 -10 60]);
title('Q1: Multiple Curves using hold on');
grid on;
hold off;

%% ----------------------------------------------------
%  3-2. Q2 - PLOT WITH MULTIPLE PANELS (subplot 활용)
% -----------------------------------------------------
figure('Name', 'Q2: Subplot Panels');

% [왼쪽 판넬] 1행 2열 중 1번째 칸: y1
subplot(1, 2, 1); 
plot(x, y1, '-or', 'MarkerFaceColor', 'g', 'LineWidth', 1.2);
xlabel('X'); ylabel('Y');
legend('Curve 1', 'Location', 'NorthWest');
axis([0 6 -10 60]);
title('Panel 1: y1 = 2x^2 + 3x - 5');

% [오른쪽 판넬] 1행 2열 중 2번째 칸: y2
subplot(1, 2, 2); 
% 검정 점선('--k')과 하늘색('c') 마커 사용
plot(x, y2, '--ok', 'MarkerFaceColor', 'c', 'LineWidth', 1.2);
xlabel('X'); ylabel('Y');
legend('Curve 2', 'Location', 'NorthWest');
axis([0 6 -10 50]);
title('Panel 2: y2 = 1.2x^2 + 4x - 3');
