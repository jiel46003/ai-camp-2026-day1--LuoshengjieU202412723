# 每日作业报告
## 1. 本日问题
- 里程碑：day-01
- 学生或小组：罗圣杰（组员信息见 `team.md`）
- 使用者：准备历史展览、希望解释数据和模型局限的博物馆教育团队
- 真实输入：Kaggle Titanic Dataset `train.csv`（891 行、12 列）
- 需要的输出：同一留出测试集上基线与候选的指标、混淆矩阵、真实错误乘客记录
- 与使用者最相关的错误：假阴性（真实幸存却被预测为未幸存）
- 本日产品边界：历史观察数据分析，不是现代救援分配工具，不能证明特征导致生存
## 2. 真实数据或真实课程输入
- 所有者/发布者：Kaggle（hesh97）
- 标题：Titanic Dataset train.csv
- 原始 URL：https://www.kaggle.com/datasets/hesh97/titanicdataset-traincsv
- 预期文件与结构：`data/raw/train.csv`，891 行、12 列
- 检查命令：`python train.py --check-data`
- 实际检查结果（2026-08-17 实测）：
```
REAL DATA CHECK PASSED
rows: 891
columns: 12
survived_counts: {0: 549, 1: 342}
missing_age: 177
missing_cabin: 687
missing_embarked: 2
```
- 已知缺失、偏差或限制：Age 缺失 177、Cabin 缺失 687、Embarked 缺失 2；缺失不等于 0，需在流水线内按训练数据填补，测试行不参与训练准备。
## 3. 可复现运行
```powershell
# 当前目录
d:\day1\student-work\day-01-titanic
# 安装
python -m pip install -r requirements.txt
# 数据检查
python train.py --check-data
# 预期：REAL DATA CHECK PASSED；rows: 891；survived_counts: {0: 549, 1: 342}
# 主程序
python train.py
# 输出：metrics.json（指标+混淆矩阵）、errors.csv（55 条错误行）
# 测试
python -m unittest discover -s tests -v
# 预期：3 个测试 ok，Ran 3 tests ... OK
```
## 4. 基线与候选
### 简单基线
- 方法：多数类基线 `DummyClassifier(strategy="most_frequent")`，永远预测训练集最常见类别 `Survived=0`
- 为什么足够简单：不读取任何特征，只需统计训练集类别频率
- 命令：`python train.py`（基线在候选前自动运行）
- 结果（223 行测试集）：accuracy=0.614，precision=0.000，recall=0.000，F1=0.000；混淆矩阵 `[[137,0],[86,0]]`，即 86 名真实幸存者全部被漏掉
### 候选方法
- 学生完成的核心改动：`train.py` 的 `build_candidate()`——在现有 `preprocessor()` 之后加入 `RandomForestClassifier(n_estimators=200, random_state=42, n_jobs=-1)`
- 保持不变的数据、划分、指标或参数：真实数据、`train_test_split(test_size=0.25, random_state=42, stratify=target)`、预处理流水线、评估函数
- 命令：`python train.py`
- 结果（同一 223 行测试集）：accuracy=0.753，precision=0.696，recall=0.640，F1=0.667；混淆矩阵 `[[113,24],[31,55]]`
| 项目 | 基线 | 候选 | 含义 |
| --- | ---: | ---: | --- |
| 主指标（幸存者 F1） | 0.000 | 0.667 | 候选开始能识别幸存者 |
| 幸存者召回率 | 0.000 | 0.640 | 86 名真实幸存者中识别出 55 名 |
| 准确率 | 0.614 | 0.753 | 总体正确比例 |
| 与使用者最相关的错误 | 假阴性 86 | 假阴性 31 / 假阳性 24 | 假阴性大幅下降但仍最多 |
## 5. 一个真实失败案例
- 样本位置/编号：`errors.csv`，source_row=2，PassengerId=348
- 真实结果：Survived=1（幸存）
- 系统输出：predicted=0（预测未幸存）→ 假阴性
- 可以观察到什么：Davison, Mrs. Thomas Henry（女性，三等舱，年龄缺失，SibSp=1，Parch=0，Fare=16.1，Embarked=S）
- 说明的限制：模型依赖统计规律，无法覆盖与"女性→高幸存率"规律相反的个体；三等舱 + 年龄缺失是边际样本
- 不能证明什么：不能证明该乘客"应当死亡"，不能证明舱位/性别"导致"结果，不能编造未记录的情节
- 下一项最小检查：统计全部 31 个假阴性的舱位/性别/年龄构成，确认错误集中在哪些人群；观察年龄缺失是否抬高假阴性比例
## 6. 智能体与学生工作边界
> 本段必须由组员自己填写，AI 不代写。
- 智能体提出/生成/修改了什么：智能体提供了基线模型与随机森林候选模型的代码结构建议，并辅助核查了流水线内缺失值处理与测试集数据泄露的潜在逻辑。
- 学生怎样核对文件、来源、输出、测试和 diff：我们独立运行 python train.py --check-data 与 python -m unittest，逐一检查控制台输出与 metrics.json、errors.csv 中的指标及混淆矩阵数值是否一致。
- 学生修改或拒绝了什么建议：拒绝了智能体提出的在预处理阶段直接全局填充均值的建议，改为在 Pipeline 内依据训练集填补以防数据泄露，并调整了随机森林的参数设定以保持与实验设计一致。
- 每名成员能独立解释的代码或证据。
## 7. 结论与限制
1.这次作业我们在 891 条泰坦尼克号数据上，对比了简单多数类基线和随机森林模型。
2.在留出的 223 条测试集上，随机森林的准确率达到了 75.3%，幸存者的召回率也从 0 提升到了 64.0%。
3.虽然模型确实能抓到一部分幸存特征，但它依然漏掉了 31 位真实幸存者，误把他们预测成了遇难。
4.像 348 号乘客这种三等舱且年龄缺失的女性，就因为处于统计规律边缘而被模型误判了。
5.这说明纯靠统计规律做分类，根本没法完全覆盖个体复杂的实际情况。
6.而且数据里有大量的年龄和船舱缺失值，这些都会直接干扰模型的判断效果。
7.我们要明确，模型学到的只是历史记录里的关联，绝不能当作导致生还的直接因果。
8.所以这个模型只适合在博物馆展览里给观众科普算法局限，千万不能当成现代救援的决策工具。
## 8. 提交复核
- [ ] README 从新环境可以开始运行
- [ ] 数据检查、测试和主程序重新运行
- [ ] 报告数字与保存输出一致
- [ ] `presentation.pptx` 在 3 分钟内讲完
- [ ] `submission.json` 路径正确
- [ ] 无密钥、大数据、私人信息、虚拟环境或缓存
- [ ] GitHub 网页复查并邮件发送 URL
