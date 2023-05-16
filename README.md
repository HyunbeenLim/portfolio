# Portfolio

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
- [간호대학 학생들의 학습성과](#간호대학-학생들의-학습성과)

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

논문 내용을 기반으로 두 가지의 simulation을 진행하였습니다.

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

저희는 해당 분포를 이용해 EM 알고리즘과 Gibb's sampling의 모수 추정(사후 분포 추정) 성능을 비교하고자 하였습니다.

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


Gibb's Sampling 방법은 베이지안 방법의 하나로, 사후분포를 추정합니다.
베이지안 방법은 사전분포에 대한 가정이 필요하므로 저희는 좀 더 검증된 사전분포를 이용하는 것이 적절하다 판단되어, Schnatter(2006)의 책을 참고하였습니다.

GMM에서 유의할 사항은 improper prior를 부여할 시 posterior 또한 improper density가 되어 추정에 어려움이 따르기 때문에 Conjugate, Independent Prior를 부여하여 추정하는 대신 사전분포의 영향이 약하지게 모수를 가정하여 추정하였습니다.

이하 책을 참고한 사전분포와 그에 따른 .

![Gibbs_para](https://github.com/HyunbeenLim/portfolio/assets/133561846/87977dd1-f461-41eb-8c46-3a12b80f3161)

이며, 여기서 S는 EM에서의 변수 z와 같습니다.


### Simulation

다음 실험입니다. 

실험에 사용한 코드는 저희가 R에서 직접 작성한 코드를 사용하였고 데이터 생성은 R의 distr 패키지를 사용했습니다. 수렴 기준은 소수점 아래 5번째 자리로 하였습니다.

실험에 사용한 데이터는 크게 두가지입니다.

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
이 문제는 $Chung$ et. al(2004)에서 언급된 문제이며, 해당 논문에서 제시한 모수 공간에 제약을 거는 방식인 permutation 방법을 통해서 어느정도 해결하는 모습을 보일 수 있었습니다. 

또한 실제 사례 적용 결과를 통해 두 방법 모두 좋은 결과를 보이는 것을 확인하여 임의의 분포를 GMM으로 잘 근사할 수 있음을 확인할 수 있었습니다. 동시에, 양천구의 사례와 같이 각 구 별 평균 실거래가 등의 기술통계로는 알 수 없는 정보를 잘 파악할 수 있음을 알 수 있었습니다.

## 따릉이




## 간호대학 학생들의 학습성과




