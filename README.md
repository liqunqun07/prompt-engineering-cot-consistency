# 中医辨证系统实现

本项目围绕慢性淋巴细胞白血病患者症状描述，构建一个基于大语言模型的中医辨证辅助流程。系统读取症状文本，预测对应的证型和治法，并通过不同提示词策略对结果进行对比。

## 文件说明

- `main.ipynb`：主要实验 notebook，包含提示词设计、模型调用、结果解析和评分流程。
- `train_data.csv`：包含症状、标准证型和标准治法的训练/评估数据。
- `无修改.csv`：Zero-Shot 基线输出。
- `修改_2_评分.csv`：加入置信度与自评分后的输出。
- `修改_3_给知识.csv`：加入领域知识、Few-Shot 示例与 CoT 引导后的输出。
- `修改_4_投票.csv`：在 Few-Shot + CoT 基础上加入 Self-Consistency 投票后的输出。
- `中医项目说明.docx`：项目过程说明文档。

## 方法概述

实验对比了四类提示工程策略：

1. Zero-Shot：只提供角色、任务和 JSON 输出格式。
2. 自检增强：要求模型额外输出理由、置信度和自评分，以约束模型自我检查。
3. Few-Shot + CoT：补充中医辨证知识和示例，并引导模型按症状到证型、治法的路径分析。
4. Few-Shot + CoT + Self-Consistency：对同一症状多次调用模型，统计证型和治法投票结果，选择出现频率最高的答案。

模型输出通过 `parse_response()` 解析为结构化 JSON，再由 `predict()` 返回证型和治法。评估部分使用文本嵌入和余弦相似度衡量预测结果与标准答案的语义接近程度，同时也可用证型精确匹配作为简单参考。

## 结果摘要

从保存的 CSV 结果看，单纯 Zero-Shot 输出不稳定，证型名称和治法表述容易偏离标准。加入领域知识、Few-Shot 示例和 CoT 后，证型精确匹配从基线的 `0/17` 提升到 `7/17`；加入 Self-Consistency 投票后，证型精确匹配仍为 `7/17`，但结果更稳定。

治法属于自由文本，直接精确匹配较严格，因此项目采用语义向量余弦相似度作为主要评分方式。notebook 中一次完整评估的语义综合得分为 `87.36` 分。

## 运行说明

安装依赖：

```bash
pip install openai requests numpy scikit-learn
```

运行前需要在本地设置 API 凭据，仓库中不保存密钥：

```bash
export BAIDU_LLM_API_KEY="your_llm_key"
export BAIDU_EMBEDDING_API_KEY="your_embedding_key"
```

然后打开并运行 `main.ipynb`。如需更换模型接口，可额外设置：

```bash
export BAIDU_LLM_BASE_URL="https://aistudio.baidu.com/llm/lmapi/v3"
```
