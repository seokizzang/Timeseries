# 기상데이터 시계열 예측 (서울 ASOS, 2023–2024)

최근 72시간의 기상 관측치를 입력으로 받아 1시간 뒤 기온(°C) 을 예측하는 프로젝트입니다.
베이스라인(Seasonal Naïve)과 GRU / LSTM / CNN+GRU 모델을 비교했습니다. 실행 환경은 Google Colab / PyTorch 기준입니다.

## 🔍 개요

- 문제 정의: 입력 (B, T=72, F) → 출력 (B, 1) (1시간 뒤 기온)

- 지표: RMSE / MAE (단위: °C)

- 데이터 기간:

  - Train/Val: 2023-01-01 ~ 2023-12-31

  - Test: 2024-01-01 ~ 2024-12-31

- 지역: 서울(ASOS)

## 📦 데이터

- data/weather_seoul_2023.csv (학습·검증)

- data/weather_seoul_2024.csv (테스트)

- 출처: 기상청 ASOS(종관기상관측) 공개자료 (서울 지점)

## 사용 컬럼(핵심)

- 입력: 습도(%), 풍속(m/s), 풍향(16방위), 현지기압(hPa), 해면기압(hPa), 지면온도(°C), 강수량(mm), 일조(hr), 적설(cm)

- 타깃: 기온(°C) → 코드에서는 temp

## 🧹 전처리

- 일시 → timestamp (datetime) 표준화

- 핵심 ASOS 변수만 선별

- 전부 결측인 열 제외

- 시간축 정규화: asfreq('1h') → interpolate('time') → ffill/bfill

- 스케일링 누수 방지: train으로 StandardScaler 학습 → val/test 변환

## 🧠 모델

- 베이스라인: Seasonal Naïve

- GRU 회귀: 입력 72시간 → GRU → Linear(1)

- LSTM 회귀: 구성 유사(양방향 옵션 불사용)

- CNN+GRU: 1D-CNN(국소 패턴) → GRU(장기 의존성) → Linear(1)

## 공통 학습 설정

- loss=MSELoss, optimizer=AdamW(lr=1e-3)

- batch(train/eval)=128/256, EarlyStopping(patience=6)

## 🧪 실험

- 입력 길이: 72시간 고정

- 타깃: 1시간 후 기온

- 데이터 분할:

  - Train: 2023-01-01 ~ 2023-10-31

  - Val: 2023-11-01 ~ 2023-12-31

  - Test: 2024-01-01 ~ 2024-12-31
