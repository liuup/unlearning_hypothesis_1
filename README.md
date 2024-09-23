# unlearning_hypothesis_1

本项目是对论文`Machine Unlearning Fails  to Remove Data Poisoning Attacks. ICML 2024 Spotlight`中`Section 6`, `Hypothesis 1`的另一个视角的证明实验。

# 思路
`Hypothesis 1`: Poison samples cause a large model shift, which cannot be mitigated by approximate unlearning.  
也就是说，毒害数据使得模型的偏移更大，而且没法用遗忘算法来缓和。作者在原论文的附录C中给出了一部分实验，但是我想用另一个视角来探讨毒害数据是否造成了比较大的模型偏移。  
受`Backdoor Federated Learning By Poisoning  Backdoor-Critical Layers. ICLR 2024`启发，我从模型间和层间分别对模型的偏移量进行测量。


# 实验设计
1. 数据集：CIFAR10，抽取特定比例的数据来作为毒害数据. 抽取后, 将这部分数据与trigger按比例混合.
2. 模型：五层CNN，具体可查看代码.
3. 训练：设置三个初始参数完全相同的模型base_model, model1, model2. 在训练时，base_model作为基准不参与训练，model1在正常数据上训练，model2在毒害数据上训练。
4. 指标：余弦相似度
5. 模型间测量：测量model1和base_model之间的指标model1base_cossim，测量model2和base_model之间的指标model2base_cossim.
6. 层间测量：测量model1和base_model之间，各自权重层的偏移量，可以得到五个指标。同理也测量model2与base_model之间的偏移量。



# 超参数
1. batch_size = 64  
2. ratio = 0.1 # 控制抽取的百分比比例  
3. alpha = 0.5  # 控制融合的比例，比例越高，原始图片的信息越多  
4. CrossEntropyLoss  
5. Adam(lr=0.0001, weight_decay=0.001)  



# 实验
详细实验结果：https://wandb.ai/akaabel/hypothesis/runs/ai7rkzxo?nw=nwuserakaabel


初步结论：
1. 模型间偏移：在异常数据上训练的model2相对于在正常数据上训练的model1，会有明显的偏移。在训练了32epoch后，相比于model1，余弦相似度相差了`4.43%`((0.42149-0.40283)/0.42149)，但是还不能确定这个相差的比例是否比较大.
2. 层间偏移：cnn.10.weight的偏移量相比于base_model最大，训练结束后，model2相比于model1，本层的指标相差了`9.39%`((0.23805-0.21569)/0.23805).
3. 越靠近模型中间的层偏移量越大.(这可能是一个有意思的结论，或许还能再研究一下)

# 不足之处
1. 在实验设计时，未遵循原始论文的实验步骤，具体表现为：(1)模型的选取与原论文不同，原论文对于图像分类任务用的是ResNet18，我只用了一个CNN.(2)毒害数据的生成方式不同.(3)毒害数据的混合比例不同.
2. 使用余弦相似度来测量模型的偏移，这种方法的有效性还有得考量