# Regressions

**—— 用统计方法表达社会结构的思想**

### ✨ 引言

在社会科学研究中，个体往往不是孤立存在的。学生属于学校，工人属于公司，病人属于医院，公民属于国家。
这些**嵌套结构**带来了统计建模上的挑战，也提供了新的研究机会。

本仓库介绍两种我使用过的常用于处理此类数据的回归方法：

* **相关随机效应模型（Correlated Random Effects, 简称 CRE）**
* **分层线性模型（Hierarchical Linear Models, 简称 HLM）**

它们帮助我们突破传统 OLS 回归的限制，在面对**层级结构数据**时做出更有效、更社会学意义丰富的推论。

> **核心思想：**
> 这两个模型让我们能够在统计层面上体现社会科学中的一个基本理念——**个体行为受到其所处社会结构的深刻影响。**

---

### ⚙️ 1. 相关随机效应模型（CRE）—— *Stata 实现*

#### 🔍 方法简介

CRE 方法（通常被称为 Mundlak 方法）是介于固定效应（FE）与随机效应（RE）之间的一种策略，它允许我们：

* 控制**个体不随时间变化的特征**（如性别、家庭背景）
* 同时估计这些时间不变变量的影响

它通过在模型中加入协变量的**个体均值**（between effect）来控制协变量与个体效应之间的相关性。

#### 💡 社会学视角

> CRE 模型体现了这样的社会学思想：**人的行为不仅受当前处境影响，还根植于其稳定的社会背景与身份结构中。**
> 我们通过引入“个体平均值”来刻画这些结构性差异。

#### 📎 Stata 代码示例

假设我们使用一个包含多期个体观察的数据集（如 `id` 表示个体，`year` 表示年份）来研究 `y` 与 `x` 的关系。

```stata
* 导入示例数据
webuse nlswork, clear

* 计算每个个体在整个观测期的 tenure 平均值
bysort id (year): gen x_mean = mean(tenure)

* CRE 模型：RE + 协变量均值
xtset id year
xtreg ln_wage tenure x_mean, re
```

#### 📌 结果解读

* `tenure` 的系数代表个体内部的变化（within effect）
* `x_mean` 的系数代表个体之间的差异（between effect）
* 两者合起来让我们既看到个体如何变化，也看到群体之间的结构性差异

---

### 🧭 2. 分层线性模型（HLM）—— *Stata 与 R 实现*

#### 🔍 方法简介

HLM（又称多层模型、混合效应模型）用于建模具有明确层级结构的数据，如：

* 学生属于学校
* 雇员属于企业
* 居民属于社区

它允许我们对每一层的误差结构进行建模，如：

* 每个学校有自己的基准成绩（随机截距）
* SES 对成绩的影响在不同学校间也可能不同（随机斜率）

#### 💡 社会学视角

> HLM 是对“人嵌入在社会情境中”这一理念的直接建模。
> 它让我们能同时估计**个体层面因素**与**群体层面背景**的作用，比如 SES 和学校质量。

---

#### ✅ Stata 代码示例

```stata
* 学生嵌套在学校中（假设数据结构类似）
webuse schdata, clear

* 拟合随机截距模型
mixed mathach ses || schoolid:
```

---

#### ✅ R 代码示例（lme4 包）

```r
# 加载包
library(lme4)

# 建立模型：学生 ~ SES + 学校随机截距
model <- lmer(mathach ~ ses + (1 | schoolid), data = schdata)
summary(model)
```

---

### 🧠 应该选择哪种模型？

| 情境             | 建议模型           |
| -------------- | -------------- |
| 面板数据，关注时间变化    | CRE（Stata）     |
| 横截面数据，明确层级结构   | HLM（Stata 或 R） |
| 需要估计时间不变变量的影响  | CRE            |
| 群体层面变量/背景有理论意义 | HLM            |

---

### 📚 推荐阅读

* Mundlak, Y. (1978). “On the Pooling of Time Series and Cross Section Data.” *Econometrica*
* Snijders, T. & Bosker, R. (2012). *Multilevel Analysis*
* Rabe-Hesketh, S. & Skrondal, A. (2012). *Multilevel and Longitudinal Modeling Using Stata*

