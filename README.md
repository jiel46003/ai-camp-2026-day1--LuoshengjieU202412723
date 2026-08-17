# D1：解释泰坦尼克号预测错误

一个可复现的 Titanic 二分类实验：比较**多数类基线**与**固定随机种子随机森林**，在同一个留出测试集上公平比较，并输出真实错误乘客记录，用于博物馆教育展项。

## 1. 问题

- **使用者**：准备历史展览、希望解释数据和模型局限的博物馆教育团队。
- **真实输入**：Kaggle Titanic Dataset 的 `train.csv`，891 行、12 列真实乘客记录。
- **输出**：同一测试集上的基线与候选指标（准确率、精确率、召回率、F1、混淆矩阵），以及真实错误乘客记录 `errors.csv`。
- **边界**：这是历史观察数据分析，不是现代救援分配工具，也不能证明某个特征导致生存。

## 2. 真实数据

- 来源：https://www.kaggle.com/datasets/hesh97/titanicdataset-traincsv
- 位置：`data/raw/train.csv`
- 校验命令：`python train.py --check-data`
- 预期校验输出：`REAL DATA CHECK PASSED`，rows: 891，survived_counts: {0: 549, 1: 342}

> 必须使用上述指定真实数据。禁止用 AI 生成"相似数据"替代；`data/raw/train.csv` 不提交到仓库。

## 3. 环境

- Python 3.12
- 依赖见 `requirements.txt`（pandas、scikit-learn、numpy）

```powershell
python -m pip install -r requirements.txt
```

## 4. 运行命令

```powershell
# 0) 确认工作目录包含 train.py、tests/ 和 data/raw/train.csv
Get-Location
Get-ChildItem

# 1) 数据校验（必须通过再继续）
python train.py --check-data

# 2) 运行主程序：基线 + 随机森林候选，输出 metrics.json 和 errors.csv
python train.py

# 3) 运行测试
python -m unittest discover -s tests -v
```

## 5. 预期输出

`python train.py --check-data`：

```
REAL DATA CHECK PASSED
rows: 891
columns: 12
survived_counts: {0: 549, 1: 342}
missing_age: 177
missing_cabin: 687
missing_embarked: 2
```

`python train.py` 关键结果（同一 223 行测试集，seed=42）：

| 指标 | 基线 | 候选 |
| --- | ---: | ---: |
| 准确率 | 0.614 | 0.753 |
| 幸存者召回率 | 0.000 | 0.640 |
| 幸存者 F1 | 0.000 | 0.667 |

- 输出文件：`metrics.json`（完整指标与混淆矩阵）、`errors.csv`（55 条真实错误行：PassengerId、Name、特征、真实/预测标签）。

`python -m unittest discover -s tests -v` 预期：3 个测试全部 `ok`，以 `OK` 结束。

## 6. 代码结构

- `train.py`：数据加载与校验、特征/目标构造、固定划分（test_size=0.25, seed=42, 分层）、预处理流水线（数值中位数填补、类别众数填补 + OneHot）、多数类基线 `DummyClassifier(strategy="most_frequent")`、候选 `RandomForestClassifier(n_estimators=200, random_state=42)`。
- `tests/test_pipeline.py`：预处理缺失值处理、划分行数守恒、候选必须是含 `prepare`/`model` 两步的 Pipeline。

## 7. 已知限制

- 结果只描述这份历史数据的预测表现，不构成因果或现代救援建议。
- 候选存在 31 个假阴性、24 个假阳性；年龄缺失（177 个）与舱位缺失（687 个）会影响特定人群的预测。
- 没有调参或特征工程来追求更高分数；改动数据、划分、指标或测试以获得漂亮结果是不允许的。

## 8. 提交清单

- `report.md`：书面报告
- `presentation.pptx`：3 分钟答辩 PPT
- `submission.json`：提交清单
- `team.md`：成员名单
- `train.py`：源码
- `team.md`：测试代码