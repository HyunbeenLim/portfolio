# Project

## Contents

- [RDA](#RDA)
    - [분석 목적](#분석-목적)
    - [RDA소개](#RDA-소개)
    - [Simulation Study](#Simulation-study)
    - [Application](#Application)
- [EM and Gibbs](#em-and-gibbs)
    - [Gaussian Mixture Model](#gaussian-mixture-model)
    - [EM과 Gibbs 소개](#em과-gibbs-소개)
    - [Simulation](#simulation)
    - [부동산 매매가 분석](#부동산)
    - [결론](#결론)
- [따릉이](#따릉이)
    - [분석 배경](#분석-배경)
    - [모형 설정](#모형-설정)
    - [분석 결과](#분석-결과)
    - [결론](#결론-및-한계점)
- [간호대학 학생들의 학습성과](#간호대학-학생들의-학습성과)
    - [의뢰 내용](#의뢰-내용)
    - [데이터 설명](#데이터-설명)
    - [결과](#결과)
    - [느낀점](#느낀점)

RDA, EM and Gibbs는 코드 파일을 첨부해 놓았지만, 다른 프로젝트들의 코드는 대체로 내장함수와 패키지/라이브러리 내 함수를 이용하여 특별하게 내세울 수 있는 점이 있다고 생각하지 않아 따로 첨부하지 않았습니다.

## RDA

### 분석 목적

일반적인 분류에서의 Discriminant Analysis 방법은 Linear Discriminant Analysis와 Quadratic Discriminant Analysis가 있습니다.
LDA는 분류할 데이터들의 그룹이 같은 분산을 가진다 가정해 선으로 그룹을 분류하는 방법론이며, QDA는 그룹 별로 다른 분산을 가진다는 가정을 하기 때문에 이차함수의 분류선을 가집니다.

하지만 데이터의 차원이 클 때엔, QDA와 LDA 모두 분류에서의 성능이 좋지 않기 때문에 1989년 Friedman이 Regularized Discriminant Analysis, RDA 방법을 고안하였습니다.
저희는 당시 Friedman이 RDA를 고안했던 논문을 참고해 Simulation과 실제 데이터에 대한 분석을 진행하였습니다.

### RDA 소개

RDA는 LASSO와 Ridge의 결합 형태인 Elastic-net 방법론과 유사하게 LDA와 RDA를 결합한 방법론이라고 할 수 있습니다.

LDA의 분류선은 $\sigma_k\left(x\right) = x^T\Sigma^{-1}\mu_k-{1\over 2}\mu^T_{}\Sigma^{-1}\mu_k+log\pi_k$이며

QDA의 분류선은 $\sigma_k\left(x\right) = -{1\over 2}\left(x - \mu_k\right)^T\Sigma_k^{-1}\left(x - \mu_k\right) - {1\over 2}log|\Sigma_k| + log\pi_k$로 쓸 수 있습니다.

최종 분류는 $\sigma_k(x)$를 최소로 하는 추정량들을 구해 대입해 얻은 함수로 분류합니다.

RDA의 분류선은 $\sigma_k\left(x\right) = \left(x - \mu_k\right)^T\hat{\Sigma_k}^{-1}\left(\lambda, \gamma\right)\left(x - \mu_k\right) + log|\hat{\Sigma_k}\left(\lambda, \gamma\right)| - 2log\pi_k$으로 쓸 수 있는데,
여기서 $\hat{\Sigma_k} = {S_k \over W_k}$이며, k번째 그룹의 표본 수를 나타내는 $W_k$와 $S_k = \Sigma_{c(\nu)=k}w_{\nu} \left(X_{\nu}-\mu_k\right)\left(X_{\nu}-\mu_k\right)^T$를 이용하여 각 그룹별 분산을 추정합니다.

Friedman은 또한 RDA에 두 가지의 regularization parameter를 제안하였는데 이 모수들을 조절하여 RDA의 성능을 끌어올릴 수 있습니다.

먼저 $\lambda$로 그룹별 분산을 조절하고 $\gamma$로 shrinkage를 조절할 수 있습니다.

$\lambda$를 포함한 각 그룹별 분산은 $\hat{\Sigma_k }(\lambda) = S_k(\lambda)/W_k(\lambda)$

$S_k(\lambda) = (1 - \lambda)S_k + \lambda S$ 

$W_k(\lambda) = (1-\lambda)W_k + \lambda W$ 이며, 따라서 $\lambda$는 각 그룹의 분산을 조절하는 모수입니다.

$\lambda$가 1이라면 LDA, $\lambda$가 0이라면 QDA 방법이 됩니다.


$\gamma$를 포함해 최종적으로 그룹별 분산 추정량을 $\hat{\Sigma_k}(\lambda, \gamma) = (1-\gamma)\hat{\Sigma_k}(\lambda) + {\gamma \over p}tr\left[\hat{\Sigma_k}(\lambda)\right]I$ 로 쓸 수 있습니다. 여기서 $\gamma$는 $\hat{\Sigma_k}(\lambda)$의 고유값에 곱해지는 것을 볼 수 있는데, 큰 고유값은 줄이고 작은 고유값은 늘리는 역할로 표본을 기반으로 추정한 고유값의 편향을 조절할 수 있습니다.

### Simulation Study

논문 내용을 기반으로 두 가지의 실험을 진행하였습니다.

R의 klaR 패키지를 이용하여 분석하였고, 실험에 쓰인 데이터는 논문의 저자와 동일하게 생성한 데이터이며 표본의 크기는 200, 표본 내 세개의 그룹이 있는 데이터를 생성하였습니다.

각 방법론의 특성을 고려하여 크게 두개의 표본을 생성하였고, 첫번째 표본은 모든 그룹의 분산이 identity matrix로 같은 표본, 두번째 표본은 그룹 별 분산이 k * identity matrix인 표본입니다.

각 실험은 데이터의 차원이 변함에 따라 LDA, QDA, RDA의 분류 성능이 어떻게 변화하는지 알아보기 위해 p가 6, 10, 20, 30일 때로 구분하여 진행하였고 데이터 생성부터 분석까지를 한번으로 하여 총 100번의 반복을 거쳐 구한 평균 오분류율과 RDA를 할 때 선택된 모수의 평균값을 구하였습니다.

LDA와 QDA는 모수를 선택할 필요가 없기에 표에 제시된 조절 모수의 평균값은 RDA에서 선택된 모수입니다.
klaR 패키지의 함수를 이용하여 따로 조절 모수를 지정해주지 않으면 최적의 모수를 선택해 보여줍니다.

먼저 모든 그룹의 분산이 같은 표본에 대한 실험 결과입니다.

![RDA_Simulation1_results](https://github.com/HyunbeenLim/portfolio/assets/133561846/0228eefb-3886-4e7d-9709-866aa9a7c6c3)

이 실험의 결과로 데이터의 차원이 작을 땐 세 가지 방법 모두 어느정도 분류 성능이 비슷한 모습이 보이지만, 데이터의 차원이 커질 수록 QDA의 성능이 안 좋아지는 모습을 볼 수 있었습니다. 이는 QDA가 그룹 별 분산이 다를 때의 분류를 위해 고안된 방법이기에 당연한 것으로 생각할 수 있습니다.

또한 RDA에서 선택된 $\lambda$의 값은 p 값이 커질 수록 점점 1에 가까워지는 모습을 볼 수 있습니다. 

$\lambda$ 값이 1에 가까워질 수록 분류선에 필요한 추정량들이 LDA에서의 추정량과 유사해 지기 때문에 해당 실험으로 알 수 있는 점은 모든 그룹의 분산이 같을 때에 p 값이 커질 수록, 즉 차원이 커질 수록 QDA보단 LDA에 가까운 분류 방법이 효과적이라고 말할 수 있습니다.

다음 실험은 그룹 별 분산이 다를 때 실험 결과입니다.

![RDA_Simulation2_results](https://github.com/HyunbeenLim/portfolio/assets/133561846/5ff3428f-2eaa-4ef0-a1fe-328eb45e2f6f)

두 번째 실험의 결과는 조금 더 의미 있는 해석이 가능하다고 생각합니다.

먼저 QDA는 그룹 별 분산이 다를 때 선형 분류선을 사용하는 LDA의 단점을 보완하기 위해 고안된 방법임에도, 데이터의 차원이 낮을때만 LDA보다 조금 나은 정도인 성능을 보이고 데이터의 차원이 커지면 LDA보다 오히려 좋지 못한 분류 성능을 보이고 있습니다.

이 실험에서도 데이터의 차원이 커질 수록 RDA가 다른 두 방법보다 좋은 분류 성능을 보이는 것을 확인할 수 있었는데, 첫 번째 실험에서는 $\lambda$, $\gamma$ 모두 1에 가까워 졌지만 두 번째 실험에선 $\lambda$가 오히려 0에 가까워지는 모습을 볼 수 있습니다.

이는 그룹 별 분산이 다를 때엔 LDA보단 QDA에 가까운 분류가 효과적임을 알 수 있습니다.
또한 두 실험 모두 데이터의 차원이 커질 때 $\gamma$ 값이 1에 가까워지는 것을 보아, 데이터의 차원이 커질 땐 큰 $\gamma$ 값을 사용해 조금 더 강력한 규제를 가해야 함을 알 수 있었습니다.

### Application

실제 사례에 대한 적용에 사용한 데이터는, 금속 실린더와 암석이 sonar signal에 대해 반응한 주파수대를 써 놓은 데이터 입니다.
독립변수의 개수는 60개이며, 분류할 그룹은 두 그룹입니다.

![RDA_realdata](https://github.com/HyunbeenLim/portfolio/assets/133561846/4171501c-24eb-4c77-94ed-9ea2877a7cc4)

분류를 진행하기 앞서 데이터의 차원이 크기 때문에 QDA, LDA 방법보다 regularization parameter들이 있는 RDA의 성능이 좋을 것이라 예상할 수 있으며, 그룹 별 분산이 같을 것이라 생각되지 않기 때문에 $\lambda$의 값이 1보단 0에 가깝고 $\gamma$의 값은 1에 가까울 것이라 예상하였습니다.

![RDA_real_results](https://github.com/HyunbeenLim/portfolio/assets/133561846/ad598445-704f-457c-b370-28f42c2c9b93)

얻은 결과는 예상대로 RDA의 성능이 좋고 $\lambda$ 값이 1보단 0에 가까운 것을 보이지만, $\gamma$ 값이 0에 더 가까운 모습을 보이고 있습니다.

따라서, RDA는 데이터의 차원이 클 때에 일반적으로 LDA, QDA보다 좋은 성능을 보임을 알 수 있으며 특히나 데이터의 차원이 크면서 그룹 별 분산이 다를 경우에는 LDA는 물론 QDA의 성능마저 떨어지기에 RDA를 선택하는 것이 적절하다고 생각합니다.

물론 데이터의 차원이 작거나, 큼에도 표본의 크기가 클 때, 그룹 별 분산이 별 다르지 않다고 생각될 때엔 LDA 방법을 사용하는 것이 해석에 더 용이하기에 데이터의 특징에 따라 적절한 방법론을 선택할 필요가 있다고 생각합니다.


## EM and Gibbs


### Gaussian Mixture Model

Gaussian Mixture Model은 정규분포가 결합된 형태로, 분석할 데이터의 분포가 여러개의 mode를 가질 때에 가정하는 분포입니다.

저희는 해당 분포를 이용해 EM 알고리즘과 Gibbs sampling의 모수 추정(사후 분포 추정) 성능을 비교하고자 하였습니다.

### EM과 Gibbs 소개


EM 알고리즘은 일반적으로 mode-finding 알고리즘의 한 종류이며 가능도함수를 최대화하는 모수의 추정량을 구하는 알고리즘입니다.

먼저 simulation에 활용할 데이터는 Gaussian mixture model에서 추출하였으므로 EM 알고리즘의 추정을 위해선 GMM 분포의 확률밀도 함수 $P(x) = \Sigma_{c=1}^K \pi_cN(x|\mu_c, \sigma_c)$를 알 필요가 있습니다. 여기서 $\pi_c$는 데이터에 K개의 그룹이 있을 때 그룹 c가 해당 데이터에서 차지하는 비율이며, 관측되지 않는 변수입니다.

확률밀도함수를 다시 로그가능도함수로 바꾸어 준다면 $loglikelihood = \Sigma_{i = 1}^n \Sigma_{c = 1}^K I(z_i = c)[log\pi_c + log N(x_i|\mu_c, \sigma_c)]$가 됩니다.
여기서 $z_i$는 i번째 관측치가 group c에 속할 때 1이 되는 확률변수($p(z_i = c) = \pi_c$)입니다.

다음은 EM 알고리즘의 E-step입니다.

변수 z가 missing data로 다뤄질 수 있기 때문에 $z|x$에 대한 조건부 기댓값을 먼저 구해줍니다.

$E_{z|x}(loglikelihood) = \Sigma_{i = 1}^n \Sigma_{c = 1}^K E_{z|x}\left(I(z_i = c)\right)[log\pi_c + log N(x_i|\mu_c, \sigma_c)] = \Sigma_{i = 1}^n \Sigma_{c = 1}^K p(z_i = c|x)[log\pi_c + log N(x_i|\mu_c, \sigma_c)]$가 됩니다.

이 식에서 $p(z_i = c|x)$를 $p(z_i = c|x) = {p(z_i = c,x) \over p(x)} = {p(z_i = c) p(x|z_i = c) \over p(x)} = {\pi_c N(x_i|\mu_c \Sigma_c) \over \Sigma_{c=1}^K \pi_c N(x_i|\mu_c, \sigma_c)} = \gamma_{ic}$로 정의하겠습니다.

따라서 $E_{z|x}(loglikelihood) = \Sigma_{i = 1}^n \Sigma_{c = 1}^K \gamma_{ic}log\pi_c + \Sigma_{i = 1}^n \Sigma_{c = 1}^K \gamma_{ic}logN(x_i|\mu_c, \sigma_C)$가 됩니다.

E-step으로 구한 식을 라그랑지 방법을 이용해(분량 상 생략하겠습니다.) 최대로 만드는 추정량을 구하면

![EM_추정](https://github.com/HyunbeenLim/portfolio/assets/133561846/84bf8a59-75e5-4f28-b026-84baaa5cd596)

이런 결과를 얻게 됩니다.


Gibbs Sampling 방법은 베이지안 방법의 하나로, 사후분포를 추정합니다.
베이지안 방법은 사전분포에 대한 가정이 필요하므로 저희는 좀 더 검증된 사전분포를 이용하는 것이 적절하다 판단되어, Schnatter(2006)의 책을 참고하였습니다.

GMM에서 유의할 사항은 improper prior를 부여할 시 posterior 또한 improper density가 되어 추정에 어려움이 따르기 때문에 Conjugate, Independent Prior를 부여하여 추정하는 대신 사전분포의 영향이 약하지게 모수를 가정하여 추정하였습니다.

이하 책을 참고한 사전분포와 그에 따른 .

![Gibbs_para](https://github.com/HyunbeenLim/portfolio/assets/133561846/87977dd1-f461-41eb-8c46-3a12b80f3161)

이며, 여기서 S는 EM에서의 변수 z와 같습니다.


### Simulation

다음 실험입니다. 

실험에 사용한 코드는 저희가 R에서 직접 작성한 코드를 사용하였고 데이터 생성은 R의 distr 패키지를 사용했습니다. 수렴 기준은 1e-4로 하였습니다.

실험에 사용한 데이터는 크게 두가지입니다.

#### Overlapped 

첫번째 데이터는 각 분포들이 overlap되지 않은, 즉 세 그룹이 잘 분리가 되어 데이터에 몇 개의 그룹이 존재하는지 잘 파악할 수 있는 데이터입니다.


아래는 생성한 데이터의 빈도 그래프와 데이터 생성시 설정한 모수값입니다.

![em_gibbs_data1](https://github.com/HyunbeenLim/portfolio/assets/133561846/8518407d-16f8-4f5c-b777-c775c0dbf6d4)


추정하기 위해 설정한 초기치는 다음과 같습니다.

![em_gibbs_sim1](https://github.com/HyunbeenLim/portfolio/assets/133561846/59b03d76-f366-4d18-a003-d001da86273e)


해당 초기치를 바탕으로 저희가 작성한 EM 알고리즘과 Gibbs sampling 코드를 실행하였을 때 얻은 결과값입니다.

![em_gibbs_sim1_re](https://github.com/HyunbeenLim/portfolio/assets/133561846/e3c4b0b0-dd7b-4eab-8e66-7860cf3bd606)

위와 같이 잘 분리가 되어 있는 분포를 따르는 데이터에 대해서, EM 알고리즘과 Gibbs Sampling의 결과는 거의 유사한 모습을 보이는 것을 알 수 있습니다.


또한 안정적인 추정이 되었는지 확인하기 위해 추정 과정에서의 각 iteration 별 추정량 그래프를 그려 보았습니다.

![sim1_conv](https://github.com/HyunbeenLim/portfolio/assets/133561846/892103ce-cd2e-48cf-868d-c6741f46ad18)


보시는 바와 같이 Gibbs sampling에서의 sampling이 안정적인 것을 확인할 수 있습니다.


마지막으로 오분류율(%)를 구하였습니다. 오분류율은 EM 알고리즘에선 최종 추정된 $\gamma_{ic} = p(z_i = c|x)$값을, Gibbs Sampling에선 사후분포 기반 확률을 이용하였습니다. 

EM 알고리즘은 group1에 대해 10.33 %, group2에 대해 3.67%, group3에 대해 12.33%를 보였으며

Gibbs Samlping은 group1에 대해 15.84 %, group2에 대해 6.53%, group3에 대해 17.75%를 보여 분리가 잘 된 분포를 따르는 데이터에 대해선 EM 알고리즘의 성능이 더 좋은 것을 확인할 수 있었습니다.

더 정확한 비교를 위해 데이터 생성부터 추정까지 100번을 반복해 얻은 추정량의 bias와 mse의 평균을 구해보았습니다.

![sim1_mc](https://github.com/HyunbeenLim/portfolio/assets/133561846/90a02ca9-d062-4155-b58e-51e0521678c9)

상단의 표는 크기가 100인 데이터를 사용해 얻은 결과이며 하단의 표는 크기가 100인 데이터를 사용해 얻은 값입니다.

표본에 크기에 상관 없이 bias와 mse 모두 작은 값인 것을 보아 분리가 잘 되어 있는 데이터에 대해선 두 방법의 성능 차이를 논하는 것은 큰 의미가 없다고 생각할 수 있겠습니다.


#### non-overlapped 

다음 두번째 데이터는 데이터가 서로 overlap 되어 mode가 하나인 것처럼 보이는 데이터입니다. 
데이터의 빈도 그래프와 생성할 때 입력한 값입니다.

![data2](https://github.com/HyunbeenLim/portfolio/assets/133561846/2933ccc5-686a-433a-b344-4bbf881df094)
보시는 바와 같이 overlap 된 정도가 심해 하나의 분포를 따르는 데이터처럼 보이는 것을 알 수 있습니다.

추정을 위한 초기치는 이전 실험에 쓰인 값들과 동일하며 코드를 한번 씩 실행했을 때의 결과값입니다.

![sim2](https://github.com/HyunbeenLim/portfolio/assets/133561846/c872152d-ec07-4216-9352-ba58258346c0)

위와 같이 겹친 정도가 심한 데이터는 그렇지 않은 데이터에 비해 추정값이 실제값과 거리가 굉장히 먼 것을 확인할 수 있었습니다. 특히나 두번째 그룹에 대해 추정이 불안정한 것을 알 수 있습니다.


이 실험에서도 수렴 그래프를 그려보았습니다.

![sim2_conv](https://github.com/HyunbeenLim/portfolio/assets/133561846/5ff88df2-5e40-446b-bb98-72015064cdaa)

그래프를 본다면, Gibbs sampling에서의 사후분포의 모수가 iteration 별로 굉장히 불안정한 것을 확인할 수 있었습니다.

이때의 오분류율은 EM 알고리즘은 group1에 대해 19.33%, group2에 대해 100%, group3에 대해 10.33%를,

Gibbs sampling은 group1에 대해 47.50%, group2에 대해 58.48%, group3에 대해 45.67% 를 보여 오분류율이 굉장히 증가한 것을 확인할 수 있었습니다.


다시 더 정확한 결과를 위해 100번 반복한 결과를 확인해 보겠습니다.

![sim2_mc](https://github.com/HyunbeenLim/portfolio/assets/133561846/af85518b-dd47-4821-b322-1feb023eccc6)

여기서 깁스 샘플링 방법은 샘플 사이즈가 커지면서 큰 성능 향상을 보인 것에 반해 EM 알고리즘은 샘플 사이즈에 상관 없이 평균 mse 값이 매우 큰 것을 확인할 수 있었습니다.

따라서, 데이터가 overlap 된 정도가 적다면 EM 알고리즘이 Gibbs sampling 보다 조금 더 낫거나 비슷한 성능을 보이고 overlap 된 정도가 크고 표본의 개수가 많다면 Gibbs sampling 방법의 성능이 월등하다는 것을 알 수 있습니다.


### 부동산 

다음은 실제 사례에 대해 적용을 해 보았습니다.

저희가 작성한 코드는 1차원 데이터에 대한 군집화를 하기 위해 만들어졌으므로, 1차원 데이터만으로도 유의미한 결과를 얻을 수 있는 분석은 집 값 데이터 분석이라고 생각하였습니다.

저희가 가져온 데이터는 크기가 44810인 2021년 서울 시 아파트 매매가 데이터이고, 사용할 정보는 제곱 미터 당 가격(만원)과 해당 아파트의 자치구 명입니다.

![application_data](https://github.com/HyunbeenLim/portfolio/assets/133561846/49e7627a-5037-46ad-aff6-5e32a6d13161)

첨부한 사진의 왼쪽에서 데이터의 대략적 정보를 확인할 수 있고, 오른쪽 그래프를 통해 데이터의 분포 그래프를 알 수 있습니다.
데이터의 분포가 오른쪽 꼬리가 긴 정규 분포의 형태로 보이기에 저희는 이것을 근거로 데이터가 세 그룹을 가진 GMM을 따른다는 가정을 하는 것이 타당하다고 생각하였습니다.

분석을 하기 위해 사용한 초기치는 세 그룹 모두 동일하게 평균이 1000, 분산이 200, 표본 내 그룹의 비율 ${1 \over 3}$으로, 사전분포의 모수는 $e_0 = 4, b_0 = 1000, B_0 = 1000000, c_0 = 1, C_0 = 1$으로 설정하였습니다.

![app_res](https://github.com/HyunbeenLim/portfolio/assets/133561846/b17b9848-97d7-4476-9cf6-6b622e54d56e)

얻은 결과를 바탕으로 그룹1, 그룹2, 그룹3 순서로 싼 가격에 매매된 아파트 그룹이라는 것을 알 수 있었고 추정한 9개의 모수 모두 두 방법론이 거의 동일한 값을 가진 것을 확인할 수 있었습니다. 

![app_conv](https://github.com/HyunbeenLim/portfolio/assets/133561846/1bde6a92-d339-4e0e-ab87-ec40e1db3446)

추정 과정을 그린 그래프 또한 Gibbs sampling 방법의 샘플링이 굉장히 안정적인 것을 확인할 수 있습니다.

![app_em_est_den](https://github.com/HyunbeenLim/portfolio/assets/133561846/b1c90e69-734b-42e4-9dcf-93304b4b019e)
EM 알고리즘으로 추정한 모수를 바탕으로 그린 분포 그래프와 그 합 그래프입니다.

빨간색 그래프가 세 분포를 합친 그래프인데, 실제 데이터의 분포와 유사한 것을 보입니다.

![app_gibbs_est_den](https://github.com/HyunbeenLim/portfolio/assets/133561846/ca28a906-f599-49d1-a5c6-9d6f4379887d)
Gibbs sampling의 그룹 별 사후 분포와 그 합 그래프입니다.

Gibbs sampling 또한 빨간 그래프가 실제 데이터의 분포와 유사합니다.


가져온 데이터의 표본의 크며 분포 그래프만을 그려보았을 때와는 다르게 overlap 된 정도가 적기 때문에 좋은 결과를 얻었다고 결론을 낼 수 있겠습니다.

마지막으로 EM 알고리즘의 E-step에서 정의된 $\gamma_{ic} = p(z_i = c|x)$를 활용하여 새로운 그래프를 그려보았습니다.

![app_gam_graph](https://github.com/HyunbeenLim/portfolio/assets/133561846/08894ac0-7eab-48f4-8a6e-38ebf6075aa5)

먼저 그래프는 분석을 통해 군집화를 완료한 후, 다시 각 데이터에 맞는 자치구를 인덱싱해 같은 자치구 별로 평균을 구한 확률 그래프입니다. 

1그룹 맨 위 도봉구가 위치하고 있는데 이는 도봉구에서 2021년 매매된 아파트 중 거의 90%가 1그룹, 즉 가장 싼 가격에 매매된 아파트 그룹에 속한다고 해석할 수 있겠습니다.

이 그래프를 통해 새로운 해석이 가능한데, 양천구으 경우 서울 시에서 평균 집 값이 평균을 웃도는 자치구임이 알려져 있습니다. 하지만 그래프에선 양천구의 50% 이상의 아파트가 1그롭, 40% 정도의 아파트가 2그룹에 속하고 10%의 아파트만이 3그룹에 속해 있는 것을 확인할 수 있겠습니다. 3그룹이 전체에서 차지하는 비율을 생각해 보았을 때, 양천구의 평균 집 값은 서울 시 평균이거나 그 이하여야 마땅하지만, 양천구에 속한 목동의 집 값이 비싼 것임이 알려져 있는 것을 감안했을 때, 목동의 집 값이 서울 시 내에서도 아주 비싼 편이며 이 것이 양천구의 평균 매매가 순위를 올려 놓았다라고 생각할 수 있겠습니다.

![asasd](https://github.com/HyunbeenLim/portfolio/assets/133561846/c62261d3-e3aa-4240-a566-deba602575b1)
2021년 구별 아파트 평당 매매가

### 결론

이번 연구에서 GMM 분석에서 흔하게 사용되는 EM 알고리즘과 Gibbs sampling의 추정에 있어 고려할 사항과 문제점을 중심으로, 두 방법론의 비교를 시뮬레이션과 실제 사례 적용을 통해 분석하였습니다.

우선 시뮬레이션에서 분석 결과엔 넣지 않았지만 overlap 된 정도가 큰 데이터에 대해서 labelling 문제가 발생하는데, 이 문제는 그룹 추정에서 그룹 1에 대한 추정치를 그룹2나 그룹3으로 labelling 하는 문제입니다.
이 문제는 $Chung$ $et.$ $al$(2004)에서 언급된 문제이며, 해당 논문에서 제시한 모수 공간에 제약을 거는 방식인 permutation 방법을 통해서 어느정도 해결하는 모습을 보일 수 있었습니다. 

또한 실제 사례 적용 결과를 통해 두 방법 모두 좋은 결과를 보이는 것을 확인하여 임의의 분포를 GMM으로 잘 근사할 수 있음을 확인할 수 있었습니다. 동시에, 양천구의 사례와 같이 각 구 별 평균 실거래가 등의 기술통계로는 알 수 없는 정보를 잘 파악할 수 있음을 알 수 있었습니다.

## 따릉이

### 분석 배경

서울 시 자전거 대여 서비스 따릉이의 이용자 수가 꾸준이 증가해 2021년엔 누적 이용량이 1억건을 돌파했다는 결과가 있습니다.

하지만 사용량이 늘어남에 따라 연간 적자가 계속 증가하여 2021년 기준 103억원에 달하며 대여소에 자리가 없음에도 반납이 가능한 특징 탓에 무질서한 주차로 인한 불만이 늘어나고 있습니다.

따라서 따릉이의 대여와 반납의 불균형이 어떤 원인으로 발생하는지 알아보고, 해결방안을 제안하고자 분석을 하게 되었습니다. 

먼저 대여소 별 어느 대여소에서 가장 많은 따릉이가 대여/반납 되었는지 확인해 보았습니다.

![대여](https://github.com/HyunbeenLim/portfolio/assets/133561846/674d5e87-189c-4781-94e2-08d877fdfe52)
![반납](https://github.com/HyunbeenLim/portfolio/assets/133561846/ac26c590-992e-4ddf-a2f1-abd1b0765ce1)

대여와 반납 모두 뚝섬유원지역 대여소 가장 많았고 여의나루역 대여소가 그 뒤를 이었습니다.

두 대여소는 모두 한강 근처 대여소이기에 따릉이의 이용이 가장 활발한 것이라 생각할 수 있습니다.

또한 대여소 별로 대여와 반납의 불균형 문제를 확인하기 위해 대여수에서 반납수를 빼 비교해 보았습니다.

![반납-대여](https://github.com/HyunbeenLim/portfolio/assets/133561846/4ee68e95-4b2c-471b-a15a-67b277d5a325)

대여수 대비 반납의 수가 가장 많았던 대여소는 뚝섬유원지역이었으며, 반납된 따릉이가 2738대나 많았음을 확인할 수 있습니다.

![대여-반납](https://github.com/HyunbeenLim/portfolio/assets/133561846/f3218c55-461f-4c18-8e96-31dbb6db7451)

여의나루역의 대여소에서 반납보다 대여소가 975대가 많아 25번째에 위치해 있는 것을 확인할 수 있었습니다.

두 대여소 모두 한강 근처 대여소이지만 상반된 결과를 보이고 있기 때문에 대여수 - 반납수의 값이 왜 정반대의 결과를 보이는지 알아보고자 하였습니다.

해당 연구는 파이썬으로 진행하였습니다.

### 모형 설정

분석에 사용한 데이터는 계절에 따른 이용량의 변화를 반영하지 않기 위해 2020-2022 3개년도의 6월 따릉이 이용자료를 사용하였습니다.

기상정보로 일별 기온, 강수량, 풍속 정보를 사용하였으며 이용 날짜의 요일, 공휴일 여부에 대한 자료를 사용하였습니다.

저희는 분석에 앞서 아래 네 가지 로지스틱 회귀 모형을 설정하였으며, 데이터의 불균형이 심한 것을 우려해 Y = 1 : Y = 0의 비율이 3:10이 되도록 임의추출하였습니다.

설명 변수로는 대여시각, 반납시각, 이용시간, 이용거리, 강수량(mm), 기온, 평균풍속(m/s), 대여요일, 공휴일 여부로 9가지를 사용하였습니다.

![모형](https://github.com/HyunbeenLim/portfolio/assets/133561846/cd66b0a5-b393-4b59-80ad-afb049edb976)

분석에 사용한 모형은 GAM, GLM 그리고 penalized GLM 3가지이며 GAM과 GLM을 통해 각 변수별 중요도의 차이를 비교하고 penalized GLM으로 변수 선택의 결과를 살펴보았습니다.

먼저 GLM(Generalized Linera Model)은 조건부 평균의 함수를 다음과 같이 각 변수들의 선형결합으로 표현합니다.

$g\left(E\left[Y|X\right]\right) = \beta_0 + \beta_1X_1 + \cdots + \beta_pX_p$

여기서 항등함수 $g(x) = x$를 사용하면 선형회귀 모형이 되며 로짓함수 $g(x) = log\left({p \over (1 - p)}\right)$를 사용하면 로지스틱 회귀 모형이 되어 로짓함수를 사용하였습니다.

일반선형회귀처럼 다른 변수들을 통제한 채 한 변수의 효과를 파악할 수 있는 장점을 GLM 역시 가지고 있습니다.


다음 GAM(Generalized Additive Model)은 다음과 같이 조건부 평균의 함수를 각 변수의 함수들의 합으로 표현합니다. 선형 결합으로 표현한 GLM과 달리 비선형 함수를 사용할 수 있어 모델이 표현할 수 있는 범위가 넓어지며 GLM과 같이 다른 변수를 통제하고 한 변수의 효과를 파악할 수 있다는 장점이 있습니다.

$g\left(E[Y|X]\right) = \beta_0 + f(X_1) + \cdots + f(X_p)$

GLM과 동일하게 g에 로짓함수를 사용했습니다.


마지막으로 GLM, GAM을 통한 분석 결과로 각 모형에서 유의미한 결과를 얻은 변수들을 비교하기 위해 GLM에 계수에 대해 penalty를 부과하여 GAM과 비교하였습니다.

대여요일 및 공휴일 여부를 나타낸 변수들은 더미 변수형태로 모형에 사용되기 때문에 변수들이 각각의 그룹을 형성합니다. 이러한 정보들이 만약 필요가 없다면 모두 선택되지 않아야 하므로 그룹 정보를 잘 포착할 수 있는 elastic-net penalty를 사용하였습니다.

$\beta^* = argmin_{\beta}\left[g\left(E[Y|X]\right) - \beta_0 - \Sigma_{i=1}^p \beta_i X_i\right] + \lambda \left[\alpha_1 \Sigma_{i=1}^p ||\beta_i|| + \alpha_2 \Sigma_{i=1}^p ||\beta_i||^2 \right], \alpha_1 + \alpha_2 = 1$

초모수 $\lambda, \alpha_1$의 선택은 k-fold cross-validation을 진행하였고, train : test = 7 : 3으로 나누어 train 데이터를 10개의 fold로 나누었습니다.

이후 Grid search를 실시하기 위해 사용한 초모수의 범위는 다음과 같습니다.

$\lambda^{-1} : \{0, 0.001, 0.002, \cdots, 0.1 \}, \\ \alpha_1 = \{0.5, 0.7, 0.9\}$



### 분석 결과

penalized GLM을 진행하기 앞서 변수들을 정규화하는 작업을 선행하였고 추정된 회귀 계수의 절대값이 0.2 보다 큰 변수를 로짓값에 대하여 유의미한 영향을 주는 변수라고 간주하였습니다.

분석의 결과는 여의나루역 대여소의 대여 결과와 반납 결과를 먼저, 이후 뚝섬유원지역 대여소의 대여 결과와 반납 결과를 이어서 보여드리도록 하겠습니다.

#### 여의나루역
먼저 여의나루역의 대여에 대한 분석 결과입니다.

![여의나루_대여](https://github.com/HyunbeenLim/portfolio/assets/133561846/9fe7c0ec-0cb7-4c0e-9458-c07c130ef780)

GLM 분석 결과 이용시간이 유의미한 변수로 나타났습니다.
계수 추정치는 -0.017로 이용시간이 1분 증가할 수록 여의나루역 대여소에서 따릉이를 대여했을 확률의 오즈가 exp(-0.017) = 0.983배 감소한다는 것을 의미합니다. 이를 통해 여의나루역 대여소에서 따릉이를 대여하는 경우 다른 대여소에서 대여하는 경우에 비해서 짧은 시간만 이용하는 것을 알 수 있습니다.

penalized GLM의 결과, 초모수 $\lambda = {1\over 0.071}, \alpha_1 = 0.7$이 선택되었고 이때 선택된 변수는 이용시간 변수입니다. 다른 변수들은 완전히 0으로 shrink되었거나 0에 가까워진 모습을 보입니다.

GAM 분석 결과, p-value를 기반으로 대여시각, 이용시간 그리고 이용거리가 유의미한 변수로 나타났으며 유의미한 변수로 나타났습니다.

![gam_여의대여](https://github.com/HyunbeenLim/portfolio/assets/133561846/33807b03-684d-4505-a336-a458ee556134)

위의 사진으로 GAM에서 추정된 변수 별 함수를 확인할 수 있는데, 대여시각 로짓에 대한 함수를 보면, 오후 1시와 2시 사이를 기점으로 대여확률이 감소하다 증가하는 형태를 보이고 있습니다. 이는 여의나루역 대여소의 따릉이는 출근 시간과 퇴근 시간에 주로 이용되며 점심시간에는 이용자가 줄어드는 것이라 해석할 수 있겠습니다.

다음 이용거리의 함수는 중위수에 가까울 수록 대여 확률이 높아지는 것을 확인할 수 있습니다. 이는 여의나루역 대여소에서 따릉이를 대여하는 이용자들은 너무 짧거나 너무 먼 거리를 이동하지 않는다는 것을 의미한다고 할 수 있겠습니다.

마지막으로 이용시간의 함수를 보면 단조 증가 함수의 형태를 띄는데, 이는 여의나루역에서 따릉이를 대여하는 사람은 오랜 시간 대여하는 경우가 많다는 것을 의미하게 되어 GLM에서의 해석과 반대되는 결과를 보입니다.

GAM에서 다른변수들로부터 비선형적 영향을 받았거나 변수들간의 상관관계가 커 GLM의 계수 추정이 올바르지 않게 되었을 가능성이 있습니다.


그 다음 여의나루의 반납에 대한 분석 결과입니다.

![여의나루_반납](https://github.com/HyunbeenLim/portfolio/assets/133561846/8bf2df6f-5b28-4f0d-afaf-64fec4a8ee91)

GLM 분석 결과 평균기온과 이용시간, 이용거리가 유의미한 변수로 나타났습니다.
계수 추정치를 기반으로 기온이 1도 상승함에 따라 여의나루 대여소에 반납할 확률의 오즈가 exp(-0.0881) = 0.916배 감소하며 이용시간이 1분 증가하면 반납 확률의 오즈가 exp(-0.019)배 감소한다고 할 수 있습니다. 또한 이용거리가 1m 증가한다면 exp(-0.0001)배 만큼 감소한다고 할 수 있겠습니다.

penalized GLM의 결과 $\lambda = {1 \over 0.016}, \alpha_1 = 0.5$가 선택되었고 이용시간과 이용거리 변수를 제외하면 나머지 변수들은 0으로 shrink된 모습을 보입니다.

다음 GLM에서 유의미한 변수는 이용시간과 이용거리 변수입니다.

![gam_여의반납](https://github.com/HyunbeenLim/portfolio/assets/133561846/86f0ac29-daa2-46b0-a784-0bec330f5cfe)

GAM 함수 추정 결과입니다 이용시간과 이용거리 변수가 유의미하게 다나타났습니다.

이용시간의 함수를 본다면, 가파르게 증가하다 완만한 감소를 하는 형태를 보이고 있습니다.
이는 여의나루 대여소에 반납된 따릉이들은 1.5시간 이상 이용된 경우가 많음을 의미합니다.

다음 이용거리의 함수는 증가함수 이므로 가까운 거리보다 먼거리를 이동한 이용자들이 여의나루 대여소에 반납을 하는 것으로 이해할 수 있겠습니다.

GLM에서 선택된 평균기온 변수가 GAM에선 유의미하지 않은 것과 이용거리 변수의 해석에서 다시 한번 반대된 모습을 보이는 것을 확인할 수 있습니다.

#### 뚝섬유원지역
다음 뚝섬유원지역의 대여에 대한 분석입니다.


![뚝섬_대여](https://github.com/HyunbeenLim/portfolio/assets/133561846/7ddbcf0e-9fde-4d0f-a2e0-a36a8cd1fa90)

GLM 분석 결과 이용시간 및 이용거리가 유의미한 변수로 나타났습니다. 이용시간이 1분 증가할때 뚝섬유원지역에서 대여할 확률의 오즈가 exp(-0.00012)배로 소폭 감소하며 이용거리가 1m 증가할 때 오즈가 exp(0.000075)배로 소폭 증가합니다 

penalized GLM의 결과 $\lambda = {1\over 0.052}, \alpha_1 = 0.7$이 선택되었으며 이용시간과 이용거리 변수를 제외하고 0이거나 0에 아주 가깝게 shrink 되었습니다.

![gam_뚝섬대여](https://github.com/HyunbeenLim/portfolio/assets/133561846/a3d40ccf-baf6-4756-b635-31a37290f0e1)

GAM 추정 결과입니다 반납시각, 이용시간, 이용거리 및 강수량이 유의미한 변수로 나타났습니다.

반납시각의 함수를 본다면 점심시간보다는 아침이나 저녁시간에 대여하는 경우가 많은 것을 확인할 수 있습니다.

이용시간의 함수는 증가함수로 이곳에서 대여된 따릉이는 비교적 오래 이용될 것으로 추정되며, 이용거리의 함수는 중위수에서 완만한 모습을 보여 짧은 거리보단 비교적 먼 거리에 이용된다는 것을 알 수 있습니다.

강수량의 함수는 감소함수로, 우리의 직관과 일치하는 결과를 보입니다.

이 경우에도 이용시간에서의 해석이 GAM과 GLM이 반대되는 결과를 보입니다.


다음은 뚝섬유원지역 반납에 대한 분석 결과입니다.

![뚝섬_반납](https://github.com/HyunbeenLim/portfolio/assets/133561846/702dd03b-b7d5-4f8d-97bf-392b4e5e9eff)

GLM 분석 결과 대여시각, 이용시간, 이용거리 변수가 유의미한 변수로 나타났습니다.
세 회귀 계수 모두 한 단위 증가할 수록 오즈가 소폭 감소함을 의미하고 있습니다.

penalized GLM의 결과 $\lambda = {1 \over 0.08}, \alpha_1 = 0.5$가 선택되었고 다른 경우와는 다르게 반납시각과 강수량 변수만 0으로 shrink 되어 대부분의 변수가 선택된 것을 확인할 수 있습니다.

![gam_뚝섬반납](https://github.com/HyunbeenLim/portfolio/assets/133561846/0ac11200-a314-4406-8209-8eadcfe3e2e9)

GAM 추정 결과입니다. 이용시간, 이용거리, 요일 변수가 유의미한 변수로 나타났습니다.

이용시간, 이용거리 함수는 단조 증가 함수 형태를 보이고 있으며 이곳에서 반납된 따릉이는 비교적 오랜시간 긴 거리에 이용된 따릉이임을 알 수 있습니다.

또한 요일변수에서 주말에 반납되는 따릉이가 많은 것을 알 수 있는데, 이는 뚝섬유원지에 방문하는 이용자들이 주말에 많다는 것을 알 수 있습니다. 따라서 이 부분에서 대여와 반납의 불균형이 이루어지고 있음을 알 수 있습니다.

여기서도 이용시간, 이용거리 변수에서 GLM과 GAM의 해석의 차이가 있었습니다.

### 결론 및 한계점 

GLM에서 변수들 간의 다중공선성으로 인해 GAM, penalized GLM과 다른 계수 추정이 이루어졌습니다. 또한 penalized GLM에서 계수의 절대값이 0.2보다 큰 변수들이 이용시간이나 이용거리만 존재해 여의나루역과 뚝섬유원지역의 차이점을 추론하기에 어려움을 겪었습니다.

따라서 GAM 모형을 통해 얻은 유의미한 변수들을 살펴본다면

![GAM](https://github.com/HyunbeenLim/portfolio/assets/133561846/797b4b92-70d3-4137-b80a-00d21637a0a3)
인 것으로 나타났습니다.

여의나루역과 뚝섬유원지역의 대여에서 차이가 나는 유의미한 변수들은 대여시각과 반납시각입니다. 하지만 두 변수의 그래프가 두 곳 모두 U자 형태의 모양을 띄고 있기 때문에 두 역의 차이점을 도출해내기엔 어려움이 있었습니다.

주목할만 한 차이점은 뚝섬유원지역 반납여부에 대한 변수로서 대여요일이 유의미하다는 것입니다. 뚝섬유원지역의 GAM 대여요일 그래프를 살펴보면 평일인 경우 모두 로짓값에 음의 영향을 끼치지만 토, 일의 경우 로짓값을 크게 상승시키는 것을 볼 수 있습니다.

여의나루에 반납하는 경우 대여요일이 유의미한 변수는 아니지만 비교를 해본다면 평일엔 로짓값에 양의 영향을, 주말엔 음의 영향을 끼치는 것을 알 수 있어 두 곳의 차이를 확인할 수 있었습니다.

결과적으로 여의나루역에 반납되는 따릉이는 출퇴근용으로 평일에 이용되는 경우가 많지만 뚝섬유원지에 반납되는 따릉이는 주말 여가 활동을 위해 이용되는 경우가 많은 것으로 추측할 수 있습니다.


아쉬운 점은 두 곳을 뚜렷하게 구분할 변수가 부족했기에 두 지역간의 차이가 모델에 제대로 반영되지 못한 점입니다. 

다양한 변수의 영향력을살피기 위해 많은 변수를 추가한 것이 오히려 다중공선성을 불러와 GLM에 악영향을 끼치게 된 것과 이용자의 개인 정보가 마스킹되어 있어 나이와 성별 정보를 활용하지 못한 점도 아쉬웠습니다.

## 간호대학 학생들의 학습성과

### 의뢰 내용

모 대학교 간호학과 대학원에 재학 중인 학생 분이 의뢰를 주셔서 진행한 프로젝트입니다.

연구의 목적은 간호대학생들의 자기결정성 학습동기, 학업적 자기효능감, 학습전이 그리고 학습성과 간의 관계를 알아보고, 학습성과에 미치는 영향을 확인하고자 하는 것이였으며, 간호대학생들 1학년 ~ 4학년 학생들을 대상으로 한 설문지를 바탕으로 분석을 진행하였습니다.

SPSS를 이용해 분석하는 것을 원하셨으므로 SPSS를 통해 분석을 진행하였습니다.

설문지의 질문은 선행연구들에서 타당성이 입증된 4가지 항목들(자기결정성 학습동기, 학업적 자기효능감, 학습전이 그리고 학습성과) 총 78 문항에 더해 응답자의 일반적 특성을 알 수 있는 10 문항으로 이루어져 있었습니다.

의뢰의 내용은 크게 

1. 일반적 특성에 대한 빈도 분석과 같은 기술 통계
2. 자기결정성 학습동기, 학업적 자기효능감, 학습전이, 학습성과 변수간의 상관분석
3. 일반적 특성에 따른 위 네 변수의 평균 분석
4. 자기결정성 학습동기, 학업적 자기효능감, 학습전이, 일반적 특성이 학습성과에 미치는 영향을 알아보기 위한 다중회귀분석

입니다.

일반적 특성은 범주형 변수이며 다른 네 항목은 5점 ~ 6점 리커트 척도로 구성되어 있어 각 항목 별 응답자의 점수 평균을 내어 분석을 진행하였습니다.

5점과 6점의 차이가 크지 않을 뿐더러 섣부른 표준화는 오히려 분석 결과에 악영향을 끼칠 가능성이 있어 리커트 척도에 대한 표준화는 진행하지 않았습니다.
유의할 점은, 몇 가지 문항이 역문항이라 역코딩 과정을 거쳐 점수를 뒤바꾸어 주는 과정이 있었다는 것입니다.

또한 일반적 특성을 제외한 네 항목 중 자기결정성 학습동기와 학업적 자기효능감 변수는 각 각 5개, 3개의 하위요인을 가졌기에 신뢰도 분석을 진행할 때에 하위요인을 가진 해당 두 항목은 하위요인 별로 신뢰도 분석을 진행하였습니다.

일반적 특성은 응답자의 성별, 학년, 평점평균, 입학동기, 전공만족도, 대인관계만족도, 선호수업방식, 비대면수업만족도, 대면수업이해도, 비대면수업이해도를 알 수 있는 문항들로 이루어져 있습니다.

### 데이터 설명

자기결정성 학습동기(30문항) - 학교에서 새로운 학습을 시작하기 위한 원동력을 제공하고 이미 학습한 행동들을 조절하고 통제하는 내적 기제

자기결정성 학습동기의 하위 요인 5가지(각 6문항) - 무동기, 외적조절동기, 내사된 조절동기, 확인된 조절동기, 내재된 동기

학업적 자기효능감(26문항) - 학습자가 학업적 상황에서 과제를 수행하기 위하여 필요한 행위를 조직하고 실행해 나가는 자신의 능력에 대한 판단하는 능력

학업적 자기효능감의 하위요인 3가지 - 과제수준선호 8문항, 자기조절효능감 10문항, 자신감 8문항

학습성과(16문항) - 교육을 통해 성취해야 하는 궁극적인 목표이자 결과

학습전이(6문항) - 학습내용을 현업에 적용하여 업무성과를 개선하는 활동으로써 교육훈련 효과성을 결정함

일반적 특성
1. 성별: 여자/남자
2. 학년: 1학년/2학년/3학년/4학년
3. 평점평균: 2.5이하/2.5-3.0/3.0-3.5/3.5-4.0/4.0 이상
4. 입학동기: 적성/성적/취업률/주변권유/간호사동경/사회봉사/기타
5. 전공만족도: 만족/보통/불만족
6. 대인관계만족도: 만족/보통/불만족
7. 선호수업방식: 강의식/토론식/사례기반/문제기반/기타
8. 비대면수업만족도: 만족/보통/불만족
9. 대면수업이해도움: 매우그렇다/그렇다/보통이다/그렇지않다/전혀그렇지않다
10. 비대면수업이해도움: 매우그렇다/그렇다/보통이다/그렇지않다/전혀그렇지않다


### 결과

결과에 앞서, 상관분석과 평균분석 등을 진행하기 전에 데이터가 정규성 가정을 만족하는지 확인해야 할 필요가 있습니다.

따라서 저희는 Shapiro-wilk Test로 정규성 검정을 먼저 시행하였으나 일반적 특성을 제외한 네 가지 변수 모두 정규성 가정을 만족하지 않았으며 평균분석을 위해 진행한 일반적 특성에 따른 그룹 별 변수들의 정규성 검정마저 표본 수가 적은 그룹(ex) 남성, 불만족 그룹 등)을 제외하고는 모두 정규성을 만족하지 않는다는 결과를 얻었습니다.

그렇기에 첫 분석을 진행했을 때에 비모수적 방법인 Spearman correlation(상관분석), Mann-Whitney U test/Kruskal-Wallis test(평균분석)을 사용하였으나, 모수적 방법을 활용하는 것이 조금 더 해석에 효과적이며 표본이 195개로 충분히 많기 때문에 큰 문제가 없을 것이라는 교수님의 조언을 바탕으로 분석 방법을 수정하였습니다.

그렇기에 모수적 방법을 사용하여 다시 분석을 하였고 상관분석에 Pearson correlation 방법을, 평균분석에 T-test/ANOVA를 사용하였고 다중회귀분석에서도 모수적 방법을 이용하였습니다. 

상관분석의 결과는 모수적 방법과 비모수적 방법에 큰 차이가 없었으며 평균분석에도 결과 자체가 뒤바뀌는 현상은 없었습니다.

#### 신뢰도 분석

![Cronbach](https://github.com/HyunbeenLim/portfolio/assets/133561846/166f354e-29e9-4173-89fd-6079281c1097)

가장 먼저 Cronbach's alpha를 사용하여 항목 별 신뢰도 분석을 진행하였습니다.

alpha 값에 뚜렷한 cut-off는 정해지지 않았지만, Nunally가 제안한 0.7 이상을 기준점으로 잡아 모든 항목에 대한 응답이 신뢰가 있다고 판단하였습니다.
문항 별로 해당 문항 제거시 alpha 값 또한 구해보았는데, alpha 값이 소폭 상승하는 문항들이 있었지만 이에 대해 Raykov가 언급한 내용을 바탕으로 제거할만한 점수 상승량은 아니라고 판단하였습니다(alpha inflation 등의 이유로 고려할 사항이 많아 제거하는 것이 언제나 좋은 것은 아니라고 하였습니다).

#### 상관분석

![Pearson](https://github.com/HyunbeenLim/portfolio/assets/133561846/7228711f-0bbd-457b-9041-b4c8c14cf2c7)

다음 네 변수에 대한 상관분석 결과입니다.

Pearson correlation 방법이며 모든 변수는 양의 상관관계를 가진다는 결과를 얻었고 각 변수 별로 다른 변수와의 상관관계를 알아보겠습니다.

자기결정성 학습동기 변수는 다른 변수들과 0.2 이하정도로 약한 상관계수를 갖는데, 학습성과 변수를 제외한 학업적 자기효능감, 학습전이 변수와의 상관계수는 유의미하지 않다는 것을 알 수 있었습니다.

학업적 자기효능감 변수는 학습성과, 학습전이와 각 0.589, 0.504의 상관계수를 갖는데 이는 두 변수와 어느 정도의 상관관계를 갖는다고 해석할 수 있습니다.

학습전이와 학습성과는 0.854로 강한 상관관계를 갖습니다.

따라서 자기결정성 학습동기의 점수는 다른 변수들의 점수와 무관하다고 해석할 수 있으며, 학업적 자기효능감 변수의 점수는 학습전이, 학습성과의 점수와 어느정도 비슷한 모습을 보인다고 해석할 수 있으며 학습성과 변수와 학습전이 변수는 강한 상관계수를 가지므로 두 변수의 점수는 유사한 모습을 보인다고 해석할 수 있습니다.

#### 다중회귀분석

##### Stepwise

학습성과를 반응변수로, 다른 변수들을 독립변수로 하여 다중회귀분석을 진행하였습니다.

먼저 변수 선택을 위해 단계선택법(Stepwise selection)을 활용하였고 일반적 특성은 one-hot encoding으로 더미변수를 사용했습니다(이 경우 각 특성 별로 그룹 -1 개만큼의 더미변수가 만들어집니다).

![variable_selection](https://github.com/HyunbeenLim/portfolio/assets/133561846/cd77dee6-2768-4dde-a665-de2c93ebfb6c)

선택된 변수는 학습전이, 학업적 자기효능감, 입학 동기 변수 중 세 번째 더미(주변사람의 권유), 자기결정성 학습동기, 비대면 수업이 도움이 되는가를 묻는 변수의 두 번째 더미(보통이다) 순서로 선택되었습니다.

![model_summary](https://github.com/HyunbeenLim/portfolio/assets/133561846/392bdbeb-f906-47f6-8270-c6523bc74b0c)

모형의 요약이며 오차의 독립성을 Durbin-Watson 통계랑으로 검증하였습니다. 2.014로 자기상관이 없는 것을 확인할 수 있습니다.

![stepwise_coefficients](https://github.com/HyunbeenLim/portfolio/assets/133561846/b417f363-a6f6-44c1-9ae4-5691b15af778)

회귀 모형의 결과표입니다.

독립변수간 다중공선성을 알아보기 위해 VIF(분산팽창요인)을 사용하였고 VIF 지수가 1.0~1.461로 모두 10 미만이므로 다중공선성이 없다고 판단하였습니다.


선택된 변수 중 입학동기 변수의 더미 변수와 비대면수업도움 변수의 더미 변수가 선택되었기에 최종 다중회귀모형에선 입학 동기 변수의 모든 더미 변수와 비대면수업도움 변수의 모든 더미 변수가 추가가 되어야 하기 때문에 해당 변수들을 모두 포함 후 다중회귀분석을 진행하였습니다.

여기서 주의할 점은, 141번 째 응답자의 표준화 잔차의 절대값이 4.808로 이상치로 확인되었다는 것입니다.
따라서 이상치를 제거한 경우와, 그렇지 않은 경우 두 가지를 모두 진행하였습니다.

![outlier](https://github.com/HyunbeenLim/portfolio/assets/133561846/867ba936-ef33-46b3-986e-63a8737d20ef)

##### 이상치를 제거하지 않은 경우

![incl_model](https://github.com/HyunbeenLim/portfolio/assets/133561846/bf684a33-86df-48f6-a9a7-502c7c089a40)
![incl_coefficients](https://github.com/HyunbeenLim/portfolio/assets/133561846/893a6548-ef1d-40a7-ae55-0816d3505762)

독립변수간 VIF 지수는 1.082~4.925로 모두 10미만이므로 다중공선성이 없다고 판단하였습니다.

또한 Durbin-Watson 통계량이 2.043로 자기상관이 없는 것으로 확인되었으며 회귀분석상 모델의 설명력을 나타내는 결정계수값은 0.891로 나타나, 이 회귀모델은 약 89%의 설명력을 지닌다고 할 수 있습니다.

학습성과에 유의한 영향을 미치는 변수는 자기결정성 학습동기, 학업적 자기효능감, 학습전이, 동기더미3(주변의 권유)인 것으로 나타났습니다.

표준화 계수는 각 독립변수들이 종속변수인 학습성과에 미치는 상대적인 영향력을 나타내는 것으로 학습전이($\beta$ = 0.708)가 가장 큰 영향을 주었으며, 학업적 자기효능감($\beta$ = 0.189), 동기더미_3($\beta$ = −0.095), 자기결정성 학습동기($\beta$ = 0.082) 순으로 나타났습니다.
이는 학습전이와 학업적 자기효능감, 자기결정성 학습동기가 높아질수록 학습성과가 증가하며 동기가 적성에 맞아서에 비해 주변의 권유일 때 학습성과가 감소한다는 것을 보여줍니다.

##### 이상치를 제거한 경우

![removed_model](https://github.com/HyunbeenLim/portfolio/assets/133561846/042515cf-6d07-42eb-9442-c1ddb5b7ce74)
![removed_coefficients](https://github.com/HyunbeenLim/portfolio/assets/133561846/3a3798db-8cdf-4aec-b393-026874df1f39)

독립변수간 VIF 지수는 1.080~4.905로 모두 10미만이므로 다중공선성이 없다고 판단하였습니다. 

오차의 독립성을 검증한 결과 Durbin-Watson 통계량이 2.148로 자기상관이 없는 것으로 확인되었으며 회귀분석상 모델의 설명력을 나타내는 결정계수값은 0.810로 나타나 이 회귀모델은 이상치를 제거하지 않았을 때보다 조금 낮은 약 81%의 설명력을 지닌다고 할 수 있습니다.

학습성과에 유의한 영향을 미치는 변수는 학업적 자기효능감, 학습전이, 동기더미3(주변의 권유)입니다.

표준화 계수는 각 독립변수들이 종속변수인 학습성과에 미치는 상대적인 영향력을 나타내는 것으로 학습전이($\beta$ = 0.742)가 가장 큰 영향을 주었으며, 학업적 자기효능감($\beta$ = 0.159), 동기더미_3($\beta$ = −0.091) 순으로 나타났습니다.
이는 학습전이와 학업적 자기효능감이 높아질수록 학습성과가 증가하며 동기가 적성에 맞아서에 비해 주변의 권유일 때 학습성과가 감소한다는 것을 보여주게 됩니다.

하지만 이 경우 이상치를 제거하지 않았을 때와는 다르게 자기결정성 학습동기 변수가 유의미한 변수로 선택되지 않았음을 확인할 수 있습니다.

원인을 알아보기 위해 이상치를 제거하지 않은 경우의 상관관계와 제거한 경우의 상관관계를 비교해 보았습니다.

![incl_correlation](https://github.com/HyunbeenLim/portfolio/assets/133561846/01435081-0ff6-42a6-a5e6-33a8223560aa)
![removed_correlation](https://github.com/HyunbeenLim/portfolio/assets/133561846/ec508ede-687a-4ef9-bc8a-b57461488f6a)

좌측은 이상치를 포함한 상관관계, 우측은 이상치를 제거한 상관관계입니다.

이상치를 제거했을 때 자기결정성 학습동기와 학습전이가 유의한 상관관계를 보이고 있습니다. 따라서 두 변수가 함께 설명변수로 사용되었을 때 한 변수의 설명력에 의해 다른 변수가 유의하지 않은 것으로 나타나 결과가 달라지는 것을 알 수 있었습니다.

따라서 이상치를 제거한 뒤 각 변수를 학습성과에 대해 단순회귀분석을 진행하였습니다.

![simple_regression](https://github.com/HyunbeenLim/portfolio/assets/133561846/7cdb8bf7-e885-461c-a832-3b0e0c573e3a)

이 때 두 변수 모두 학습성과에 유의미한 변수임을 알 수 있습니다.

이상치가 존재하여 실제로 유의하지 않은데 유의하다고 분석 결과를 왜곡하는 경우에는 이상치를 제거해야하지만 이상치라 하더라도 중요한 관측치일 경우에는 모형에 포함해도 괜찮습니다.

또한 단순선형회귀분석의 결과를 보았을 때 각각의 변수가 학습성과에 유의한 영향을 미치는 것으로 보여 141번 째 응답의 경우 설문에 일부러 이상하게 응답한 것이 아닌 성실히 응답한 것으로 보입니다.

따라서 연구 목적에 따라 제거하지 않고 사용해도 괜찮다고 생각합니다.

#### 평균비교

일반적 특성에 따른 항목 별 평균 비교를 진행하였습니다.

일반적 특성이 10가지에 그룹 또한 2 그룹 ~ 7 그룹으로 나누어지기 때문에 평균분석을 진행한 결과와 빈도분석 그래프 등의 양이 방대해 모든 내용을 첨부할 순 없으나 분석의 방법을 소개하겠습니다.

그룹이 두 그룹인 경우 T-test 사용하였고 세 그룹 이상인 경우 ANOVA를 사용한 뒤 그룹 간 유의미한 차이가 있다고 나온 변수들에 대해선 Scheffe 사후검증하였습니다.

사후 분석을 진행할 때에 주의할 점은 Scheffe 사후 분석의 보수적인 경향 때문에 ANOVA에선 그룹 간 유의미한 차이가 있다는 결과를 얻게 되어도 Scheffe 사후분석에선 차이가 있는 그룹이 없다는 결과를 얻을 수 있게 됩니다. 자주 발생하는 문제이기에 주의해야 한다고 생각합니다.

### 느낀점

이번 프로젝트는 분석의 계획, 결과의 활용 부분에 참여하지 않고 의뢰 받은 분석 그 자체에만 집중할 수 있었으며 실제 데이터 분석에 익숙해질 수 있던 프로젝트였습니다.

연구를 진행하면서 가장 크게 느낀 점은 통계 분석의 유용함보단 분석의 결과와 분석의 이유 등을 비전공자에게 설명할 때에 얼마나 잘 설명할 수 있냐 또한 굉장히 중요하단 것이었습니다.

시각화 등의 과정을 거쳐 최대한 전문적이지 않은 용어로 전달할 줄 아는 능력 또한 기를 수 있는 시간이었습니다.




