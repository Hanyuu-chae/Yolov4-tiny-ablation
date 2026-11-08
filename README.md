# Yolov4-tiny-ablation

# 数据集来源与说明
* **数据集名称**：PASCAL VOC 2007(Pattern Analysis, Statistical Modelling and Computational Learning)
* **官方网址**：[http://host.robots.ox.ac.uk/pascal/VOC/voc2007/](http://host.robots.ox.ac.uk/pascal/VOC/voc2007/)
* **数据分布与规模**：
  * 包含 20 个常见目标类别（如人、车、飞机、动物、日常用品等）。
  * 训练集/验证集：`VOC2007 Trainval`（共计 5,011 张图像）。
  * 测试集：`VOC2007 Test`（共计 4,952 张图像，用于最终 mAP 性能评估）。
* **实验配置**：采用 `VOC2007 Trainval` 进行模型训练，并在 `VOC2007 Test` 上进行严格的评估。

# 参考开源项目与声明
* **基准框架参考**：本项目底层网络架构、数据加载器（Dataloader）及损失函数计算逻辑，参考并改编自 GitHub 上的开源项目**[bubbliiiing/yolov4-tiny-pytorch](https://github.com/bubbliiiing/yolov4-tiny-pytorch)**。
* **自主二次开发内容**：
  1. 重构并实现了 `nets/attention.py` 模块，集成了 SE-Net、ECA、CA、CBAM 等注意力机制的类定义与前向传播逻辑。
  2. 在 `nets/yolo.py` 与 `CSPdarknet53_tiny.py` 中嵌入了灵活的模块挂载接口，通过超参数 `phi` 实现切换。
  3. 针对CPU训练环境，进行了超参数的调优适配（固定输入分辨率 `256×256`、Batch Size 设为 4、使用 SGD 优化器）。

# 核心创新与改进模块
本项目探讨了以下四大类改进模块：
1. **SE-Net (Squeeze-and-Excitation)**：通过显式建模通道之间的相关性，利用全局池化与全连接学习通道权重，自适应地重标定特征响应。
2. **ECA-Net (Efficient Channel Attention)**：摒弃了 SE 中的降维全连接层，通过一维卷积（`Conv1D`）实现局部跨通道交互，在极低参数量下实现通道增强。
3. **CA (Coordinate Attention)**：将通道注意力与精确的位置/方向编码结合，将水平和垂直方向的聚合特征分别编码，使网络能够同时关注通道与长距离位置信息。
4. **CBAM (Convolutional Block Attention Module)**：串联通道注意力与空间注意力子模块，从“是什么”和“在哪里”两个维度对特征图进行精细化联合筛选。
5. **DWConv & GELU**：引入深度可分离卷积降低计算复杂度，并探索现代连续激活函数对非线性拟合能力的提升。

# 项目目录结构
```text
YOLOv4-Tiny-Ablation-Study/
├── nets/
│   ├── yolo.py                 # YOLOv4-Tiny 主网络结构及检测头
│   ├── CSPdarknet53_tiny.py    # 轻量级主干网络（支持注意力模块嵌入）
│   └── attention.py            # SE、ECA、CA、CBAM 注意力模块实现
├── utils/
│   ├── dataloader.py           # VOC 数据集加载与 Mosaic 等数据增强
│   ├── callbacks.py            # Loss 记录、TensorBoard 及 mAP 评估回调
│   └── utils_fit.py            # 训练与验证单轮迭代逻辑
├── VOCdevkit/
│   └── VOC2007/                # PASCAL VOC 2007 数据集存放目录
├── train.py                    # 主训练与评估入口脚本
├── predict.py                  # 单张图像/视频推理预测脚本
├── get_map.py                  # 测试集 mAP 计算与评测脚本
└── README.md                   # 项目说明文档
