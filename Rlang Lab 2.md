## 一. 实验目的
1. 理解鸢尾花数据集的结构； 
2. 掌握R语言利用机器学习算法进行处理的流程和语法；
3. 掌握R语言数据可视化的基本方法；
4. 掌握机器学习模型的建立与模型选择的方法；
5. 掌握常用的机器学习算法在鸢尾花数据集上的应用。

## 二、实验环境
Windows系统，RGui（32-bit）

## 三、实验内容
鸢尾花（Iris）数据集是常用的分类实验数据集，由Fisher, 1936收集整理。
数据集包含150个数据，分为3类，每类50个数据，每个数据包含4个属性。
可通过花萼长度，花萼宽度，花瓣长度，花瓣宽度4个属性预测鸢尾花卉属于（Setosa，Versicolour，Virginica）三个种类中的哪一类。
本实验对鸢尾花数据各个特征的相关性进行分析，接下来实验几种常用的机器学习算法对该数据进行分类的效果，最后选出分类效果较好的方法。

## 四、实验内容

### 4.1 安装并加载caret包
```
install.packages("caret")
library(caret)
```

### 4.2 加载数据并保存到dataset
```
data(iris)
dataset <- iris
head(dataset)
```

<img width="362" alt="{929E94B7-711C-499E-AD80-53471BDAC792}" src="https://github.com/user-attachments/assets/34f6afb1-6a83-4a82-99b9-2323cd9ea8f4">

### 4.3 将数据集分为训练与验证
```
set.seed(34)
index <- createDataPartition(dataset$Species, p = 0.8, list = FALSE)

train_data <- dataset[index, ]
test_data <- dataset[-index, ]
```

### 4.4 查看数据的属性
<img width="342" alt="{447FC9A7-D72F-4540-9D18-F72606216703}" src="https://github.com/user-attachments/assets/207bc34f-cc02-4452-811b-f7eea5424c42">

<img width="450" alt="{3E172FF7-93A6-47DF-8CBA-5CAEC87CCE3C}" src="https://github.com/user-attachments/assets/142bd4e9-4ebc-4679-9a85-5ce628d3213e">

<img width="431" alt="{784BA4BD-1123-483F-B946-A330ECCA72E9}" src="https://github.com/user-attachments/assets/ef5d3409-b8dc-4ba0-9999-a7f6f76e3d82">

<img width="380" alt="{9DC3201E-EDE7-40AB-B187-866023C4268C}" src="https://github.com/user-attachments/assets/75b0e420-1085-4557-a7d0-9535f41f2539">

<img width="424" alt="{B7C0BA1E-3033-46E1-A10A-911B1DEA3782}" src="https://github.com/user-attachments/assets/5671142d-a648-493c-a358-602f8d3dc823">

<img width="421" alt="{F3458F14-9833-4881-862B-98674A833B6B}" src="https://github.com/user-attachments/assets/db53893d-97ca-4081-9100-4422db1619dc">

### 4.5 数据集的可视化
```
# 单变量图
x <- dataset[, 1:4]  # 特征变量：Sepal.Length, Sepal.Width, Petal.Length, Petal.Width
y <- dataset[, 5]    # 目标变量：Species

# 设置图形布局为一行四列
par(mfrow=c(1,4))

for(i in 1:4) {      # 为每个变量：
  boxplot(x[,i], main=names(iris)[i])  # 为每个特征绘制箱线图，并标注特征名
}

# 绘制目标变量（类别）的简单散点图
plot(y)
```

<img width="341" alt="{4EA23E30-972A-4051-B8B1-D7E609808A5C}" src="https://github.com/user-attachments/assets/2e49855b-58bc-4d45-86ee-ef35d593cd48">

```
# 多变量图

# 绘制3个类别的椭圆图，分别使用 1 和 2 列的特征（Sepal.Length, Sepal.Width）进行绘制
featurePlot(x = x[y == "setosa", 1:2], y = y[y == "setosa"], plot = "ellipse")
featurePlot(x = x[y == "versicolor", 1:2], y = y[y == "versicolor"], plot = "ellipse")
featurePlot(x = x[y == "virginica", 1:2], y = y[y == "virginica"], plot = "ellipse")
```

<img width="326" alt="{A5FE2606-51A8-40D0-AE77-3A08B4628B7C}" src="https://github.com/user-attachments/assets/101a90b5-9f96-49b5-bb5f-6d948a3bc527">

<img width="286" alt="{B3A7D6A0-2841-48A9-A62E-15B8A531B08D}" src="https://github.com/user-attachments/assets/3ccc1b81-9582-411b-91c9-36f9c9fe4945">

**如何知道什么线条表示的是哪种类型？**
* 默认情况下，分类变量（Species）的每个类别会用不同颜色表示。

**椭圆图上 Sepal.Length 是第一列的横坐标，第四行的纵坐标吗？**
* 横坐标表示长度，纵坐标表示宽度
* 是的

### 4.6 算法评估
#### 4.6.1 十折交叉验证
十折交叉验证将数据集划分为 10个子集，每个子集依次作为验证集，其他9个子集作为训练集，进行模型的训练和验证。
```
# 十折交叉验证
control <- trainControl(method = "cv", number = 10)
metric <- "Accuracy"
```

#### 4.6.2 LDA线性判别模型
在多个自变量和一个因变量之间建立线性模型，通过线性空间可分性将因变量分类
```
# 线性判别模型LDA
set.seed(34)
# 用数据集中所有特征预测目标变量
fit.lda <- train(Species ~ ., data = train_data, method = "lda", trControl = control, metric = metric)
```

#### 4.6.3 分类和回归树
通过`cart`树将数据分类
```
# 分类和回归树
set.seed(34)
fit.cart <- train(Species ~ ., data = train_data, method = "rpart", trControl = control, metric = metric)
```

#### 4.6.4 k邻近
KNN的基本思想是：给定一个新的输入样本，KNN会计算该样本与训练集中所有样本的距离，
然后选取与该输入样本最接近的 K个邻居，并根据这K个邻居的类别来进行预测
```
# kNN
set.seed(34)
fit.knn <- train(Species ~ ., data = train_data, method = "knn", trControl = control, metric = metric)
```

#### 4.6.5 支持向量机
```
# SVM
set.seed(34)
fit.svm <- train(Species ~ ., data = train_data, method = "svmLinear", trControl = control, metric = metric)
```

#### 4.6.6 随机森林
```
# RF
set.seed(34)
fit.rf <- train(Species ~ ., data = train_data, method = "rf", trControl = control, metric = metric)
```

#### 4.6.7 展示结果
```
# 创建一个包含所有模型的列表并进行比较
results <- resamples(list(lda = fit.lda, cart = fit.cart, knn = fit.knn, svm = fit.svm, rf = fit.rf))

# 显示模型的准确性
summary(results)
```

<img width="394" alt="{A8DD331B-AB39-41BC-907D-47C556469816}" src="https://github.com/user-attachments/assets/624c0865-b021-4105-a465-3e793f1b86da">

```
# 将模型的准确率可视化
dotplot(results)
```

<img width="316" alt="{257398B7-EFFE-4E84-8FCF-159CB71E2E1F}" src="https://github.com/user-attachments/assets/413b5191-3888-4ffa-9df5-f26de8b2d57f">

```
# 查看LDA模型的详细信息
print(fit.lda)
```

<img width="352" alt="{B12AE61C-872E-4640-8541-04EB4BDCB13F}" src="https://github.com/user-attachments/assets/3dd49aa7-5714-4685-9c3a-b706410af6c4">

### 4.7 做出预测

```
# 在验证集上做出预测
predictions <- predict(fit.lda, test_data)

# 显示混淆矩阵
confusionMatrix(predictions, test_data$Species)
```

<img width="460" alt="{AC4D0150-8FF7-4FD6-A6A3-00B82A8EE4A3}" src="https://github.com/user-attachments/assets/54d3081f-b210-4e50-9b81-70f4d0d9053f">
