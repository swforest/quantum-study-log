# 4주차 - 회로 조작과 기초 알고리즘

## 주간 목표

반복되는 회로를 instruction·gate로 묶고, 매개변수를 분리하며, 저장 형식을 목적에 맞게 고릅니다. 그 도구를 Deutsch-Jozsa, Bernstein-Vazirani, Grover 알고리즘에 적용하여 `oracle → 간섭 → 측정` 구조를 이해합니다.

## 일정

| 요일 | 주제 | 결과물 |
|---|---|---|
| 월 | 합성·분해·사용자 gate | 재사용 가능한 Bell preparation instruction |
| 화 | 매개변수화 회로 | parameter sweep 회로 |
| 수 | OpenQASM·QPY | round-trip 비교 보고서 |
| 목 | Deutsch-Jozsa·Bernstein-Vazirani | oracle 기반 두 알고리즘 |
| 금 | Grover | 2큐비트 search와 성공 확률 분석 |

## 주간 완료 기준

- compose의 qubit mapping을 명시합니다.
- parameter binding 전후 회로를 구분합니다.
- OpenQASM을 완전한 Qiskit 저장 형식으로 오해하지 않습니다.
- oracle이 답을 직접 출력하는 장치가 아니라 phase·function 정보를 encode한다는 점을 설명합니다.
- 알고리즘 성공 조건과 query 이점을 문제 가정과 함께 설명합니다.
