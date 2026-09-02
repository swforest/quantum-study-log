# 5주차 - Transpiler, BackendV2, 노이즈

## 주간 목표

추상적인 논리 회로가 실제 backend의 gate set·연결성·오류 특성에 맞는 ISA 회로로 바뀌는 과정을 이해합니다. 이상적인 알고리즘 정답과 유한 샷·하드웨어 노이즈가 섞인 관측 결과를 구분합니다.

## 일정

| 요일 | 주제 | 핵심 결과물 |
|---|---|---|
| 월 | transpilation·ISA | 원본과 ISA 회로 비교표 |
| 화 | optimization·PassManager | level·seed별 지표 실험 |
| 수 | BackendV2·Target·Layout | logical→physical mapping 보고서 |
| 목 | Aer noise model | readout·gate noise 시뮬레이션 |
| 금 | 이상·노이즈 비교 | 정량 비교와 완화 기초 |

## 완료 기준

- [ ] 논리 회로와 ISA 회로를 구분합니다.
- [ ] transpiler 출력의 논리적 동등성을 검사합니다.
- [ ] layout, routing, basis translation을 각각 설명합니다.
- [ ] BackendV2의 `target`을 정보의 중심으로 사용합니다.
- [ ] 유한 샷 편차와 noise bias를 구분합니다.
- [ ] noise mitigation과 error correction을 같은 것으로 부르지 않습니다.
