一种用于斗地主的AI，功能强大，竞赛第一

使用的方法是强化传统的MC方法，使用深度神经网络，动作编码，并行actor来强化，说明MC方法在复杂空间上也能有好的结果。

斗地主两大困难。首先，有限制交流下，对于非完备信息空间，参赛者不仅要竞争还要合作。其次，信息空间巨大。

## 1.介绍 ##
与德州扑克这种游戏相比，斗地主因为三人竞争，状态复杂且信息不全，所以无法使用CFR，又因为动作空间多，且动作空间不容易抽象，所以使得搜索计算的成本很贵，如果像AlphaGo一样使用蒙特卡洛树来遍历，计算成本高。

已有的模型人类启发式搜索加Q-learning的Combination Q-Network (CQN)，效果差，基于贝叶斯的DeltaDou，有效果，但是计算资源大

提出了一种不需要任何人类知识的模型，DouZero通过深度神经网络、动作编码和取样增强了传统的蒙特卡罗方法。

这种算法有两种优势：

1.相比于DQN，不容易受高估偏差影响，因为其结果并非估计，而是真实采样的结果值。

2.通过将动作编码到矩阵中，可以自然的泛化不常见但是同一类的动作（这里的隐含条件是编码的时候会将动作空间按类规定编码方式），通过这一泛化可以不使用全局搜索，从而减少计算量。
DouZero的出现说明经典的蒙特卡罗方法可以在大规模和复杂的纸牌游戏中提供强大的结果（大规模空间，多智体，稀疏奖励，复杂行动空间，复杂编码模型，非完备信息），并可以实现竞争与合作
## 3. DMC ##
### 3.1 DMC ###
MC有几个特点：

1.结果为平均样本返回，或者像n-MC那样以γ为权重向结尾展望折现。

2.直接使用统计结果，偏差小，但是方差大。

3.一次取样直接运行到结束，不存在中途停止，中途停止是TD。

4.易于并行取样。

5.在应用时选取Q的max，一般为了平衡探索与开发会用ε-greedy或者其他启发式办法。

6.一般认为因为取样有限，又有采样结果利用率低，不能用于非完整链，且方差高的问题，不适合用于大空间。

DMC对MC进行改进主要在三个方面：

1.用神经网络代替q表

2.用MSE更新网络

3，并行采样

DMC很适合用于斗地主，因为斗地主运行链条完整，且容易并行化，编码的行动可泛化，因此可以用足够多的MC采样环节高方差。
### 3.2 与策略梯度类相比较 ###
策略梯度直接用梯度下降建模，并且在大动作空间中效果很好，但是不存在泛化效果，无法推理未取得的动作，因为采样有限，所以动作的泛化很有用。而DMC可以通过将动作特征作为输入，来泛化看不见的动作（为什么PG没有泛化效果？）

A2C的方法经过实验得知（q网络用语critic），actor仍然受大空间影响，实验证明不行。（常规MC与A2C有结合方法没，为什么这里的actor要受大空间影响？）
### 3.3 与DQN比较 ###
DQN有高估问题且不稳定，DMC无偏差且很稳定。另一方面DQN在稀疏奖励中迭代收敛的很慢，而NC收敛和长度无关。最后，DQN要在更新中找最大值，对于大动作空间来说计算量会很大。

（这一块的主要问题就在于DMC的那个泛化能力上，为什么在这种动作编码的下他会有泛化能力，这对于有规律的大动作空间很有帮助）
## 4.DouZero System ##
### 4.1 动作空间编码与模型 ###
编码模式如下：

输入是状态和动作的连接表示。

对于每个15×4卡矩阵，首先将矩阵压平成一个大小为60的一维向量。

删除6个总是零的条目，因为只有一对大小王，卡矩阵被转换为一个大小为54的独热码。

使用独热码来表示其他两个玩家当前的手卡。对于农民，使用一个大小为17的向量，其中每个条目对应于当前状态下的手卡的数量。对于地主来说，矢量的大小是20。类似地，使用一个15维向量来表示当前状态下的炸弹数量。

对于历史出牌，考虑最近的15个动作，并连接每三个连续动作的表示；也就是说，历史出牌被编码到一个5×162矩阵中，并被输入到一个LSTM中，使用最后一个单元格中的隐藏表示来表示历史。如果历史上的出牌少于15次，我们就使用零矩阵来表示缺失。

具体矩阵编码内容如下图：
![image](/非完备信息相关论文记录/images/a1.png)
对于q-网络，LSTM用于编码历史信息，并且输出与其他状态/动作特征连接起来，最后，我们使用隐藏大小为512的6层MLP来生成q值，该网络根据动作和状态的连接表示来预测给定的状态-动作对的值，网络在这里的作用相当于一个critic。模型结构如下图：
![image](/非完备信息相关论文记录/images/b1.png)
### 4.2 actor并行采样 ###
![image](/非完备信息相关论文记录/images/c1.png)
![image](/非完备信息相关论文记录/images/d1.png)
## 5. 实验结果 ##
比较DouZero，DeltaDou，SL（人类监督基线），传统方法（RHCP-v2，RHCP，RLCard），CQN，Random他们的WP,ADP指标和Elo值，实验结果显示效果很好，并且虽然斗地主本身具有运气效果，但是相比于带搜索的DeltaDou表现更高。
加上抢地主这一环节的网络，依旧很好。
加入TD没有改善，说明TD在稀疏奖励中没什么帮助，实验证明A2C失败，因为仅仅像critic加入动作特性不足与解决复杂动作空间问题。
模型运行初期与人类的运行结果相似度加强，后期相似度减少，说明学到了新东西。
## 6. related work ##
非完备信息游戏-search：CFR是扑克领域领先算法，有许多变种。但是traversing the game tree是计算密集问题，要遍历数空间，除此之外往期的研究集中于零和设计，对于合作问题研究有限，因此CFR难以用在此处。

非完备信息游戏-RL：RL可以在扑克游戏中实现竞争，RL基于采样，可以很容易的推广到大规模游戏中，而RL+搜索在众多游戏中被证明有用，比如DeltaDou首先推断隐藏信息，然后使用MCTS（RL+搜索）。这种方法计算昂贵，并且有时需要人类先验知识。
## 7. 展望与未来 ##
未来将探讨以下方向。

计划尝试其他神经结构，如卷积神经网络和ResNet。

为强化学习在循环过程中加入选择（involve bidding in the loop for reinforcement learning0。

在DouZero中研究如何平衡RL和搜索。

探索off-policy方法，提高样本效率（现在是一次一用）。

尝试明确为农民合作建模。

尝试可扩展的框架，如SEEDRL。

测试蒙特卡罗方法在其他任务上的适用性

（存留问题，代码，作者没给出整个程序的运行过程）
