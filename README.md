# 수치 모델 앙상블을 활용한 강수량 예측

<br>


<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ec09474d-2d96-45f8-8e20-93b2fc0b7c7c" width="70%">
    </td>
  </tr>
</table>

<br>

## 수상결과

**입선(사실은 1등이었던 것..!): 최하단에 설명**

<br>


## 진행기간
> **2024/05/22~08/07**
<br>


## 사용 데이터
- **수치모델 데이터: 전국 20개 지점**
    - 예측자료
        - **수치 앙상블 계급별 강수확률**
        - 3시간 주기 자료(일 2회(3~240시간))
    - 관측자료
        - **강수량(관측값)**
        - 3시간 단위 누적 강수량
- 학습 데이터 기간: A\~C년의 각 5\~9월(총 3년 중 5~9월)
- 검증 데이터 기간: D년 5~9월(특정 5지점의 확률 자료)
<br>

## 개발 환경
> **Python, HIVE**
<br>

### 역할(팀장)
> **EDA, Data Prerprocessing, Modeling, 보고서 작성, 발표**
<br>

## 분석 주제
> **수치 모델 앙상블을 활용한 강수량 예측**
<br>


## 공모 범위 및 분석 목표
- **공모 범위**
    - 앙상블 확률 자료를 활용하여 3시간 누적 강수량의 계급구간 예측
    - 10개의 강수계급(mm)을 반영하여 모델 구성 및 예측
    - 제공된 수치모델 데이터만 사용하여 분석
- **분석 목표**
    - 강수량은 의류, 토목, 관광사업 등 대부분의 분야에 영향을 주며, 농업, 수자원 관리, 재난 관리 등의 분야에는 직접적이며 치명적인 영향을 끼침
        - 침수 사고로 인한 인명피해나 야구에서 우천 취소 등 `강수로 인한 피해는 특정 분야에 국한 되지 않음`
        - 뿐만 아니라  수자원 관리나 농업 분야에서의 작물 관리 등 단순히 폭우 뿐만 아니라 `비가 온다는 것 자체를 예측하는 것도 중요`
    - 현재 기상청에서는 상세한 기상 및 예상 강수량을 제공하고 있으나, 예보의 불확실성이 소수 존재하기에 이를 개선할 필요성
    - 정확한 강수량 예측은 각 분야에서의 피해를 최소화하고 실질적인 이득을 얻는 데에 큰 기여
    - `주어진 수치모델 앙상블 강수 확률 자료를 활용하여 예측 모델을 생성하는 것을 목표`
<br>

## 분석 요약
1. 데이터 수집
2. EDA
    1. 시각화
    2. 파생변수
3. Data Preprocessing
    1. 데이터 정렬(시계열 요소 유지)
    2. Yeo-Jonhson Transform
    3. 결측치 제거
4. Modeling(Bi-LSTM)
    1. Layer 설계 및 학습
    2. Custom Loss Function
5. Varification
    1. Cross Validation
    2. 연도 분리
    3. 결과 해석

<br>

## 분석 과정

### *데이터 수집*

> **주최 측 HIVE에 접속 후 제공된 데이터를 SQL 쿼리로 수집**

<br>

### *EDA&파생변수*

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/939832b0-d070-46b8-97ab-a6f113138472" width="100%">
    </td>
  </tr>
</table>

- 주최 측에서 제공한 데이터는 슈퍼컴퓨터로 만들어진 모델을 앙상블하여 만들어진, 구간별 강수량의 누적확률을 나타내는 수치모델 데이터
- **좌측 표에서 흰색 배경은 기존 변수, 회색 배경은 생성한 파생 변수, 하늘색은 예측해야할 타겟**
- 기존 변수를 살펴보면 지점별로 특정 발표 시각에 3시간 간격으로 10일 이후 까지의 누적확률값을 제공함을 파악
    - **이에 시계열 모델을 사용 시 예측 시각을 기준으로 정렬을 진행해야한다고 판단했고, 지점 코드와 결합하여 basis_index라는 파생 변수 생성**
        - 관측소별로 강수확률을 제공하기 때문에 이를 고려
        - 예측 시각의 정보를 담고 있는 ef_year, ef_month, ef_day, ef_hour 네 변수를 통합하고, stn4contest를 추가
- 추가적으로 누적확률이 아닌 `개별 확률을 나타내는 변수` 및 `강수 기댓값(V_expect)`이라는 `파생변수`를 생성
    - ex. V00_ind는 강수량이 0.1mm 미만일 확률
    - $v\_{expect} =
    0.05 \times v_{00\_ind} + 0.15 \times v_{01\_ind} + 0.35 \times v_{02\_ind} + 0.75 \times v_{03\_ind} +
    1.5 \times v_{04\_ind} + 3.5 \times v_{05\_ind} + 7.5 \times v_{06\_ind} + 15 \times v_{07\_ind} +
    25 \times v_{08\_ind} + 30 \times v_{09\_ind}$

<br>

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/8b93d8c4-0171-4e4c-b984-feb1b9cb7cff" width="100%">
    </td>
  </tr>
</table>

- 좌측 그래프에서 강수량의 분포를 살펴본 결과 연도 별로 분포가 상이한 것으로 파악
    - **이에 시간과 관련된 변수를 추가할 시 특정 연도에 과적합 될 수 있다고 판단하여 추가적인 변수를 생성하지는 않음(계절 변수 등)**
- 각 연도별로 5월부터 9월까지만 존재하기 때문에 교차검증이나 학습 데이터 지정 시 연도 분리가 필요하다고 판단
- **우측 그래프에서 기존 변수와 파생변수에서 분포가 전부 skewed 된 상태임을 확인**
    - 이에 모델 학습 시 학습 속도 향상을 위해 `정규화를 진행`해야한다고 판단

<br>

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/cdc47d35-1dc3-4604-8688-a4580197deb0" width="100%">
    </td>
  </tr>
</table>

- 발표 시각과 예측 시각간의 차이를 나타내는 dh를 활용해서 실강수량과 여러 변수간의 상관계수를 나타낸 그래프
    - **dh가 작을 수록 상관계수가 증가함을 확인**
    - 이는 강수량을 예측할 때 dh가 작은 행의 신뢰도가 더 높을 수 있다는 것을 알 수 있음
- 이에 특정 시점의 입력에 더 많은 가중치를 부여하는 `Attention Mechanism`을 활용하거나 `dh가 큰 행의 경우 학습에 덜 영향이 가도록 하는 방법`을 사용
    - Attention Mechanism의 경우 성능이 저하
    - 행 가중치를 달리하여 학습을 진행시킨 것은 미비한 성능 향상
        - 위의 수식처럼 적절한 알파값을 설정해야하는데 시간 소요가 많이 될 것으로 예상되어 보류(성능 향상에 크게  도움이 안됨)

<br>


### *Data Preprocessiing*

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/318a7b1d-29c7-4746-9759-361da77845b4" width="100%">
    </td>
  </tr>
</table>

- 전처리는 크게 3가지를 진행
    - **데이터 정렬**
        - 지점과 예측 시각을 결합한 파생변수인 basis_index를 기준으로 정렬 후 dh 기준으로 오름차순 정렬을 추가적으로 진행
        - 이는 모델 학습 과정에서 시계열 요소를 유지하게 만듦
            - 활용한 모델의 예시로 내일을 예측할 때 오늘의 확률 값의 정보를 이용할 수 있도록 한 것
    - **결측치 제거**
        - 실강수량 값이 -999인 행은 학습에 노이즈 발생 위험이 있어 제거
    - **Yeo-Johnson Transformation**
        - skewed된 변수의 정규화 및 표준화를 진행
        - Box Cox 변환의 확장된 형태로 0을 포함한 데이터를 정규분포에 가깝게 변환할 수 있다는 장점

<br>

### *Modeling*

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/f04b6574-8381-44dc-9e27-431a3c405323" width="100%">
    </td>
  </tr>
</table>

- 시계열 요소를 고려하기 위해 `LSTM, GRU, Bidirectional LSTM` 모델을 활용
    - 단순 성능을 살펴보았을 때는 GRU를 제외한 두 모델이 비슷했고, `Bidirectional LSTM`의 학습 안정성이 좀 더 좋다는 것을 파악하여 이를 최종 모델로 선정
    - 시간에 따라 정보를 전달하는 RNN 모델의 장기 의존성 문제를 해결한 모델
    - 즉, 오늘을 예측할 때, 오래 전의 정보를 기억하여 예측.  이때 Bidirectional LSTM은 단방향이 아닌 양방향으로 정보를 처리해서 학습
- Bidirectional LSTM Layer 2개와 Dense Layer로 간단한 구조로 설계
    - 변수가 그리 많지 않아 복잡한 모델 설계 시 과적합이 일어나거나 학습 시간이 매우 소요되어 이처럼 간단한 구조로 설계
- 첫 번째 Layer 내부에만 `Dropout`을 설정해서 과적합을 방지하도록 했고, `시간 종속성 유지 및 앙상블 효과`

<br>

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/35b1b469-2b0e-44be-b39d-87c54e52dd2e" width="100%">
    </td>
  </tr>
</table>

- 실강수량, 즉 `연속형 변수를 예측`하는 Loss를 사용하면 `Label 순서에 민감`하게되고, 단순히 `계급 구간을 예측`하는 Loss를 사용하면 `데이터 불균형에 매우 민감`하다는 단점
- **이러한 점을 해결하기 위해 RMSE와 CSI라는 평가지표를 결합하는 방식으로 손실함수를 설계**
    - 보통 Loss 값을 계산할 때 행 단위로 계산하여 배치 단위로 평균 혹은 합산하는 방식을 사용하나 CSI의 경우 행단위로 계산하면 0과 1만 나오게 되는 문제가 발생
    - 이에 CSI를 계산할 때는 배치 단위로 계산할 수 있도록 설계 후 다음 수식과 같은 손실 함수를 생성하여  모델 학습에 사용

<br>

### *Verification*

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/f2972477-d348-4ea9-bd55-c08a8cafd668" width="100%">
    </td>
  </tr>
</table>

- `최종 모델 학습 이전 먼저 시계열 교차 검증을 진행`
    - 각 학습데이터와 검증데이터 셋이 연속적이지 않기 때문에, 일반적인 시계열 교차 검증과는 달리 각 연도를 분리
    - C년도를 검증데이터로 설정 후, A, B년도 데이터를 각각 학습
    - 이후 각 모델과 두 모델의 앙상블까지 총 3가지 모델을 비교해서 검증
    - **그 결과 직전 연도, 즉 B년도로 학습한 모델이 C년도를 예측할 때 성능이 가장 높음**
- **최종 모델의 학습데이터를 C년도로 설정 하고, 모델의 일반화를 위해 A,B년도 Loss의 평균을 최종 Loss로 설정**
    - 실제 타겟은 실강수량으로 설정하여 예측하고 이를 계급으로 다시 변환
- 모델 파라미터는 우측 표와 같이 설정(Grid Search 방식으로 선정)

<br>

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ab765644-1e22-4b3e-8238-52fa26d0a1f4" width="80%">
    </td>
  </tr>
</table>

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/c01b7f31-37ed-461c-be6f-f0ff57c5292f" width="100%">
    </td>
  </tr>
</table>

- `좌측은 dh별로 정확도를 나타내는 그래프`
    - 앞선 EDA에서는 dh가 증가하면 변수와 실강수량간의 상관관계가 급격히 낮아짐을 확인
    - **모델 예측 결과에서는 dh가 증가해도 비교적 높은 정확도를 유지하는 것을 볼 수 있음**
    - **이는 시간 요소를 고려하는 시계열 모델이 유의했음을 시사**
- `우측은 구간별 평가지표를 나타내는 그래프`
    - 계급구간을 우측 표와 같이 기상청에서 제공하는 명칭으로 변경해서 살펴보았을 때, 모든 구간에서 무작위 모형 대비 성능이 우수
    - **이는 불균형 데이터 학습으로 인한 문제가 발생하지 않았고, 데이터의 패턴을 효과적으로 학습했음을 확인**
    - 막대그래프는 데이터에 각 계급 구간이 차지하는 비율
        - **무작위 모형의 recall, precision으로 해석할 수 있음**
    - `주황색 선은 예측 모델의 Recall`, `보라색 선은 예측 모델의 Precision`을 나타내는데 여기서 `Recall은 미탐지 평가지표`로, `Precision은 오탐지 평가지표`로 해석
        - **무강수의 경우를 보면 두 지표 모두 매우 우수**
        - **강한비의 경우 Precision이 비교적 높기 때문에 강한비라고 모델이 예측했다면 그럴 확률이 매우 높을 것으로 미리 대책을 마련하는데 있어 불필요한 비용이 생기지 않을 것으로 해석**

<br>

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/ca6c981c-c98f-4608-8ce1-0f366dc492da" width="100%">
    </td>
  </tr>
</table>

- 시간 요소를 고려한 모델을 사용함으로써 보다 정확하고 먼 시일의 강수량을 예측 가능
- **Loss 부분을 직접 설계함으로써 연속형 및 이산형 평가기준을 동시에 고려하여 불균형 데이터라는 단점에 큰 영향을 받지 않음**
- 자동 관개 시스템이 강수량 예측에 따라 작동하도록 하여 스마트 농업 시스템을 구축하거나, 외부 작업 및 행사의 대체 계획 마련 등 다양한 산업에서 도움
- 오보로 인한 불필요한 비용을 감소, 재난 예방, 실시간 정보 제공 등 여러 방면에서 큰 도움

<br>


## 시사점 및 보완할 점

- **Modeliing**
    - `맞춤형 손실 함수 설계`
        - **Loss 부분을 직접 설계함으로써 연속형 및 이산형 평가기준을 동시에 고려하여 불균형 데이터라는 단점에 큰 영향을 받지 않음**
            - `기본 손실함수를 사용했을 때보다 성능이 약 80% 높아짐`
    - `모델 구조 및 학습 전략`
        - 잔차 연결(Residual Connection), Attention Mechanism 등을 실험하며 다양한 딥러닝 모델 구조 탐색
            - 과소적합이 심하게 일어나 이를 활용하기 위해서는 매우 긴 학습시간이 소요될 것으로 예상되어 간결한 모델 구조로 진행
        - Early Stop 설정과 다양한 Activation Function 실험
        - 시계열 교차 검증 방식에서 연도별 데이터 분리를 도입해 일반화 성능 확보
        - 시간 요소를 고려한 모델을 사용함으로써 보다 정확하고 먼 시일의 강수량을 예측 가능
- **결과 분석**
    - `성능 평가`
        - Recall은 미탐지 평가지표로, Precision은 오탐지 평가지표로 해석
        - 무강수를 예측하는데 있어 두 지표 모두 우수
        - 강한비의 경우 Precision이 비교적 높으므로 강수 대응 계획 수립 시 신뢰도 향상 및 비용 절감 가능
    - `EDA`
        - dh가 작을 수록 상관계수가 증가함을 확인
            - 강수량을 예측할 때 dh가 작은 행의 신뢰도가 더 높을 수 있다는 것을 확인
            - 이에 특정 시점의 입력에 더 많은 가중치를 부여하는 Attention Mechanism을 활용하거나 dh가 큰 행의 경우 학습에 덜 영향이 가도록 하는 방법 실험
            - 결과적으로 성능 향상에는 도움이 되지 않았음
        - 연도별 강수량 분포 차이를 확인, 시간 관련 변수 추가 시 과적합 위험 고려
- **기대효과**
    - 자동 관개 시스템이 강수량 예측에 따라 작동하도록 하여 스마트 농업 시스템을 구축하거나, 외부 작업 및 행사의 대체 계획 마련 등 다양한 산업에서 도움
    - 오보로 인한 불필요한 비용을 감소, 재난 예방, 실시간 정보 제공 등 여러 방면에서 큰 도움
- **향후 과제**
    - 충분한 학습 시간과 자원 확보 시 batch size와 학습률을 낮추어 더 안정적인 학습 가능.
    - dh 값 기반 가중치 조정 등 세부 모델 개선 방향 모색.
    - 적절한 alpha 값 설정을 통해 모델 적용 범위 확장 연구 필요


<br>

## 대회 운영에서 아쉬운 점(사실 1등인 이유..!)

- **Data Leakage**
    - `데이터의 구조를 살펴보면 발표시각 기준으로 여러 예측 날짜의 강수확률을 제공`
        - 발표 시각과 예측 날짜 사이의 간격이 길수록 해당 데이터의 신뢰도가 낮음(dh가 큰 행은 신뢰도가 낮다)
    - `이에 첫 회의에서 예측 날짜를 기준으로 묶어서 pivot table 형태로 바꾼 후 예측을 진행하자는 의견이 제안`
        - 동일한 예측 날짜에 대한 데이터를 하나로 합쳐 모델에 입력
    - `하지만 이렇게 진행할 경우 test 셋을 예측 시 Data Leakage가 발생`
        - 발표 시각이 1월 1일인 데이터를 사용해 1월 5일의 강수 확률을 예측한다고 가정
        - pivot table로 데이터를 묶으면, **발표 시각이 1월 3일인 데이터에 포함된 1월 5일의 강수 확률 정보**도 모델이 사용할 수 있게 됨
        - 특정 예측 날짜(1월 5일)의 강수 확률이 여러 발표 시각(1월 1일, 2일, 3일  등)에 대해 동일하게 처리되면서 미래 정보를 학습하게 되는 문제 발생
- **문의**
    - 기상청 문의 처에 위와 같은 문제점이 발생할 수 있지 않은가 문의를 넣었으나 처음에는 괜찮다고 답변함
    - **아무리 생각해도 문제가 있다고 판단해서 추가적으로 4번의 문의를 드리며 직접 표를 그리고 자세하게 설명한 결과 Data Leakage가 맞다는 답변을 받음**
- **가설**
    - `예측 시각을 기준으로 묶는 방법 혹은 dh가 가장 작은 행만 추출해서 학습을 하는 경우가 만일 Data Leakage가 발생하지 않는다고 가정`
        - 제공되는 데이터가 의미가 퇴색됨(발표시각 기준 최대 10일 이후의 강수확률 제공)
        - 최대 12시간 이후의 강수량만 예측가능하게 바뀜
        - 모델의 활용성이 매우 떨어진다고 판단
        - `Data Leakage 발생 가능성이 없다고 하더라도, 모델의 신뢰성과 실질적 의미를 고려하는 것이 중요하다고 팀원들에게 제안`
            - **답변을 받기 전부터 예측 시각으로 데이터를 묶는 방식은 사용하지 않는 방향으로 의견을 제시하고 방향성 확립**
- **결과**
    - 수상자 발표 이후 오랜 시간이 지나고 홈페이지에 수상자들의 보고서가 올라와 확인한 결과 모두 예측 시각을 기준으로 묶거나 dh가 작은 행만 사용함을 확인
    - **모델 구조나 전처리 방법 등 우리 팀의 방법이 훨씬 고도화 되어 있음을 확인**
        - dh에 따른 행 가중치 변화
        - 파생변수
        - 모델 구조 설계
        - …
    - ***운영 측에서 문제가 있었는지, 아니면 단순히 성능 만을 중시하여 해당 방식을 고려하지 않았는지는 불분명. 그러나 모델의 활용성과 신뢰성을 고려할 때, 우리의 접근 방식이 더 적절하다고 판단하여 후회는 없음!***









