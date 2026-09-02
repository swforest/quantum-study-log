# 6주차 - Primitives V2와 IBM Quantum Runtime

## 주간 목표

현재 Qiskit의 대표 실행 추상화인 Sampler·Estimator V2를 이해합니다. 회로를 무조건 `execute()`하는 구형 모델에서 벗어나, `무엇을 알고 싶은가`에 따라 샘플링 또는 기대값 계산을 선택합니다.

## 일정

| 요일 | 주제 | 결과물 |
|---|---|---|
| 월 | V2·PUB·결과 container | Primitive 선택 의사결정표 |
| 화 | StatevectorSampler | parameter sweep와 BitArray 해석 |
| 수 | StatevectorEstimator | observable broadcast와 기대값 |
| 목 | Runtime SamplerV2 | ISA·service·job 흐름 설계 |
| 금 | Runtime EstimatorV2·실행 모드 | 실행 전략 비교 보고서 |

## 안전 원칙

- 계정 token을 문서·출력·Git에 기록하지 않습니다.
- 사용자가 명시적으로 승인하지 않으면 실제 QPU job을 제출하지 않습니다.
- 로컬 reference primitive로 논리를 먼저 검증합니다.
- 현재 공식 문서의 package version과 API signature를 확인합니다.
- V1 `quasi_dists`, V1 `EstimatorResult.values`를 V2 결과에 적용하지 않습니다.
