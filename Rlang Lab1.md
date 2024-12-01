## 一、实验目的
1. 掌握R语言数据结构； 
2. 掌握R语言绘制直方图、密度估计曲线、经验分布图和QQ图的方法；
3. 掌握R语言绘制茎叶图、箱线图的方法；
4. 掌握描述性统计分析中常用的统计量；
5. 掌握R语言简单线性回归分析；
6. 掌握R语言的各项功能和函数，能够通过完成试验内容对R语言有一定的了解，会运用软件对数据进行分析。

## 二、实验环境
Windows系统，RGui（32-bit）

## 三、实验内容

### 1. 创建三百个学生并设置学号
```
set.seed(34)

num_students <- 300
start_id <- 210222001
xuehao <- seq(start_id, by = 1, length.out = num_students)
```

### 2. 生成各科数据

#### 2.1 高数
```
gaoshu <- rnorm(num_students, mean = 70, sd = sqrt(10))
gaoshu <- round(pmin(pmax(gaoshu, 0), 100))
```

#### 2.2 线代
```
gaoshu <- rnorm(num_students, mean = 70, sd = sqrt(10))
gaoshu <- round(pmin(pmax(gaoshu, 0), 100)) 
```

#### 2.3 英语
```
yingyu <- runif(num_students, min = 56, max = 99)
yingyu <- round(yingyu)
```

#### 2.4 程序设计
```
chengshe <- rnorm(num_students, mean = 85, sd = sqrt(12))
chengshe <- round(pmin(pmax(chengshe, 0), 100))
```

### 3. 组合为数据框并保存
```
classscore <- data.frame(
  xuehao = xuehao,
  gaoshu = gaoshu,
  xiandai = xiandai,
  yingyu = yingyu,
  chengshe = chengshe
)
```

```
write.table(classscore, file = "classscore.txt", row.names = FALSE, sep = "\t")
```

<img width="392" alt="{AE3C8D6A-AF5E-42F0-80F2-E20F47655EFC}" src="https://github.com/user-attachments/assets/d12599e7-8442-4571-90c8-b4b3855f7b7d">


### 4. 计算平均分
```
classscore$average <- rowMeans(classscore[, 2:5]) 
classscore$total <- rowSums(classscore[, 2:5])
```

```
col_means <- colMeans(classscore[, 2:5]) # 各科平均分
```

### 5. 计算指标
#### 5.1 计算各科最高最低分
```
max_scores <- apply(classscore[, 2:5], 2, max)  # 各科最高分
min_scores <- apply(classscore[, 2:5], 2, min)  # 各科最低分
```

#### 5.2 按分位数分为四个等级
```
quantiles <- quantile(classscore$total, probs = c(0.2, 0.4, 0.6)) 

classscore$grade <- cut(
  classscore$total,
  breaks = c(-Inf, quantiles, Inf), 
  labels = c("D", "C", "B", "A"),   
)

table(classscore$grade) 
```
* 错误一：本题只区分了四个等级，后40%的全部是D，出现
`Error in cut.default(classscore$total, breaks = c(-Inf, quantiles, Inf), : number of intervals and length of 'labels' differ`
的原因是拆成了五个部分（20分位）

<img width="147" alt="{DFDF5898-934E-493E-BEA7-E665DBF6D312}" src="https://github.com/user-attachments/assets/0a8db857-8ff2-4d35-ae64-4054f6496cd9">

### 6. 求总分最高
```
max_total <- max(classscore$total)  
max_xuehao <- classscore$xuehao[classscore$total == max_total] 

print(paste("总分最高的学生学号为:", max_xuehao))
```
<img width="417" alt="{2392AF3D-AC92-4A04-B148-BB0CCA37BD92}" src="https://github.com/user-attachments/assets/df289460-fa52-44c4-98e0-f7b889e2851d">


### 7. 绘图

#### 7.1 高数直方图
```
hist(classscore$gaoshu, 
     main = "Math hist", 
     xlab = "score", 
     col = "skyblue", 
     breaks = 10)
```

<img width="364" alt="{2C14E1ED-EFD8-4058-95B6-2213C6754230}" src="https://github.com/user-attachments/assets/4668ea56-13ca-4f31-ad02-0ea97b7a2dcc">


#### 7.2 高数柱状图
```
barplot(table(classscore$gaoshu), 
        main = "Math barplot", 
        xlab = "score", 
        ylab = "num", 
        col = "orange")
```
<img width="484" alt="{5A28C631-AB69-49BA-A768-34562077A52A}" src="https://github.com/user-attachments/assets/d7314bb7-cb7b-4a88-b2e7-1df94291cdfe">

#### 7.3 高数饼图
```
gaoshu_freq <- table(cut(classscore$gaoshu, breaks = c(0, 59, 69, 79, 89, 100)))
pie(gaoshu_freq, 
    labels = c("E", "D", "C", "B", "A"), 
    main = "Math pie", 
    col = rainbow(length(gaoshu_freq)))
```

<img width="283" alt="{A608D907-98F1-4507-92C3-09E783BCAE7F}" src="https://github.com/user-attachments/assets/0f8fd632-03f6-420e-8393-cc02a37cc376">

#### 7.4 高数-线代散点图
```
plot(classscore$gaoshu, classscore$xiandai, 
     main = "Math-Algebra plot", 
     xlab = "Math", 
     ylab = "Algebra", 
     col = "blue", 
     pch = 16)
```

<img width="441" alt="{080D6707-F539-49D5-B9B3-2052EEBA7144}" src="https://github.com/user-attachments/assets/5998f8f5-35f3-4cc9-bae6-dc1f017901f6">


#### 7.5 高数-英语散点图
```
plot(classscore$gaoshu, classscore$yingyu, 
     main = "Math-English plot", 
     xlab = "Math", 
     ylab = "English", 
     col = "green", 
     pch = 16)
```

<img width="395" alt="{7D395842-DC27-4480-9D46-E19FA51BF7A0}" src="https://github.com/user-attachments/assets/a942501c-9dcd-42df-b977-e154d0a00716">

### 8.画星相图，解释其含义

#### 8.1 检查fmsb包
```
if (!require(fmsb)) install.packages("fmsb")
library(fmsb)
```

#### 8.2 准备数据
* 本题选取前五位学生
* 星象图的数据依赖于学生的成绩最大最小值

```
star_data <- classscore[1:5, c("gaoshu", "xiandai", "yingyu", "chengshe")]

max_values <- apply(star_data, 2, max)  
min_values <- apply(star_data, 2, min)  

star_data <- rbind(max_values, min_values, star_data)
```

#### 8.3 绘制星象图
```
radarchart(star_data,
           axistype = 1,
           pcol = c("red", "blue", "green", "purple", "orange"),  # 各学生的颜色
           plwd = 4, 
           cglcol = "grey", 
           axislabcol = "black",  
           caxislabels = seq(min(min_values), max(max_values), length.out = 6),  # 根据动态范围设置刻度
           title = "Score") 

legend("topright", legend = paste("学生", 1:5), col = c("red", "blue", "green", "purple", "orange"), lty = 1, lwd = 2)
```

<img width="481" alt="{2525632F-8499-4643-A27B-0008816B3F54}" src="https://github.com/user-attachments/assets/bdc0e3ef-2221-4d6b-92d6-d64ad89265d8">

#### 8.4 解释含义
* 轴线长度代表该学生学科相对水平
* 面积代表该学生总体水平
* 可以看出四号学生总体实力更强，三号学生偏科英语

### 九、相关性分析
#### 9.1 散点图
```
# 线性回归模型：高等数学与线性代数
lm_gaoshu_xiandai <- lm(xiandai ~ gaoshu, data = classscore)

plot(classscore$gaoshu, classscore$xiandai, 
     main = "Math-Algobra", 
     xlab = "Math", 
     ylab = "Algobra", 
     col = "blue", 
     pch = 16)

abline(lm_gaoshu_xiandai, col = "red", lwd = 2)

summary(lm_gaoshu_xiandai)
```

<img width="508" alt="{CB8C57E2-F955-4401-951C-862021CFC5F8}" src="https://github.com/user-attachments/assets/d40420fd-776d-4f5a-a4f7-04877d506b39">

* `Multiple R-squared:  0.8084` 说明`R2`在0.8以上
* `p-value: < 2.2e-16`说明置信度极强
* 可以认为高数成绩与线代成绩成正相关

#### 9.2 ggplot2绘制回归线
```
ggplot(classscore, aes(x = gaoshu, y = xiandai)) +
  geom_point(color = "blue") + 
  geom_smooth(method = "lm", col = "red") +  
  labs(
    title = "Math-Algobra",
    x = "Math",
    y = "Algobra"
  ) +
  theme_minimal()
```
<img width="445" alt="{7D319C46-CD5A-426C-A814-67CDEB118888}" src="https://github.com/user-attachments/assets/af53c9a8-2a5b-4445-bba1-b63fc1c19e88">

### 10. 生成社会实践成绩

```
level <- c("A", "B", "C", "D", "E")

shijian <- sample(level, 300, replace = TRUE)

classscore$shijian <- shijian

head(classscore)
```
<img width="485" alt="{FE4D63CE-402C-4908-9BFA-E89310485DB2}" src="https://github.com/user-attachments/assets/b6a11f67-3fea-4d26-8df3-a8149ac279be">

### 11. 绘制饼图
```
shijian_table <- table(classscore$shijian)

print(shijian_table)

plot(shijian_table,
     main = "plot",
     xlab = "grade",
     ylab = "num",
     col = "skyblue")

pie(shijian_table,
    labels = paste(names(shijian_table), ":", shijian_table),
    main = "pie",
    col = rainbow(length(shijian_table)))
```

<img width="440" alt="{77A065DA-2C20-4ACC-9422-62373AE9F2A4}" src="https://github.com/user-attachments/assets/5b912510-957d-4fd8-a502-74e465fe845e">
