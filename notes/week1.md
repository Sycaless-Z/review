# review撰写第1周笔记：6.16 - 6.21

目标：先把 review 的开头搭起来，重点是论文在解决什么问题、为什么要解决、模型怎么定义。证明和相关工作先不急着写细。

## review 框架

暂时按这个顺序写：

1. 论文基本信息与核心结论
2. 问题背景和重要性
3. 模型定义
4. 相关工作
5. 协议机制
6. 核心直觉
7. 正确性和效率证明
8. 贡献、局限、开放问题

这周先写了 1-3。

## 读论文开头的理解

contention resolution 暂时翻成“冲突消解”。这个词在别的地方也能指锁竞争、数据库事务之类的问题，但这篇论文里主要就是多个设备抢共享信道。

最简单的图像：

- 没人发：silence
- 一个人发：success
- 多个人发：noise

要解决的是：设备不知道系统里有多少人，也没有中心调度，怎么靠本地随机和信道反馈让成功时间隙尽量多。

## 为什么是 $1/e$

如果有 $n$ 个设备，并且大家都知道 $n$，每个人用 $1/n$ 的概率发送，则成功概率是

$$
n\cdot \frac{1}{n}\left(1-\frac{1}{n}\right)^{n-1}
=\left(1-\frac{1}{n}\right)^{n-1}
\to 1/e.
$$

所以 $1/e$ 是一个很自然的基准。本文不是在已知 $n$ 的情况下做到这个，而是在设备不知道 $n$、还有对抗性注入的情况下，用反馈慢慢调到这个状态附近。

## 目前觉得最重要的变量

不是单个设备的 $p_i$，而是

$$
c=\sum_i p_i
$$

也就是总冲突度。$c$ 太小会静默，$c$ 太大会冲突，$c$ 接近 1 最好。

因为更新是乘法，所以取

$$
\gamma=\ln c.
$$

这样听到 silence 时 $\gamma$ 加 $\epsilon$，听到 noise 时 $\gamma$ 减 $\epsilon/(e-2)$，比较像一个被拉回 0 的随机游走。

这里一开始我有个疑问：所有设备都更新，为什么 $\gamma$ 不是变很多倍？后来想明白了，因为 $c$ 是求和，所有 $p_i$ 同乘 $k$ 后，$c$ 也只是同乘 $k$：

$$
c'=\sum_i kp_i=kc.
$$

## 模型词汇

multiple-access channel：共享信道。

ternary feedback：三元反馈，区分 silence / success / noise。

packet injection：数据包不是一开始全部出现，可以后续被注入。

jammer：可以故意制造 noise 的干扰者。

channel utilization：长期成功传输比例。

stateless：不是完全没有变量，而是不记录复杂历史。设备只保留当前 $p_i$，再根据当前反馈更新。

## 相关工作待查

- Aldous：BEB 在泊松注入下不稳定。
- Bender 等：BEB 稳定 / 不稳定阈值，同步到达时的复杂度。
- Bender, Fineman, Gilbert, Young：jammer 模型下的稳定协议。
- Awerbuch 等：MWU 类型更新和 jammer。
- Bender, Kopelowitz, Pettie, Young：能量受限模型。
- Mosely & Humblet、Tsybakov & Mikhailov：泊松注入下超过 $1/e$。
- Pippenger：如果反馈更强，能做到接近 $1-o(1)$。

## 后面要补

下周先把 related work 查清楚，再写协议机制。证明部分需要先整理势函数，不然容易只是照着论文翻译。
