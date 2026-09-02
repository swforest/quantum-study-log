# 7주차 - QAOA와 조합 최적화

## 주간 목표

작은 이진 최적화 문제를 QUBO와 Ising Hamiltonian으로 표현하고, QAOA parameterized circuit의 기대값을 고전 optimizer로 줄인 뒤, 최종 bitstring을 원래 문제의 해로 해석합니다.

## 일정

| 요일 | 주제 | 결과물 |
|---|---|---|
| 월 | 최적화·QUBO | 문제 정의와 brute-force 기준해 |
| 화 | Ising 변환 | QUBO↔Pauli Z 수식 검증 |
| 수 | QAOAAnsatz·비용함수 | parameterized cost circuit |
| 목 | 고전 최적화 loop | iteration 기록과 수렴 진단 |
| 금 | 로컬 종합 실습 | exact 기준과 비교한 QAOA 보고서 |

## 주간 원칙

- quantum algorithm 전에 고전 목적함수와 제약을 명확히 정의합니다.
- 작은 문제는 brute force로 정답을 먼저 구합니다.
- maximize/minimize와 Hamiltonian 부호를 기록합니다.
- constant offset을 버릴 때 에너지 비교에 미치는 영향을 기록합니다.
- QAOA가 작은 문제에서 고전 알고리즘보다 유리하다고 주장하지 않습니다.
- 한 번의 optimizer 결과를 알고리즘 성능으로 일반화하지 않습니다.
