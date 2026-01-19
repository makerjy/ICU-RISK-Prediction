# ICU Risk Prediction Pipeline (MIMIC-IV)

MIMIC-IV ICU 데이터를 기반으로,
중증 환자의 사망 위험 및 Failure-to-Rescue(FTR) 가능성을 조기에 예측하는 것을 목표로 한
의료 데이터 분석 및 머신러닝 파이프라인 구축 프로젝트임.

본 프로젝트는 단일 모델 성능 개선이 아니라,
EDA → 전처리 → 모델링 → 검증 → 시각화까지 이어지는 전체 연구 흐름을
재현 가능한 구조로 설계하고 비교 실험을 수행하는 데 초점을 둠.

---

## Overview

ICU 환경에서는 환자 상태가 빠르게 악화될 수 있으며,
의료진의 개입이 지연될 경우 사망으로 이어질 가능성이 높아짐.

본 프로젝트는 다음 질문에서 출발함.

* ICU 입실 이후 조기 시점에서 사망 또는 구조 실패 가능성을 탐지할 수 있는가
* 극심한 클래스 불균형 환경에서 어떤 평가 전략이 합리적인가
* 결측치 처리 방식이 모델 성능에 얼마나 큰 영향을 미치는가
* 단순 ML 모델로도 의미 있는 임상적 신호를 포착할 수 있는가

이를 위해 데이터 구조 분석부터 전처리 전략, 모델 구조, 검증 방식까지
각 단계를 분리하여 실험하고 연결함.

---

## Dataset

* Source: MIMIC-IV (v2.x)
* Population: ICU first stays only
* Data types:

  * Demographics
  * Vital signs (time-series)
  * Laboratory measurements
  * Clinical scores
* Characteristics:

  * Severe class imbalance (mortality is rare)
  * Irregular time-series sampling
  * Large amount of missing values

의료 데이터 특성상, 전처리 방식과 cohort 정의가
모델 성능에 직접적인 영향을 미치므로 해당 부분을 별도 단계로 분리하여 검증함.

---

## Overall Pipeline

```text
┌───────────────────────────────┐
│        MIMIC-IV Dataset        │
│  (ICU stays, vitals, labs, etc)│
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│        Exploratory Analysis    │
│        (ICU-RISK_EDA)          │
│ - cohort validation            │
│ - variable distributions       │
│ - missingness patterns         │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│    Feature Engineering & Prep  │
│     (ICU-RISK_Modeling)        │
│ - preprocessing pipelines      │
│ - imbalance handling           │
│ - baseline feature sets        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│        Model Training          │
│     (ICU-RISK_Modeling)        │
│ - LR / GB / XGB / LGBM models  │
│ - ensemble voting              │
│ - cross-validation             │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│     Validation Pipelines       │
│  (REALMIP_ValidModel_ML)       │
│ - model structure comparison   │
│                                │
│ (REALMIP_imputation_cmp)       │
│ - missing value strategy test  │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│      Service Prototype         │
│          (icu-risk)            │
│ - prediction visualization     │
│ - simple API + UI structure    │
└───────────────────────────────┘
```

각 단계는 독립적인 GitHub 레포로 관리되며,
본 메인 레포는 전체 연구 흐름을 구조적으로 연결하는 인덱스 역할을 수행함.

---

## Evaluation Strategy

### Metrics

* AUROC
* AUPRC (주요 지표)
* Recall 중심 해석

클래스 불균형 환경에서 정확도는 의미가 거의 없다고 판단하여,
고위험 환자를 얼마나 잘 상위에 정렬하는지를 주요 기준으로 사용함.

### Validation Levels

* Row-level evaluation
* Stay-level aggregated evaluation
* Preprocessing strategy comparison
* Model structure comparison

단일 split 성능이 아닌,
구조적 안정성을 확인하는 방향으로 검증을 설계함.

---

## Research Focus

본 프로젝트에서 중점적으로 다룬 주제는 다음과 같음.

* ICU 시계열 데이터에서 cohort 정의의 중요성
* 결측치 처리 방식이 예측 성능에 미치는 구조적 영향
* 단순 ML 모델과 전처리 전략 조합의 한계와 가능성
* 재현 가능한 의료 데이터 분석 파이프라인 설계

모델 복잡도보다 데이터 구조와 전처리 전략의 영향이
더 클 수 있다는 가설을 검증하는 데 초점을 둠.

---

## Repository Structure

단계별 상세 구현은 아래 레포지토리에서 관리됨.

* EDA
  [https://github.com/makerjy/ICU-RISK_EDA](https://github.com/makerjy/ICU-RISK_EDA)

* Modeling Pipeline
  [https://github.com/makerjy/ICU-RISK_Modeling](https://github.com/makerjy/ICU-RISK_Modeling)

* Validation — Model Comparison
  [https://github.com/makerjy/REALMIP_ValidModel_ML](https://github.com/makerjy/REALMIP_ValidModel_ML)

* Validation — Imputation Strategy Comparison
  [https://github.com/makerjy/REALMIP-MIMIC_IV_imputation_cmp](https://github.com/makerjy/REALMIP-MIMIC_IV_imputation_cmp)

* Service Prototype
  [https://github.com/makerjy/icu-risk](https://github.com/makerjy/icu-risk)

각 레포는 단독 실험이 아니라,
하나의 ICU 위험 예측 파이프라인의 서로 다른 단계와 관점을 담당함.

---

## Limitations & Future Work

* Calibration curve 및 decision curve analysis 미포함
* 시계열 기반 딥러닝 모델(GRU, Transformer)과의 체계적 비교 필요
* 멀티모달 데이터(ECG waveform, imaging) 결합 구조 확장 가능성
* 다중 시간 예측(horizon-based risk forecasting) 구조 확장 가능

향후 연구에서는 단일 시점 분류 문제를 넘어,
시간 흐름에 따른 위험 변화 추적 구조로 확장할 계획임.

---

본 레포지토리는 전체 연구 흐름을 정리하는 메인 인덱스 역할을 하며,
각 단계별 실험 코드는 서브 레포지토리에서 관리됨.

