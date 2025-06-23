# Hallucination-Attribution
#友情说明：本项目源于https://github.com/TianyunYoung/Hallucination-Attribution.git
#本着复现论文的方向去的，至于创建仓库的目的是因为课程老师需要我们创建一个github链接储存我们的代码
#数据集来源：（二选一即可）
1、官方链接：https://cocodataset.org/#download
2、百度网盘链接：
COCO2014
链接: https://pan.baidu.com/s/1XiNE3M-MzENWYKEUm2r7cg 
提取码: 5m60

## 简介
Hallucination-Attribution是一个用于研究和缓解多模态大型语言模型(MLLMs)中幻觉现象的项目。项目提供了一系列方法来识别、归因和减轻视觉语言模型在生成描述时产生的幻觉内容。

## 背景与动机
多模态大型语言模型在处理图像和生成文本描述方面取得了显著进展，但它们往往会生成包含图像中不存在对象或错误描述的"幻觉"内容。此项目旨在：
1. 识别负责产生幻觉的注意力头(attention heads)
2. 开发对模型进行干预的方法以减少幻觉
3. 评估干预方法的有效性

## 项目结构
```
Hallucination-Attribution/
├── LLaVA/                        # LLaVA模型相关代码
│   ├── llava/                    # 核心模型代码
│   ├── scripts/                  # 脚本文件
│   ├── eval_scripts/             # 评估脚本
│   ├── bash_scripts/             # Bash脚本
│   └── results/                  # 结果存储目录
├── baselines/                    # 基准模型
│   ├── transformers-4.37.2/      # Transformers库
│   ├── minigpt4/                 # MiniGPT-4模型
│   ├── decoder_zoo/              # 解码器集合
│   ├── models/                   # 模型文件
│   ├── eval_scripts/             # 评估脚本
│   ├── eval_configs/             # 评估配置
│   ├── bash_scripts/             # Bash脚本
│   └── results/                  # 结果存储目录
├── newer_models/                 # 新模型实现
│   ├── eval_caption.py           # 用于评估模型生成的图像描述
│   ├── intervention_decoding.py  # 干预解码实现
│   └── decoding.sh               # 解码脚本
├── dataset/                      # 数据集
│   ├── coco/                     # COCO数据集
│   └── nocaps/                   # NoCaps数据集
└── figs/                         # 可视化图表
    ├── attention_bias.png        # 注意力偏差可视化
    ├── halhead.png               # 幻觉头可视化
    ├── nonhalhead.png            # 非幻觉头可视化
    ├── text_reweight.png         # 文本重新加权可视化
    ├── img_reweight.png          # 图像重新加权可视化
    └── js_div_in_training.png    # 训练过程中的JS散度
```

## 安装与环境配置
### 依赖项
项目使用Conda进行环境管理，主要依赖项在`baselines/environment.yml`文件中定义：

```bash
# 创建并激活环境
conda env create -f baselines/environment.yml
conda activate baselines
```

### LLaVA模型安装
```bash
cd LLaVA
pip install -e .
```

## 使用方法

### 1. 识别幻觉头
使用以下命令来识别和分析模型中的幻觉头：

```bash
cd LLaVA
bash bash_scripts/attribute.sh
```

此脚本将：
1. 为COCO数据集图像生成描述
2. 评估生成的描述并提取幻觉和非幻觉对象
3. 识别负责产生幻觉的注意力头

### 2. 使用干预解码来减少幻觉
对于较新的模型（如Chameleon和Llama-3.2-11B-Vision），可以运行：

```bash
cd newer_models
bash decoding.sh
```

此脚本通过在解码过程中干预特定注意力头来减少幻觉。

### 3. 评估模型输出
可以使用以下命令评估模型在图像描述生成任务上的表现：

```bash
python newer_models/eval_caption.py \
    --model-path [MODEL_PATH] \
    --image-folder [IMAGE_FOLDER] \
    --caption_file_path [CAPTION_FILE] \
    --answers-file [OUTPUT_FILE] \
    --dataset [DATASET_NAME] \
    --prune  # 启用干预解码
```

## 主要技术与方法

### 1. 幻觉归因
项目采用了零消融(zero ablation)方法来识别负责幻觉的注意力头。通过观察消融特定注意力头对模型输出的影响，我们可以识别出对幻觉生成贡献最大的头。

### 2. 干预技术
项目实现了以下干预技术：
- **选择性消融**：在推理过程中选择性地禁用幻觉头
- **注意力重新加权**：调整图像和文本令牌的注意力权重
- **自适应消融**：根据注意力模式动态调整干预强度

## 实验结果
实验表明，通过适当的干预，可以显著减少模型生成的幻觉内容，同时保持描述的质量和准确性。详细结果可以参考`figs/`目录下的可视化图表。

## 支持的模型
- LLaVA系列模型
- Chameleon 7B/30B
- Llama-3.2-11B-Vision

## 致谢与参考
该项目受到了多项幻觉研究和视觉语言模型研究的启发。感谢COCO和NoCaps数据集提供用于评估的图像和标注。 
