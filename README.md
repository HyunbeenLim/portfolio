# Portfolio

## Contents

- [RDA](#RDA)
    - [분석 목적](#분석-목적)
    - [RDA소개](#RDA-소개)
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

Friedman은 또한 RDA에 두 가지의 hyper parameter를 제안하였는데 이 모수들을 조절하여 RDA의 성능을 끌어올릴 수 있습니다.

먼저 $\lambda$로 그룹별 분산을 조절하고 $\gamma$로 shrinkage를 조절할 수 있습니다.

$\lambda$를 포함한 각 그룹별 분산은 $\hat{\Sigma_k (\lambda)} = S_k(\lambda)/W_k(\lambda)$
$S_k(\lambda) = (1 - \lambda)S_k + \lambda S$ 
$W_k(\lambda) = (1-\lambda)W_k + \lambda W$ 이며, 여기서 $\lambda$가 1이라면 LDA, $\lambda$가 0이라면 QDA 방법이 됩니다.

$\gamma$를 포함해 최종적으로 그룹별 분산 추정량을 $\hat{\Sigma_k (\lambda, \gamma)} = (1-\gamma)\hat{\Sigma_k(\lambda)} + {\gamma \over p}tr\left[\hat{\Sigma_k(\lambda)}\right]I$ 로 쓸 수 있습니다. 여기서 $\gamma$는 $\hat{\Sigma_k(\lambda)}$의 고유값에 곱해지는 것을 볼 수 있는데, 큰 고유값은 줄이고 작은 고유값은 늘리는 역할로 표본을 기반으로 추정한 고유값의 편향을 조절할 수 있습니다.

## EM and Gibbs




## 따릉이




## 간호대학 학생들의 학습성과




