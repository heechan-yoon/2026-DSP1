# [DSP 실습 02] MATLAB 기초: 행렬 생성 및 연산 규칙 이해

## 1. 실습 목적
- MATLAB의 기본 데이터 구조인 행렬(Matrix)의 생성 및 차원 제어 방법을 익힌다.
- `size`, `sum` 등 기초 함수를 활용하여 행렬의 속성을 추출한다.
- **행렬 곱(Matrix Multiplication)**과 **원소별 곱(Element-wise)**의 수학적 차이와 MATLAB 구현법을 학습한다.

---

## 2. 실습 예제 및 통합 코드 (실행 결과 포함)

MATLAB에서 수행한 전체 코드와 명령 창(Command Window)의 출력 결과입니다.

```matlab
%% 1. 배열 B 생성 및 크기/합계 추출
B = [10 15 20 ; 3 5 7 ; 4 6 8 ; 5 8 9];
[r_B, c_B] = size(B);
b = sum(B, 'all');

fprintf('B의 행 개수(r_B): %d, 열 개수(c_B): %d\n', r_B, c_B);
% 출력: B의 행 개수(r_B): 4, 열 개수(c_B): 3

fprintf('B의 모든 원소 합(b): %d\n', b);
% 출력: B의 모든 원소 합(b): 100

%% 2. 행렬 연산 비교 (Matrix Mult vs Element-wise)
C = [1 2 3];

% [테스트] 행렬 곱셈(*) - 차원 불일치 에러 발생 확인
try
    Result_Matrix_Mult = B * C;
catch ME
    fprintf('행렬 곱셈 결과: %s\n', ME.message);
end
% 출력: 행렬 곱셈 결과: 에러 발생! (행렬 곱셈의 차원이 잘못되었습니다...)

% [핵심] 원소별 곱셈(.*) - Broadcasting 활용
B_dot_C = B .* C; 
disp('B .* C 결과 (원소별 곱셈):');
disp(B_dot_C);
% 출력:
%    10    30    60
%     3    10    21
%     4    12    24
%     5    16    27
