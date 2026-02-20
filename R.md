# Libraries

数据获取 → 清洗 → 探索 → 可视化 → 特征工程 → 建模 → 评估 → 报告输出

## 数据获取（Data Import）

### `readr`读取 CSV

```
library(readr)
data <- read_csv("data.csv")

country_metrics <- read.csv("data/country_metrics.csv")
```

### `readxl`读取 Excel

### `haven`读取 SPSS / Stata

### `DBI` + `RSQLite`数据库连接

### `httr`API 请求

## 数据清洗（Data Cleaning / Manipulation）- dplyr

对数据进行筛选、分组、汇总、变换

是 tidyverse 中的数据操作核心

数据是 **data frame / tibble**

支持链式操作（pipe）

### filter() - 筛选行

#### 单条件

#### 单条件多值，或者范围区间

filter( 

  country %in% c("Mexico", "Canada"), 

   between(year, 2019, 2022)

) 

#### 多条件

### select() - 选择列

### mutate() - 新增/修改列

### summarise() - 汇总统计

### group_by() - 分组

### arrange() - 排序

### slice() - 选取行

## 数据整理（Data Reshaping / wrangling）- tidyr

改变数据的“形状”（宽 ↔ 长）

Tidy data 原则：

1. 每个变量一列

2. 每个观测一行

3. 每个值一个单元格

### pivot_longer()   宽 → 长

```
pivot_longer(
  cols = -country,
  names_to = "metric",
  values_to = "value"
)
```

它在做什么？

一句话：

> 把“多个列”堆叠成两列：  
> 一列存原来的列名（metric），  
> 一列存原来的数值（value）。

![](/Users/xiongmin/Library/Application%20Support/marktext/images/2026-02-16-19-37-10-image.png)

![](/Users/xiongmin/Library/Application%20Support/marktext/images/2026-02-16-19-39-33-image.png)

![](/Users/xiongmin/Library/Application%20Support/marktext/images/2026-02-16-19-39-59-image.png)

### pivot_wider()长 → 宽

### separate()拆分列

### unite()合并列

### drop_na()删除缺失

## 探索性数据分析（EDA - exploratory data analysis）

### `skimr`自动摘要

### `summary()`基础统计

### `psych`描述统计

### `GGally`散点矩阵

## 数据可视化（Visualization） - ggplot2

基于 Grammar of Graphics 构建图形

#### 介绍

##### 核心思想：

> 图形 = 数据 + 美学映射 + 几何对象 + 统计变换 + 坐标系统 + 分面 + 主题

##### 标准模版

```
ggplot(data = 数据, aes(美学映射)) + # 负责定义数据与映射
  geom_XXX() +  # 负责定义几何表现
  scale_XXX() + # 负责定义比例与转换 (scale_y_continuous(breaks = seq(65, 90, 5)))
  facet_XXX() + # 负责定义分面
  coord_XXX() + # 负责定义坐标系统 (coord_cartesian(ylim = c(65, 90)))
  theme_XXX() # 负责定义样式
```

##### aes

###### 核心参数

| 参数        | 作用                                                            |
| --------- | ------------------------------------------------------------- |
| x         | x轴变量                                                          |
| y         | y轴变量                                                          |
| color     | 线颜色（分组）                                                       |
| group     | 分组变量                                                          |
| linetype  | 线型 （"solid"，"dashed"，"dotted"，"dotdash"，"longdash"，"twodash"） |
| linewidth | 线宽                                                            |
| alpha     | 透明度                                                           |

aes(x = year, y = life_expectancy, color = country)

**当有多组图形时要有group，否则线会乱连，出现竖线问题**

aes(x = year, y = life_expectancy, group = country, color = country)

### 几何对象

| 目的    | 推荐 geom                 |
| ----- | ----------------------- |
| 看趋势   | geom_line               |
| 看增长   | geom_line + geom_smooth |
| 看分布   | geom_histogram          |
| 看结构   | geom_area               |
| 看类别对比 | geom_col                |
| 看异常值  | geom_boxplot            |
| 看相关性  | geom_point              |

#### 点图类 - 相关性分析、回归分析

##### geom_point() - 散点 - 两个连续变量关系

示例代码

```
filter_country_metrics("life_expectancy") |>
ggplot() +
  geom_point(
    aes(x = year, y = life_expectancy, color = country, group = country)
  ) +
  # geom_point(aes(year, life_expectancy, color = country), size = 2) +
  labs(
    title = "Life Expectancy In Northern Amarica countries between 2015 and 2023",
    x = "Year",
    y = "Life Expectancy",
    color = "Country"
  ) +
  theme_minimal()
```

图形样例

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/4.png)

##### geom_jitter() - 抖动散点 - 分类 + 连续

##### geom_count() - 点大小表示频数 - 离散重复数据

#### 线图类 - 时间序列

##### geom_line()  - 折线 - 时间趋势

核心参数

| 参数          | 作用                                                                         |
| ----------- | -------------------------------------------------------------------------- |
| aes         |                                                                            |
| linetype    | 线型 （"solid"，"dashed"，"dotted"，"dotdash"，"longdash"，"twodash"）              |
| linewidth   | 线宽                                                                         |
| alpha       | 透明度                                                                        |
| position    | 控制图层位置调整("identity" - 默认; position_dodge() - 多组并排; position_jitter() - 抖动) |
| stat        | y 值来源 （identity - y 值直接来自数据）                                               |
| na.rm       | 是否忽略 NA （geom_line(na.rm = TRUE)）                                          |
| show.legend | 控制是否显示图例                                                                   |
| inherit.aes | 继承 ggplot() 里的 aes                                                         |

示例代码

```
ggplot(na_data,
       aes(x = year,
           y = life_expectancy,
           color = country,
           group = country)) +
  geom_line(
    linewidth = 1.2,
    linetype = "solid",
    alpha = 0.9
  ) +
  theme_minimal()
```

图形样例

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/1.png)

##### geom_path() - 按数据顺序连线 - 轨迹图

##### geom_step() - 阶梯线 - 累计函数

#### 面积图类 - 不确定性区间

##### geom_area()面积

示例

```
filter_country_metrics("life_expectancy") |>
ggplot() +
  geom_area(
    aes(x = year, y = life_expectancy, fill = country)
  ) +
  # geom_point(aes(year, life_expectancy, color = country), size = 2) +
  labs(
    title = "Life Expectancy In Northern Amarica countries between 2015 and 2023",
    x = "Year",
    y = "Life Expectancy",
    color = "Country"
  ) +
  theme_minimal()
```

图例

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/3.png)

##### geom_ribbon() - 区间带 - 置信区间

#### 柱状图类 - 分布 / 分类对比

##### geom_histogram() - 直方图 - 单变量分布

核心参数

| 参数       | 作用      |
| -------- | ------- |
| binwidth | 每个箱子的宽度 |
| bins     | 分成多少箱   |
| fill     | 填充颜色    |
| color    | 边框颜色    |
| alpha    | 透明度     |
| boundary | 起始边界    |
| center   | 箱中心     |

```
geom_histogram(bins = 30) 
```

bins默认为30，当不指定bins时，ggplot 会给你提示： `stat_bin()` using `bins = 30`.

直方图做的是：

1. 把 x 轴范围切成若干区间（bin）

2. 统计每个区间里有多少观测值

3. 画出柱子

如果你的能源消费范围是：0 到 3000 TWh，且每个数值都有对应的国家，即国家A，消费1，国家B消费2，国家C消费3

每个 bin 宽度 ≈ 3000 / 30 = 100 TWh

每个柱子表示：100 TWh 范围内有多少国家

一共会有30个柱子，如果不设定bin，那么就会有3000个柱子

##### geom_bar() - 计数柱 - 分类频数

##### geom_col() - 已计算数值柱 - 汇总数据

样例代码

```
filter_country_metrics("life_expectancy") |>
ggplot() +
  geom_col(
    aes(x = year, y = life_expectancy, fill = country),
    position = "dodge", alpha = 0.8
  ) +
  # geom_point(aes(year, life_expectancy, color = country), size = 2) +
  labs(
    title = "Life Expectancy In Northern Amarica countries between 2015 and 2023",
    x = "Year",
    y = "Life Expectancy",
    color = "Country"
  ) +
  theme_minimal()
```

1️⃣ 柱状图通常 x 轴用离散变量（建议 `factor(year)`）  
2️⃣ 多国家需要 `position = "dodge"` 才能并排  
3️⃣ 必须指定 `group` 或 `fill` 来区分国家

图形样例

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/2.png)

#### 分布分析类 - 统计分布分析

##### geom_density() - 密度曲线 - 连续分布

##### geom_boxplot() - 箱线图 - 分布比较

##### geom_violin() - 小提琴图 - 分布形态

##### geom_dotplot() - 点图分布 - 小样本

#### 关系与趋势类

##### geom_smooth() - 回归/平滑线 - 趋势分析

##### geom_abline() - 参考直线 - 回归基准

##### geom_hline() - 水平线 - 阈值

##### geom_vline() - 垂直线 - 时间标记

#### 文本与标注类

`geom_text()`文本标签

`geom_label()`带框标签

`annotate()`添加标注

#### 区间与误差类

`geom_errorbar()`误差线

`geom_linerange()`区间线

`geom_pointrange()`点+区间

`geom_crossbar()`均值区间

#### 地图与空间类

`geom_polygon()`区域地图

`geom_map()`地图数据

`geom_sf()`空间数据

#### 二维密度类

`geom_bin2d()`二维直方图

`geom_hex()`六边形分箱

`geom_density2d()`等高线密度

### 比例控制

#### scale_*()

### 分面

#### facet_wrap()  - 一定要使用pivot_longer吗？

```
facet_wrap(~metric) # 默认使用 scales = "fixed"， 所有子图共用同一坐标轴

facet_wrap(~metric, scales = "free") # 每个子图使用自己的坐标范围

facet_wrap(~metric, ncol = 1) # 每列一个子图（竖排）

facet_wrap(~metric, nrow = 1) # 横排
```

意思是：

> 按 metric 这个变量的不同取值，  
> 把图拆成多个子图。

### 坐标系统

#### coord_XXX()

### 主题

#### theme_XXX()

## gt — 表格美化（Table Presentation）

生成“出版级”美观表格

gt 专门负责“展示层”

### gt() - 创建表

### fmt_number() - 数字格式

### tab_header() - 添加标题

### cols_label() - 重命名列

### tab_row_group() - 行分组

### tab_style() - 条件样式

## 分类变量处理 - forcats

## 时间数据处理 - lubridate

## 相关分析 - corrplot

## 建模（Modeling） - tidymodels

## 模型评估 - yardstick

## 报告输出 -

### `rmarkdown`报告生成

### `knitr`自动化输出

### `gt`表格美化

### `patchwork`图合并

# Others

## 管道

把左边的结果传给右边函数的第一个参数。

```
country_metrics |>
  filter(region23 == "Northern America") |>
  select(year, population) |>
  pivot_longer(cols = -year) |>
  ggplot()
```

逻辑变成：可读性强

> 数据 → 过滤 → 选择列 → 变形 → 画图

## factor变量

`factor` 是 R 用来表示“分类变量（categorical variable）”的特殊数据类型。

它不是普通字符串，而是：带有“水平（levels）”结构的分类编码变量。

```
gender <- c("Male", "Female", "Male", "Female")

str(gender)
Factor w/ 2 levels "Female","Male": 2 1 2 1

```

实际存储是整数：1, 2

对应 标签（levels）："Female", "Male"

建模中的应用

```
data <- data.frame(
  income = c(5000, 6000, 7000, 8000),
  gender = factor(c("M", "F", "M", "F"))
)
# 根据gender来预测income
lm(income ~ gender, data = data)

# R 会自动建模成线性回归模型
income = β0 + β1 * genderF

# 如果将gender使用自身的值Female或者Male，则无法做乘法，线性回归模型错误。
# 所以要将gender通过factor分类化，做乘法的是Female对应的R分类后的数字
```

## str(epa2021_raw)

主要是观察哪些变量是数值，哪些变量需要factor后才能被用于建模

str后是R能识别的数据类型，和直接查看源文件看到的可能不一样。源文件里看到的虽然是数字，但有可能字符串。如engine_displacement，值为3.5 ，有可能R读到的是3.5L

```
data.frame':	1108 obs. of  28 variables:
 $ model_yr             : int  2021 2021 2021 2021 2021 2021 2021 2021 2021 2021 ...
 $ mfr_name             : chr  "Honda" "aston martin" "aston martin" "Volkswagen Group of" ...
 $ division             : chr  "Acura" "Aston Martin Lagonda Ltd" "Aston Martin Lagonda Ltd" "Audi" ...
```

## summary(epa2021_raw)

观察是否有缺失值，数值是否合理，以决定该怎么清洗数据

```
model_yr      mfr_name           division           carline            mfr_code         model_type_index engine_displacement  no_cylinders    transmission_speed    city_mpg    
 Min.   :2021   Length:1108        Length:1108        Length:1108        Length:1108        Min.   :  1.0    Min.   :1.000       Min.   : 3.000   Length:1108        Min.   : 8.00  
 1st Qu.:2021   Class :character   Class :character   Class :character   Class :character   1st Qu.: 40.0    1st Qu.:2.000       1st Qu.: 4.000   Class :character   1st Qu.:17.00  
 Median :2021   Mode  :character   Mode  :character   Mode  :character   Mode  :character   Median :114.0    Median :3.000       Median : 6.000   Mode  :character   Median :20.00  
 Mean   :2021                                                                               Mean   :259.4    Mean   :3.125       Mean   : 5.604                      Mean   :20.89  
 3rd Qu.:2021                                                                               3rd Qu.:507.0    3rd Qu.:3.800       3rd Qu.: 6.000                      3rd Qu.:23.00  
 Max.   :2021                                                                               Max.   :999.0    Max.   :8.000       Max.   :16.000                      Max.   :58.00  
```

```
例如看到这样的数据，排量25L合理吗？
engine_displacement
Min.   0
Max.  25

Min.   :1.2  
1st Qu.:2.0  # 第一4分位
Median :2.5  
Mean   :2.6  
3rd Qu.:3.0  
Max.   :6.2  
NA's   : 15

```

运行 `summary()` 时，你要问：

1. 有多少 NA？ -- 只有数值型的列会统计NA

2. 数值范围是否合理？

3. 是否存在异常值？

4. 分类变量是否干净？

5. 是否有明显数据错误？

## dplyr::mutate_if

```
epa2021 <- epa2021 %>% mutate_if(is.character, as.factor)


'data.frame':	1108 obs. of  13 variables:
 $ mfr_code           : Factor w/ 22 levels "ASX","BMX","CRX",..: 8 1 1 21 21 21 21 21 2 2 ...
 $ engine_displacement: num  3.5 4 4 5.2 5.2 5.2 5.2 2 3 2 ...
 $ no_cylinders       : int  6 8 8 10 10 10 10 4 6 4 ...
 $ transmission_speed : Factor w/ 25 levels "Auto(A10)","Auto(A6)",..: 8 25 21 6 6 6 6 6 21 21 ...
 $ air_aspir_method   : Factor w/ 4 levels "OT","SC","TC",..: 3 3 3 NA NA NA NA 3 3 3 ...
 $ transmission       : Factor w/ 7 levels "A","AM","AMS",..: 3 5 6 3 3 3 3 3 6 6 ...
 $ no_gears           : int  9 7 8 7 7 7 7 7 8 8 ...
 $ trans_lockup       : Factor w/ 2 levels "N","Y": 2 1 2 1 1 1 1 1 2 2 ...
 $ drive_sys          : Factor w/ 5 levels "4","A","F","P",..: 2 5 5 2 5 2 5 2 5 5 ...
 $ fuel_usage         : Factor w/ 6 levels "D","DU","G","GM",..: 6 5 5 6 6 6 6 3 5 5 ...
 $ class              : Factor w/ 22 levels "Compact Cars",..: 21 21 21 21 21 21 21 21 21 21 ...
 $ car_truck          : Factor w/ 3 levels "??","1","car": 3 3 3 3 3 3 3 3 3 3 ...
 $ comb_mpg           : int  21 17 20 16 17 16 17 26 25 28 ...
```

`mutate_if()` 已经逐步被替代，推荐用：

```
epa2021 <- epa2021_raw %>%
  mutate(across(where(is.character), as.factor))

## 只转换某几列
epa2021 <- epa2021_raw %>%
  mutate(across(c(State, Sector), as.factor))

## 想排除某列
epa2021 <- epa2021_raw %>%
  mutate(across(where(is.character) & !matches("ID"), as.factor))
```

通用判断函数

| 函数              | 说明      |
| --------------- | ------- |
| `is.na()`       | 是否缺失    |
| `is.null()`     | 是否 NULL |
| `is.finite()`   | 是否有限值   |
| `is.infinite()` | 是否无限    |

数值类

| 函数             | 说明               |
| -------------- | ---------------- |
| `is.numeric()` | 数值（int + double） |
| `is.integer()` | 整数               |
| `is.double()`  | 小数               |
| `is.logical()` | TRUE / FALSE     |

字符与分类类

| 函数               | 说明   |
| ---------------- | ---- |
| `is.character()` | 字符   |
| `is.factor()`    | 因子   |
| `is.ordered()`   | 有序因子 |

日期时间类

| 函数             | 说明   |
| -------------- | ---- |
| `is.Date()`    | 日期   |
| `is.POSIXct()` | 日期时间 |
| `is.POSIXlt()` | 日期时间 |

结构类

| 函数                | 说明  |
| ----------------- | --- |
| `is.list()`       | 列表  |
| `is.data.frame()` | 数据框 |
| `is.matrix()`     | 矩阵  |

## skim

`skim()` 是一个比 `summary()` 更强大的数据概览工具，来自 **skimr 包**

- 按变量类型分组展示

- 显示缺失值

- 显示分布统计

- 显示常见类别

- 显示分位数

- 显示直方图（小 sparkline）

```
library(skimr)
skim(epa2021)
skim(epa2021, comb_mpg, engine_displacement) # 只skim某几列

```

| 项目            | 含义    |
| ------------- | ----- |
| n_missing     | 缺失数量  |
| complete_rate | 非缺失比例 |
| mean          | 平均数   |
| sd            | 标准差   |
| p0            | 最小值   |
| p25           | 第一四分位 |
| p50           | 中位数   |
| p75           | 第三四分位 |
| p100          | 最大值   |
| hist          | 小型分布图 |

| 项目            | 含义      |
| ------------- | ------- |
| n_missing     | 缺失数量    |
| complete_rate | 完整比例    |
| ordered       | 是否有序    |
| n_unique      | 类别数量    |
| top_counts    | 出现最多的类别 |

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/skim1.png)

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/skim2.png)

![](/Users/xiongmin/Data/Min/Documents/Technologies/Homework/R/images/skim3.png)


