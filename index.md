---
layout: home
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
  I am 4th year Ph.D. student at [Peking University](https://english.pku.edu.cn/), a member of [Programming Languages Lab](https://pl.cs.pku.edu.cn/en/). I am advised by Prof. [Zhenjiang Hu](https://zhenjiang888.github.io/). My main interest is in programming languages, and type systems, formal semantics, program verification in particular. I am working on topics related to simplifying DSL and IDE implementation. Welcome to discuss with me!

  <p class="itcomment">
  My email address consists of the first three languages I learned: Visual Basic, C/C++, and Pascal.
  </p>
</div>


## Education

- Ph.D. Student in Computer Science, 2021-
  <p class="comment">Programming Languages Lab, School of Computer Science, Peking University</p>
- BSc in Computer Science, 2017-2021
  <p class="comment">School of EECS, Peking University</p>

## Publications

{% include publications.md %}

## Teaching

- *Teaching assistant.* [Introduction to Functional Programming](https://zhenjiang888.github.io/FP/) (Fall 2022 and 2023).
- *Teaching assistant.* [Software Foundations](https://xiongyingfei.github.io/SF/) (Spring 2022).
- *Teaching assistant.* Introduction to Computing (A) (Fall 2019).
