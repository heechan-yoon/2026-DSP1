# 1주차 DSP 실습: MATLAB 기초 및 삼각함수 시각화

## 1. 실습 목적
- MATLAB의 기본 명령어(`clear`, `clc`, `close all`) 숙달
- `sind`, `cosd` 함수를 이용한 삼각함수 데이터 생성
- `subplot`과 `hold on`을 이용한 데이터 시각화 방법 비교 및 습득

## 2. 주요 코드 정리

### 📝 예제 1: subplot을 이용한 그래프 분할 출력
두 개의 그래프를 상하로 나누어 각각 출력하는 방식입니다.

```matlab
clear;
clc;
close all;

theta = 0 : 10 : 720; % 0도부터 720도까지 10도 간격 설정
x = sind(theta);
y = cosd(theta);

% 첫 번째 행에 사인 그래프 출력
subplot(2,1,1);
plot(theta, x, 'ro', 'LineWidth', 1); % 'ro' -> 빨간색 원형 마커
xlabel('theta');
ylabel('Amplitude');
title('sin(red)&cos(blue)');

% 두 번째 행에 코사인 그래프 출력
subplot(2,1,2);
plot(theta, y, 'b-', 'LineWidth', 1); % 'b-' -> 파란색 실선
xlabel('theta');
ylabel('Amplitude');
title('sin(red)&cos(blue)');
