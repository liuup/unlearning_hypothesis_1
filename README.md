# unlearning_hypothesis_1

本项目是对论文`Machine Unlearning Fails to Remove Data Poisoning Attacks. ICML 2024 Spotlight`中`Section 6`, `Hypothesis 1`的另一个视角的证明实验。

# 思路
> Hypothesis 1: Poison samples cause a large model shift, which cannot be mitigated by approximate unlearning.  

也就是说，毒害数据使得模型的偏移更大，而且没法用遗忘算法来缓和。作者在原论文的附录C中给出了一部分实验，但是我想用另一个视角来探讨毒害数据是否造成了比较大的模型偏移。  
因此，受`Backdoor Federated Learning By Poisoning Backdoor-Critical Layers. ICLR 2024`启发，我从模型间和层间分别对模型的偏移量进行测量。


# 实验设计
1. 数据集：CIFAR10，抽取特定比例的数据来作为毒害数据. 抽取后, 将这部分数据与trigger按比例混合.
2. 模型：五层CNN，具体可查看代码.
3. 训练：设置三个初始参数完全相同的模型base_model, model1, model2. 在训练时，base_model作为基准不参与训练，model1在正常数据上训练，model2在毒害数据上训练。
4. 指标：余弦相似度，L1距离
5. 模型间测量：对于这三个模型，两两之间分别测量余弦相似度和L1距离，可以得到6个指标
6. 层间测量：对于这三个模型，两两之间分别测量层之间的余弦相似度和L1距离，可以得到3 * 5 * 2 = 30个指标



# 超参数
1. batch_size = 128
2. ratio = 0.1 # 控制抽取的百分比比例  
3. alpha = 0.5  # 控制融合的比例，比例越高，原始图片的信息越多  
4. CrossEntropyLoss  
5. Adam(lr=0.0001, weight_decay=0.01)  



# 实验
详细实验结果：https://wandb.ai/akaabel/hypothesis/runs/541uhcvk?nw=nwuserakaabel


初步结论：
1. 模型间偏移：对于model1和model2，在训练了32个epoch后，cossim逐步减小到0.76754，L1增大到5.3654，意味着两个模型之间的距离越来越大，通过对比model1_base_cossim和model2_base_cossim，可以发现model2的偏移速度比model1更大。
2. 层间偏移：通过观察六张图表的cossim和L1指标，可以发现cnn.10.weight的偏移量最大，其次是cnn.6.weight。偏移量最小的是输入端的cnn.0.weight（对于本次使用的CNN，从输入端到输出端，层的名称依次为`'cnn.0.weight', 'cnn.3.weight', 'cnn.6.weight', 'cnn.10.weight', 'cnn.13.weight'`，不考虑bias）。
3. 越靠近模型中间的层偏移量越大.(还不太确定，或许还能再研究一下)

# 不足之处
1. 在实验设计时，未遵循原始论文的实验步骤，具体表现为：(1)模型的选取与原论文不同，原论文对于图像分类任务用的是ResNet18，我只用了一个CNN.(2)毒害数据的生成方式不同.(3)毒害数据的混合比例不同.
2. 使用余弦相似度来测量模型的偏移，这种方法的有效性还有得考量

# What's Next
遗忘+知识蒸馏，但似乎只考虑了线性层的最后一层，还没细看
> Kim, Hyunjune, Sangyong Lee, and Simon S. Woo. "Layer Attack Unlearning: Fast and Accurate Machine Unlearning via Layer Level Attack and Knowledge Distillation." Proceedings of the AAAI Conference on Artificial Intelligence. Vol. 38. No. 19. 2024.