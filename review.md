# Review for Simple Contention Resolution via Multiplicative Weight Updates

author 张语桐

## 1. 论文基本信息与核心结论

本文研究的是经典的 contention resolution（冲突消解 / 竞争解决）问题。先把它想成一个共享信道问题：很多设备都想把自己的一个数据包发出去，但同一时间只有一个设备能成功占用信道。一个时间隙内，如果没有设备发送，就是静默；如果正好一个设备发送，就是成功；如果两个或更多设备同时发送，就发生冲突，信道表现为噪音。协议要解决的问题是，在设备数量未知、数据包还可能不断被注入的情况下，怎样让尽可能多的时间隙变成成功传输。

这篇论文的核心贡献是提出了一个非常简单的无状态协议。每个还没有成功发送的设备只维护一个数值参数 $p_i$，并根据信道的三元反馈做乘法更新：

$$
p_i \leftarrow
\begin{cases}
p_i e^\epsilon, & \text{听到静默时},\\
p_i, & \text{听到成功时},\\
p_i e^{-\epsilon/(e-2)}, & \text{听到噪音时}.
\end{cases}
$$

其中 $\epsilon>0$ 是步长，也是算法的主要参数。我的理解是，静默说明当前设备整体太保守，所以应该把发送概率调大；噪音说明当前整体太激进，所以应该把发送概率调小；成功说明当前信道负载比较合适，所以暂时不用调整。

论文证明，对于任意 $\epsilon>0$，该协议可以达到 $1/e-O(\epsilon)$ 的通道利用率。这里的 $1/e$ 不是随便出现的常数：如果系统里有 $n$ 个设备，并且它们知道 $n$，那么最自然的做法是每个设备以 $1/n$ 的概率发送，此时一个时间隙恰好有一个设备成功发送的概率是

$$
n\cdot \frac{1}{n}\left(1-\frac{1}{n}\right)^{n-1}
= \left(1-\frac{1}{n}\right)^{n-1}
\rightarrow \frac{1}{e}.
$$

也就是说，在这种随机接入模型里，$1/e$ 是一个很自然的效率基准。本文的意义在于，它不要求设备一开始知道 $n$，而是只依靠所有设备都能听到的三元反馈，把系统的总发送倾向逐渐调到接近最合适的位置。

## 2. 研究问题为什么重要

冲突消解问题重要，是因为它刻画了许多分布式系统里的基本矛盾：多个参与者共享同一个资源，但它们不能依赖中心调度器，也未必知道同时竞争的人数。网络中的多路访问信道是最直接的例子，例如多个无线设备共享同一段频谱。如果每个设备都保守等待，信道会大量空闲；如果每个设备都积极发送，又会发生大量冲突。一个好的协议需要在“空闲浪费”和“冲突浪费”之间自动平衡。

传统方法是 exponential backoff，特别是 Binary Exponential Backoff (BEB)。BEB 的优点是简单：设备失败后降低发送概率或扩大等待窗口。但论文在引言中指出，BEB 在理论上有明显问题。即使数据包按照低速率的泊松过程到达，BEB 也可能不稳定；当所有 $n$ 个数据包同时到达时，它通常需要 $O(n\log n)$ 时间和每个设备 $\Theta(\log n)$ 次尝试，而理想表现应该接近 $O(n)$ 时间和 $O(1)$ 次尝试。

所以我觉得本文真正想做的是：保留 BEB 那种简单、分布式的风格，但避免它在高负载或长期注入下的坏理论性质。它不是通过复杂的调度或很多状态变量来解决问题，而是只用一个乘法更新规则来控制发送概率。

## 3. 模型与基本定义

### 3.1 多路访问信道

论文考虑离散同步时间隙中的 multiple-access channel。每个活跃设备在每个时间隙决定是否尝试发送自己的数据包。一个时间隙的反馈只有三类：

- $0$ 或 silence：没有设备发送。
- $1$ 或 success：恰好一个设备发送，该设备的数据包成功传输并退出系统。
- $2^+$ 或 noise：至少两个设备发送，发生冲突；在 jammer 模型中，也可以是对手主动制造的噪音。

所有活跃设备在每个时间隙结束后都能听到这个三元反馈。这个假设很关键，因为协议正是依赖 silence / success / noise 的差别来更新 $p_i$。

### 3.2 数据包注入与活跃设备

设备不是必须在一开始全部出现。对手可以在不同时间注入新的数据包，使新设备或新任务变为活跃。新数据包加入系统后，设备按照初始化规则设置自己的参数：

$$
p_i \leftarrow \epsilon^2.
$$

如果某个设备成功发送了自己的数据包，它就停止参与后续竞争。系统中尚未成功发送的数据包数量记为 $n$。

### 3.3 聚合冲突度

本文分析的关键变量不是单个 $p_i$，而是所有活跃设备发送倾向的总和

$$
c=\sum_i p_i.
$$

这个量可以理解为 aggregate contention（聚合冲突度）。如果 $c$ 太小，静默概率高，信道被浪费；如果 $c$ 太大，噪音概率高，冲突严重。最佳状态应该在 $c\approx 1$ 附近，因为此时成功概率接近最大。

由于更新规则是乘法形式，论文在对数尺度上分析冲突度：

$$
\gamma=\ln c.
$$

当所有活跃设备听到同一个反馈时，它们的 $p_i$ 会同时乘上同一个系数。例如听到静默时所有 $p_i$ 乘以 $e^\epsilon$，因此

$$
c'=\sum_i p_i e^\epsilon=e^\epsilon c,
\quad
\gamma'=\ln c'=\ln c+\epsilon=\gamma+\epsilon.
$$

同理，听到噪音时 $\gamma$ 减少 $\epsilon/(e-2)$；听到成功但暂不考虑成功设备退出造成的影响时，$\gamma$ 不变。所以后面论文把 $\gamma$ 的变化看成一种带有“回到 0”趋势的随机游走。

### 3.4 通道利用率

通道利用率可以粗略理解为长期时间内成功传输数据包的比例。本文的目标是证明，在对抗性注入甚至存在 jammer 干扰的情况下，除去与干扰次数成比例的不可避免损失后，协议仍然能让未被干扰的时间隙达到

$$
1/e-O(\epsilon)
$$

的利用率。这里的 $O(\epsilon)$ 表示通过选择足够小的步长，可以让效率任意接近 $1/e$。

### 3.5 Jammer

jammer 是可以主动制造噪音的对手。它可以让某些时间隙即使没有真实冲突也表现为 noise。本文的结论比普通随机接入模型更强：如果 jammer 总共干扰了 $J$ 个时间隙，那么只需额外扣除 $O(J)$ 个被浪费的时间隙，协议在剩余时间上的利用率仍然接近 $1/e$。

### 3.6 Stateless Protocol

论文强调该协议是 stateless 的。这里的无状态不是说设备完全没有变量，而是说每个设备除了当前的 $p_i$ 以外，不记录复杂历史，也不需要知道自己失败过多少次、系统中有多少设备、其他设备的状态或完整信道历史。每个时间隙的行为只由当前 $p_i$ 和刚刚收到的三元反馈决定。

这一点是本文和很多复杂稳定协议相比的重要优势。它把协议做得接近 BEB 一样简单，但分析结果明显更强。

## 4. 相关工作

这篇论文的位置可以从三个方向看：传统退避协议、已有的稳定冲突消解协议，以及 $1/e$ 这个效率界限到底意味着什么。

首先是传统的 Binary Exponential Backoff。BEB 来自以太网一类随机接入协议，思想很直接：如果一个设备发送失败，就降低自己的发送概率，或者等价地扩大随机等待窗口。它的优点是非常简单，而且在很多实际系统里确实好用。但理论结果说明，BEB 在长期高负载下并不可靠。Aldous 在 acknowledgement-based 模型中证明过，二进制指数退避在任意正到达率下都会出现 ultimate instability，也就是长期成功传输率趋向 0。Bender, Farach-Colton, He, Kuszmaul, Leiserson 从 worst-case / batched arrivals 的角度分析 backoff，证明当所有 $n$ 个包同时到达时，BEB 的 makespan 是 $\Theta(n\log n)$，而且更一般的指数退避、 polynomial backoff 对参数也很敏感。他们还给出 loglog-iterated backoff，使单调协议做到 $\Theta(n\log\log n/\log\log\log n)$，并说明这已经是单调 backoff 能达到的最优量级。

这说明 BEB 的问题不是实现细节，而是它的调节方式本身不够稳定。设备只根据自己的失败历史退避，可能会让系统在“大家都太保守”和“大家又同时冲突”之间来回摆动。本文想保留退避协议的简单性，但把调节依据换成所有设备都能听到的三元反馈。

第二类相关工作是更复杂的稳定协议。Bender, Fineman, Gilbert, Young 的 Re-Backoff 研究怎样 scale exponential backoff，希望同时满足 constant throughput、较少 access attempts 和 robustness。它可以处理动态到达和资源被 disrupted 的情况，并保证平均 $O(\log^2(\eta+D))$ 次 access attempts，其中 $\eta$ 表示系统中同时存在过的最大包数，$D$ 表示 disruption 数量。本文摘要中把它作为主要比较对象：Re-Backoff 已经能做到常数吞吐量和 robustness，但效率常数小于本文的 $1/e$，协议也更复杂。Awerbuch, Richa, Scheideler 研究的是 single-hop wireless networks 中的 jamming-resistant MAC protocol，对手可以 jam 掉 $(1-\epsilon)$ 比例的时间步；他们的协议也通过反馈调节发送概率，目标是在 non-jammed time steps 上保持常数吞吐量。Bender, Kopelowitz, Pettie, Young 进一步从 energy 角度看问题。他们的协议能保证 expected constant throughput，每个设备发送 $O(1)$ 次、监听 $O(\log(\log^*N))$ 次。这个方向说明，通道利用率不是唯一指标；如果监听信道也有能量成本，那么本文“每个时隙都监听反馈”的假设还留下了后续问题。

和这些工作相比，本文的优势是把协议形式压得很简单：每个设备只维护一个 $p_i$，然后根据 silence、success、noise 做乘法更新。同时它把通道利用率提高到 $1/e-O(\epsilon)$，并且每个设备平均发送尝试次数是常数级的 $e+O(\epsilon)$，不再依赖 $n$ 或 $J$。

第三个方向是 $1/e$ 阈值和反馈强弱。本文的 $1/e$ 对 stateless algorithms 来说是最优的，但这不表示所有随机接入模型都不能超过 $1/e$。在泊松到达模型里，Mosely and Humblet 设计的算法可以达到约 $0.48776$ 的 throughput，超过 $1/e$。主论文还提到 Tsybakov and Mikhailov 有类似效率的早期结果，这篇我暂时没有查到原文，只先把它作为主论文中的引用保留。Pippenger 从信息论角度分析多路访问广播信道，说明如果协议只能区分有限种冲突类型，那么吞吐量不可能接近 1；但如果信道能报告准确的发送者数量，就可以接近满利用率。Goldberg, Jerrum, Kannan, Paterson 则研究反馈更弱的 backoff 和 acknowledgement-based protocols，证明 backoff protocols 的 capacity 至多为 0.42，acknowledgment-based protocols 的对应上界为 0.531。后来 Bender, Kopelowitz, Kuszmaul, Pettie 又研究了 without collision detection 的模型，也就是监听者不能区分空闲时隙和碰撞时隙。他们的结果说明，即使没有 collision detection，也可以在对抗性到达下做到常数 throughput；但如果加入 jamming adversary，collision detection 又会变得必要。这篇后续工作正好从反方向说明，本文依赖的三元反馈假设不是随便的技术细节，而是影响协议能力的核心条件。

所以本文的贡献不是说 $1/e$ 是所有模型的绝对极限，而是在“无状态、三元反馈、对抗性注入、可有 jammer”这一组限制下，用非常简单的 MWU 风格规则达到接近最优的效率。

## 5. 本文协议机制

本文协议分成初始化、发送、更新三个部分。

新设备被激活时，按照 initialization rule 设置

$$
p_i\leftarrow \epsilon^2.
$$

这里 $\epsilon$ 是算法步长。初始化成 $\epsilon^2$ 的直觉是，新来的包一开始不要把总冲突度突然推得太高，否则可能立刻制造大量冲突；同时它又不能太小，否则后面需要很久才能参与竞争。这个初始化值在第 3 节势函数分析里也会用到。

论文为了方便分析，先给出一个比较“人工”的 transmission rule：

$$
\begin{cases}
\text{以概率 } e^{-p_i} \text{ 保持静默},\\
\text{以概率 } p_i e^{-p_i} \text{ 发送数据包},\\
\text{以概率 } 1-(1+p_i)e^{-p_i} \text{ 制造噪音}.
\end{cases}
$$

这个规则看起来有点奇怪，因为设备会主动“制造噪音”。它的作用主要是让分析更干净：如果令

$$
c=\sum_i p_i,
$$

那么三种反馈的概率可以精确写成

$$
p_{sil}=e^{-c},
$$

$$
p_{suc}=ce^{-c},
$$

$$
p_{noi}=1-(1+c)e^{-c}.
$$

也就是说，反馈分布完全由总冲突度 $c$ 决定，而不依赖每个 $p_i$ 的具体分布。这样后面只需要分析 $c$ 或 $\gamma=\ln c$ 的变化。

真正的更新规则是本文最核心的部分：

$$
p_i \leftarrow
\begin{cases}
p_i e^\epsilon, & \text{silence},\\
p_i, & \text{success},\\
p_i e^{-\epsilon/(e-2)}, & \text{noise}.
\end{cases}
$$

这个规则的设计理由可以从 $c=1$ 看出来。当 $c=1$ 时，

$$
p_{sil}=1/e,\qquad p_{noi}=1-2/e=(e-2)/e.
$$

如果希望 $c=1$ 附近没有系统性偏移，那么 silence 带来的上调和 noise 带来的下调应该在期望上抵消。也就是

$$
\frac{1}{e}\cdot \epsilon+\frac{e-2}{e}\cdot\left(-\frac{\epsilon}{e-2}\right)=0.
$$

这解释了为什么噪音时的指数是 $-\epsilon/(e-2)$，而不是简单的 $-\epsilon$。

最后，论文说明前面的“制造噪音”其实不是实际协议必须做的事。更自然的版本是：

$$
\begin{cases}
\text{以概率 } e^{-p_i} \text{ 保持静默},\\
\text{以概率 } 1-e^{-p_i} \text{ 发送数据包}.
\end{cases}
$$

也就是设备不再主动制造噪音，而是把那部分概率也用来尝试发送。论文后面证明，这个 simpler transmission rule 不会让系统表现变差，反而从设备角度更合理。

## 6. 核心直觉

我目前对这篇论文的理解是：它不是让每个设备分别猜测系统规模，而是让所有设备通过同一个反馈信号共同调节一个总量

$$
c=\sum_i p_i.
$$

当 $c$ 太小时，系统大概率静默，说明设备们整体发得太少。此时所有设备都乘上 $e^\epsilon$，于是 $c$ 也乘上 $e^\epsilon$，向 1 靠近。当 $c$ 太大时，系统大概率噪音，说明整体发得太多。此时所有设备都乘上 $e^{-\epsilon/(e-2)}$，于是 $c$ 下降，也向 1 靠近。

因为更新是乘法的，所以论文改用

$$
\gamma=\ln c
$$

来分析。这样 $c$ 的乘法变化就变成 $\gamma$ 的加法变化：

$$
\gamma'\in\left\{\gamma+\epsilon,\gamma,\gamma-\frac{\epsilon}{e-2}\right\}.
$$

这使得 $\gamma$ 看起来像一个随机游走。不同的是，它不是普通的无偏随机游走，而是带有回到 0 的趋势。$\gamma<0$ 表示 $c<1$，总发送概率偏小，所以 silence 更容易出现，$\gamma$ 更容易被往上推；$\gamma>0$ 表示 $c>1$，冲突偏多，所以 noise 更容易出现，$\gamma$ 更容易被往下拉。

论文把这种直觉类比为 homesick random walk，也就是“想回家”的随机游走。这里的“家”就是 $\gamma=0$，也就是 $c=1$。如果 $\gamma$ 经常待在 0 附近，那么成功概率

$$
p_{suc}(\gamma)=e^\gamma e^{-e^\gamma}
$$

就会接近最大值 $1/e$。在 $\gamma=0$ 附近做泰勒展开，有

$$
p_{suc}(\gamma)=\frac{1}{e}-\frac{\gamma^2}{2e}-\frac{\gamma^3}{6e}+O(\gamma^4).
$$

这说明只要 $\gamma$ 偏离 0 不太多，效率损失就是二阶的小量。后面的势函数证明就是把这个直觉严谨化：即使某些时隙没有成功传输，也不能简单看作浪费，因为它们可能让 $\gamma$ 往更好的方向移动，从而为后面的成功“铺路”。
