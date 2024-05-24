---
title: "学习统计模型——通过R模拟"
author: "Zhou Bolin"
date: "2024-05-24"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib]
biblio-style: apa
csl: include/apa.csl
link-citations: yes
---



# 概述 {-}

本教材采用在R语言环境下模拟广义线性模型(General Linear Model, GLM)的方法来介绍统计分析。总体目标是教会学生如何将研究设计的描述转化为线性模型，来分析该研究的数据。重点是分析心理学实验数据所需的技能。

<div class="small_right"><img src="images/logos/logo.png" 
     alt="Stat Models Hex Logo" /></div>

本教材包括以下内容：

* 线性模型工作流程;
* 方差-协方差矩阵;
* 多元回归;
* 交互作用(连续与分类; 分类与分类);
* 线性混合模型;
* 广义线性混合模型。

本课程的内容构成了[格拉斯哥大学心理学院](https://www.gla.ac.uk/schools/psychologyneuroscience/)(University of Glasgow School of Psychology)由[Dale Barr](https://www.gla.ac.uk/schools/psychologyneuroscience/staff/dalebarr/)讲授的大学三年级一学期课程的基础。这也是由格拉斯哥大学心理学院工作人员开发的[PsyTeachR](https://psyteachr.github.io/)系列课程材料的一部分。

与你可能遇到的其他教材不同，这是一本**互动教材**。每一章都包含嵌入式练习和网页应用来帮助学生更好地理解内容。你只有通过浏览器访问这些材料，这些互动内容才能正常工作。因此，不建议打印这些材料。如果你希望在没网的情况下访问教材或保存本地版本以防止网站变化或迁移，可以[下载离线使用版本](https://psyteachr.github.io/stat-models-v1/offline-textbook.zip)。只需要从ZIP压缩包中提取文件，在`docs`目录中找到`index.html`文件，然后使用浏览器打开这个文件即可。

## 如何引用本书 {-}

Barr, Dale J. (2021). *Learning statistical models through simulation in R: An interactive textbook*. Version 1.0.0. Retrieved from <https://psyteachr.github.io/stat-models-v1>.

## 发现问题？ {-}

如果你发现错误或书写错误，有问题或建议，请在<https://github.com/psyteachr/stat-models-v1/issues>提交问题。谢谢！

## 教育者需知 {-}

您可以根据自己的需求免费重复使用和修改本教材中的材料，但需要注明原作出处。请注意关于重复使用本材料的 [Creative Commons CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) 许可证的其他条款。

本书是使用R [**`bookdown`**](https://bookdown.org)包构建的。源文件可在[github](https://github.com/psyteachr/stat-models-v1)上获得。
