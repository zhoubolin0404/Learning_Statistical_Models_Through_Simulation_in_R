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

::: {style="border: 2px solid blue; padding: 10px; background-color: #e6f7ff;"}

对于任意$n$个测量值，你可以计算$\frac{n!}{2(n - 2)!}$个各值之间的成对相关。符号$!$称为**阶乘(factorial)**运算符，定义为从1到$n$所有数字的乘积。因此，如果你有6个测量值，你可以得到

$$
\frac{6!}{2(6-2)!} = \frac{1 \times 2 \times 3 \times 4 \times 5 \times 6}{2\left(1 \times 2 \times 3 \times 4\right)} = \frac{720}{2(24)} = 15
$$
:::

<br>

你可以使用R中的`base::cor()`或`corrr::correlate()`来创建相关矩阵。我们更喜欢后者函数，因为`cor()`要求你的数据存储在矩阵中，而我们将处理的大多数数据是存储在数据框中的表格数据。`corrr::correlate()`函数将数据框作为第一个参数，并提供“整洁”的输出，因此它可以更好地与tidyverse系列函数和管道操作符(`%>%`)联动。

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

你可以在任意给定的行或列的交叉处查找任何双变量相关系数。因此，`height`和`mass`之间的相关系数是.131，你可以在第1行，第2列或第2行，第1列找到它——它们是相同的。请注意，这里只有`choose(3, 2)` = 3个唯一的双变量关系，但每个关系在表中出现了两次。我们可能只想显示唯一的组合，这可以通过在管道中附加`corrr::shave()`来实现。


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

现在我们只有相关矩阵的下三角部分，但`NA`看起来很难看，前导0也不美观。**`corrr`**包还提供了`fashion()`函数，可以对其进行清理(更多选项请参见`?corrr::fashion`)。


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

