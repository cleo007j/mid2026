# 문제 5번
1. 빅오 표기법($O$-notation)의 개념
 알고리즘의 시간 복잡도(Time Complexity)를 나타내는 수학적 표기법입니다.
 입력 크기($N$)가 무한히 커질 때, 연산 횟수가 증가하는 최고차항의 추세를 나타내며 상수나 계수는 무시합니다.
 C언어 프로그램 작성 시 효율적인 코드와 비효율적인 코드를 객관적으로 비교하는 기준이 됩니다
2. 주요 시간 복잡도 종류 및 속도 비교 (빠른 순서)
   $O(1)$ - 상수 시간 (Constant Time)입력 크기($N$)와 상관없이 일정한 시간이 소요됩니다.   예: 배열의 인덱스로 값에 직접 접근하기 (arr[5]).   속도: 가장 빠름

   $O(\log N)$ - 로그 시간 (Logarithmic Time)데이터가 증가할수록 처리해야 할 양이 절반씩 줄어듭니다.   예: 정렬된 배열에서의 이진 탐색(Binary Search).   속도: 매우 빠름

   $O(N)$ - 선형 시간 (Linear Time)입력 크기($N$)에 비례하여 연산 횟수가 증가합니다.   예: 단일 for 루프를 사용한 순차 탐색(Sequential Search), 배열 전체 순회.   속도: 보통
   $O(N \log N)$ - 선형 로그 시간 (Linearithmic Time)효율적인 정렬 알고리즘에서 주로 나타납니다.   예: 합병 정렬(Merge Sort), 퀵 정렬(Quick Sort의 평균 경우).   속도: 양호함
   $O(N^2)$ - 이차 시간 (Quadratic Time)입력 크기의 제곱에 비례하여 연산이 증가합니다.   예: 이중 for 루프(Nested Loop), 버블 정렬(Bubble Sort), 선택 정렬(Selection Sort).   속도: 느림 ($N$이 커지면 성능 급격히 저하)
   $O(2^N)$ - 지수 시간 (Exponential Time)입력이 증가할수록 연산량이 2배씩 폭발적으로 증가합니다.   예: 피보나치 수열을 구하는 단순 재귀 함수.   속도: 매우 느림
   $$O(1) < O(\log N) < O(N) < O(N \log N) < O(N^2) < O(2^N)$$

   <img width="1002" height="315" alt="image" src="https://github.com/user-attachments/assets/389e915a-1eb1-46dd-9299-e14ff55dab73" />
 <img width="1007" height="578" alt="image" src="https://github.com/user-attachments/assets/3f1d82dd-535b-4912-ab3a-562aa15bb8f9" />
 <img width="1069" height="581" alt="image" src="https://github.com/user-attachments/assets/cf743116-ad1c-4cae-9512-541fd9080e2b" />
<img width="987" height="428" alt="image" src="https://github.com/user-attachments/assets/d344044c-10bb-4071-9db5-538be7249246" />
<img width="1129" height="346" alt="image" src="https://github.com/user-attachments/assets/c2ee18d1-83bf-4cce-995d-f545b5660303" />

