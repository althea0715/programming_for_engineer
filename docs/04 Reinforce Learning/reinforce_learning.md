# Policy Gradient Algorithms

## 개요

국내 최고의 전문가께 강화학습을 배웠지만, 배울 때는 그저 처음 접한 이론이라 많은걸 받아들이지 못했습니다. 그래서 상당기간 지났으나 해당 내용들을 정리하고 기억하고자 합니다.

`lilianweng.github.io`에 작성되어 있는 [Policy Gradient Algorithms](https://lilianweng.github.io/posts/2018-04-08-policy-gradient/) 포스트가 주요 대상이며 이는 강화학습 알고리즘에 대해 비교적 자세히 정리가 되어있습니다. 저도 처음에는 해당 내용을 번역하면서 공부하려고 하였는데 이미 번역해주신 분이 있어서 참고하였습니다.

책이나 검색으로 접할 수 있는 알고리즘은 REINFORCE, Actor-Critic, DQN 일 뿐입니다. 또한 위 블로그에 작성되어 있는 수 많은 알고리즘들의 경우에도 구현체는 구할 수가 없었습니다. 그래서 저는 실제로 논문으로만 존재한 구현체를 직접 구현하고자 합니다.

가급적이면 제가 이해한 내용과 최소의 구현을 통해 저를 포함한 다른분들께도 도움이 되면 좋겠다는 생각으로 이 글을 작성하니 좋은 참고가 되셨으면 좋겠습니다.
 
- 원문 : [Policy Gradient Algorithms](https://lilianweng.github.io/posts/2018-04-08-policy-gradient/) 
- 국내 번역 : [Policy Gradient Algorithms](https://talkingaboutme.tistory.com/entry/RL-Policy-Gradient-Algorithms?category=538748)

## Notatinos

<center>

|         Symbol         | 의미                                                                                                                                     |
| :--------------------: | :--------------------------------------------------------------------------------------------------------------------------------------- |
|       $s \in S$        | States; 상태                                                                                                                             |
|       $a \in A$        | Actions; 행동                                                                                                                            |
|       $r \in R$        | Rewards; 보상                                                                                                                            |
|    $S_t, A_t, R_t$     | t 시점의 States, Actions, Rewards ($s_t, a_t, r_t$라고 표기하기도 합니다.)                                                               |
|        $\gamma$        | Discount factor; 미래에 얻게된 보상의 패널티입니다. ($0<\gamma\leq 1$)                                                                   |
|         $G_t$          | Return; $\gamma$가 적용된 모든 Rewards ($G_t=\sum_{k=0}^{\infty}\gamma^kR_{t+k+1}$)                                                      |
| $P(S^\prime, r\|s, a)$ | 현재 States와 Actions 상태에서 얻게될 Rewards와 다음 States가 될 확률입니다.                                                             |
|      $\pi(a\|s)$       | Stochastic policy(확률적 정책); States에 따라서 Actions를 정할 전략, $\pi_\theta(\cdot)$은 $\theta$로 파라미터화 되어있는 것을 말합니다. |
|        $\mu(s)$        | Deterministic policy(결정적 정책); $\pi(s)$라고 작성할 수 있지만 문자에 따라 강화학습의 목표가 달라지기 때문에 가급적 나누는게 낫습니다. |
|         $V(s)$         | State-value function, $V_w(\cdot)$라고 작성될 땐 $w$로 파라미터화 되어있는 것을 말합니다.                                                |
|       $V^\pi(s)$       | Policy $\pi$를 따를 때 state-value funciton 입니다.                                                                                      |
|        $Q(s,a)$        | $V(s)$와 비슷하지만 Actions를 고려한 값입니다.                                                                                           |
|      $Q^\pi(s,a)$      | $V^\pi(s)$와 비슷하지만 Policy \pi를 따를 때의  Actions를 고려한 값입니다.                                                               |
|        $A(s,a)$        | Advantage function; Q-Value의 variance를 줄여주기 위해 사용합니다. $A(s,a) = Q(s,a)-V(s)$                                                |
</center>

## Policy Gradient

Agent가 최고의 Reward를 얻을 수 있는 최적의 전략을 찾는 것이 강화학습의 목표입니다. Policy Gradient에서 Parameter $\theta$로 구성되어있는 Policy $\pi$를 최적화하는데 노력합니다. 즉, 최고의 Rewards를 얻기 위해 $\theta$를 최적화하려고 노력합니다.

$$
\begin{aligned}
J(\theta)
&=\sum_{s \in S}d^\pi(s)V^\pi(s)\\
&=\sum_{s \in S}d^\pi(s)\sum_{a \in A}\pi_\theta(a|s)Q^\pi(s,a)
\end{aligned}
$$
<center>Reward Function</center><br>

$d^\pi(s)$는 Policy $\pi_\theta$를 가진 상태의 markov chain에 대한 stationary distribution이라는 데, $d^\pi(s)=\lim_{t \rightarrow \infty}P(s_t=s|s_0, \pi_\theta)$로 설명 가능합니다. State $s_0$와  $\pi_\theta$였을 때 t시간에 따라 $s_t=s$가 될 확률입니다. 어차피 게임을 계속해서 정답을 맞출 확률이 높아질 테니 시간이 무한히 흘렀을 때 State가 될 확률이라고 보면 됩니다.

## Policy Gradient Theorem

보통 주어진 환경이 어떻게 될지 모르기 때문에 $d^\pi(s)$를 구하는 것 자체가 어렵습니다. 하지만 Policy Gradient Theorem이 이 문제를 해결해줄 수 있습니다. 어차피 관심 있는 변수는 Reward $J(\theta)$이므로 $\theta$를 Gradient하면 됩니다.

$$
\begin{aligned}
\nabla_\theta J(\theta)
&=\nabla_\theta\sum_{s \in S}d^\pi(s)\sum_{a \in A}\pi_\theta(a|s)Q^\pi(s,a)\\
&=\sum_{s \in S}d^\pi(s)\sum_{a \in A}Q^\pi(s,a)\nabla_\theta\pi_\theta(a|s)\\
\end{aligned}
$$

$\pi_\theta(a|s)$를 제외한 나머지 변수들은 $\theta$에 관한 변수가 아니므로 Gradient의 대상이 아닙니다.

## On-Policy VS Off-Policy

On-Policy의 경우

$$
Q(s,a) \leftarrow Q(s,a) + \alpha(r+\gamma Q(s^\prime, a^\prime)-Q(s,a))
$$

Off-Policy의 경우

$$
Q(s,a) \leftarrow Q(s,a) + \alpha(r+\gamma \max_{a^\prime} Q(s^\prime, a^\prime)-Q(s,a))
$$

여기서 $s^\prime$, $a^\prime$은 Next State, Next Action을 뜻합니다. 

Off-Policy는 $Q(s^\prime, a^\prime)$ 상태에서 모든 상태를 고려하는 것이 아니라 최고의 상태만 찾아가는 형태를 취합니다. 이를 탐욕 정책 Greedy Policy라 합니다. 대체적으로 Off-Policy를 대상으로 강화학습을 진행하게 됩니다.

## Overview

강화학습을 공부하려고 시작하면 대표적으로 REINFORCE, Actor-Critic를 처음 접하게 됩니다. 이 알고리즘들은 On-Policy 형태이며 글의 의도상 게임 한판을 다 하면 (한 에피소드를 완료하면) 그간 모았던 State, Action, Reward, Next State 를 기록하여 한꺼번에 모델을 업데이트 합니다. 그런 내용을 담고 있는 수식이 바로 아래의 수식입니다.

$$
\begin{aligned}
\nabla_\theta J(\theta)
&=\mathbb{E}_\pi[G_t\nabla_\theta\ln\pi_\theta(a|s)]&\text{REINFORCE}\\
&=\mathbb{E}_\pi[Q^\pi(s,a)\nabla_\theta\ln\pi_\theta(a|s)]&\text{Actor-Critic}
\end{aligned}
$$

$\mathbb{E}_\pi$라는 Expectation이 존재하는데 이는 계산된 점수(Reward)
를 한게임이 끝났을때 한꺼번에 연산해야한다는 의미입니다.

이제 실제로 관심 가지게 될 Actor-Critic 알고리즘은 구현상에 여러가지 애로사항이 있었습니다. 아무리 자료를 찾아봐도  Actor-Critic 알고리즘이 구현되지 않고 A2C 알고리즘이 구현되어 있습니다. 향후 찾아보시는 분들께 도움되었으면 좋겠습니다. (Actor-Critic에서 Actor는 정책이고 Critic은 정책값입니다.)

$$
\begin{aligned}\nabla_\theta J(\theta)
&=\mathbb{E}_{\pi_\theta }[\nabla_\theta\log\pi_\theta(s,a)G_t]&\text{REINFORCE}\\
&=\mathbb{E}_{\pi_\theta }[\nabla_\theta\log\pi_\theta(s,a)Q^w(s,a)]&\text{Q Actor-Critic}\\
&=\mathbb{E}_{\pi_\theta }[\nabla_\theta\log\pi_\theta(s,a)A^w(s,a)]&\text{Advanced Actor-Critic(A2C)}\\
&=\mathbb{E}_{\pi_\theta }[\nabla_\theta\log\pi_\theta(s,a)\delta]&\text{TD Actor-Critic}\\\end{aligned}
$$

위 수식들은 뒷 부분만 변경이 되는데, Actor-Critic 알고리즘을 파해쳐기위해 $Q^w(s,a)$가 무엇인지 알 필요가 있습니다. Bellman optimality equation을 작성하면 다음과 같습니다.

$$
Q_*(s,a)=R_s^a+\gamma\sum_{s^\prime \in S}P_{ss^\prime}^a V_*(s^\prime)
$$

위 수식을 조금 자세히 설명하면 각 Notation은 다음과 같은 의미입니다.

- $R_s^a$ : 현재 State에서 Action을 취할 때 얻는 Reward
- $P_{ss^\prime}^a$ : 현재 State에서 Action을 취해서 다음 State 갈 확률
- $V_*(s^\prime)$ : 현재 State의 가치 혹은 현재 State에서 얻을 수 있는 Reward 값

우리는 Deep Learning으로 정책(Policy, $\pi$)을 설정하고 그 정책에서 정책값을 계산할 것이므로 이 수식을 아래와 같이 다시 작성할 수 있습니다. 

$$
Q^{\pi^\theta}(s,a)=R_t+\gamma V^{\pi^\theta}(s^\prime)
$$

정책을 담당하는 Actor에서 계산해야하는 정책값을 Critic에서 추정하게 끔 변형합니다. (원래 정책에서 정책값을 계산하는 것 자체가 어려워서 Critic에서 Actor의 정책값을 학습하는 Actor-Critic 알고리즘을 쓰는 겁니다.) 그래서 $\pi^\theta=w$로 치환하면 다음과 같이 수식을 작성할 수 있습니다.

$$
Q^w(s,a)=R_t+\gamma V^w(s^\prime)
$$

여기까지가 Actor-Critic 알고리즘의 설명입니다. Advanced Actor-Critic은 Baseline을 사용하는 건데 해당 내용은 상단에 설명되어있습니다. 그러므로 Baseline에 대한 설명은 넘기고 수식만 가져오면 $A(s,a)=Q(s,a)-V(s)$ 입니다. 우리는 어차피 $V(s)$ 값은 Critic으로 쉽게 알 수 있으므로 계산을 바로 할 수 있습니다.

마지막으로 TD Actor-Critic인데, 이는 다음 State의 값과 현재 State의 값의 차이를 계산해야합니다. 한가지 알아두어야할 수식은 $V(s_t) = R_{t+1}+\gamma V(s_{t+1})$ 입니다. 아주 이상적이라면 해당 수식은 성립해야합니다. 하지만 학습해야한다는 점을 고려하면 두 값은 같지 않습니다. 그래서 저 두 값의 차이를 TD error, $\delta$라 언급할 수 있습니다. 수식으로 다음과 같이 작성할 수 있습니다.

$$
\delta=R_{t+1}+\gamma V(s_{t+1})-V(s_t)
$$

사실 이렇게 되면 A2C와 TD Actor-Critic은 같은 알고리즘이라고 할 수 있습니다. 여기서 얻을 수 있는 중요한 것은 Critic 함수를 어떻게 업데이터 할 것인가를 알 수 있다는 것입니다. 위에 언급한 loss 함수는 전부 Actor의 loss 함수입니다. Actor-Critic에서 Critic은 어떻게 업데이트 해야할지 명확하진 않습니다. (사실 아래 Actor-Critic 알고리즘 구현할 때 많이 헤맸습니다.) 요약하면 바로 전에 언급한 TD error를 줄이는 방향으로 업데이트하면 됩니다.

즉, 저 $\delta$가 0으로 수렴하는 방향으로 업데이트 하면되는데, Deep Learning 알고리즘 기준으로 L1, mse 등 아무거나 0으로 수렴하게 만드는 함수를 쓰면 됩니다. 

Advanced 함수를 만드는 것이랑 모양은 같지만, loss 함수를 만들때 유의해야할 점이 있습니다. 이러니 저러니 해도 결국 Gradient Descent 방식으로 Deep Learning 모델의 Parameter $\theta$를 업데이트 하는데, 이는 무언가의 변수를 편미분한다는 것을 뜻합니다. 편미분을 할 타겟을 명확히해아합니다.

- Actor의 loss 함수에서 편미분의 대상은 $\nabla_\theta\log\pi_\theta(s,a)$입니다.
- Critic의 loss 함수에서 편미분의 대상은 $V(s,a)$입니다.
저 둘을 제외한 나머지 값들은 미분이 되지 않도록 고정시켜야합니다.

향후 소개할 알고리즘의 Notation과 다소 다를 수 있으나, 가급적 맞추려고 노력하였습니다. 아래 수식에도 따로 작성해 놓은 고찰이 있으니 참고해주세요.



작성중...ㄴㅇㅀㄴㅇㄹㅀ

## Policy Gradient Algorithms


상단에 공유된 링크의 블로그에 작성된 Section 순서대로 실제 알고리즘을 설명합니다. 해당 링크에는 개선사항이 설명되어 있긴하지만 가급적이면 논문이나 알고리즘 그대로의 알고리즘을 구현하겠습니다.

강화학습은 기본적으로 어떠한 시스템의 시뮬레이션이 필요하며, 시뮬레이션의 입력값은 Discrete 방식과 Continuous 방식이 있습니다. 강화학습을 공부하는데 유명한 시뮬레이션인 CatPole은 Discrete 방식입니다. 선택하는 문제뿐만 아니라 실수를 입력할 경우를 대비한 강화학습도 필요합니다. 이를 위해 Pendulum을 사용하며, Pendulum은 Continuous 방식입니다. 

<center>

|        기능        | CartPole-v0 |    Pendulum-v1     |
| :----------------: | :---------: | :----------------: |
|    입력 값 종류    |  Descrete   |     Continuous     |
| 입력 범위 (action) |  0 또는 1   | $-2 \leq x \leq 2$ |
|   상태값 (state)   | 4개의 변수  |     3개의 변수     |
|  보상값 (reward)   |  0 또는 1   |   실수값 (음수)    |
| 한 게임 최대 횟수  |     200     |        200         |
</center>

CartPole의 경우 한개의 에피소드에 200 Stage까지 갈 수 있습니다. Reward에 아무런 튜닝을 하지 않았을 때 20 Stage까지 가면 20점 17 Stage까지 가면 17점 입니다. 생각해보면 두 개의 점수차이는 큰 차이가 나지 않는 것으로 보입니다. 특히 알고리즘에 따라 평균을 구하게 되면 더욱 차이가 발생하지 않습니다. 그래서 목표에 도달하지 않았을 때의 <span style="color:red">**패널티가 굉장히 중요합니다.**</span> 그래서 저는 기본적으로 CartPole의 Reward를 다음과 같이 설계하였습니다.

```python
# 목표에 도달하지 못하면 -1점, 일반적으로 동작할 땐 0.1점
reward = 0.1 if not done or i_iter >= 199 else -1
```



ㅁㄴㅇㄻㅇㄹ 발췌









