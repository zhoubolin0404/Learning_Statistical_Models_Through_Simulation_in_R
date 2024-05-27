# 相关和回归



## 相关矩阵(Correlation matrices)

你可能已经通过阅读心理学论文对**相关矩阵**这个概念有所了解。相关矩阵是总结同一个体的多个测量值之间关系的常用方法。

假设你用多个量表测量了心理幸福感。一个问题是这些量表在多大程度上测量了相同的东西。通常，你会查看相关矩阵来探索测量之间所有成对关系。

回忆一下，相关系数量化了两个变量之间关系的**强度**和**方向**。它通常用符号$r$或$\rho$(希腊字母"rho")表示。相关系数的范围在-1到1之间，其中0表示没有关系，正值反映正相关(一个变量增加，另一个也增加)，负值反映负相关(一个变量增加，另一个减少)。

<div class="figure">
<img src="02-Correlation_and_Regression_files/figure-html/correlation-relationships-1.png" alt="不同类型的二元关系" width="576" />
<p class="caption">(\#fig:correlation-relationships)不同类型的二元关系</p>
</div>

如果你有$n$个测量值，你可以计算多少个成对相关？你可以用下面蓝框中的公式来计算，也可以更简单地通过R中的`choose(n, 2)`函数直接计算。例如，要获得6个测量值之间可能的成对相关数量，你可以输入`choose(6, 2)`，这会告诉你有15对组合。

<div class = "bluebox">

对于任意$n$个测量值，你可以计算$\frac{n!}{2(n - 2)!}$个各值之间的成对相关。符号$!$称为**阶乘(factorial)**运算符，定义为从1到$n$所有数字的乘积。因此，如果你有6个测量值，你可以得到

$$
\frac{6!}{2(6-2)!} = \frac{1 \times 2 \times 3 \times 4 \times 5 \times 6}{2\left(1 \times 2 \times 3 \times 4\right)} = \frac{720}{2(24)} = 15
$$

</div>

你可以使用R中的`base::cor()`或`corrr::correlate()`来创建相关矩阵。我们更喜欢后者函数，因为`cor()`要求你的数据存储在矩阵中，而我们将处理的大多数数据是存储在数据框中的表格数据。`corrr::correlate()`函数将数据框作为第一个参数，并提供"整洁"的输出，因此它可以更好地与tidyverse系列函数和管道操作符(`%>%`)联动。

让我们创建一个相关矩阵来看看它是如何工作的。首先加载我们需要的包。


```r
library("tidyverse")
library("corrr")  # 如缺失(missing)，在控制台(console)中输入install.packages("corrr")
```

我们将使用`starwars`数据集，这是在加载tidyverse包后可用的内置数据集。该数据集包含了出现在星球大战电影系列中的各种角色的信息。让我们来看看之间的相关性


```r
starwars %>%
  select(height, mass, birth_year) %>%
  correlate()
```

```
## Correlation computed with
## • Method: 'pearson'
## • Missing treated using: 'pairwise.complete.obs'
```

```
## # A tibble: 3 × 4
##   term       height   mass birth_year
##   <chr>       <dbl>  <dbl>      <dbl>
## 1 height     NA      0.131     -0.404
## 2 mass        0.131 NA          0.478
## 3 birth_year -0.404  0.478     NA
```

你可以在任意给定的行或列的交叉处查找任何双变量相关系数。因此，`height`和`mass`之间的相关系数是.131，你可以在第1行，第2列或第2行，第1列找到它------它们是相同的。请注意，这里只有`choose(3, 2)` = 3个唯一的双变量关系，但每个关系在表中出现了两次。我们可能只想显示唯一的组合，这可以通过在管道中附加`corrr::shave()`来实现。


```r
starwars %>%
  select(height, mass, birth_year) %>%
  correlate() %>%
  shave()
```

```
## Correlation computed with
## • Method: 'pearson'
## • Missing treated using: 'pairwise.complete.obs'
```

```
## # A tibble: 3 × 4
##   term       height   mass birth_year
##   <chr>       <dbl>  <dbl>      <dbl>
## 1 height     NA     NA             NA
## 2 mass        0.131 NA             NA
## 3 birth_year -0.404  0.478         NA
```

现在我们只有相关矩阵的下三角部分，但`NA`看起来很难看，前导0也不美观。**`corrr`**包还提供了`fashion()`函数，可以对其进行清理(更多选项请查阅`?corrr::fashion`)。


```r
starwars %>%
  select(height, mass, birth_year) %>%
  correlate() %>%
  shave() %>%
  fashion()
```

```
## Correlation computed with
## • Method: 'pearson'
## • Missing treated using: 'pairwise.complete.obs'
```

```
##         term height mass birth_year
## 1     height                       
## 2       mass    .13                
## 3 birth_year   -.40  .48
```

相关性只有在关系(大致)线性且没有严重的异常值对结果产生过大影响时才能很好地描述关系。因此，可视化相关性通常和量化它们一样是个好主意。`base::pairs()`函数可以实现这一点。`pairs()`的第一个参数形式为`~ v1 + v2 + v3 + ... + vn`，其中`v1`、`v2`等是你想要进行相关分析的变量名。


```r
pairs(~ height + mass + birth_year, starwars)
```

<div class="figure">
<img src="02-Correlation_and_Regression_files/figure-html/pairs-1.png" alt="星球大战数据集相关关系" width="576" />
<p class="caption">(\#fig:pairs)星球大战数据集相关关系</p>
</div>

我们会发现一个巨大的离群值影响了我们的数据。具体来说是有个体重超过1200kg的生物。让我们找出它并从数据集里面删掉它。


```r
starwars %>%
  filter(mass > 1200) %>%
  select(name, mass, height, birth_year)
```

```
## # A tibble: 1 × 4
##   name                   mass height birth_year
##   <chr>                 <dbl>  <int>      <dbl>
## 1 Jabba Desilijic Tiure  1358    175        600
```

好了，让我们看看没有了这个庞然大物的数据会是什么样子。


```r
starwars2 <- starwars %>%
  filter(name != "Jabba Desilijic Tiure")

pairs(~height + mass + birth_year, starwars2)
```

<div class="figure">
<img src="02-Correlation_and_Regression_files/figure-html/massive-creature-1.png" alt="去除体重离群值后星球大战数据集相关关系" width="672" />
<p class="caption">(\#fig:massive-creature)去除体重离群值后星球大战数据集相关关系</p>
</div>

好多了，但还有个生物的离群出生年份可能是我们不想要的。


```r
starwars2 %>%
  filter(birth_year > 800) %>%
  select(name, height, mass, birth_year)
```

```
## # A tibble: 1 × 4
##   name  height  mass birth_year
##   <chr>  <int> <dbl>      <dbl>
## 1 Yoda      66    17        896
```

是尤达大师！他和宇宙一样古老。让我们抛开他看看图会怎么样。


```r
starwars3 <- starwars2 %>%
  filter(name != "Yoda")

pairs(~height + mass + birth_year, starwars3)
```

<div class="figure">
<img src="02-Correlation_and_Regression_files/figure-html/bye-yoda-1.png" alt="去除体重和出生年份离群值后星球大战数据集相关关系" width="576" />
<p class="caption">(\#fig:bye-yoda)去除体重和出生年份离群值后星球大战数据集相关关系</p>
</div>

看起来更好了。让我们看看它是怎样改变我们的相关矩阵的。


```r
starwars3 %>%
  select(height, mass, birth_year) %>%
  correlate() %>%
  shave() %>%
  fashion()
```

```
## Correlation computed with
## • Method: 'pearson'
## • Missing treated using: 'pairwise.complete.obs'
```

```
##         term height mass birth_year
## 1     height                       
## 2       mass    .73                
## 3 birth_year    .44  .24
```

请注意，这些值与我们开始时的值有很大不同。

有时移除离群值不是一个好办法。处理离群值的另一个办法是使用一种更稳健(robust)的方法。使用`corrr::correlate()`默认计算的相关系数是Pearson积差相关(Pearson product-moment correlation)系数。我们也能通过改变`correlate()`的`method()`参数来计算Spearman相关系数。这将在计算相关性之前用排名替换原始值，因此仍会包括离群值，但影响将大大减小。


```r
starwars %>%
  select(height, mass, birth_year) %>%
  correlate(method = "spearman") %>%
  shave() %>%
  fashion()
```

```
## Correlation computed with
## • Method: 'spearman'
## • Missing treated using: 'pairwise.complete.obs'
```

```
##         term height mass birth_year
## 1     height                       
## 2       mass    .72                
## 3 birth_year    .15  .15
```

顺便一提，如果你用R Markdown生成报告，并希望你的表格有个好看的格式，可以使用`knitr:: able()`。


```r
starwars %>%
  select(height, mass, birth_year) %>%
  correlate(method = "spearman") %>%
  shave() %>%
  fashion() %>%
  knitr::kable()
```



|term       |height |mass |birth_year |
|:----------|:------|:----|:----------|
|height     |       |     |           |
|mass       |.72    |     |           |
|birth_year |.15    |.15  |           |

## 模拟二元数据

你已经学会了使用`rnorm()`函数从正态分布中模拟数据。回忆一下，`rnorm()`允许你指定单个变量的均值和标准差。那我们怎么模拟相关变量呢？

应该很明确，你不能仅仅运行两次`rnorm()`后组合变量就完事。因为这会得到两个不相关的变量，即相关性为零。

**`MASS`**包提供`mvrnorm()`函数，这是rnorm的"多元"(multivariate)版(因此函数的名字是'mv' + 'rnorm'，这样更容易记住)。

<div class = "blueborder">

R预装了**`MASS`**包。但**`MASS`**包中你唯一可能会用到的函数只是`mvrnorm()`，因此相比于使用`library("MASS")`加载包，使用`MASS::mvrnorm()`是更好的办法，尤其是在**`MASS`**和**`tidyverse`**里的**`dplyr`**包不太合得来的情况下(因为两个包都有`select()`函数)。因此，如果在加载**`tidyverse`**之后加载**`MASS`**，那么最终得到的`select()`是**`MASS`**版本，而不是**`dplyr`**版本。这会让你绞尽脑汁来找出代码的问题所在，所以总在不加载的情况下使用`MASS::mvrnorm()`吧。

这里作者贴了一则他在Twitter(现X)上吐槽**`MASS`**的打油诗，不过现在已经查不到了QwQ，考虑到翻译水平不佳，附上原文！

> MASS before dplyr, clashes not dire; <br> dplyr before MASS, pain in the ass.
>
> ------ Dale Barr(September 30, 2014)

</div>

请查看`mvrnorm()`函数的文档(在控制台输入`?MASS::mvrnorm`)。

有3个参数需要注意：

| 参数  | 描述                                       |
|:------|:-------------------------------------------|
| n     | 所需样本数                                 |
| mu    | 一个给出变量均值的向量                     |
| Sigma | 一个正定对称矩阵，用于指定变量的协方差矩阵 |

对`n`和`mu`的描述可以理解，但“一个正定对称矩阵(positive-definite symmetric matrix)，用于指定变量的协方差矩阵”是什么意思呢？

当你有个多元数据时，**协方差矩阵**(也叫**方差-协方差矩阵**)反映了各个变量的方差及其相互关系。它类似于**标准差**的多维版本。要充分描述单变量正态分布，你只需要知道均值和标准差；要描述双变量正态分布，你需要分别知道两个变量的均值、标准差和他们的相关性；对于包含两个以上变量的多元分布，你需要知道所有变量的均值、它们的标准差以及所有可能的成对相关性。**这些概念在我们开始讨论混合效应模型时会变得非常重要。**

你可以将协方差矩阵看作类似于之前见过的相关矩阵；实际上，通过一些计算，你可以将协方差矩阵转换为相关矩阵。

<div class = "bluebox">

**你在说什么《黑客帝国(Matrix)》？那不是从上世纪90年代开始的科幻电影系列吗？**

在数学中，矩阵只是向量概念的推广:向量被认为只有一个维度，而矩阵可以是任意数量的维度。

那么矩阵

$$
\begin{pmatrix}
1 & 4 & 7 \\
2 & 5 & 8 \\
3 & 6 & 9 \\
\end{pmatrix}
$$

是一个3(行) x 3(列)矩阵，包括了列向量$\begin{pmatrix} 1 \\ 2 \\ 3 \\ \end{pmatrix}$，$\begin{pmatrix} 4 \\ 5 \\ 6 \\ \end{pmatrix}$和$\begin{pmatrix} 7 \\ 8 \\ 9 \\ \end{pmatrix}$。通常我们用$i$ x $j$的形式表示矩阵，$i$是行数，$j$是列数。因此，一个3x2矩阵有3行2列，像这样：

$$
\begin{pmatrix}
a & d \\
b & e \\
c & f \\
\end{pmatrix}
$$

**方阵**是行数等于列数的矩阵。

你可以用`matrix()`函数在R中创建一个方阵，或者使用base R的 `cbind()`和`rbind()`将向量连接在一起，它们分别将向量按列和按行连接在一起。在控制台试试`cbina(1:3, 4:6, 7:9)`吧。

</div>

那么“正定”和“对称”是什么呢？这是对可以表示多元正态分布的这类矩阵的数学要求。换句话说，你提供的协方差矩阵必须表示一个合法的多元正态分布。在这点上，你真的不需要知道再多了。

<iframe src="https://dalejbarr.github.io/bivariate/index.html" width="420" height="620" class = "noborder">

</iframe>

让我们从模拟假设的人类身高和体重的数据开始。我们知道这些是相关的。为了能够模拟数据，我们需要这两个变量的均值和标准差以及它们的相关性。

我找到了一些[数据](https://www.geogebra.org/m/RRprACv4)并把它转换为CSV文件。如果你想跟上，可以下载这个文件[heights_and_weights.csv](data/heights_and_weights.csv){download="heights_and_weights.csv"}。这是散点图：


```r
handw <- read_csv("data/heights_and_weights.csv", col_types = "dd")

ggplot(handw, aes(height_in, weight_lbs)) + 
  geom_point(alpha = .2) +
  labs(x = "height (inches)", y = "weight (pounds)") 
```

<div class="figure">
<img src="02-Correlation_and_Regression_files/figure-html/heights-and-weights-1.png" alt="475人的身高和体重(包括婴儿)" width="672" />
<p class="caption">(\#fig:heights-and-weights)475人的身高和体重(包括婴儿)</p>
</div>

这不是一个线性关系。我们可以先对每个变量进行对数(log)变换。


```r
handw_log <- handw %>%
  mutate(hlog = log(height_in),
         wlog = log(weight_lbs))
```

<div class="figure">
<img src="02-Correlation_and_Regression_files/figure-html/handw-log-1.png" alt="对数变换后的身高和体重" width="672" />
<p class="caption">(\#fig:handw-log)对数变换后的身高和体重</p>
</div>

散点图右上侧尾部有一个大的点簇，这可能表示在这个样本中成年人比儿童更多，因为成年人更高和更重。

对数身高的均值是4.11 (SD = 0.26)，而对数体重的均值是4.74 (SD = 0.65)。对数身高和对数体重之间的相关性我们可以用`cor()`函数获得，高达0.96。

我们现在有了模拟500人身高和体重所需的所有信息。但我们如何将这些信息输入到`MASS::mvrnorm()`呢？我们知道函数调用的第一部分是`MASS::mvrnorm(500, c(4.11,4.74), ...)`，但`Sigma`——那个协方差矩阵呢？我们从上面知道$\hat{\sigma}_x = 0.26$和$\hat{\sigma}_y = 0.65$，以及$\hat{\sigma}_y = 0.65$, and $\hat{\rho}_{xy} = 0.96$。

表示二元数据`Sigma` ($\mathbf{\Sigma}$)的协方差矩阵如下：

$$
\mathbf{\Sigma} =
\begin{pmatrix}
{\sigma_x}^2                & \rho_{xy} \sigma_x \sigma_y \\
\rho_{yx} \sigma_y \sigma_x & {\sigma_y}^2 \\
\end{pmatrix}
$$

