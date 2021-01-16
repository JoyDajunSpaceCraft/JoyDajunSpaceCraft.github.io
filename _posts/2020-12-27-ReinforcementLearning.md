---
layout: post
title: ' ReinforcementLearning  '
subtitle: ' Machine Learning '
date: 2020--12-27
author: 'Joy'
header-img: 'img/reinforcementLearning.jpg'
tags:
  - Python
  - Reinforcement Learning
  - self-learning
---

# 一些数学概念

## 行列式公式

c<sub>i,j</sub> = (-1)<sup>i+j</sup>det(A)<sub>i,j</sub>

det(A) = a11·c11 + a12·c12···+a1n·c1n

![](/img/rl/math_pic.jpg)

## 行列式的值
+ 其实行列式的值是高维空间中的体积
+ 如果两个row的值相同说明这个矩阵的行列式是0
+ 


# 概念解析
首先根据2020年李宏毅机器学习中的概念将reinforcement learning 作为一种 非 supervice learning的形式作出对比
## 以AlphaGo为例
+ supervice learning 表示从一个老师那里学习
+ reinforcement learning 是根据过去的经验来学习

## 以电玩为例
+ 比如一款星球大战游戏，玩家需要操纵机器往左还是往右，射击还是移动，每次做出动作都会有一个reward作为结果

## 应用领域 
+ Chatbot作用于 如果人也不知道怎么做，但是用reinforcement learning 可以更快的会的结果。
+ 最常用的Application是 
## 难点
+ reward有滞后性，短期牺牲会有好结果
+ agent要会探索世界

## A3C 新概念？
### policy based v.s. value based
+ policy based -> learning an actor： 找一个function
![](/img/rl/policy_actor.jpg)
+ value based -> learning a critic：找一个评价指标

所以最强的是将policy 和 value结合的一种方法 Asynchronous Advatage Actor-Crtic(A3C) Alpha Go 是以上两种方法的结合

### policy based -> learning an actor

#### 步骤
![](/img/rl/policySteps.jpg)

#### NN as Actor 第一步
+ input: the observation of machine represented as a vector or a matrix
+ output: each action corresponds to a neuron in putput layer
 比如 上面的电玩例子，input是每次的图片 pixels 经过NN 获得结果 左移，右移 或者开火
看到哪个output的几率会采取哪种action
![](/img/rl/step1.jpg)
好处就是：比较general，毕竟机器可以预测没有看到过的信息。
+ 对于supervised learning 的 review 
#### Goodness of Actor 第二步
+ give an actor
+ use actor to play the video game 
最大化每次游戏结束的total reward而不是每次action的reward，即使是给了相同的action还是会有不同的reward因为游戏本身有随机性。其实是max reward的期望。

![](/img/rl/step2/expectation.jpg)

#### pick best function 第三步
最大化R用Gradient Ascent
假设在sample data中 计算 gradient 其实就是很直觉的几率看到某个1observation 会的reward 会变好。
为什么要取log 对 p的微分 再除 p的几率 偏好 action b出现的几率多 出掉某个比较大的值相当于 normalization
+ 问题 env 和 Reward是黑盒子需要用 policy gradient train 
### value based -> learning a critic

+ a critic is a function depending on the actor pπie it is evaluated 
  + the function is represented by a neural netword
+ state value function V<sup>π</sup>(s)
  + when using actor π, the cumulated reward expects to be obtained after seeing observation(state)
critic 工作就是衡量一个actor好不好， critic会随着actor的不同而改变
+ 怎么评估 Critic
1. 蒙特卡洛 在游戏结束才知道 reward是多少
2. temporal-difference 方法 $V^π(S_t)=V^T(S_t+1)+r_t$ 
  + 虽然不知道两个实验中获得的V是多少，但是要让差值越接近r_t越好，好处是在游戏没有结束的时候可以开始train

 
+ another critic 另一种 critic方式，在每次对V输入的时候不光是输入 observation 的S 还要输入 actor，算出取得的reward，假设 action是可以**穷举**的。可以用Qfunction找出一个比较好的 actor

Q function 的妙用 1）初始的π 2）蒙特卡洛或者 temporal-difference 方法学出 Q value 3）学出的Qfunction 给了一个新的π 比原来的 π更好

重点在2）到3）步骤 需要在 Voldπ大于等于Vnewπ，也就是π的更新值要一直大于 原先的值

详情见一片rainbow文章

### actor 和 critic A3C （Asynchronous Advatage Actor-Critic）
<a hrrf="https://github.com/JoyDajunSpaceCraft/pytorch-a2c-ppo-acktr-gail/blob/master/a2c_ppo_acktr/model.py" target="_blank">A3C实现</a>

#### A3C 的精神
一般都是根据 Actor 来学习reward 但是A3C是根据 Critic学

#### Asynchronous
Asynchronous是什么意思：有一个global的 Actor 和 gobal 的 Critic 到学习的时候
1. copy global paramters
2. sampling some data 
3. 

### policy gradient 方法 针对 env 和 reward 
因为env 和 reward是不可微分的黑盒子，所以需要实现对这两个内容的train 就用 policy gradient


### inverse reinforcement learning 
imitation learning 的一种 ，没有reward function ，因为多数的情况下是没有reward的，所以用这个来获得reward function 


# 小概念 PPO Proximal Policy Optimization 
从policy gradient 到 on-policy 和 off-policy 再到 add constraint
## policy gradient 
action env reward-function 
+ policy π is a network with parameter Θ
  + input 电玩： 游戏画面
  + output 机器产生什么样的行为 电玩 ： 左右 开火
游戏初始画面 s1 做出action a1 获得 reward r1 
新的游戏画面s2 ...
直到游戏决定结束了，一场游戏叫做episode R表示total reward 
trajectory τ = {s1,a1,s2,a2...}
获得 的几率是：pΘ(τ) = p(s1)pΘ(a1|s1)p(s2|s1,a1)pΘ(a2|s2)...
                  = p(s1)∏pΘ(at,st)p(St+1|St,at)
p(St+1|St,at)是env也就是游戏环境，pΘ(at,st)是actor的行为。
所以前后的图片几率还是有关系的。
获得的R是有规律的，穷举每一个可能的τ，R<sub>Θ</sub>= ∑<sub>τ</sub>R(τ)pΘ(τ) 
就是得到期望值R(τ)，最大化 这个期望， 用到gradient deciant

+  实作的时候的细节 loss function 是 crossentropy 要记得乘上 total reward 

+ tips 1. add baseline 很多游戏只有正的reward ，因为reward一直是乘在几率之前，表示了上升的多少，但是我们是sample所以的action如果有一个reward没有被sample到，它的reward就是0，下次增加的几率就是0，所以将 reward要减去某个概率保证reward有正有负。减去的值就是平均值。

imag
+ tips 2. assign suitable credit 每次游戏可能会是某件事情的累加，因为sample次数不多，合理的contribution需要计算。所以前面的R变为从某个时间t开始直到游戏结束，还要再乘上 一个小于1的参数

imag
## from on-policy to off-policy
