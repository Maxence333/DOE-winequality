# Wine Quality Optimization via Experimental Design

**红葡萄酒质量优化试验设计项目**

[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 项目简介

本项目利用**试验设计（Design of Experiments, DoE）**方法，对红葡萄酒的理化指标与感官质量之间的关系进行建模与优化。基于UCI公开的Wine Quality数据集，我们先后采用**均匀设计（Uniform Design）** 和**响应曲面设计（Central Composite Design, CCD）**，识别关键影响因子、建立预测模型，并找到使质量评分最大化的工艺参数组合。

项目完整实现了“问题定义 → 试验设计 → 数据分析 → 模型改进 → 最优条件求解”的闭环流程，展示了试验设计在工业配方/工艺优化中的实际应用。

## 目录

- [背景与目标](#背景与目标)
- [数据来源](#数据来源)
- [方法流程](#方法流程)
- [代码结构](#代码结构)
- [环境依赖](#环境依赖)
- [快速开始](#快速开始)
- [主要结果](#主要结果)
- [局限性及改进方向](#局限性及改进方向)
- [许可证](#许可证)
- [参考文献](#参考文献)

## 背景与目标

葡萄酒的最终质量由多种理化因子（酸度、糖分、二氧化硫、酒精浓度等）共同决定。传统“单因子试验”耗时且无法捕捉交互作用。本项目旨在：

1. **筛选**出对红葡萄酒质量有显著影响的关键因子；
2. 建立**高精度**的预测模型（优于简单线性回归）；
3. 求解**最优工艺条件**，获得比历史数据更高的预期质量评分。

## 数据来源

- **数据集**：Wine Quality Dataset (Red Wine)  
- **发布者**：P. Cortez et al., University of Minho, 2009  
- **链接**：[UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/wine+quality)
- **规模**：1599个样本，11个理化因子 + 1个质量响应（3–10分）

> 注：原始数据仅包含观测样本，并非按照特定试验设计收集。本项目以之作为“历史数据”起点，并**模拟**了第二次主动设计试验的响应。

## 方法流程

整体工作流如下图所示：

```
[历史数据] → [均匀设计抽样] → [线性逐步回归] → [发现模型缺陷]
       ↓
[固定次要因子] → [响应曲面设计(CCD)] → [二阶回归建模] → [岭分析寻优]
```

### 具体步骤

| 阶段 | 方法 | 目的 |
|------|------|------|
| 第一次试验 | 均匀设计（32点） + 逐步线性回归 | 初步筛选显著主效应因子 |
| 诊断改进 | 残差分析、VIF、模型不足讨论 | 发现缺失交互项和曲率 |
| 第二次试验 | 中心复合设计（27点） + 二次响应曲面 | 拟合二阶模型，捕捉交互与曲率 |
| 优化 | 岭分析（Ridge Analysis） | 在因子空间内最大化质量评分 |

## 代码结构

```
wine_doe_project/
├── data/
│   ├── winequality-red.csv          # 原始数据（UCI）
│   ├── uniform_design_samples.csv   # 第一次试验32个样本点
│   └── ccd_design_points.csv         # 第二次试验27个设计点（含模拟响应）
├── notebooks/
│   ├── 1_UniformDesign_Analysis.ipynb   # 均匀设计建模
│   ├── 2_CCD_Design_Generation.ipynb    # 生成CCD表
│   └── 3_ResponseSurface_Optimization.ipynb # 二次模型拟合与寻优
├── src/
│   ├── doe_utils.py                 # 均匀/CCD设计辅助函数
│   ├── regression.py                # 逐步回归、二次项构造
│   └── optimization.py              # 岭分析、约束优化
├── results/
│   ├── first_model_summary.txt      # 第一次回归结果
│   ├── second_model_summary.txt     # 第二次响应曲面结果
│   └── optimal_conditions.json      # 最优参数及预测值
├── README.md
├── requirements.txt
└── LICENSE
```

## 环境依赖

- Python 3.8+
- 核心库：
  - `numpy`, `pandas` – 数据处理
  - `scipy`, `pyDOE2` – 试验设计表生成（CCD、拉丁超立方）
  - `statsmodels`, `scikit-learn` – 回归分析、特征选择、优化
  - `matplotlib`, `seaborn`, `plotly` – 可视化
  - `jupyter` – 交互式分析

安装依赖：

```bash
pip install -r requirements.txt
```

`requirements.txt` 内容示例：
```
numpy==1.24.3
pandas==2.0.1
scipy==1.10.1
pyDOE2==1.3.0
statsmodels==0.14.0
scikit-learn==1.2.2
matplotlib==3.7.1
seaborn==0.12.2
jupyter==1.0.0
```

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/yourusername/wine_doe_project.git
cd wine_doe_project
```

### 2. 安装依赖

```bash
pip install -r requirements.txt
```

### 3. 运行分析（方式一：Jupyter Notebook）

```bash
jupyter notebook notebooks/1_UniformDesign_Analysis.ipynb
```

依次执行三个 notebook 即可复现完整流程。

### 4. 运行分析（方式二：Python脚本）

```python
from src.doe_utils import generate_uniform_design
from src.regression import stepwise_fit
from src.optimization import ridge_optimize

# 加载数据
df = pd.read_csv('data/winequality-red.csv')

# 第一次试验：均匀设计+逐步回归
design_ud = generate_uniform_design(df, n_samples=32)
model1, summary1 = stepwise_fit(design_ud, response='quality')

# 第二次试验：CCD + 二次模型
# ... (详见notebook)
```

## 主要结果

### 第一次模型（线性）

\[
\hat{y} = -1.23 - 0.92x_2 + 0.38x_{11} + 0.24x_{10} - 0.18x_8
\]

- \(R^2 = 0.62\)，RMSE = 0.73  
- 显著因子：挥发性酸度（负）、酒精（正）、硫酸盐（正）、密度（负）

**缺陷**：未考虑交互作用，预测精度低，无法外推最优。

### 第二次模型（响应曲面）

\[
\begin{aligned}
\hat{y} = 5.82 - 0.78x_2 + 0.31x_{11} + 0.18x_{10} - 0.23x_8 \\
- 0.21x_2x_{11} + 0.12x_{10}x_{11} - 0.09x_2^2 - 0.06x_{11}^2
\end{aligned}
\]

- \(R^2 = 0.89\)，RMSE = 0.21  
- 交互项 \(x_2 \times x_{11}\) 显著，表明低挥发性酸度配合高酒精可显著提升质量。

### 最优工艺条件

| 因子 | 最优值 | 单位 |
|------|--------|------|
| 挥发性酸度 | 0.28 | g/dm³ |
| 密度 | 0.9935 | g/cm³ |
| 硫酸盐 | 1.62 | g/dm³ |
| 酒精 | 12.8 | % vol |

**预期最大质量评分：7.96**（10分制），高于原始数据最高分（8分）。

## 局限性及改进方向

### 当前局限性

1. **第二次试验响应为模拟数据**：依赖于第一次模型的预测值加噪声，并非真实湿化学试验，可能放大初始偏差。
2. **因子降维主观**：将7个次要因子固定为常数，可能丢失与主因子的高阶交互作用。
3. **质量评分主观性**：原始评分由品尝专家给出，未考虑评委间差异。
4. **设计范围限制**：CCD设计基于原始数据的可行域，未必覆盖真正的全局最优（如酒精>14.9%）。

### 未来改进

- ✅ 执行真实发酵试验，收集实际响应数据。
- ✅ 使用**贝叶斯优化**或**D-最优设计**处理因子约束。
- ✅ 引入**区组设计**（以评委为区组）消除个体偏差。
- ✅ 考虑**稳健参数设计**，同时优化质量均值和抗噪声能力。

## 许可证

本项目采用 [MIT 许可证](LICENSE)。数据集遵循UCI数据集的使用条款（无限制，引用即可）。

## 参考文献

1. Cortez, P., Cerdeira, A., Almeida, F., Matos, T., & Reis, J. (2009). Modeling wine preferences by data mining from physicochemical properties. *Decision Support Systems*, 47(4), 547-553.  
2. Fang, K. T., Lin, D. K., & Liu, M. Q. (2003). Uniform design: Theory and application. *Technometrics*, 45(3), 217-230.  
3. Box, G. E. P., & Wilson, K. B. (1951). On the experimental attainment of optimum conditions. *Journal of the Royal Statistical Society B*, 13(1), 1-45.

---

**作者**：[Your Name]  
**联系方式**：[your.email@example.com]  
**项目日期**：2026-04-11

> 本README随代码仓库一并提交。如有疑问，欢迎提交Issue或联系作者。