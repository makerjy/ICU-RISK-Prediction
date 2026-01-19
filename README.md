

# ICU Risk Prediction Pipeline (MIMIC-IV)

MIMIC-IV ICU 데이터를 기반으로 중증 환자의 사망 위험(mortality) 및 Failure-to-Rescue(FTR) 가능성을 ICU 입실 초기 시점에 예측하기 위한 의료 데이터 분석/머신러닝 파이프라인 프로젝트입니다.

이 프로젝트의 초점은 “단일 모델 SOTA”가 아니라,
	•	EDA → 전처리 → 피처링 → 모델링 → 검증 → 시각화/프로토타입
	•	각 단계를 분리 가능한 실험 단위로 만들고
	•	비교 실험이 가능한 형태로 재현 가능한 연구 구조를 구축하는 데 있습니다.

본 메인 레포는 단계별 서브 레포지토리를 연결하는 연구 흐름 인덱스 역할을 수행합니다.

---

## Problem Statement

ICU 환경에서는 환자 상태가 빠르게 악화되며, 개입 지연은 사망으로 이어질 수 있습니다. 본 프로젝트는 아래 질문에서 출발합니다.
	•	ICU 입실 이후 조기 시점에서 사망 또는 구조 실패 위험을 탐지할 수 있는가?
	•	극심한 클래스 불균형(사망 rare) 환경에서 어떤 평가 전략이 합리적인가?
	•	결측치 처리(imputation) 전략이 성능에 어느 정도의 구조적 영향을 주는가?
	•	복잡한 딥러닝 없이도 단순 ML + 전처리 조합으로 임상적 신호를 포착할 수 있는가?

---

## Dataset
	•	Source: MIMIC-IV (v2.x)
	•	Population: ICU first stays only
	•	Modalities
	•	Demographics
	•	Vital signs (irregular time-series)
	•	Laboratory measurements
	•	Clinical scores
	•	Key characteristics
	•	Severe class imbalance (mortality is rare)
	•	Irregular sampling & variable-length time-series
	•	Large missingness (feature-dependent & time-dependent)

의료 데이터 특성상 cohort 정의와 전처리 선택이 성능에 직접 영향을 주므로, 이를 별도 실험 축으로 분리해 검증합니다.

---

## Project Scope
	•	단일 split 성능을 보고 끝내는 구조가 아니라,
	•	전처리/모델 구조/평가 레벨을 분리해 “성능이 어디서 생기고 무너지는지”를 확인하는 연구 설계를 목표로 합니다.

---

## Overall Pipeline
```text
┌───────────────────────────────┐
│        MIMIC-IV Dataset       │
│ (ICU stays, vitals, labs, etc)│
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│     Exploratory Analysis      │
│        (ICU-RISK_EDA)         │
│ - cohort validation           │
│ - variable distributions      │
│ - missingness patterns        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│  Feature Engineering & Prep   │
│     (ICU-RISK_Modeling)       │
│ - preprocessing pipelines     │
│ - imbalance handling          │
│ - baseline feature sets       │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│        Model Training         │
│     (ICU-RISK_Modeling)       │
│ - LR / GB / XGB / LGBM        │
│ - ensemble voting             │
│ - cross-validation            │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│     Validation Pipelines      │
│ (REALMIP_ValidModel_ML)       │
│ - model structure comparison  │
│ (REALMIP-MIMIC_IV_imputation) │
│ - imputation strategy test    │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│      Service Prototype        │
│          (icu-risk)           │
│ - prediction visualization    │
│ - simple API + UI structure   │
└───────────────────────────────┘
```
각 단계는 독립 레포로 관리되며, 본 레포는 전체 흐름을 구조적으로 연결합니다.

---

## Evaluation Strategy

Metrics
	•	AUROC
	•	AUPRC (primary)
	•	Recall 중심 해석 (고위험 환자 조기 탐지 관점)

Validation Levels
	•	Row-level evaluation
	•	Stay-level aggregated evaluation
	•	Preprocessing strategy comparison
	•	Model structure comparison

핵심은 “점수 한 번 잘 나온 모델”이 아니라, 설정이 바뀌어도 구조적으로 유지되는지를 확인하는 것입니다.

⸻

## Research Focus

본 프로젝트가 실제로 검증하려는 포인트는 아래입니다.
	•	ICU 시계열 데이터에서 cohort 정의의 중요성
	•	결측치 처리 방식이 예측 성능에 미치는 구조적 영향
	•	단순 ML 모델 + 전처리 전략 조합의 가능성과 한계
	•	비교 실험이 가능한 형태의 재현 가능한 파이프라인 설계

가설: 모델 복잡도보다 데이터 정의/전처리 설계의 영향이 더 클 수 있다.

---

## Repositories
	•	EDA
	•	https://github.com/makerjy/ICU-RISK_EDA
	•	Modeling Pipeline
	•	https://github.com/makerjy/ICU-RISK_Modeling
	•	Validation — Model Comparison
	•	https://github.com/makerjy/REALMIP_ValidModel_ML
	•	Validation — Imputation Strategy Comparison
	•	https://github.com/makerjy/REALMIP-MIMIC_IV_imputation_cmp
	•	Service Prototype
	•	https://github.com/makerjy/icu-risk

각 레포는 단독 프로젝트가 아니라, 하나의 ICU 위험 예측 파이프라인에서 서로 다른 단계/실험 축을 담당합니다.

---

## How to Use This Repo

이 레포는 코드를 “한 곳에 모아 실행”하는 목적이 아니라,
	1.	전체 연구 흐름을 한눈에 이해하고
	2.	실험 축(전처리/모델/평가)을 분리해 따라가며
	3.	필요한 단계 레포로 이동할 수 있게 하는

인덱스/허브 역할을 합니다.

추천 탐색 순서:
	1.	ICU-RISK_EDA → 2) ICU-RISK_Modeling → 3) REALMIP validation repos → 4) icu-risk prototype

---

## Limitations & Future Work
	•	멀티모달 확장: ECG waveform, imaging 등 결합 구조로 확장 가능
	•	다중 시간 예측: 단일 시점 분류를 넘어 horizon-based risk forecasting으로 확장 가능
	•	임상적 사용을 위한 추가 과제: calibration, subgroup fairness, prospective validation 등 (현재 범위 밖)

---

## Notes
	•	MIMIC-IV는 접근 권한(PhysioNet credentialing)이 필요한 데이터입니다.
	•	본 프로젝트는 연구/학습 목적의 재현 가능한 실험 구조를 목표로 하며, 임상 의사결정을 대체하지 않습니다.

