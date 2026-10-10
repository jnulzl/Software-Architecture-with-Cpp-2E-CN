# 《C++ 软件架构》（Software Architecture with C++，第二版）—— LaTeX 工程（AI 翻译）

原著：*Software Architecture with C++* (2nd Edition, Packt, 2025)
副标题：*Designing Robust C++ Systems with Modern Architectural Practices*
作者：Adrian Ostrowski、Piotr Gaczkowski、Andrey Gavrilin

## 目录结构

```
latex_source/
├── main.tex                 根文件
├── build.bat                一键编译脚本（纯 ASCII，请勿加中文注释）
├── README.md
├── book/
│   ├── ccs.tex              导言区：字体、版式、全部自定义命令与环境  ← 改版式改这里
│   ├── index.tex            封面、目录、部分页、全部 19 章的装配（gen_index.py 生成）
│   ├── manning-pygments-style.tex   代码高亮配色
│   └── content/
│       ├── front/           00-title（书名/版权）、01-contributors、02-preface、index（索引）
│       ├── part/            01–04（四个「部分」扉页 + 部分引言）
│       ├── main/            01–19 正文十九章
│       └── back/            assessments（测评题解答）、newsletter、other-books
├── fonts/                   JetBrainsMonoNL（等宽字体回退用；代码默认 Consolas）
└── images/                  cover.png + 148 张插图
```

## 编译

需要 **TeX Live 2026**（自带 `latexminted`）与 **Python 3**（`latexminted` 是被 `xelatex`
调起的 Python 脚本，代码块由它与 Pygments 做语法高亮）：

```bat
build.bat                          REM Windows 双击即可
build.bat noidx                    REM 跳过索引生成（只改正文时用，快很多）
```

`build.bat` 的步骤（共 4 遍 xelatex）：

> 如果环境配置遇到问题或者编译失败可以直接到[releases](https://github.com/jnulzl/Software-Architecture-with-Cpp-2E-CN/releases)下载最新的pdf文件

## 官方代码

[Software-Architecture-with-Cpp-2E](https://github.com/PacktPublishing/Software-Architecture-with-Cpp-2E)