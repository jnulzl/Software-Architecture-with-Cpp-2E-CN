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

1. `gen_index.py --phase plan` —— 写 `book/index.tex`；索引正文若不存在则先写占位 stub
2. **xelatex 第 1 遍** —— 把全书 2090 个 `\label{idx:NNN}` 收进 `main.aux`
3. `gen_index.py --phase final` —— 从 `main.aux` 取真实页码，写正式索引
4. **xelatex 第 2–4 遍** —— 目录、书签、页码收敛

> 索引必须这么做（两阶段），因为词条页码是真页码：LaTeX 里没法给 `\pageref` 排序、
> 去重、把连号压成 `188–190`，只能在 Python 侧拿到数字后再写死。
> 索引排在全书最后，它的长短不会改变任何正文页码，所以采集一次就够。

git-bash 会话里等价的封装：

```bash
bash work/tools/full_build.sh      # 四遍编译 + 索引，并自动核验 main.log
bash work/tools/compile.sh 3       # 只跑 xelatex（索引用上一次的）
```

> ⚠ 使用 **Consolas**（系统字体）作代码字体、**思源宋体 / 微软雅黑**（系统字体）作中文字体；
> 换机器若缺字体会自动回退（见 `ccs.tex`）。
> ⚠ `TEXMF_OUTPUT_DIRECTORY` 必须是**项目根的绝对路径**，否则代码块全部高亮失败。
> ⚠ `build.bat` 是纯 ASCII 文件，**请勿在其中加入中文注释**（cmd 按 GBK 代码页解析会乱码）。
> ⚠ 在脚本里把 `/c/Windows/System32` 前置到 PATH 之后，`bash` 会解析成
> `C:\Windows\System32\bash.exe`（WSL 启动器），它看不懂 `/d/...` 路径。所以
> `work/tools/full_build.sh` 是自包含的，不再嵌套调用 `bash xxx.sh`。

## 上游流水线（在项目根的 `../work/`）

本书的翻译不是手写的，而是「EPUB → 结构化 → 分单元翻译 → 回填 LaTeX」的流水线产物：

| 步骤 | 脚本 | 作用 |
|---|---|---|
| 1 | `work/tools/setup.py` | 解包 EPUB：XHTML → `work/epub_full/`，图片 → `latex_source/images/` |
| 2 | `work/tools/extract.py` | XHTML → `work/tmpl/*.tex`（带 `@@id@@` 占位的 LaTeX 骨架）+ `work/units/*.json`（待翻译单元）+ `work/index_entries.json`（索引层级 + 页码锚点）+ `_manifest.json` |
| 3 | `work/tools/make_batches.py` | 按字符预算切批 → `work/batch/tNN.json` |
| 4 | （并行子代理翻译） | 按 `work/TRANSLATION_GUIDE.md` 翻译 → `work/out/tNN.json` |
| 5 | `work/tools/build_latex.py` | `tmpl + out` → `book/content/**/*.tex`（并做锚点去重、图片名归一化、裸 URL 包扎） |
| 6 | `work/tools/gen_index.py` | 生成 `book/index.tex` 与 `book/content/front/index.tex` |

**翻译规范**：`work/TRANSLATION_GUIDE.md`（输入输出 JSON 格式、HTML 保留规则、
语言风格、架构/C++ 术语对照表、交叉引用与图注格式）。

### 规模

- **7499** 个翻译单元，**59** 个批次，全部 id 一一对应、零漏译。
- 正文 19 章 + 4 个部分 + 148 张插图 + 465 个代码块（C++/CMake/YAML/Dockerfile/Python/JSON）+ 表格若干。
- 书末索引：**784 个主词条 + 573 个子词条**，1555 个页码引用，按拼音排序、按首字母分节。

## 索引是怎么做的（`book/content/front/index.tex`）

1. `extract.py` 解析 EPUB 的 `Index.xhtml`：`p.index-level-1/2` → `work/index_entries.json`
   （词条层级 + 它引用的 `_idIndexMarkerNNN` 锚点），词条文字进翻译单元 `work/units/theindex.json`。
2. 正文里每个被索引的段落/标题/列表项前都输出 `\label{idx:NNN}`（全书 2090 个，
   模板里 3 处重号已在 `build_latex.py` 里去重）。
3. `gen_index.py --phase final`：
   - 从 `main.aux` 解出每个锚点的真实页码（中文版分页，不是原书页码）；
   - 页码去重、升序、≥3 个连号压成 `188--190`；
   - 词条按 **拼音** 排序：借 Windows 的 `Chinese_China.936` 排序规则
     （`locale.strxfrm`），并用一组「各声母首字」标尺（阿/八/擦/搭…）把汉字定位到首字母区间；
     拉丁字母开头的词条自然落到各自字母下（ASCII 在 936 规则里排在汉字之前）；
     数字/符号开头的落到「符号与数字」节。
4. 排版命令在 `ccs.tex`：`\idxhead{字母}` / `\idxmain{词条}{页码}` / `\idxsub{词条}{页码}`，
   两栏（`multicol`），子词条缩进 1.6em 并悬挂对齐。

## 自定义命令与环境（`book/ccs.tex`）

| 命令 | 用途 |
|---|---|
| `\myChapter{章号}{标题}{文件}` | 编号正文章（章首有巨型浅灰章号 + 钢蓝横线） |
| `\myPart{部号}{标题}{文件}` | 部分扉页（「第 N 部分」+ 钢蓝斜体部名，同页接部分引言） |
| `\myFront{标题}{文件}` | 前言类（进目录、无编号） |
| `\myFrontNoToc{文件}` | 扉页/版权页（不进目录） |
| `\myGraphic{宽度比例}{图片}{图注}` | 插图（紧随正文、不浮动） |
| `\begin{codebox}[语言]` | 代码块（minted 高亮，`语言` 缺省 `cpp`） |
| `\begin{myQuote}` | 引用块（左侧钢蓝细竖线 + 缩进） |
| `\begin{myNotic}{注意}` / `\begin{myTip}{提示}` | 原书 NOTE / TIP 提示段 |
| `\idxhead` / `\idxmain` / `\idxsub` | 书末索引的三类条目 |
| `\texthref{url}` | 译文里的「裸 URL」：链接目标 = 显示文字，并自动获得断行点 |
| `\myChapterIntro{标题}` | 章首「本章涵盖」米色盒（本版未使用） |
