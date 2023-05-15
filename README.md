# Portfolio

## Contents

- [RDA](#RDA)
    - [분석 목적](#분석-목적)
    - [RDA소개](#RDA-소개)
    - [Simulation Study](#Simulation-study)
    - [Application](#Application)
- [EM and Gibbs](#em-and-gibbs)
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



## EM and Gibbs




## 따릉이




## 간호대학 학생들의 학습성과




