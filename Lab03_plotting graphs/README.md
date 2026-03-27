# [DSP 실습 03] MATLAB 그래프 그리기: `plot`, `hold on`, `subplot` 이해

## 1. 실습 목적
- MATLAB의 `plot()` 함수를 이용하여 연속 신호 형태의 그래프를 그리는 방법을 익힌다.
- `hold on` 명령어를 사용하여 **하나의 그래프 창에 여러 곡선(Multiple Curves)**을 동시에 표시하는 방법을 학습한다.
- `subplot()` 함수를 이용하여 **하나의 figure 창을 여러 패널(Multiple Panels)**로 나누어 그래프를 배치하는 방법을 이해한다.

---

## 2. 실습 예제 및 결과

### Q1. 하나의 그래프 창에 두 곡선 그리기

Q1에서는 `hold on` 명령어를 사용하여 하나의 좌표축 위에 두 개의 곡선을 동시에 표시하였다.  
`x`의 범위는 0부터 5까지 0.5 간격으로 설정하였고, 두 개의 함수 `y1`, `y2`를 정의하여 같은 figure 창에 함께 나타내었다.

```matlab
%% Q1 - Draw Multiple Curves
% x 범위를 0부터 5까지 0.5 간격으로 생성
x = 0:0.5:5;

% 두 개의 곡선 정의
y1 = 2*x.^2 + 3*x - 5;
y2 = 1.2*x.^2 + 4*x - 3;

% figure 생성
figure;

% 첫 번째 곡선
plot(x, y1, '-or', 'LineWidth', 1.5, 'MarkerFaceColor', 'g');
hold on;

% 두 번째 곡선
plot(x, y2, 'b--o', 'LineWidth', 1.5, 'MarkerFaceColor', 'c');

% 그래프 정보 추가
xlabel('X');
ylabel('Y');
title('Q1 - Multiple Curves in One Plot');
legend('Curve 1', 'Curve 2', 'Location', 'NorthWest');
grid on;
hold off;
```

#### Q1 결과 그래프
<img width="998" height="671" alt="lab03_Q1_capture" src="https://github.com/user-attachments/assets/5aa640c7-1e80-4aa0-a91b-7d68a7960e3e" />

Q1 결과에서는 두 개의 곡선이 하나의 그래프 위에 함께 표시되는 것을 확인할 수 있다.  
이때 `hold on`을 사용하지 않으면 두 번째 `plot()` 명령이 첫 번째 그래프를 덮어쓰게 되므로, 여러 곡선을 동시에 표현할 때 반드시 필요한 명령어임을 알 수 있다.

---

### Q2. `subplot`을 이용한 multiple panels 구성

Q2에서는 `subplot(1,2,1)`과 `subplot(1,2,2)`를 사용하여 하나의 figure 창을 1행 2열 구조로 나누고, 각 패널에 서로 다른 곡선을 표시하였다.  
이를 통해 여러 그래프를 한 화면에서 비교하는 방법을 확인할 수 있다.

```matlab
%% Q2 - Plot with Multiple Panels
% x 범위를 0부터 5까지 0.5 간격으로 생성
x = 0:0.5:5;

% 두 개의 곡선 정의
y1 = 2*x.^2 + 3*x - 5;
y2 = 1.2*x.^2 + 4*x - 3;

% figure 생성
figure;

% 첫 번째 패널
subplot(1,2,1);
plot(x, y1, '-or', 'LineWidth', 1.5, 'MarkerFaceColor', 'g');
xlabel('X');
ylabel('Y');
title('Curve 1');
legend('Curve 1', 'Location', 'NorthWest');
grid on;

% 두 번째 패널
subplot(1,2,2);
plot(x, y2, 'b--o', 'LineWidth', 1.5, 'MarkerFaceColor', 'c');
xlabel('X');
ylabel('Y');
title('Curve 2');
legend('Curve 2', 'Location', 'NorthWest');
grid on;
```

#### Q2 결과 그래프
<img width="1036" height="677" alt="lab03_Q2_capture" src="https://github.com/user-attachments/assets/b634eaf7-1140-45ef-82a3-6bca69ed8575" />

Q2 결과에서는 하나의 figure 창이 두 개의 패널로 나뉘고, 왼쪽에는 첫 번째 곡선, 오른쪽에는 두 번째 곡선이 각각 표시된다.  
이 방식은 여러 데이터를 개별적으로 비교하거나 시각적으로 정리할 때 매우 유용하다.

---

## 3. 실습 내용 정리

이번 실습에서는 MATLAB의 기본적인 그래프 출력 방식과 함께, 하나의 창에 여러 곡선을 겹쳐 그리는 방법과 여러 패널로 나누어 표현하는 방법을 학습하였다.

Q1에서는 `hold on`을 사용하여 하나의 좌표축 위에 두 개의 곡선을 동시에 표시하였다. 이를 통해 여러 함수의 형태를 직접 비교할 수 있었다.

Q2에서는 `subplot()`을 이용하여 하나의 figure를 여러 영역으로 나누고 각 영역에 개별 그래프를 배치하였다. 이를 통해 여러 그래프를 한 화면에서 효율적으로 비교할 수 있음을 확인하였다.
