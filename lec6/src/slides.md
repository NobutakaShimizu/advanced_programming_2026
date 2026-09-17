---
theme: neversink
layout: cover
title: プログラミング応用第6回
githubPages:
  ogp: true
author: 清水 伸高
mdc: true
css: unocss
style: |
  @import 'styles/custom.css';

fonts:
  sans: 'Roboto'
  mono: 'Fira Code'
  weights: '400,500,700'
  italic: true
favicon: 'https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6/svgs/solid/book.svg'
themeConfig:
  primary: '#1976d2'
lineNumbers: true
transition: none
---

# プログラミング応用 第6回: <br> 線形計画問題と組合せ最適化

[清水 伸高](https://sites.google.com/view/nobutaka-shimizu/home) (塩浦研 助教)

<div style="position: absolute; bottom: 20px; font-size: 0.8em; width: 100%; text-align: center;">
2025年 11月11日
</div>


---
layout: top-title
color: amber-light
---

::title::
# 今日の内容

::content::

- [線形計画問題](/3)
- [LPの双対](/14)
  - [弱双対定理と強双対定理](/22)
- [組合せ最適化への応用](/30)
  - [最短経路問題](/37)
  
<div class="topic-box">

今回の目標: **線形計画問題**の視点から組合せ最適化アルゴリズムを理解する.

</div>
 
---
layout: section
color: amber-light
---

# 線形計画問題

---
layout: top-title
color: amber-light
---

::title::
# 生産計画問題

::content::

<div class="question">

あなたはレストランを経営している.
必要なメニューを全て提供するための**仕入れコストを最小化**せよ.

</div>

<v-clicks>

- 1日あたり **カレー** `30` kg, **パスタ** `20` kg, **サラダ** `15` kg を用意しなければならず, 材料は以下の通りである:
  - カレー1 kgを作るには 野菜 `0.3` kgと肉 `0.4` kgを使う
  - パスタ1 kgを作るには肉 `0.6` kgとチーズ `0.2` kgを使う
  - サラダ1 kgを作るには野菜 `0.8` kgとチーズ `0.1` kgを使う

- それぞれの材料の1 kgあたりの価格は次の通りである：
  - 野菜 `400`円/kg, 肉 `900`円/kg, チーズ `600`円/kg

</v-clicks>


---
layout: top-title
color: amber-light
---

::title::
# LPの例

::content::

野菜, 肉, チーズの仕入れ量をそれぞれ **$x_{\mathrm{v}}, x_{\mathrm{m}}, x_{\mathrm{c}}$** とすると, LPとして記述できる:

$$
\boxed{
\begin{align*}
\text{min} \quad & 400 x_{\mathrm{v}} + 900 x_{\mathrm{m}} + 600 x_{\mathrm{c}} \\
\text{s.t.} \quad & 0.3 x_{\mathrm{v}} + 0.4 x_{\mathrm{m}}  \ge 30 \\
                  & 0.6 x_{\mathrm{m}} + 0.2 x_{\mathrm{c}} \ge 20 \\
                  & 0.8 x_{\mathrm{v}} + 0.1 x_{\mathrm{c}} \ge 15 \\
                  & x_{\mathrm{v}}, x_{\mathrm{m}}, x_{\mathrm{c}} \geq 0
\end{align*}
}
\longrightarrow 
\boxed{
\begin{align*}
\text{min} \quad & \begin{bmatrix} 400 & 900 & 600 \end{bmatrix} \begin{bmatrix} x_{\mathrm{v}} \\ x_{\mathrm{m}} \\ x_{\mathrm{c}} \end{bmatrix} \\
\text{s.t.} \quad & \begin{bmatrix} 0.3 & 0.4 & 0  \\ 0 & 0.6 & 0.2 \\ 0.8 & 0 & 0.1 \\ 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x_{\mathrm{v}} \\ x_{\mathrm{m}} \\ x_{\mathrm{c}} \\ x_{\mathrm{v}} \\ x_{\mathrm{m}} \\ x_{\mathrm{c}} \end{bmatrix} \ge \begin{bmatrix} 30 \\ 20 \\ 15 \\ 0 \\ 0 \\ 0 \end{bmatrix}
\end{align*}
}
$$

<v-click>

- 目的関数と制約が**線形関数**を用いて記述

<div class="topic-box">

線形関数 = 変数の1次式 (定数項を含む場合もある)

</div>

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 線形計画問題(Linear Programming; LP)

::content::

<div class="definition">

**線形計画問題(LP)** とは、線形の目的関数を線形の等式・不等式制約のもとで最大化または最小化する連続最適化問題である.
一般に、次の形で表される:

$$
\begin{align*}
\text{min} \quad & c^\top x \\
\text{s.t.} \quad & Ax \ge b
\end{align*}
$$

ここで, 問題の入力は$c\in\Real^n,A\in\Real^{m\times n},b\in\Real^m$ (定数と呼ぶ) であり, $x\in\Real^n$ は変数である.
ベクトル間の不等号“$\ge$"は全ての成分について大小関係$\ge$が成り立つことを意味する.

</div>

- 実用上の多くの最適化問題を記述できる: 生産計画, 資源配分, 物流の最適化, 構造設計

<v-click>

- 最小化問題だが, 目的関数を$-1$倍すれば**最大化問題**も記述できる
- 制約の不等式の向き $\ge$ も両辺を$-1$倍すれば $\le$ にできる

</v-click>


---
layout: top-title
color: amber-light
---

::title::
# LPの構造

::content::

$$
\boxed{
\begin{align*}
\text{min} \quad & 400 x_{\mathrm{v}} + 900 x_{\mathrm{m}} + 600 x_{\mathrm{c}} \\
\text{s.t.} \quad & 0.3 x_{\mathrm{v}} + 0.4 x_{\mathrm{m}}  \ge 30 \\
                  & 0.6 x_{\mathrm{m}} + 0.2 x_{\mathrm{c}} \ge 20 \\
                  & 0.8 x_{\mathrm{v}} + 0.1 x_{\mathrm{c}} \ge 15 \\
                  & x_{\mathrm{v}}, x_{\mathrm{m}}, x_{\mathrm{c}} \geq 0
\end{align*}
}
$$

- 各制約は不等式 $a^\top x \ge b$ の形になっている (**線形制約**と呼ぶ)
- 実行可能解の集合は, $0$個の線形不等式全てを満たす点の集合となる.

<div class="definition" v-click>

- $ax=b$で表される点の集合を**超平面**という.
- 0個以上の線形制約を満たす点の集合を**多面体**という.

</div>

---
layout: top-title
color: amber-light
---

::title::
# $2x+y+z \leq 1$, $x+2y+z \leq 1$, $x+y+2z \leq 1$, $x, y, z \geq 0$

::content::

<Polyhedron3D2 />

<div style="font-size: 0.8em; margin-top: 10px; text-align: center;">

**ドラッグ(左クリック)** で回転, **ドラッグ(右クリック)** で並行移動, **マウスホイール**でズーム. 表示がおかしかったらリロード.

</div>



---
layout: top-title
color: amber-light
---

::title::
# 制約: $x+y \leq 1$, $x, y, z \geq 0$ (z方向に非有界)

::content::

<Polyhedron3DUnbounded />

<div style="font-size: 0.8em; margin-top: 10px; text-align: center;">

**ドラッグ(左クリック)** で回転, **ドラッグ(右クリック)** で並行移動, **マウスホイール**でズーム. 表示がおかしかったらリロード.

</div>

---
layout: top-title
color: amber-light
---

::title::
# LPは解けるか?

::content::

<div class="topic-box">

LPは三つの場合に分類される:

  1. 実行可能解が存在しない (**実行不可能**)
  2. 実行可能解は存在するが, 最適解が存在しない (**非有界**)
  3. 実行可能解と最適値が存在する (**最適解存在**)


</div>

- 実行不可能の例
  - 制約 $x+y \leq 1$, $x+y \geq 3$ の場合
- 実行可能解は存在するが最適解が存在しない例 (前ページ)
  - max $z$ s.t. $x+y \leq 1$, $x,y,z \geq 0$
  - $z$の値はいくらでも大きくできるので最適解は存在しない

---
layout: top-title
color: amber-light
---

::title::
# LPを「解く」

::content::

- コンピュータでLPを場合は入力の行列やベクトルは全て**有理数**となる
  - 入力長 $\approx$ 全ての有理数の桁数の総和

<div class="proposition">

最適解が存在するならば, 入力長($A$などの桁数も考慮)に関して**多項式長の桁数を持つ最適解**が存在.

</div>

<div class="toggle-box" markdown="1">
<details>
  <summary>証明のイメージ <span style="font-size:0.9em;">(クリックで展開)</span></summary>
  
  <div style="padding-left: 2em;" markdown="1">
  
  - 最適解を達成する多面体の「端点」が存在する
  - この「端点」は連立方程式 $A'x=b'$ の解になっている ($A'$は正則行列)
  - 逆行列の公式 (クラメールの公式) を使って, 各成分の桁数を評価する
  
  </div>
</details>
</div>

- LPを解く = 最適値とそれを達成する最適解の一つを出力する

---
layout: top-title
color: amber-light
---

::title::
# 代表的なアルゴリズム

::content::


| アルゴリズム       | 計算量         | 特徴                      | 備考          |
|----------------------|---------------|---------------------------|-----------------------|
| 単体法     | 最悪指数時間   | 多面体の端点を探索       | 実用上は非常に高速  |
| 楕円体法     | 多項式時間 | 最適解を含む楕円を縮小           | 初の**多項式時間アルゴリズム**だが非実用的      |
| 内点法               | 多項式時間     | 多面体の内部を探索           | 実用性と理論保証を両立        |
| 主双対内点法 |  多項式時間   |   主・双対の内部を同時に探索     | 多くの商用ソルバーのベース    |

<v-clicks>

- 単体法は**Dantzig (1947)** によって提案された
- **Khachiyan (1979)** が世界初の多項式時間アルゴリズムとして楕円体法を発表
- **Karmarkar (1984)** が実用的かつ多項式時間アルゴリズムとして内点法を発表
- 主双対内点法については吉瀬先生(筑波大学)の[講演資料](https://www.kurims.kyoto-u.ac.jp/coss/coss2024/slides/yoshise-lecture1.pdf)を参照

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# これからの内容

::content::

<div class="topic-box">

- とりあえずLPは多項式時間で解けることを前提
- LPは連続的だが, 最適化問題にも応用できることを解説
- 次やること: 重要な役割を果たす**双対**の概念を導入する
  - 最適化問題の定式化や理論的な性質の理解に役立つ
</div>

---
layout: section
color: amber-light
---

# LPの双対

---
layout: top-title
color: amber-light
---

::title::
# 制約がなければ最適化は簡単

::content::

<div class="topic-box">

制約がなければ, 最適化問題 

$$
\mathrm{min}\quad f(x)
$$

は ($f$が簡単な形なら) 解きやすい.

</div>

<v-clicks>

- 例:　$f(x) = (x-1)^2 + (y-2)^3$
  - 制約がなければ, 最適解は $x=1,y=2$ であることがすぐにわかる
- [勾配法](https://advanced-programming-2025.pages.dev/lec2/31): $f$が凸かつ滑らか

<div class="topic-box">

制約があるLPを制約なし最適化問題で近似できれば良いのでは?

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最適化問題の制約を「外す」

::content::

<div class="topic-box">
次の形のLPを考える (簡単のため最適解が存在すると仮定):
$$
\begin{align*}
\text{min} \quad & c^\top x \\
\text{s.t.} \quad & Ax \ge b
\end{align*}
$$

</div>

<v-clicks>

- 次の関数 $\delta$ を考える:
  
  $$
    \begin{align*}
      \delta(x) = \begin{cases}
        0 & \text{if } Ax \ge b \\
        +\infty & \text{otherwise}
      \end{cases}
    \end{align*}
  $$
  
- このとき, 上のLPは制約なし最適化問題 $\text{min} \quad c^\top x + \delta(x)$ と等価.

<div class="topic-box">

イメージ: 制約条件を違反する$x$に対して**ペナルティ** $\delta(x)$ を課すことで制約条件を「外す」

</div>

</v-clicks>
  
---
layout: top-title
color: amber-light
---

::title::
# 最適化問題の制約を「外す」

::content::

- しかし, $\delta(x)$ は扱いづらい (微分不可能)

<v-clicks>

- ペナルティを線形関数にするため, **固定した $y\in \Real^m$** に対して
  $$
    \begin{align*}
      \textcolor{c2185b}{\delta_y(x) = y^\top (b - Ax) = \sum_{i=1}^m y_i (b_i - (Ax)_i)}
    \end{align*}
  $$
  を考える (ただし $y\ge 0$).

- $x$が元のLPの制約 $Ax\ge b$ を**違反し**, $(Ax)_i< b_i$ となるならば, ペナルティとして
  $$
    \begin{align*}
      (b_i - (Ax)_i)\cdot y_i
    \end{align*}
  $$
  が課される.

<div class="topic-box">

- 元のペナルティ $\delta(x)$ を, 扱いやすいペナルティ $\delta_y(x)$ に置き換えた.

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最適化問題の制約を「外す」

::content::


- これを用いた制約なし最適化問題
  $$
    \begin{align*}
      \text{min} \quad & c^\top x + y^\top (b - Ax)
    \end{align*}
  $$
  を考える.

<div class="remark">

ベクトル$y\ge 0$は固定し, $x$について最適化を行うことに注意.

</div>

<v-click>

- 元のLPの任意の実行可能解 $x$ (つまり$Ax\ge b$を満たす) に対して
$$
  \begin{align*}
    y^\top (b-Ax) \le 0
  \end{align*}
$$
-  つまり, $x$が元のLPの制約を満たすならば目的関数値は下がるが, 違反するときはペナルティとなる.

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# ここまでのまとめ

::content::

<div class="topic-box">

- 元のLPと線形ペナルティを課した最適化問題:


$$ \boxed{
  \begin{align*}
    \text{min} \quad & c^\top x \\
    \text{s.t.} \quad & Ax \ge b 
  \end{align*}}
  \longrightarrow
  \boxed{\begin{align*}
    \text{min} \quad c^\top x + y^\top (b-Ax)
  \end{align*}}
$$

- 固定した各 $y\ge 0$ に対して 左の最適値 $\ge$ 右の最適値

</div>

<v-clicks>

- **制約をペナルティに置き換えて**元の最適値の下界を得た
- 必ずしも等号が成り立つとは限らない. あくまで「下からの近似」

<div class="remark">

解きたい最適化問題を解きやすい最適化問題で**近似する**というアイデアは単純だが非常に強力.
今回は「制約つきの最適化問題」をペナルティを用いて「制約なしの最適化問題」で近似している.

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最適値の下界を最大化

::content::

<div class="question">

最適値の下界を **$\calL^*(y):=\inf_x\{c^\top x + y^\top (b-Ax)\}$** とおく.
$y\ge 0$を色々動かして, 最適値 $\min_{x\colon Ax\ge b}\{c^\top x\}$ に最も近い $\calL^*(y)$ を求めよ.

</div>

<div class="remark">

**$\min$ と $\inf$ の違い**: 最適値の有界性が確定している場合は$\min$を使い, 最適値が非有界になりうる場合は$\inf$を使う.
今回は, 元のLPが最適解を持つことを仮定したので $\min_{x\colon Ax\ge b} \{c^\top x\}$ と書いてよい.

</div>

<v-clicks>


- $\calL^*(y)$について式変形すると

  $$
    \begin{align*}
      \calL^*(y) &= \inf_x \{c^\top x + y^\top (b-Ax)\} \\
                 &= \inf_x \{y^\top b + (c^\top - y^\top A)x\} \\
                 &= y^\top b + \inf_x \{(c^\top - y^\top A)x\}
    \end{align*}
  $$

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最適値の下界を最大化

::content::

- 先の式変形より, $\calL^*(y) = y^\top b + \inf_x \{(c^\top - y^\top A)x\}$
  - $c^\top = y^\top A$ の場合, $\calL^*(y)=y^\top b$
  - $c^\top \ne y^\top A$ の場合, $x$を動かして$(c^\top - y^\top A)x$ はいくらでも小さくできるので, $\calL^*(y)=-\infty$.

<div class="topic-box" v-click>

- すなわち, 最もよい下界 $\max_{y\ge 0}\calL^*(y)$ は

  $$
    \begin{align*}
      \text{max} \quad & y^\top b \\
      \text{s.t.} \quad & y^\top A = c^\top \\
                        & y \ge 0
    \end{align*}
  $$

  を解くことで与えられる.
- このLPを **双対問題** といい, 元のLPを**主問題**という.
- 双対問題の変数や制約をそれぞれ**双対変数**, **双対制約**と呼ぶ

</div>  

---
layout: top-title
color: amber-light
---

::title::
# 主問題と双対問題

::content::


$$
  \boxed{
  \begin{align*}
    \text{min} \quad & c^\top x \\
    \text{s.t.} \quad & Ax \ge b
  \end{align*}
  }

  \longleftrightarrow

  \boxed{
  \begin{align*}
    \text{max} \quad & y^\top b \\
    \text{s.t.} \quad & y^\top A = c^\top \\
                      & y \ge 0
  \end{align*}
  }
$$

<v-clicks>

<div class="theorem">

- 主問題の最適値 **$\ge$** 双対問題の最適値 (**弱双対定理**)
- 主問題が最適解を持つならば, 双対問題も最適解を持つ, **最適値は同じ** (**強双対定理**)
</div>

- 弱双対定理はこれまでの議論から明らか ($\because$ $\calL^*(y)$ は主問題の最適値の下界)
  
</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 非負制約つきの場合

::content::

- 主問題が非負制約を持つ場合を考える:
$$
  \begin{align*}
    \text{min} \quad & c^\top x \\
    \text{s.t.} \quad & Ax \ge b \\
                      & x \ge 0
  \end{align*}
$$

- 同様にして $\calL^*(y) = \inf_{\textcolor{c2185b}{x\ge 0}}\{c^\top x + y^\top (b-Ax)\}$ を考えると
$$
  \begin{align*}
    \text{max} \quad & y^\top b \\
    \text{s.t.} \quad & y^\top A \le c^\top \\
                      & y \ge 0
  \end{align*}
$$

<div class="remark" v-click>

LPの文脈では, 非負制約 $x\ge 0$ は扱いやすいので, 線形制約 $Ax\ge b$ には組み込まず, 別途扱うことが多いので,
講義によっては上の形の主問題を扱うことがある.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 双対の取り方

::content::

- レストランの例で扱ったLPの双対を考えよう:

$$
\boxed{
\begin{align*}
\text{min} \quad & 400 x_{\mathrm{v}} + 900 x_{\mathrm{m}} + 600 x_{\mathrm{c}} \\
\text{s.t.} \quad & 0.3 x_{\mathrm{v}} + 0.4 x_{\mathrm{m}}  \ge 30 \\
                  & 0.6 x_{\mathrm{m}} + 0.2 x_{\mathrm{c}} \ge 20 \\
                  & 0.8 x_{\mathrm{v}} + 0.1 x_{\mathrm{c}} \ge 15 \\
                  & x_{\mathrm{v}}, x_{\mathrm{m}}, x_{\mathrm{c}} \geq 0
\end{align*}
}
$$

- それぞれの制約はカレー, パスタ, サラダの提供量に対応している.
  - これらに対応した双対変数 **$y_{\mathrm{curry}}, y_{\mathrm{pasta}}, y_{\mathrm{salad}}\ge 0$** を導入する.

<div class="topic-box" v-click>

双対変数は, 主問題の**各制約**に対して定義する.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 双対の取り方

::content::

<div class="topic-box">

主問題の各制約の両編を, 対応する双対変数で掛け算し, 足し合わせる:

</div>

$$
\boxed{
\begin{align*}
\text{min} \quad & 400 x_{\mathrm{v}} + 900 x_{\mathrm{m}} + 600 x_{\mathrm{c}} \\
\text{s.t.} \quad & \textcolor{c2185b}{y_{\mathrm{curry}}} (0.3 x_{\mathrm{v}} + 0.4 x_{\mathrm{m}})  \ge 30 \textcolor{c2185b}{y_{\mathrm{curry}}} \\
                  & \textcolor{c2185b}{y_{\mathrm{pasta}}} (0.6 x_{\mathrm{m}} + 0.2 x_{\mathrm{c}}) \ge 20 \textcolor{c2185b}{y_{\mathrm{pasta}}}\\
                  & \textcolor{c2185b}{y_{\mathrm{salad}}} (0.8 x_{\mathrm{v}} + 0.1 x_{\mathrm{c}}) \ge 15 \textcolor{c2185b}{y_{\mathrm{salad}}}\\
                  & x_{\mathrm{v}}, x_{\mathrm{m}}, x_{\mathrm{c}} \geq 0
\end{align*}
}
$$

<v-click>

- **$y\ge 0$** なので不等号の向きは変わらない. 足し合わせたものを整理すると
$$
\begin{align*}
& (0.3 y_{\mathrm{curry}} + 0.8y_{\mathrm{salad}}) x_{\mathrm{v}} + (0.4 y_{\mathrm{curry}} + 0.6 y_{\mathrm{pasta}}) x_{\mathrm{m}} + (0.2 y_{\mathrm{pasta}} + 0.1 y_{\mathrm{salad}}) x_{\mathrm{c}} \\
& \quad \ge 30 y_{\mathrm{curry}} + 20 y_{\mathrm{pasta}} + 15 y_{\mathrm{salad}}
\end{align*}
$$

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 双対の取り方

::content::

<div class="topic-box">

制約の右辺を全て足し合わせて整理したものを, 目的関数の下界とする (最大化問題の場合は上界)

</div>

$$
\begin{align*}
&400 x_{\mathrm{v}} + 900 x_{\mathrm{m}} + 600 x_{\mathrm{c}} \\
&\ge (0.3 y_{\mathrm{curry}} + 0.8y_{\mathrm{salad}}) x_{\mathrm{v}} + (0.4 y_{\mathrm{curry}} + 0.6 y_{\mathrm{pasta}}) x_{\mathrm{m}} + (0.2 y_{\mathrm{pasta}} + 0.1 y_{\mathrm{salad}}) x_{\mathrm{c}} \\
&\ge 30 y_{\mathrm{curry}} + 20 y_{\mathrm{pasta}} + 15 y_{\mathrm{salad}}
\end{align*}
$$

<v-clicks>

- 最後の式 $30 y_{\mathrm{curry}} + 20 y_{\mathrm{pasta}} + 15 y_{\mathrm{salad}}$ が双対の目的関数 (最大化したい)
- 変数$x$の各係数を比較して$\ge$で結んだもの, および$y\ge 0$が双対制約
  
  $$
    \begin{align*}
      \begin{cases}
      400 \ge 0.3 y_{\mathrm{curry}} + 0.8y_{\mathrm{salad}}, \\
      900 \ge 0.4 y_{\mathrm{curry}} + 0.6 y_{\mathrm{pasta}}, \\
      600 \ge 0.2 y_{\mathrm{pasta}} + 0.1 y_{\mathrm{salad}}, \\
      y_{\mathrm{curry}}, y_{\mathrm{pasta}}, y_{\mathrm{salad}} \ge 0.
      \end{cases}
    \end{align*}
  $$

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 双対の取り方

::content::

<div class="topic-box">

制約の右辺を全て足し合わせて整理したものを, 目的関数の下界とする (最大化問題の場合は上界)

</div>

$$
\begin{align*}
&\textcolor{red}{400} x_{\mathrm{v}} + 900 x_{\mathrm{m}} + 600 x_{\mathrm{c}} \\
&\ge \textcolor{red}{(0.3 y_{\mathrm{curry}} + 0.8y_{\mathrm{salad}})} x_{\mathrm{v}} + (0.4 y_{\mathrm{curry}} + 0.6 y_{\mathrm{pasta}}) x_{\mathrm{m}} + (0.2 y_{\mathrm{pasta}} + 0.1 y_{\mathrm{salad}}) x_{\mathrm{c}} \\
&\ge 30 y_{\mathrm{curry}} + 20 y_{\mathrm{pasta}} + 15 y_{\mathrm{salad}}
\end{align*}
$$

- 最後の式 $30 y_{\mathrm{curry}} + 20 y_{\mathrm{pasta}} + 15 y_{\mathrm{salad}}$ が双対の目的関数 (最大化したい)
- 変数$x$の各係数を比較して$\ge$で結んだもの, および$y\ge 0$が双対制約
 
  $$
    \begin{align*}
      \begin{cases}
      \textcolor{red}{400 \ge 0.3 y_{\mathrm{curry}} + 0.8y_{\mathrm{salad}}}, \\
      900 \ge 0.4 y_{\mathrm{curry}} + 0.6 y_{\mathrm{pasta}}, \\
      600 \ge 0.2 y_{\mathrm{pasta}} + 0.1 y_{\mathrm{salad}}, \\
      y_{\mathrm{curry}}, y_{\mathrm{pasta}}, y_{\mathrm{salad}} \ge 0.
      \end{cases}
    \end{align*}
  $$
  


---
layout: top-title
color: amber-light
---

::title::
# 双対の取り方

::content::

- まとめると, 双対問題は次のようになる:

$$
\boxed{
\begin{align*}
\text{max} \quad & 30 y_{\mathrm{curry}} + 20 y_{\mathrm{pasta}} + 15 y_{\mathrm{salad}} \\
\text{s.t.} \quad & 0.3 y_{\mathrm{curry}} + 0.8y_{\mathrm{salad}} \le 400 \\
                  & 0.4 y_{\mathrm{curry}} + 0.6 y_{\mathrm{pasta}} \le 900 \\
                  & 0.2 y_{\mathrm{pasta}} + 0.1 y_{\mathrm{salad}} \le 600 \\
                  & y_{\mathrm{curry}}, y_{\mathrm{pasta}}, y_{\mathrm{salad}} \ge 0
\end{align*}}
$$

<v-click>

- 大体のLPはこのようにして双対を取ることができる
  - 主問題が最大化問題ならば, 双対は最小化問題になるので不等号の向きを適切に逆にする
  - (慣れた人は脳内で双対をとれるらしい...)

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# ラグランジュ双対 (おまけ)

::content::

- $\calL^*$を用いて導出した双対は**ラグランジュ双対** と呼ばれるものの一種

<v-clicks>

- LPをより一般化した次の形の最適化問題にも適用できる:
  $$
  \begin{align*}
    \text{min} \quad & f(x) \\
    \text{s.t.} \quad & g_i(x) \le 0 \quad (i=1,2,\ldots,m)
  \end{align*}
  $$
- それぞれの制約に対応した双対変数 $y_i \ge 0$ を導入し, **ラグランジュ関数** $\calL$ を定義する:
  $$
  \begin{align*}
    \calL(x,y) = f(x) + \underbrace{\sum_{i=1}^m y_i g_i(x)}_{\text{ペナルティ項}}
  \end{align*}
  $$
- $x$が制約を満たすとき, $f(x) \ge \calL(x,y)$ が成り立つ.
- $\calL^*(y) = \inf_x\{\calL(x,y)\}$ として, **$\max_{y\ge 0} \calL^*(y)$** を解く. この最大化問題がラグランジュ双対.

</v-clicks>

---
layout: section
color: amber-light
---

# 組合せ最適化問題への応用


---
layout: top-title
color: amber-light
---

::title::
# 整数計画問題とLP緩和

::content::

- 様々な組合せ最適化問題は**整数計画問題 (Integer Programming; IP)** として表現できる:
  
$$
\boxed{
\begin{align*}
\text{min} \quad & c^\top x \\
\text{s.t.} \quad & Ax \ge b \\
                  & \textcolor{c2185b}{x \in \mathbb{Z}^n}
\end{align*}}
$$

<v-click>

- 整数制約 $x\in \Int^n$ を除去したものを**線形計画問題の緩和 (LP緩和)** という:

$$
\boxed{
\begin{align*}
\text{min} \quad & c^\top x \\
\text{s.t.} \quad & Ax \ge b
\end{align*}}
$$

</v-click>

<v-click>

- 緩和することによって, 実行可能解の集合が広がる
  - したがって, LP緩和の最適値 $\le$ 元のIPの最適値
  - 最大化問題の場合は不等号が逆
  
</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 例: ナップザック問題

::content::

<div class="question">

- 重さ $w_i$ と価値 $v_i$ を持つ品物 $i=1,2,\ldots,n$ がある.
- 重さの上限 $W$ を超えないように品物を選び, 価値の総和を最大化せよ.

</div>

<v-click>

- $x_i$ = 「品物$i$を選ぶ個数」とすると, ナップザック問題は次のIPで表現できる:
$$
\boxed{
\begin{align*}
\text{max} \quad & \sum_{i=1}^n v_i x_i \\
\text{s.t.} \quad & \sum_{i=1}^n w_i x_i \le W \\
                  & x_i \ge 0  && (i=1,2,\ldots,n) \\
                  & x_i \in \Int && (i=1,2,\ldots,n)
\end{align*}}
$$

</v-click>


---
layout: top-title
color: amber-light
---

::title::
# 例: ナップザック問題

::content::

<div class="question">

- 重さ $w_i$ と価値 $v_i$ を持つ品物 $i=1,2,\ldots,n$ がある.
- 重さの上限 $W$ を超えないように品物を選び, 価値の総和を最大化せよ.

</div>

- LP緩和すると:
$$
\boxed{
\begin{align*}
\text{max} \quad & \sum_{i=1}^n v_i x_i \\
\text{s.t.} \quad & \sum_{i=1}^n w_i x_i \le W \\
                  & \textcolor{c2185b}{x_i \ge 0}  && (i=1,2,\ldots,n)
\end{align*}}
$$

- 直感: アイテムを「連続的に」選べるようになる

---
layout: top-title
color: amber-light
---

::title::
# 例: 01ナップザック問題

::content::

<div class="question">

- 重さ $w_i$ と価値 $v_i$ を持つ品物 $i=1,2,\ldots,n$ がある.
- 重さの上限 $W$ を超えないように品物を選び, 価値の総和を最大化せよ
- ただし, 同じ品物は高々1つしか選べない.

</div>

<v-click>

- ナップザック問題は次のIPで表現できる:
$$
\boxed{
\begin{align*}
\text{max} \quad & \sum_{i=1}^n v_i x_i \\
\text{s.t.} \quad & \sum_{i=1}^n w_i x_i \le W \\
                  & x_i \in \binset && (i=1,2,\ldots,n)
\end{align*}}
$$

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 例: 01ナップザック問題

::content::

<div class="question">

- 重さ $w_i$ と価値 $v_i$ を持つ品物 $i=1,2,\ldots,n$ がある.
- 重さの上限 $W$ を超えないように品物を選び, 価値の総和を最大化せよ.
- ただし, 同じ品物は高々1つしか選べない.

</div>

- LP緩和すると:
$$
\boxed{
\begin{align*}
\text{max} \quad & \sum_{i=1}^n v_i x_i \\
\text{s.t.} \quad & \sum_{i=1}^n w_i x_i \le W \\
                  & \textcolor{c2185b}{0 \le x_i \le 1}  && (i=1,2,\ldots,n)
\end{align*}}
$$


---
layout: top-title
color: amber-light
---

::title::
# IPと定式化

::content::


<div class="proposition">

整数計画問題(IP)はNP困難である.

</div>

- 実際, ナップザック問題をIPとして定式化できる (ナップザック問題はNP困難)
- 従って, IPが多項式時間で解ける$\Rightarrow$ナップザック問題が多項式時間で解ける$\Rightarrow$$\mathsf{P}=\mathsf{NP}$ (証明終)

<div class="remark">

- 解きたい最適化問題をIPとして表現することを **IPとして定式化する** という.
- 定式化してIPを解く**IPソルバー**(CPLEX, GUROBIなど)を用いて解くことができる
  - IPソルバーは理論保証はないが, 実用上は高速に解けることが多い

</div>

---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題の定式化

::content::

- 非負重みつき有向グラフ $G=(V,E,w)$ と二頂点$s,t\in V$が与えられたとき, $st$-[最短経路問題](https://advanced-programming-2025.pages.dev/lec4/20)を考える
- 各辺$e\in E$について, 変数 $x_e \in \{0,1\}$ を導入.
  - $x_e=1$ $\iff$ 辺$e$が最短経路に含まれる
- 頂点$v$に入る辺集合を **$\delta^{\mathrm{in}}(v)$**, 頂点$v$から出る辺集合を **$\delta^{\mathrm{out}}(v)$** とする

$$
\boxed{
\begin{align*}
\text{min} \quad & \sum_{e\in E} w(e) x_e \\
\text{s.t.} \quad & \sum_{e\in \delta^{\mathrm{in}}(v)} x_e - \sum_{e\in \delta^{\mathrm{out}}(v)} x_e =
  \begin{cases}
    -1 & v=s \\
    1 & v=t \\
    0 & \text{otherwise}
  \end{cases} && (\forall v\in V) \\
                  & x_e \in \{0,1\} && (\forall e\in E)
\end{align*}}
$$


---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題の定式化

::content::

$$
\boxed{
\begin{align*}
\text{min} \quad & \sum_{e\in E} w(e) x_e \\
\text{s.t.} \quad & \textcolor{c2185b}{\sum_{e\in \delta^{\mathrm{in}}(v)} x_e - \sum_{e\in \delta^{\mathrm{out}}(v)} x_e =
  \begin{cases}
    -1 & v=s \\
    1 & v=t \\
    0 & \text{otherwise}
  \end{cases}} && (\forall v\in V) \\
                  & x_e \in \{0,1\} && (\forall e\in E)
\end{align*}}
$$

<v-click>

<div class="topic-box">

始点$s$から終点$t$までバケツリレーしたとき, 受け取るバケツの個数 $-$ 渡すバケツの個数を考えている.　

</div>

<div style="text-align:center;">
  <img src="/images/bucket_relay.png" alt="最短経路" style="display:inline-block; width:23%;">
</div>
</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題のLP緩和

::content::

$$
\boxed{
\begin{align*}
\text{min} \quad & \sum_{e\in E} w(e) x_e \\
\text{s.t.} \quad & \sum_{e\in \delta^{\mathrm{in}}(v)} x_e - \sum_{e\in \delta^{\mathrm{out}}(v)} x_e =
  \begin{cases}
    -1 & v=s \\
    1 & v=t \\
    0 & \text{otherwise}
  \end{cases} && (\forall v\in V) \\
                  & \textcolor{c2185b}{0 \le x_e \le 1} && (\forall e\in E)
\end{align*}}
$$

<div class="topic-box" v-click>

始点$s$から終点$t$まで辺に沿って水を流しており, $x_e$は辺$e$を流れる水の量を表す.

</div>


<div style="text-align:center;" v-click>
  <img src="/images/shortest_path.svg" alt="最短経路" style="display:inline-block;">
<div style="text-align:center; font-size:0.9em; color:#555; margin-top:0.5em;">
  <em>LP緩和で水を流すときは枝分かれしたり合流してもよい </em>
</div>
</div>

---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題のLP緩和

::content::

- 一般に, IPのLP緩和は, 最適解がIPの最適解と**異なる**ことがある.
  - 整数性制約が失われるため, 実行可能解の集合が真に広がるから

- ところが, 最短経路は特別!

<div class="theorem">

最短経路問題のLP緩和は, 元のIPの定式化と同じ最適値を持つ (**整数性**).

</div>

<div class="topic-box">

例外はあるが, 解きやすさ $\approx$ LP緩和しても不変

</div>

- 証明の方針:
  - 弱双対定理より, (IPの最適値) $\ge$ (LP緩和の最適値) $\ge$ (LP緩和の双対問題の最適値)
  - (LP緩和の双対問題の最適値) $\ge$ (IPの最適値) を示せば定理が示せる


---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題の双対

::content::

$$
\boxed{
\begin{align*}
\text{min.} \quad & \sum_{e\in E} w(e) x_e \\
\text{s.t.} \quad & \sum_{e\in \delta^{\mathrm{in}}(v)} x_e - \sum_{e\in \delta^{\mathrm{out}}(v)} x_e =
  \begin{cases}
    -1 & v=s \\
    1 & v=t \\
    0 & \text{o.w.}
  \end{cases} & & (\forall v\in V)\\
                  & 0 \le x_e \le 1 & & (\forall e\in E)
\end{align*}}

$$

- 主問題の制約 $0\le x_e \le 1$ は **$x_e \ge 0$ に置き換えても最適解は変わらない**ので置き換える ($x_e$を$1$より大きくしても最小化問題なので得をしない)
<v-click>

- 頂点 $v\in V$ での制約に対応する双対変数を **$y_v$** とおき, 各制約にかける


$$
  \begin{align*}
     \textcolor{c2185b}{y_v} \cdot \rbra{\sum_{e\in \delta^{\mathrm{in}}(v)} x_e - \sum_{e\in \delta^{\mathrm{out}}(v)} x_e} =
  \begin{cases}
    -\textcolor{c2185b}{y_s} & v=s \\
    \textcolor{c2185b}{y_t} & v=t \\
    0 & \text{o.w.}
  \end{cases}
  \end{align*}
$$

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題の双対

::content::

$$
  \begin{align*}
     y_v \cdot \rbra{\sum_{e\in \delta^{\mathrm{in}}(v)} x_e - \sum_{e\in \delta^{\mathrm{out}}(v)} x_e} =
  \begin{cases}
    -y_s & v=s \\
    y_t & v=t \\
    0 & \text{o.w.}
  \end{cases}
  \end{align*}
$$

<hr>

- 右辺の和をとると, **$y_t - y_s$** になる -> 双対の目的関数 (max)
- 左辺の和は整理すると

  $$
    \begin{align*}
      \sum_{e=(u,v)\in E} (y_v - y_u)\cdot x_e \tag{1}
    \end{align*}
  $$

  になる (実際, 各$x_e$で偏微分してみればわかる)
  
<v-click>
  
- 主問題の目的関数 $\sum_{e\in E} w(e) \cdot x_e \ge (1)$ を**各$x_e$の係数毎に**成立させるのが双対制約:

  $$
    \begin{align*}
      \textcolor{c2185b}{w(e) \ge y_v - y_u } &&  (\forall e=(u,v)\in E)
    \end{align*}
  $$

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題の双対

::content::

- 従って最短経路問題のLP緩和の双対は以下のようになる:

$$
\boxed{
\begin{align*}
\text{max.} \quad & y_t - y_s \\
\text{s.t.} \quad & y_v - y_u \le w(e) &&  (\forall e=(u,v)\in E)
\end{align*}}
$$

<v-clicks>

- **$y_v := \mathrm{dist}(s,v)$** とする (ただし, $y_s = \mathrm{dist}(s,s)=0$)
  - これは双対問題の制約を満たす実行可能解となる
  - さらに, $y_t - y_s = \mathrm{dist}(s,t)$ は(IPの最適値)に等しい. よって

$$
  \text{(IPの最適値)} = y_t - y_s \le \text{(LP緩和の双対の最適値)} \le \text{(LP緩和の最適値)} \le \text{(IPの最適値)}
$$

<div class="topic-box">

結論: 最短経路問題はLP緩和をしても最適値が変わらない

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最短経路問題の双対

::content::

<div class="topic-box">

最適値(最短経路長)を求めるだけなら, 双対問題を解けばよい.

</div>


$$
\boxed{
\begin{align*}
\text{max.} \quad & y_t - y_s \\
\text{s.t.} \quad & y_v - y_u \le w(e) &&  (\forall e=(u,v)\in E)
\end{align*}}
$$

<v-click>

- 双対変数を更新していく自然なアルゴリズムが考えられる:

<div class="algorithm">

- $y_s = 0$, $y_v = \infty$ ($v\ne s$) で初期化する
- 任意の辺$e=(u,v)$について, $y_v > y_u + w(e)$ ならば $y_v := y_u + w(e)$ と更新する
- 更新が終了したときの $(y_v)_{v\in V}$ を出力する

</div>

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# アルゴリズムの動作例

::content::

<DualVariableVisualizer />

<script setup>
import DualVariableVisualizer from './components/DualVariableVisualizer.vue'
</script>




---
layout: top-title
color: amber-light
---

::title::
# 単一始点最短経路問題を解くアルゴリズム

::content::

<div class="algorithm">

1. $y_s = 0$, $y_v = \infty$ ($v\ne s$) で初期化する
2. 任意の辺$e=(u,v)$について, $y_v > y_u + w(e)$ ならば $y_v := y_u + w(e)$ と更新する
3. 更新が終了したときの $(y_v)_{v\in V}$ を出力する

</div>

- ステップ2の更新回数は$O(\abs{V}\abs{E})$回で終了することが知られている (負閉路がない場合)
  - これは**Bellman--Ford法**と一致する ([単一始点最短経路問題を解くアルゴリズム](https://advanced-programming-2025.pages.dev/lec4/22?clicks=2))
- 辺重みが全て非負の場合, 辺$e$を「$s$から近い順に」選ぶと更新回数が $O(\abs{E})$ に減らせる
  - これは**Dijkstra法**と一致する

<div class="topic-box" v-click>

最適化問題を解くアルゴリズムは双対の視点から理解することもでき, 逆に, 双対の視点からアルゴリズムを設計することもできる.

</div>

---
layout: top-title
color: amber-light
---

::title::
# まとめ

::content::

- 線形計画法の定義
- 双対の取り方
- ラグランジュ双対 (おまけ)
- 組合せ最適化問題への応用
  - IP定式化
  - 最短経路問題の整数性
  - 双対に基づく最短経路アルゴリズム