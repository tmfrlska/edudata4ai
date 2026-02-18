# edudata4ai

**AI·데이터 사이언스 교육을 위한 합성 데이터 생성 패키지**

[![PyPI Downloads](https://static.pepy.tech/personalized-badge/edudata4ai?period=total&units=INTERNATIONAL_SYSTEM&left_color=BLACK&right_color=GREEN&left_text=downloads)](https://pepy.tech/projects/edudata4ai)

[![Python](https://img.shields.io/badge/python-%3E%3D3.6-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.1.5-green)](https://pypi.org/project/edudata4ai/)

`edudata4ai`는 교육 현장에서 활용할 수 있는 30개의 내장 데이터셋과 다양한 통계 기법을 활용한 합성 데이터 생성기(Synthpop)를 제공하는 Python 패키지입니다.

## 설치

```bash
pip install edudata4ai
```

개발 모드 설치:

```bash
git clone https://github.com/tmfrlska/edudata.git
cd edudata
pip install -e .
```

## 빠른 시작

```python
from edudata.synthpop import Synthpop
from edudata.datasets.student import df

# 1. Synthpop 인스턴스 생성
synth = Synthpop(method='randomforest', seed=42)

# 2. 원본 데이터로 모델 학습
synth.fit(df, dtypes)

# 3. 합성 데이터 생성
synthetic_df = synth.generate(k=1000)

# 4. 원본 vs 합성 데이터 품질 비교
pMSE, pMSE_ratio = synth.compare(df, synthetic_df, visualize=True)
```

## 내장 데이터셋

30개의 교육용 데이터셋이 포함되어 있으며, pandas DataFrame과 dtypes 딕셔너리로 바로 사용할 수 있습니다.

```python
from edudata.datasets.pokemon import df
print(df.head())
print(dtypes)
```

| 카테고리 | 데이터셋 |
|---------|---------|
| 교육/인문 | `student`, `mbti`, `culturereview` |
| 스포츠/게임 | `baseball`, `pokemon`, `lolgame`, `amongus`, `eplresults`, `videogames` |
| 건강/의료 | `health`, `disease`, `bodyperformance` |
| 환경/과학 | `energy`, `earthquake`, `mosquito`, `notionaltemp`, `sensorytemp` |
| 산업/비즈니스 | `bakerysales`, `mobileprice`, `twitch`, `tshirtsize` |
| 기타 | `ageculture`, `datedelivery`, `heroesinformation`, `noodle`, `travelplan`, `weatherdelivery`, `weatherpark` |

## 합성 데이터 생성 방법 (Methods)

| 방법 | 키워드 | 설명 |
|-----|--------|-----|
| **RandomForest** | `'randomforest'` | 랜덤 포레스트 기반 합성 (기본값) |
| **CART** | `'cart'` | 의사결정트리(CART) 기반 합성 |
| **Parametric** | `'parametric'` | 데이터 타입에 따라 자동 선택 (수치형→NormRank, 범주형→Polyreg) |
| **NormRank** | `'normrank'` | 순위 기반 정규 회귀 |
| **Norm** | `'norm'` | 선형 회귀 (Ridge) |
| **Polyreg** | `'polyreg'` | 로지스틱 회귀 (범주형 데이터용) |
| **Sample** | `'sample'` | 부트스트랩 샘플링 |

```python
# 열별로 다른 방법 지정
synth = Synthpop(
    method={'age': 'normrank', 'gender': 'polyreg', 'score': 'cart'}
)
```

## 주요 API

### `Synthpop` 생성자

```python
Synthpop(
    method=None,            # 합성 방법 (str 또는 dict)
    visit_sequence=None,    # 열 합성 순서
    predictor_matrix=None,  # 열 간 의존성 행렬
    proper=False,           # 베이지안 적정 합성 여부
    cont_na=None,           # 특정 수치값을 NaN으로 매핑
    smoothing=False,        # 커널 밀도 스무딩 적용
    default_method='randomforest',  # 기본 합성 방법
    numtocat=None,          # 수치형→범주형 변환 열 목록
    catgroups=None,         # 범주 그룹 수
    seed=None,              # 랜덤 시드
    missing=False,          # 결측값 처리/주입
    outliers=False,         # 이상치 주입
    numtype=False,          # 수치형 출력 타입 ('int' 또는 False)
    save=False              # 합성 데이터 CSV 저장
)
```

### `fit(df, dtypes=None)`

원본 데이터로 합성 모델을 학습합니다.

- `df`: pandas DataFrame (원본 데이터)
- `dtypes`: 열별 데이터 타입 딕셔너리 (None이면 자동 감지)

### `generate(k=None)`

합성 데이터를 생성합니다.

- `k`: 생성할 레코드 수 (None이면 원본 데이터 크기와 동일)
- 반환: 합성 데이터 DataFrame

### `compare(df, synth, visualize=True)`

원본과 합성 데이터의 품질을 비교합니다.

- 로지스틱 회귀 기반 **pMSE**(Propensity Mean Squared Error) 산출
- **pMSE Ratio** < 1이면 합성 데이터 품질이 양호
- `visualize=True`이면 분포 비교 시각화 출력

## 고급 기능

### 결측값 처리

```python
synth = Synthpop(missing=False)     # 결측값 무시 (기본값)
synth = Synthpop(missing=True)      # 원본 결측 패턴 보존
synth = Synthpop(missing=0.1)       # 10% 결측값 랜덤 주입
```

### 이상치 주입

```python
synth = Synthpop(outliers=False)    # 이상치 없음 (기본값)
synth = Synthpop(outliers=True)     # 1% 이상치 주입
synth = Synthpop(outliers=0.05)     # 5% 이상치 주입 (IQR 기반)
```

### 수치형→범주형 변환

```python
synth = Synthpop(
    numtocat=['age', 'income'],
    catgroups={'age': 5, 'income': 4}
)
```

### 데이터 저장

```python
synth = Synthpop(save=True)          # 'synth.csv'로 저장
synth = Synthpop(save='my_data')     # 'my_data.csv'로 저장
```

## 패키지 구조

```
edudata/
├── __init__.py              # 패키지 진입점
├── synthpop.py              # 핵심 Synthpop 클래스
├── method/                  # 합성 방법 구현
│   ├── base.py              # 추상 기반 클래스
│   ├── ramdomforest.py      # RandomForest
│   ├── cart.py              # CART (의사결정트리)
│   ├── sample.py            # 부트스트랩 샘플링
│   ├── norm.py              # 선형 회귀
│   ├── normrank.py          # 순위 기반 정규 회귀
│   ├── polyreg.py           # 로지스틱 회귀
│   └── helpers.py           # 유틸리티 함수
├── processor/               # 데이터 전처리/후처리
├── validator/               # 입력 검증
└── datasets/                # 30개 내장 데이터셋
```

## 데이터 타입

`edudata4ai`는 다음과 같은 데이터 타입을 지원합니다:

- **수치형**: `int64`, `float64`, `datetime64[ns]`
- **범주형**: `category`, `bool`, `object`

## 의존성

- `numpy >= 1.17.3`
- `pandas >= 0.25.3`
- `scipy >= 1.3.1`
- `scikit-learn >= 0.22`
- `seaborn >= 0.11.2`

## 라이선스

MIT License

## 저자

**Seulki Kim** — [tmfrlska85@gmail.com](mailto:tmfrlska85@gmail.com)

GitHub: [https://github.com/tmfrlska/edudata](https://github.com/tmfrlska/edudata)
