一个有点老的会议报告，论文研究了深度学习算法如何在部分可观测(PO)网格地图中导航。
实现了流行的强化学习体系结构，包括价值迭代网络(VIN)、策略梯度(PG)和DQN来解决这一任务。
最终证明了VIN的性能很强，而其他深度强化学习算法不能推广。
并将VIN于A*，D* 算法结果相比较。

## 贡献 ##
给出了一个基准的po网格数据集，证明VIN在po中效果良好。

## 近来工作 ##
传统解决po问题的算法是A* 算法的变体D* 算法，通过将未知空间转换为自由空间，现在用深度强化学习的办法

## 模型结构 ##
![image](/非完备信息相关论文记录/images/a2.png)
与传统的VIN相似，用了一个规划模块。
## 实验结果 ##
设置一个网格环境，将fo环境下的A* 算法作为基准，计算po环境下D*，VIN等算法与基准的平均路径差，其中DQN和PG无法实现搜索，VIN是通过模仿学习进行训练的，缺乏显式的规划模块可能是另外两个不成功的原因之一
