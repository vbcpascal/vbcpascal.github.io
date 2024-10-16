---
layout: home
language: zh
---

<div class="col col-left">
  <div class="profile">
    <img class="avatar" src="/assets/author/self2.jpg">
    <div class="profile-info">
      {%- if page.language == "zh" -%}
      <h2> {{ site.author.name_zh }} </h2>
      {%- else -%}
      <h2> {{ site.author.name }} </h2>
      {%- endif -%}
      <h3> vbcpascal@outlook.com </h3>
    </div>
  </div>

</div>

<div class="col col-right" markdown="1">
  你好！这里是我的个人主页。我目前就读于[北京大学](https://www.pku.edu.cn)博士四年级，是[程序设计语言研究室](https://pl.cs.pku.edu.cn/)的一员，导师是[胡振江](https://zhenjiang888.github.io/)教授。我钟爱对程序语言的研究，特别是类型系统、形式化语义、程序验证的相关问题。我目前的研究致力于简化领域特定语言（DSL）及其集成开发环境（IDE）的开发。欢迎与我讨论交流！

  <p class="itcomment">
  我的邮箱名由我最先学习的三门语言组成：Visual Basic、C/C++、Pascal。
  </p>
</div>


## 教育经历

- 计算机软件与理论 博士（2021-）
  <p class="comment">北京大学 计算机学院 程序设计语言研究室</p>
- 计算机科学 学士（2017-2021）
  <p class="comment">北京大学 信息科学技术学院</p>

## 论文

{% include publications.md %}

## 课程

- 助教：[计算概论（函数式程序设计）](https://zhenjiang888.github.io/FP/) （2022秋、2023秋）
- 助教：[软件科学基础](https://zhenjiang888.github.io/FP/) （2022春）
- 助教：计算概论（A）（2019秋）
