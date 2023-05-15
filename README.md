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
    - [부동산 매매가 분석](#부동산-매매가-분석)
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

LDA의 분류선은 $\sigma_k\left(x\right) = x^T\Sigma^{-1}\mu_k-{1\over 2}\mu^T_{-1}\Sigma^{-1}\mu_k+log\pi_k$이며

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

이 식에서 $p(z_i = c|x)$를 $p(z_i = c|x) = {p(z_i = c,x) \over p(x)} = {p(z_i = c) p(x|z_i = c) \over p(x)} = {\pi_c N(x_i|\mu_c \Sigma_c) \over \sigma_{c=1}^K \pi_c N(x_i|\mu_c, \sigma_c)} = \gamma_{ic}$로 정의하겠습니다.

따라서 $E_{z|x}(loglikelihood) = \Sigma_{i = 1}^n \Sigma_{c = 1}^K \gamma_{ic}log\pi_c + \Sigma_{i = 1}^n \Sigma_{c = 1}^K \gamma_{ic}logN(x_i|\mu_c, \sigma_C)$가 됩니다.

E-step으로 구한 식을 라그랑지 방법을 이용해(분량 상 생략하겠습니다.) 최대로 만드는 추정량을 구하면

![EM_추정](https://github.com/HyunbeenLim/portfolio/assets/133561846/84bf8a59-75e5-4f28-b026-84baaa5cd596)

이런 결과를 얻게 됩니다.


Gibb's Sampling 방법은 베이지안 방법의 하나로, 사후분포를 추정합니다.
베이지안 방법은 사전분포에 대한 가정이 필요하므로 저희는 좀 더 검증된 사전분포를 이용하는 것이 적절하다 판단되어, Schnatter(2006)의 책을 참고하였습니다.

GMM에서 유의할 사항은 improper prior를 부여할 시 posterior 또한 improper density가 되어 추정에 어려움이 따르기 때문에 Conjugate, Independent Prior를 부여하여 추정하는 대신 사전분포의 영향이 약하지게 모수를 가정하여 추정하였습니다.

이하 책을 참고하여 부여한 사전분포와 얻은 사후분포입니다.

![Gibbs_para](https://github.com/HyunbeenLim/portfolio/assets/133561846/87977dd1-f461-41eb-8c46-3a12b80f3161)

이며, 여기서 $S$는 EM에서의 변수 z와 같습니다.


### Simulation
### 부동산 

## 따릉이




## 간호대학 학생들의 학습성과




