# [DSP 실습 01] MATLAB 기초: 삼각함수 생성 및 시각화 기법

## 1. 실습 목적
- MATLAB 라이브 편집기 환경에서 데이터를 생성하고 시각화하는 기초 과정을 학습한다.
- `subplot`을 활용한 다중 그래프 출력과 `hold on`을 활용한 중첩 그래프 출력의 차이점을 익힌다.

---

## 2. 실습 예제 및 코드

### 파트 1: Subplot을 이용한 그래프 분할 출력
`subplot` 함수를 사용하면 하나의 그림 창(Figure)을 행과 열로 나누어 여러 개의 독립된 그래프를 동시에 표시할 수 있습니다.

```matlab
clear;
clc;
close all;

% 각도 데이터 생성 (0도 ~ 720도)
theta = 0 : 10 : 720; 
x = sind(theta); % 사인 신호
y = cosd(theta); % 코사인 신호

% 2행 1열 구조의 1번(위) 위치에 출력
subplot(2,1,1);
plot(theta, x, 'ro', 'LineWidth', 1);
xlabel('theta');
ylabel('Amplitude');
title('sin (subplot)');

% 2행 1열 구조의 2번(아래) 위치에 출력
subplot(2,1,2);
plot(theta, y, 'b-', 'LineWidth', 1);
xlabel('theta');
ylabel('Amplitude');
title('cos (subplot)');

```
### 파트 2: Overlapping (hold on)을 이용한 그래프 중첩 출력
`hold on` 명령어를 사용하면 하나의 좌표축 위에 여러 개의 그래프를 겹쳐 그려서 신호 간의 위상 차이나 형태를 직접 비교할 수 있습니다.

```matlab
clear;
clc;
close all;

% 각도 데이터 생성 (0도 ~ 720도)
theta = 0 : 10 : 720; 
x = sind(theta); % 사인 신호
y = cosd(theta); % 코사인 신호

% 새로운 창 생성 및 중첩 출력
figure(1);

% 첫 번째 신호(Sine) 출력
plot(theta, x, 'ro', 'LineWidth', 1); 
hold on                               % 현재 그래프 축 유지

% 두 번째 신호(Cosine) 추가 출력
plot(theta, y, 'b-', 'LineWidth', 1); 

xlabel('theta');
ylabel('Amplitude');
title('Overlapping sin & cos');
grid on;
