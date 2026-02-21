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

#### convert-na-into-a-factor-level
```
library(forcats)

epa2021 <- epa2021 %>%
  mutate(
    car_truck = fct_na_value_to_level(car_truck, "Missing"),
    car_truck = fct_recode(car_truck, NotSure = "??")
  )
```
#### replace-na-in-a-factor-column
```
library(forcats)

epa2021 <- epa2021 %>%
  mutate(
    car_truck = fct_recode(car_truck, NotSure = "??")
  )
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




