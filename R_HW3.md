## 管道

把左边的结果传给右边函数的第一个参数。

```
country_metrics |>
  filter(region23 == "Northern America") |>
  select(year, population) |>
  pivot_longer(cols = -year) |>
  ggplot()
```
filter的完整格式为：filter(.data, 条件1, 条件2, ...)
select的完整格式为：select(.data, 列名1, 列名2, ...)
因此上方代码块的分步写法为
```
filtered_metrics <- filter(country_metrics, region23 == "Northern America")
selected_columns <- select(filtered_metrics, year, population)
longer_data <- pivot_longer(selected_columns, cols = -year)
ggplot(longer_data)
```

逻辑变成：可读性强

> 数据 → 过滤 → 选择列 → 变形 → 画图

## group_by

<img width="276" height="138" alt="image" src="https://github.com/user-attachments/assets/094e6efd-ff4c-4a0a-a1eb-aa3b380c54ed" />
group_by(Text1) 得到的结果将是

| Text1 |
| - |
| A |
| A1 |
| A2 |

group_by(Text1， Text2) 得到的结果将是

| Text1 | Text2 |
| - | - |
| A | B |
| A | B1 |
| A1 | B |
| A2 | B2 |

<img width="823" height="170" alt="image" src="https://github.com/user-attachments/assets/df5d4108-f34d-49fb-9b6c-79eca5df806c" />


group_by(Text1， Text2, Text3) 得到的结果将是

| Text1 | Text2 | Text3 |
| - | - | - |
| A | B | C |
| A | B1 | C1 |
| A1 | B | C1 |
| A2 | B2 | C |
| A | B | C2 |

```
epa2021 %>% 
   group_by(air_aspir_method) %>% # why do we need to use group_by??
   summarise(
    total = n(),
    missing = sum(is.na(air_aspir_method)),
    missing_rate = mean(is.na(air_aspir_method))
  )
```
<img width="1085" height="200" alt="image" src="https://github.com/user-attachments/assets/b7ee208d-7003-444b-b6f0-09d236947827" />

## factor变量

`factor` 是 R 用来表示“分类变量（categorical variable）”的特殊数据类型。

它不是普通字符串，而是：带有“水平（levels）”结构的分类编码变量。

```
gender <- factor("Male", "Female", "Male", "Female")

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
data.frame':    1108 obs. of  28 variables:
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


'data.frame':    1108 obs. of  13 variables:
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

<img width="360" height="219" alt="image" src="https://github.com/user-attachments/assets/668764a3-94f1-4e1e-afb0-ac32513948fa" />

<img width="1308" height="292" alt="image" src="https://github.com/user-attachments/assets/2b3522a5-7135-4090-86e0-b2d68c28397e" />

<img width="1641" height="174" alt="image" src="https://github.com/user-attachments/assets/26a4bde1-477a-49a3-80fd-8284c264d387" />

## 数据的处理
### NA 数据的处理
如果保持 NA：
- lm() 会自动删除这些行（listwise deletion）
- 数据量减少
- 可能造成偏差
如果改成一个 level（例如 "Missing" 或 "Unknown"）：
- 保留数据
- 把“缺失”当成一种类别

处理前需注意数据是否已经被转换成了factor，还是原始的character。因为转换方法是不一样的

| 函数                        | 所属包     | 作用对象   |
| ------------------------- | ------- | ------ |
| `fct_recode()`            | forcats | factor |
| `fct_na_value_to_level()` | forcats | factor |
| `na_if()`                 | dplyr   | 向量     |
| `replace_na()`            | tidyr   | 向量     |
```
Error in `mutate()`:
ℹ In argument: `car_truck = na_if(car_truck, "??")`.
Caused by error in `na_if()`:
! Can't convert from `y` <character> to `x` <factor<b5ce7>> due to loss of generality.
• Locations: 1
Run `rlang::last_trace()` to see where the error occurred.
```

#### convert-na-into-a-factor-level
```
library(forcats)

epa2021 <- epa2021 %>%
  mutate(
    car_truck = fct_na_value_to_level(car_truck, "Missing")
  )
```
#### replace-na-in-a-factor-column
```
library(forcats)

# 做法 A：直接用 fct_recode
epa2021 <- epa2021 %>%
  mutate(
    car_truck = fct_recode(car_truck, NotSure = "??")
  )

# 做法 B：先 na_if，再 fct_na_value_to_level
epa2021 <- epa2021 %>%
  mutate(
    car_truck = fct_recode(car_truck, NULL = "??"),
    car_truck = fct_na_value_to_level(car_truck, "Missing")
  )
```
两种做法最终的实际结果是一样的，但概念，或者说对"??"的理解则不一样。
做法A，将"??"理解为一个合法类别，只不过名字不好听，改为一个恰当名字。
做法B，将"??"理解为不合法数据，可能是录入错误等。所以先按照基本操作，将不合法数据改为NA，再将所有NA改为默认值
数据清洗原则
```
错误值 → NA
NA → 判断是否结构性缺失
再决定是否：
    - 删除
    - 填补
    - 转成 level
```
### 绝大部分都是数字，但同时存在少数字符串？
### 某些数字过大？

## 定义Function
将代码细化为细小颗粒的功能组，是一个非常良好的代码习惯。优点有：
1. 功能越简单，代码越容易实现。类似于拼积木
2. 每个功能单独定义一个function，通过function名，就能大致知道他实现了什么。特别是在今后修改或维护时，甚至都不用进去细看每一行代码
3. 特别是功能被反复调用时，修改这一处就可以改变所有调用者

```
ggplot(epa2021, aes(x = transmission, y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by Transmission",
    x = "Transmission",
    y = "Combined MPG"
  ) +
  theme_minimal()

# We have done some data clean to set air_aspir_method NA data to Missing
ggplot(epa2021, aes(x = air_aspir_method, y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by Air Aspir Method",
    x = "Air Aspir Method",
    y = "Combined MPG"
  ) +
  theme_minimal()

# We have done some data clean to set car_truck NA data to Missing 
ggplot(epa2021, aes(x = car_truck, y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by Car Truck",
    x = "Car Truck",
    y = "Combined MPG"
  ) +
  theme_minimal()

# Class has 22 levels
ggplot(epa2021, aes(x = class, y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by Class",
    x = "Class",
    y = "Combined MPG"
  ) +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))

# engine_displacement is number type
ggplot(epa2021, aes(x = factor(engine_displacement), y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by Engine Displacement",
    x = "Engine Displacement",
    y = "Combined MPG"
  ) +
  theme_minimal()

# no_cylinders is int type
ggplot(epa2021, aes(x = factor(no_cylinders), y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by No Cylinders",
    x = "No Cylinders",
    y = "Combined MPG"
  ) +
  theme_minimal()
```
通过提取function后的代码结构
```
getAesBasedOnFactor <- function(x, isXFactor){
  if (isXFactor){
    return (aes(x = {{ x }}, y = comb_mpg))
  }
  
  return (aes(x = factor({{ x }}), y = comb_mpg))
}

getThemeBasedOnRecordNumbers <- function(isXTooMany){
  if (isXTooMany){
    return (theme(axis.text.x = element_text(angle = 90, hjust = 1)))
  }
  
  return (theme_minimal())
}

showBoxplot <- function(x, xName, isXFactor, isXTooMany){
  ggplot(epa2021, getAesBasedOnFactor({{ x }} , isXFactor)) +
  geom_boxplot() +
  labs(
    title = paste("Combined MPG by", xName),
    x = paste0(xName),
    y = "Combined MPG"
  ) +
  getThemeBasedOnRecordNumbers(isXTooMany)
}

showBoxplot(transmission, "Transmission", TRUE, FALSE)

# We have done some data clean to set air_aspir_method NA data to Missing
showBoxplot(air_aspir_method, "Air Aspir Method", TRUE, FALSE)

# We have done some data clean to set car_truck NA data to Missing 
showBoxplot(car_truck, "Car Truck", TRUE, FALSE)

# Class has 22 levels
showBoxplot(class, "Class", TRUE, TRUE)

# engine_displacement is number type
showBoxplot(engine_displacement, "Engine Displacement", FALSE, FALSE)

# no_cylinders is int type
showBoxplot(no_cylinders, "No Cylinders", FALSE, FALSE)
```
## geom_boxplot()
```
## 样例代码
ggplot(epa2021, aes(x = engine_displacement, y = comb_mpg)) +
  geom_boxplot() +
  labs(
    title = "Combined MPG by Engine Displacement",
    x = "Engine Displacement",
    y = "Combined MPG"
  ) +
  theme_minimal()
```
问题在于：
engine_displacement 是 numeric， comb_mpg 是 numeric。 而 geom_boxplot() 的逻辑是：数值变量 ~ 分类变量
但现在两个都是连续变量。ggplot 不知道：是按 x 分组？还是按 y 分组？
所以给出警告：
```
Warning: Orientation is not uniquely specified when both the x and y aesthetics are continuous. Picking default orientation 'x'.
Warning: Continuous x aesthetic ℹ did you forget `aes(group = ...)`?
```
<img width="741" height="434" alt="image" src="https://github.com/user-attachments/assets/1fe0a0af-aea4-48d1-b074-8fbd114a309f" />

需要将engine_displacement改为factor： aes(x = factor(engine_displacement), y = comb_mpg)

<img width="722" height="441" alt="image" src="https://github.com/user-attachments/assets/be300030-9224-495b-8ac7-cb918022385f" />

## theme(), theme_minimal()
theme_minimal() 是一个完整的“预设主题”。如碰到X轴值特别多会重叠在一起等特殊情况时，需要使用theme()来自定义，如修改：字体， 颜色， 网格线， 背景， 图例位置， 轴线等
```
theme(
    axis.text.x = element_text(angle = 90, hjust = 1)
  )
```
<img width="719" height="440" alt="image" src="https://github.com/user-attachments/assets/c317c4f3-373b-4ca5-8de9-3a16fc95594a" />

除了theme_minimal()外，还有其他预设主题如下：

| 主题              | 特点    |
| --------------- | ----- |
| theme_gray()    | 默认主题  |
| theme_minimal() | 极简风   |
| theme_classic() | 经典论文风 |
| theme_bw()      | 黑白主题  |
| theme_void()    | 无轴无网格 |

## Correlation
相关性本质是在回答：两个连续变量之间是否存在“线性关系”？

如果两个变量高度相关（>0.8）那这两个变量不能同时进线性回归，否则：
 - 系数不稳定
 - 标准误变大
 - p 值失真

| r 值  | 含义      |
| ---- | ------- |
| 1    | 完全正线性关系 |
| 0.8  | 强正相关    |
| 0.5  | 中等相关    |
| 0.2  | 弱相关     |
| 0    | 无线性关系   |
| -0.8 | 强负相关    |
| -1   | 完全负线性关系 |

<img width="548" height="302" alt="image" src="https://github.com/user-attachments/assets/9db6a9dc-ffd5-471c-a4ca-a62cc557e4cb" />
所以在做线性回归分析时，通常会只保留强相关的某一个参数用于分析

只能计算 numeric 变量之间的相关性
<img width="506" height="156" alt="image" src="https://github.com/user-attachments/assets/d80cd442-0406-4d79-ab44-d8217bfd777f" />
相关性分析只是了解下变量之间是否存在线性相关，并不是只有通过相关性分析的变量才能做线性回归分析，所以并不能说factor不能用于做线性回归分析

```
# Select numeric variables, since correlation can be computed only between them
numeric_data <- epa2021 %>%
  select(where(is.numeric))

# Used use = "complete.obs" to remove NA data, even we have transferred them at above, but used here one more time just in case
cor_matrix <- cor(numeric_data, use = "complete.obs")

# View correlation
corrplot(
  cor_matrix,
  method = "color",
  type = "upper",
  tl.col = "black",
  tl.cex = 0.8
)
```
| 参数                | 作用        |
| ----------------- | --------- |
| method = "color"  | 用颜色表示相关系数 |
| method = "circle" | 用圆形表示     |
| type = "upper"    | 只显示上三角    |
| tl.col            | 标签颜色      |
| tl.cex            | 标签大小      |
| addCoef.col = "black"          | 相关系数数字颜色      |
| number.cex = 0.7            | 相关系数数字标签大小      |

<img width="543" height="449" alt="image" src="https://github.com/user-attachments/assets/e1ff4ea9-a1e3-4144-9ced-a59bb236bea0" />

## lumping - Factor recoding
典型的 低频类别合并（rare category lumping），在建模前非常常见 - 将某个变量如（transmission_speed）样本数 < 某个数，就合并成 "Other"
为什么要做 lump？
- 避免 dummy 变量过多
- 避免样本太小类别导致不稳定
- 提高模型泛化能力
  
### fct_lump_min
```
epa2021$transmission_lump <- epa2021 |>
  mutate(
    transmission_lump = fct_lump_min(
      transmission_speed,
      min = 50, # 样本数量小于50的处理，大于50的保留
      other_level = "Other"
    )
  )

epa2021$transmission_lump 
```
<img width="1318" height="354" alt="image" src="https://github.com/user-attachments/assets/ff3135f2-4c45-49d3-8a34-3eb49d95a526" />
### fct_lump_***

| 函数                   | 控制方式    | 保留规则        | 典型用途    |
| -------------------- | ------- | ----------- | ------- |
| `fct_lump()`         | 通用接口    | 通过参数选择      | 旧版本统一入口 |
| `fct_lump_min()`     | 最小样本数   | 保留 n ≥ min  | 按频数阈值   |
| `fct_lump_n()`       | 保留前 n 个 | 保留最常见 n 个   | 限制类别数量  |
| `fct_lump_prop()`    | 比例阈值    | 保留占比 ≥ prop | 按比例控制   |
| `fct_lump_lowfreq()` | 只保留最常见  | 仅最大频数保留     | 极端简化    |
| `fct_other()`        | 手动指定    | 只保留指定类别     | 精确控制    |

#### fct_lump
```
fct_lump(
  f,
  n = NULL,
  prop = NULL,
  w = NULL,
  other_level = "Other",
  ties.method = c("min", "average", "first", "random", "max")
)
```

| 参数            | 类型         | 作用          | 说明         |
| ------------- | ---------- | ----------- | ---------- |
| `f`           | factor     | 要处理的因子变量    | 必填         |
| `n`           | integer    | 保留前 n 个类别   | 按频数排序      |
| `prop`        | numeric    | 保留比例 ≥ prop | 0–1        |
| `w`           | numeric 向量 | 权重          | 用于加权频数     |
| `other_level` | character  | 合并后类别名称     | 默认 "Other" |
| `ties.method` | character  | 并列处理方式      | 控制边界情况     |


## Data partitioning
将数据划分为训练集和测试集 - 为了评估模型在“未见过数据”上的泛化能力
因为： 测试集数据在训练过程中“完全不可见”
这模拟了真实世界场景：未来的数据是模型没见过的。

| 比例        | 用途   |
| --------- | ---- |
| 70% / 30% | 常见   |
| 80% / 20% | 数据量大 |
| 60% / 40% | 数据少时 |

```
library(rsample)

set.seed(123)

split <- initial_split(data, prop = 0.8)

train_data <- training(split)
test_data  <- testing(split)
```
### set.seed(123)
固定随机数生成器的起点，使结果可重复
保证：
- 每次训练集和测试集划分一样
  
否则：
- 每次运行都会不同
- 模型结果变化

123只是个数字，可以是任意数字

## Null model
```
# Create a linear regression object with the default mode and engine
epa2021_lm0 <- linear_reg()

# Using the overall mean as our null model is equivalent to fitting a model with just a y-intercept
epa2021_lm0_fit <- 
  epa2021_lm0 %>% 
  fit(comb_mpg ~ 1, data = epa2021_train)
```
<img width="525" height="305" alt="image" src="https://github.com/user-attachments/assets/a66a4bc2-23c6-488f-8374-9611ab79b6c9" />

```
# y-intercept
coef(epa2021_lm0_fit$fit)
# 因变量comb_mpg在训练集中的均值
mean(epa2021_train$comb_mpg, na.rm = TRUE)
```
## tidy()
<img width="1315" height="157" alt="image" src="https://github.com/user-attachments/assets/aa89c00c-aaea-4e9c-a261-e3318e784551" />

| 列名          | 含义    | 数学意义             |
| ----------- | ----- | ---------------- |
| `term`      | 变量名称  | 系数对应变量           |
| `estimate`  | 系数估计值 | (\hat{\beta})    |
| `std.error` | 标准误   | 系数不确定性           |
| `statistic` | t 值   | (\hat{\beta}/SE) |
| `p.value`   | p 值   | 显著性检验            |

## glance() 
<img width="1301" height="113" alt="image" src="https://github.com/user-attachments/assets/93954c16-257f-4f9e-b5cf-dd7d50b38472" />

| 字段              | 含义       | 数学定义 / 解释                     | 如何解读          |
| --------------- | -------- | ----------------------------- | ------------- |
| `r.squared`     | 决定系数     | ( R^2 = 1 - \frac{SSE}{SST} ) | 越接近 1 说明解释力越强 |
| `adj.r.squared` | 调整后 R²   | 对变量个数进行惩罚                     | 用于模型比较，更稳健    |
| `sigma`         | 残差标准误    | ( \sqrt{SSE/df} )             | 预测误差的平均波动大小   |
| `statistic`     | F 统计量    | 模型整体显著性检验                     | 越大说明模型越显著     |
| `p.value`       | F 检验 p 值 | 检验所有系数是否为 0                   | <0.05 通常认为显著  |
| `df`            | 模型自由度    | 自变量个数                         | 解释变量数量        |
| `df.residual`   | 残差自由度    | ( n - p - 1 )                 | 样本量减去参数数      |
| `logLik`        | 对数似然     | ( \log L )                    | 用于 AIC/BIC    |
| `AIC`           | 赤池信息准则   | ( -2\log L + 2k )             | 越小越好          |
| `BIC`           | 贝叶斯信息准则  | ( -2\log L + k\log n )        | 惩罚更强，越小越好     |
| `deviance`      | 残差平方和    | ( SSE )                       | 模型未解释的误差      |
| `nobs`          | 样本量      | 观测数量                          | 用于确认数据规模      |

## 线性回归

<img width="534" height="79" alt="image" src="https://github.com/user-attachments/assets/53fe83c7-1828-4624-859e-86c05e774efd" />

用一个或多个自变量 𝑋 来预测连续因变量 Y 的统计模型。 用一条“直线”去拟合数据。

Y=β0​+β1​X1​+β2​X2​+⋯+βp​Xp​+ϵ

| 符号         | 含义   |
| ---------- | ---- |
| (Y)        | 因变量  |
| (X)        | 自变量  |
| (\beta_0)  | 截距   |
| (\beta_1)  | 回归系数 |
| (\epsilon) | 误差项  |

<img width="586" height="268" alt="image" src="https://github.com/user-attachments/assets/4add258a-d9e5-4c61-b2ef-c71a1131b7fd" />

<img width="644" height="454" alt="image" src="https://github.com/user-attachments/assets/1924e708-5826-4a84-afdd-7294a4fafc20" />

<img width="799" height="408" alt="image" src="https://github.com/user-attachments/assets/2d0a24d3-d242-4b53-8027-1e25bf8b574b" />

### 模型定义
#### 1. 定义模型类型
```
epa2021_lm1 <- linear_reg()
```

#### 2. 指定变量
```
# Null 模型， 只有截距 Y^=β0
epa2021_lm0_fit <- 
  epa2021_lm0 %>% 
  fit(comb_mpg ~ 1, data = epa2021_train)

# 单变量 - 一条直线 Y^=β0​+β1​X1
epa2021_lm0_fit <- 
  epa2021_lm0 %>% 
    fit(comb_mpg ~ engine_displacement, data = epa2021_train)​

# 多变量 - 多维平面 Y^=β0​+β1​X1​+β2​X2
epa2021_lm0_fit <- 
  epa2021_lm0 %>% 
    fit(comb_mpg ~ x1 + x2, data = epa2021_train)

# 所有变量 .表示除了因变量外的所有变量 Y^=β0​+β1​X1​+β2​X2​+。。。
epa2021_lm0_fit <- 
  epa2021_lm0 %>% 
    fit(comb_mpg ~ ., data = epa2021_train)

# 所有变量 .表示除了因变量外的所有变量，并排出某些相关变量 Y^=β0​+β1​X1​+β2​X2​+。。。
epa2021_lm0_fit <- 
  epa2021_lm0 %>% 
    fit(comb_mpg ~ . - mfr_code - model_yr, data = epa2021_train)

```
#### 3. 验证模型

| 指标          | 公式                                        | 单位         | 特点    | 什么时候用  |    |        |
| ----------- | ----------------------------------------- | ---------- | ----- | ------ | -- | ------ |
| MAE         | ( \frac{1}{n}\sum                         | y - \hat y | )     | 与Y相同   | 稳健 | 业务误差直观 |
| RMSE        | ( \sqrt{\frac{1}{n}\sum (y - \hat y)^2} ) | 与Y相同       | 惩罚大误差 | 对大偏差敏感 |    |        |
| MSE         | 平方误差                                      | Y²         | 不直观   | 理论分析   |    |        |
| R²          | 解释变异比例                                    | 无单位        | 解释能力  | 回归解释   |    |        |
| Adjusted R² | 调整R²                                      | 无单位        | 惩罚变量多 | 多变量比较  |    |        |
| MAPE        | 百分比误差                                     | %          | 可跨尺度  | 金融业务   |    |        |
| AIC/BIC     | 信息准则                                      | 无单位        | 模型比较  | 模型选择   |    |        |


##### mean absolute error (MAE)

<img width="591" height="159" alt="image" src="https://github.com/user-attachments/assets/82315b39-30dc-4071-93fe-5368f4bc5b5a" />

###### yardstick

```
# 1️⃣ 得到训练集预测值（null model）
epa2021_lm0_fitted_vals <- 
  predict(epa2021_lm0_fit, new_data = epa2021_train)

# 2️⃣ 合并真实值和预测值
results_train_lm0 <- data.frame(
  actual = epa2021_train$comb_mpg,
  .pred  = epa2021_lm0_fitted_vals$.pred
)

# 3️⃣ 计算 MAE
epa2021_train_lm0_mae <- 
  yardstick::mae(
    results_train_lm0,
    truth = actual,
    estimate = .pred
  )

# 4️⃣ 打印结果
sprintf(
  'lm0: train : MAE = %.1f mpg',
  epa2021_train_lm0_mae$.estimate
)
```
###### MLmetrics

```
MLmetrics::MAE(results_train_lm0$actual, results_train_lm0$.pred)
```
##### RMSE
```
yardstick::rmse(
  results_train_lm0,
  truth = actual,
  estimate = .pred
)
```

#### 4. 通过散点图看预测值和真实值的关系

```
results_test_lm %>%
  ggplot( aes(x = actual, y = pred)) +
  geom_point(alpha = 0.5, color = "steelblue") +
  geom_abline(intercept = 0, slope = 1, 
              color = "red", linetype = "dashed") +
  labs(
    title = "Actual vs Predicted",
    x = "Actual MPG",
    y = "Predicted MPG"
  ) +
  theme_minimal()
```

<img width="709" height="448" alt="image" src="https://github.com/user-attachments/assets/b2deb336-aa3d-401e-9334-5ba9de551c6a" />

## 如何确认调用的方法来源于哪个包
```
> find("fit")
[1] "package:workflows" "package:tailor"    "package:parsnip"   "package:infer"    
> find("predict")
[1] "package:stats"
> find("select")
[1] "package:dplyr"
> find("mae")
[1] "package:yardstick"
> 
```

## 报错
### 数据只在测试集有而训练集没有
```
Error in model.frame.default(Terms, newdata, na.action = na.action, xlev = object$xlevels) :
factor transmission_speed has new levels Auto(AM-S9)
```
解决办法： 使用 recipe + step_novel()
```
library(recipes)
library(workflows)
library(parsnip)

rec <- recipe(comb_mpg ~ ., data = epa2021_train) %>%
  step_rm(mfr_code, model_yr, division) %>%   # 删除变量
  step_novel(all_nominal_predictors()) %>%  # 处理新 levels
  step_dummy(all_nominal_predictors())      # 生成 dummy

wf <- workflow() %>%
  add_model(linear_reg()) %>%
  add_recipe(rec)

fit_wf <- fit(wf, data = epa2021_train)

predict(fit_wf, new_data = epa2021_test)
```
