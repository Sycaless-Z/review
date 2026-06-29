# review撰写第2周笔记：6.22 - 6.28

目标：补相关工作、协议机制、核心直觉。

## 本周主要梳理

这周先读了论文第 2 节和第 4 节。后面又补看了下载到 `papers/` 里的相关论文首页、摘要、主要定理和结论。review 里新增了三块：

1. 相关工作
2. 本文协议机制
3. 核心直觉

证明还没细写，只把后面会用到的直觉先铺出来。

## 相关工作怎么分

我暂时把 related work 分成三类：

### BEB 和退避协议

BEB 很简单，但是理论稳定性差。

需要记住的点：

- Aldous：任意常数泊松注入率下 BEB 不稳定。
- Bender 等：同步到达时 BEB 是 $\Theta(n\log n)$；他们给出 loglog-iterated backoff，单调协议最优量级是 $\Theta(n\log\log n/\log\log\log n)$。
- 系统里还有很多 backoff 变体，比如 exponential increase/decrease、multiplicative increase/linear decrease 等，但这些更像工程启发式。

### 更复杂的稳定协议

这一类工作解决了 BEB 的一些问题，但是协议复杂度更高，常数效率也不如本文。

- Bender, Fineman, Gilbert, Young：Re-Backoff，constant throughput，polylog access attempts，可以处理 disrupted slots。本文主要和它比。
- Awerbuch, Richa, Scheideler：single-hop wireless network，jammer 可以 jam 掉 $(1-\epsilon)$ 比例的时间步，目标是在 non-jammed steps 上保持常数吞吐量。
- Bender, Kopelowitz, Pettie, Young：能量模型，发送和监听都算 energy；结果里有每个设备 listens $O(\log(\log^*N))$ 次。

### $1/e$ 阈值和反馈强弱

$1/e$ 不是所有模型的绝对上限。

- 对 stateless algorithm，本文说 $1/e$ 是最优。
- 泊松注入模型下，Mosely & Humblet 可以到约 $0.48776$。Tsybakov & Mikhailov 原文没找到，先只作为主论文引用保留。
- 如果反馈更强，Pippenger 的结果说明可以接近满利用率。
- 如果反馈更弱，Goldberg 等证明 backoff protocol capacity 至多 0.42，acknowledgment-based protocol 至多 0.531。

## 查阅文献笔记

### Aldous 1987

题目：*Ultimate instability of exponential back-off protocol for acknowledgment-based transmission control of random access communication channels*

先看了摘要和 Theorem 1。结论很强：binary exponential backoff 在任意 $v>0$ 下 unstable，成功传输率 $N(t)/t\to0$。这可以用来说明 BEB 的理论问题不是只在极端高负载下才出现。

### Bender et al. 2005

题目：*Adversarial contention resolution for simple channels*

这里主要看 batched arrivals 和 worst-case backoff。需要用到的点：BEB makespan 是 $\Theta(n\log n)$；一般 $r$-exponential backoff 也有相应下界；loglog-iterated backoff 可以做到 $\Theta(n\log\log n/\log\log\log n)$，并且对 monotone backoff 最优。

### Bender, Fineman, Gilbert, Young 2016

题目：*How to Scale Exponential Backoff: Constant Throughput, Polylog Access Attempts, and Robustness*

这篇是本文最直接比较对象。它提出 Re-Backoff，目标是 constant throughput、few attempts、robustness。笔记里先记：它做到常数吞吐量和 polylog access attempts，但本文说它的效率常数小于 0.05，且每个设备平均尝试次数仍然是 $O(\log^2(n+J))$ 量级，不如本文的 $e+O(\epsilon)$。

### Awerbuch, Richa, Scheideler 2008

题目：*A jamming-resistant MAC protocol for single-hop wireless networks*

这篇关注 jammer。对手可以知道协议和历史，并且 jam 掉 $(1-\epsilon)$ 比例的 time steps。协议根据 idle / success 等反馈调节概率，在 non-jammed time steps 上保持 constant throughput。和本文的关系：都是根据反馈调概率，但本文的更新规则更简单，目标常数也更接近 $1/e$。

### Bender, Kopelowitz, Pettie, Young 2016

题目：*Contention resolution with log-logstar channel accesses*

这篇把 energy 作为核心指标，因为监听 channel 也有成本。结果是 expected constant throughput，每个 player sends $O(1)$ times，listens $O(\log(\log^*N))$ times。本文最后的 open problem 和这篇直接有关：能否同时做到低 energy 和 $1/e-O(\epsilon)$ utilization。

### Mosely & Humblet 1985

题目：*A Class of Efficient Contention Resolution Algorithms for Multiple Access Channels*

这篇在泊松到达模型里做到 throughput 约 0.48776。这个数比 $1/e\approx0.3679$ 高。说明本文的 $1/e$ 不是所有模型的绝对极限，而是 stateless / ternary feedback / adversarial injection 这一类模型下的目标。

### Pippenger 1981

题目：*Bounds on the Performance of Protocols for a Multiple-Access Broadcast Channel*

这篇主要是反馈强弱和 capacity 的关系。如果只能区分有限种冲突类型，不能接近满利用率；如果能知道准确发送者数量，就能接近 capacity。这个可以放在 related work 里说明三元反馈假设为什么重要。

### Goldberg et al. 2004

题目：*A bound on the capacity of backoff and acknowledgment-based protocols*

摘要里最直接的结论：backoff protocol 的 capacity 至多 0.42；acknowledgment-based protocol 的对应上界是 0.531。这个用来说明反馈弱时上限会下降。

### Bender, Kopelowitz, Kuszmaul, Pettie 2020

题目：*Contention Resolution Without Collision Detection*

这是论文发表之后的相关工作。主要看了摘要和 introduction。它研究没有 collision detection 的 channel：监听者不能区分“没人广播”和“两个或更多人同时广播”。这比本文的 ternary feedback 弱，因为本文能区分 silence 和 noise。

需要记住的点：

- 没有 collision detection 时，仍然可以在 adversarial arrivals 下做到 constant throughput。
- 但如果再加入 jamming adversary，论文证明 collision detection fundamentally necessary。
- 所以它适合放在“反馈强弱”那一段，说明本文的三元反馈假设很关键。

## 协议机制

### initialization rule

新设备：

$$
p_i\leftarrow \epsilon^2.
$$

这里先记住是为了让新包不要一下子把总冲突度冲太高。具体为什么势函数里好用，下周证明部分再看。

### transmission rule

论文先用一个方便分析的版本：

$$
\begin{cases}
e^{-p_i} & \text{silent}\\
p_i e^{-p_i} & \text{transmit}\\
1-(1+p_i)e^{-p_i} & \text{make noise}
\end{cases}
$$

这个 make noise 很怪，实际不合理，但分析方便。它让

$$
p_{sil}=e^{-c},\quad p_{suc}=ce^{-c},\quad p_{noi}=1-(1+c)e^{-c}.
$$

### update rule

$$
p_i \leftarrow
\begin{cases}
p_i e^\epsilon & \text{silence}\\
p_i & \text{success}\\
p_i e^{-\epsilon/(e-2)} & \text{noise}
\end{cases}
$$

为什么是 $e-2$：在 $c=1$ 时，

$$
p_{sil}=1/e,\quad p_{noi}=(e-2)/e.
$$

所以

$$
\frac{1}{e}\epsilon+\frac{e-2}{e}\left(-\frac{\epsilon}{e-2}\right)=0.
$$

这说明在最优点附近更新是无偏的。

### simpler transmission rule

后面改成更自然的：

$$
\begin{cases}
e^{-p_i} & \text{silent}\\
1-e^{-p_i} & \text{transmit}
\end{cases}
$$

也就是不主动制造噪音。

## 核心直觉

最关键是总冲突度

$$
c=\sum_i p_i.
$$

$c<1$：大家太保守，容易 silence，所以要上调。

$c>1$：大家太激进，容易 noise，所以要下调。

取

$$
\gamma=\ln c
$$

之后，乘法更新变成加法移动：

$$
\gamma'\in\left\{\gamma+\epsilon,\gamma,\gamma-\frac{\epsilon}{e-2}\right\}.
$$

这就是 homesick random walk 的直觉：$\gamma$ 会被拉回 0。0 对应 $c=1$，也就是成功概率最高的位置。

效率函数：

$$
p_{suc}(\gamma)=e^\gamma e^{-e^\gamma}.
$$

在 0 附近：

$$
p_{suc}(\gamma)=1/e-\gamma^2/(2e)-\gamma^3/(6e)+O(\gamma^4).
$$

所以只要 $\gamma$ 不离 0 太远，效率就接近 $1/e$。

## 未找到的文献

Tsybakov & Mikhailov, *Slotted multiaccess packet broadcasting feedback channel*, 1978 没找到。主论文里说它和 Mosely & Humblet 一样能到约 0.48776，review 里暂时只提“主论文提到”，不展开它的细节。
