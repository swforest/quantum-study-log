# 구형 학습자료와 Qiskit 2.x API 현대화 가이드

## 목적

한국어 저장소의 설명은 회로와 게이트를 복습하는 데 유용하지만 환경은 Python 3.8과 Qiskit `0.29.1`에 고정되어 있습니다. 이 문서는 구형 코드를 보았을 때 무엇을 개념으로 남기고, 무엇을 현재 방식으로 바꾸어야 하는지 판단하기 위한 기준입니다.

이 가이드는 2026년 9월의 Qiskit 2.x 문서를 기준으로 작성했습니다. 패키지는 계속 바뀔 수 있으므로 실제 수업에서는 설치된 버전을 먼저 출력하고 해당 버전의 공식 API 문서를 확인합니다.

## 가장 중요한 원칙

1. **회로의 수학적 의미와 특정 Python API를 분리합니다.** Hadamard가 중첩을 만든다는 사실은 유지되지만 실행 함수의 이름은 바뀔 수 있습니다.
2. **상태 계산, 샷 기반 샘플링, 기대값 계산을 구분합니다.** 예전에는 모두 `execute()`에 의존하기 쉬웠지만 현재는 목적에 맞는 도구를 선택합니다.
3. **로컬 학습과 실제 QPU 실행을 분리합니다.** 로컬에서 상태와 논리를 검증한 뒤에만 Runtime 흐름을 다룹니다.
4. **실행 전 트랜스파일과 ISA 적합성을 의식합니다.** 추상 회로를 실제 장치가 직접 실행한다고 생각하지 않습니다.
5. **결과 객체의 구조를 추측하지 않습니다.** V1의 `quasi_dists`와 V2의 `BitArray` 기반 결과는 다릅니다.

## 핵심 변환표

| 구형 표현 | 상태 | 현재 학습 경로 |
|---|---|---|
| `from qiskit import execute` | Qiskit 1.0에서 제거 | `transpile()` 후 `backend.run()`, 또는 목적에 맞는 Primitive V2 |
| `from qiskit import BasicAer` | 제거 | `BasicSimulator`, `Statevector`, `Operator` |
| `from qiskit import Aer` | 패키지 분리 | `from qiskit_aer import AerSimulator` |
| `IBMQ.load_account()` | 구형 provider | `QiskitRuntimeService()` |
| `IBMQ.get_provider()` | 구형 계정 구조 | 서비스에서 `backend()` 또는 `least_busy()` 사용 |
| `qiskit.tools` | 제거 | 기능별 현재 모듈 또는 직접 상태·Job API 사용 |
| `qiskit.tools.jupyter` | 제거 | 일반 Jupyter 표시와 현재 Runtime API 사용 |
| `qiskit.test.mock.Fake...` | 공개 경로 제거 | `qiskit_ibm_runtime.fake_provider` 또는 `GenericBackendV2` |
| `qc.qasm()` | 제거 | `qiskit.qasm2.dumps(qc)` |
| `QuantumCircuit.from_qasm_file()` | 일부 호환되지만 학습상 구식 | `qiskit.qasm2.load()` |
| V1 `Sampler`, `Estimator` | Qiskit 2.0에서 구현 제거 | 로컬 `StatevectorSampler`, `StatevectorEstimator` |
| Runtime `Sampler`, `Estimator` V1 | 구형 | `SamplerV2`, `EstimatorV2` |
| `result.quasi_dists` | V1 결과 | V2 `PrimitiveResult` → PUB 결과 → `BitArray`/데이터 필드 |
| `backend.configuration()` | BackendV1 중심 | `BackendV2.target`, `num_qubits`, `operation_names` 등 |
| `backend.properties()` | BackendV1 중심 | `target`의 `InstructionProperties`, backend qubit properties |
| `qiskit.opflow` | 제거 | `SparsePauliOp`, `Operator`, Primitives V2 |

공식 참고 문서:

- [Qiskit 1.0 feature changes](https://quantum.cloud.ibm.com/docs/en/guides/qiskit-1.0-features)
- [Qiskit 2.0 migration guide](https://quantum.cloud.ibm.com/docs/en/guides/qiskit-2.0)
- [Primitive inputs and outputs](https://quantum.cloud.ibm.com/docs/en/guides/primitive-input-output)
- [OpenQASM 2 and Qiskit](https://quantum.cloud.ibm.com/docs/en/guides/interoperate-qiskit-qasm2)

## 실행 목적별 현재 선택

### 1. 회로의 정확한 상태벡터가 필요한 경우

측정 전 순수 상태를 보고 싶다면 백엔드 샷 실행이 아니라 `Statevector`를 우선합니다.

```python
from qiskit import QuantumCircuit
from qiskit.quantum_info import Statevector

circuit = QuantumCircuit(2)
circuit.h(0)
circuit.cx(0, 1)

state = Statevector.from_instruction(circuit)
print(state.data)
```

측정 명령은 비단일 연산이므로 상태벡터 계산 전에 넣지 않습니다. 측정이 이미 있으면 복사본에서 최종 측정을 제거하거나, 상태 준비 회로와 측정 회로를 분리합니다.

### 2. 회로 전체의 유니터리 행렬이 필요한 경우

```python
from qiskit.quantum_info import Operator

unitary = Operator(circuit)
print(unitary.data)
```

측정, reset, classical control 등이 포함된 회로는 하나의 유니터리 행렬로 표현할 수 없습니다.

### 3. 로컬에서 측정 샘플이 필요한 경우

Primitive V2의 로컬 기준 구현을 사용합니다.

```python
from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorSampler

circuit = QuantumCircuit(2)
circuit.h(0)
circuit.cx(0, 1)
circuit.measure_all()

sampler = StatevectorSampler(default_shots=1024, seed=7)
result = sampler.run([circuit]).result()
counts = result[0].data.meas.get_counts()
print(counts)
```

측정 레지스터 이름이 `meas`가 아닐 수 있습니다. V2 결과는 고정된 단일 `counts` 속성이 아니라 회로의 classical register 이름을 따라 데이터가 구성됩니다.

### 4. 로컬에서 기대값이 필요한 경우

```python
from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorEstimator
from qiskit.quantum_info import SparsePauliOp

circuit = QuantumCircuit(1)
circuit.h(0)
observable = SparsePauliOp.from_list([("X", 1.0)])

estimator = StatevectorEstimator()
result = estimator.run([(circuit, observable)]).result()
expectation = result[0].data.evs
print(expectation)
```

Estimator에는 측정 게이트를 넣지 않습니다. Estimator가 관측가능량에 필요한 처리를 담당합니다.

### 5. Aer 시뮬레이터와 노이즈가 필요한 경우

```python
from qiskit import transpile
from qiskit_aer import AerSimulator

simulator = AerSimulator()
isa_circuit = transpile(circuit, simulator)
job = simulator.run(isa_circuit, shots=1024)
result = job.result()
print(result.get_counts())
```

Aer는 Qiskit SDK 코어와 별도 패키지입니다. 상태벡터만 필요하다면 항상 Aer부터 선택할 필요는 없습니다.

### 6. IBM Quantum Runtime을 사용하는 경우

다음은 개념적 골격입니다. 실제 계정 설정과 QPU 실행은 6주차에 명시적으로 확인한 후 진행합니다.

```python
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager

service = QiskitRuntimeService()
backend = service.least_busy(operational=True, simulator=False)

pass_manager = generate_preset_pass_manager(
    backend=backend,
    optimization_level=1,
)
isa_circuit = pass_manager.run(circuit)

sampler = SamplerV2(mode=backend)
job = sampler.run([isa_circuit], shots=1024)
```

실제 QPU는 비용·쿼터·대기시간·계정 권한이 관련되므로 학습자가 요청하지 않은 제출은 하지 않습니다.

## OpenQASM과 저장 형식

구형 예제:

```python
text = circuit.qasm()
circuit.qasm(filename="sample.qasm")
```

현재 OpenQASM 2 상호운용:

```python
from qiskit import qasm2

text = qasm2.dumps(circuit)
qasm2.dump(circuit, "sample.qasm")
loaded = qasm2.load("sample.qasm")
```

OpenQASM은 다른 시스템과 회로를 주고받기 위한 언어이지 Qiskit 객체를 완전하게 보존하는 일반 저장 형식이 아닙니다. Qiskit 회로 구조를 보존하려면 QPY를 고려합니다.

```python
from qiskit import qpy

with open("circuits.qpy", "wb") as stream:
    qpy.dump(circuit, stream)

with open("circuits.qpy", "rb") as stream:
    restored = qpy.load(stream)[0]
```

## 구형 노트북을 읽는 순서

구형 셀을 발견하면 바로 실행하지 말고 다음 질문에 답합니다.

1. 이 셀이 설명하려는 양자 개념은 무엇인가?
2. 입력은 회로, 상태, 연산자, 백엔드 중 무엇인가?
3. 원하는 출력은 상태벡터, 유니터리, 샷 결과, 기대값 중 무엇인가?
4. 계정·실제 장치가 꼭 필요한가?
5. 제거된 import나 BackendV1 속성이 있는가?
6. 현재 도구로 바꾸었을 때 결과의 의미가 동일한가?

예를 들어 `execute(qc, statevector_simulator)`는 `Statevector.from_instruction(qc)`로 바꿀 수 있지만, `execute(qc, qasm_simulator, shots=1024)`는 샷 기반 결과가 목적이므로 `StatevectorSampler`나 Aer 실행으로 옮겨야 합니다. 이름만 기계적으로 치환하면 학습 목표를 놓치기 쉽습니다.

## 버전 기록 양식

실습 노트북 첫 셀에는 다음 정보를 남깁니다.

```python
import platform
import qiskit

print("Python:", platform.python_version())
print("Qiskit:", qiskit.__version__)
```

별도 패키지는 각 패키지의 `__version__` 또는 패키지 메타데이터로 확인합니다. 오류가 발생하면 오류 메시지, 설치 버전, 공식 문서의 대상 버전을 함께 기록한 뒤 수정합니다.

## 이 과정에서 사용하지 않는 학습 습관

- 작동하지 않는 구형 코드를 버전만 낮춰 무조건 재현하지 않습니다.
- `from qiskit import *`를 사용하지 않습니다. 필요한 namespace를 명시합니다.
- 실제 QPU에서 먼저 시험하지 않습니다.
- 샷 수가 유한한 결과를 정확한 확률과 동일시하지 않습니다.
- 결과 딕셔너리의 문자열을 큐비트 번호 순서라고 단정하지 않습니다.
- 시각화 그림만 보고 전역위상과 상대위상을 혼동하지 않습니다.
