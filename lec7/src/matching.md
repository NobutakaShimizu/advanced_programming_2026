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

# プログラミング応用 第7回: <br> マッチングの理論と応用

[清水 伸高](https://sites.google.com/view/nobutaka-shimizu/home) (塩浦研 助教)

<div style="position: absolute; bottom: 20px; font-size: 0.8em; width: 100%; text-align: center;">
2025年 11月18日
</div>

---
layout: top-title
color: amber-light
---

::title::
# 今日の内容

::content::

1. 二部グラフとマッチング
2. 増加路に基づくアルゴリズム
3. LPに基づくアルゴリズム(ハンガリアン法)
4. 先端的な話題


---
layout: top-title
color: amber-light
---

::title::
# マッチングとは?

::content::


- マッチングは, **誰と誰をどのように結びつけるか**という「社会設計の数学」
- 実際の制度・産業で稼働中または導入が検討中：
  - [NRMP制度](https://www.nrmp.org/): 医師と病院の配属 (アメリカ)
  - [Boston Mechanism → Gale–Shapley方式](https://people.duke.edu/~aa88/articles/ChangingBoston.pdf): 公立学校の生徒–学校割当 (アメリカ)
  - [公立高校入試の単願制見直し](https://www.mdc.e.u-tokyo.ac.jp/news/6531/) (日本)
- 目的：
  - 公平・安定・効率の3要素を同時に満たすアルゴリズム設計
- 応用範囲はさらに広く, ネットワーク設計, タスク割り当て, ロボット制御, 交通計画など

---
layout: top-title
color: amber-light
---

::title::
# 数理的奥深さ（グラフ理論の核心）

::content::

- **マッチング = グラフ上の独立な辺集合**
- 組合せ最適化の出発点：
  - 最大マッチング → 組合せ的最適化の典型例
  - Tutte行列 → 代数的アプローチ（行列式と完全マッチング）
  - LP定式化 → 幾何的アプローチ（多面体理論・整数性）
  - マトロイド交叉 → 抽象的構造の理解

<div class="topic-box">

理論的に洗練されており, 応用も広く, 現実社会の制度設計にも重要な影響を及ぼしている.

</div>

---
layout: section
color: amber-light
---

# 二部グラフとマッチング



---
layout: top-title
color: amber-light
---

::title::
# 二部グラフ

::content::

<div class="definition" style="display: flex; flex-direction: column;">

グラフ $G=(V,E)$ であって, 頂点集合 $V$ が2つの部分集合 $L$ と $R$ に分割され, すべての辺が $L$ の頂点と $R$ の頂点を結ぶものを **二部グラフ** という.
このような二部グラフを **$G=(L\sqcup R,E)$** と表す.

<div style="margin-top: auto; font-size: 0.7em; color: #666; font-style: italic;">

($L$と$R$はそれぞれ左と右に配置された頂点集合であることを明示するための記号)

</div>

</div>

<figure style="text-align: center; margin: 2em 0;">
  <img src="/images/bipartite_example.png" alt="二部グラフの例" style="max-width: 200px; display: block; margin: 0 auto;">
</figure>

<div class="remark">

集合$A,B$の和集合は一般に$A\cup B$と書くが,
**$A\cap B=\emptyset$** であることを強調する場合は **$A\sqcup B$** と書く.

</div>


---
layout: top-title
color: amber-light
---

::title::
# マッチング

::content::

<div class="definition">

二部グラフ $G=(L\sqcup R,E)$ の辺部分集合 $M\subseteq E$ は, 各辺が互いに共有点を持たないとき**マッチング**という.
すなわち, 任意の相異なる$e,e'\in M$に対して **$e\cap e'=\emptyset$** であるとき, $M$はマッチングである.
</div>

- [無向グラフの定義](https://advanced-programming-2025.pages.dev/lec4/5)では各辺 $e=\set{u,v}$ は二つの頂点からなる集合だったことに注意

<figure style="text-align: center; margin: 2em 0;">
  <img src="/images/matching_example.png" alt="マッチングの例" style="max-width: 400px; display: block; margin: 0 auto;">
</figure>

- 一般のグラフに対してマッチングは定義できるが, この講義では二部グラフのマッチングを扱う

---
layout: top-title
color: amber-light
---

::title::
# 最大マッチング問題

::content::

<div class="question">

与えられた二部グラフ $G=(L\sqcup R,E)$ に対して, **辺数最大のマッチングを求めよ**.

</div>

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; margin: 2em 0; align-items: start;">
  <!-- 左側: 操作説明 -->
  <div style="font-size: 0.9em; color: #666;">
    <h3 style="margin-top: 0; font-size: 1.1em; color: #333;">操作方法</h3>
    <ul style="line-height: 1.8;">
      <li>黒い辺をクリック: マッチングに追加（赤になる）</li>
      <li>赤い辺をクリック: マッチングから削除（黒に戻る）</li>
      <li>マッチングでなくなる場合は赤にならない</li>
    </ul>
  </div>

  <!-- 右側: ビジュアライザとボタン -->
  <BipartiteMatchingVisualizer />
</div>

<script setup>
import BipartiteMatchingVisualizer from './components/BipartiteMatchingVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# 完全マッチング問題

::content::

<div class="definition">

二部グラフ $G=(L\sqcup R,E)$ に対して, $L$ のすべての頂点と $R$ のすべての頂点がちょうど一つずつ辺で結ばれるようなマッチング $M\subseteq E$ を **完全マッチング** という ($|L|=|R|$ のときに存在しうる).

</div>

<div class="question">

- 与えられた二部グラフ $G=(L\sqcup R,E)$ に**完全マッチングが存在するかどうか**判定せよ (判定問題)
- 存在するならばその完全マッチングを一つ求めよ (探索問題)

</div>

- グラフ理論でも最も基本的な問題の一つ

<div class="topic-box">

この講義では, **代数的な視点に基づいた非常に簡潔な**乱択アルゴリズムを紹介する.

</div>

---
layout: top-title
color: amber-light
---

::title::
# Tutte行列に基づく乱択アルゴリズム

::content::

<div class="definition">

$\abs{L}=\abs{R}=n$を満たす二部グラフ $G=(L\sqcup R,E)$ に対して, $n\times n$の**行列 $T$** を以下で定義する:
- 各辺 $\set{u,v} \in E$ に対して**変数 $x_{u,v}$** を用意し,
$$
  T_{u,v} =
  \begin{cases}
    x_{\set{u,v}} & \text{if } \set{u,v} \in E \\
    0 & \text{otherwise}
  \end{cases}
$$

</div>

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; margin: 2em 0; align-items: center;">
  <!-- 左側: Tutte行列の例 -->
  <div>
  <h3 style="margin-top: 0; font-size: 1.1em; color: #333;">Tutte行列の非ゼロ成分は変数</h3>    
  
  $$  T = \begin{bmatrix}
    x_{\set{1,a}} & 0 \\
    x_{\set{2,a}} & x_{\set{2,b}}
  \end{bmatrix}  $$

  </div>

  <div style="text-align: center;">
    <img src="/images/tutte.svg" alt="Tutte行列に対応する二部グラフ" style="max-width: 100%; height: auto;">
  </div>
</div>


---
layout: top-title
color: amber-light
---

::title::
# Tutte行列と完全マッチング

::content::

<div class="topic-box">

Tutte行列の行列式 $\det(T)$ は変数 $(x_{u,v})_{\set{u,v}\in E}$ に関する **$n$次多項式** である.

</div>

  $$\det(T) = \sum_{\sigma\in S_n} \mathrm{sgn}(\sigma) \prod_{i=1}^n T_{i,\sigma(i)} $$
  - $S_n$ は$1$から$n$までの置換全体の集合
  - $\mathrm{sgn}(\sigma)\in\{-1,1\}$ は置換$\sigma$の符号 (紙に置換を図示したときの交差の個数の偶奇)

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; margin: 2em 0; align-items: center;">
  <!-- 左側: Tutte行列の例 -->
  <div>
  <h3 style="margin-top: 0; font-size: 1.1em; color: #333;" markdown="1">
  
  $\det(T)$の各項は完全マッチングに対応
  
  </h3>    
  
  $$  \det(T) = x_{\textcolor{red}{\set{1,a}}}x_{\textcolor{red}{\set{2,b}}} $$
    

  </div>

  <div style="text-align: center;">
    <img src="/images/tutte2.svg" alt="Tutte行列に対応する二部グラフ" style="max-width: 100%; height: auto;">
  </div>
</div>

---
layout: top-title
color: amber-light
---

::title::
# Tutte行列と完全マッチング

::content::

<TutteMatrixVisualizer />

<script setup>
import TutteMatrixVisualizer from './components/TutteMatrixVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# Tutte行列と完全マッチング

::content::

<div class="proposition">

$G=(L\sqcup R,E)$ が完全マッチングを持つ $\iff$ Tutte行列 $T$ の行列式 $\det(T)$ が**零多項式でない**

</div>

- つまり, $n$ 次多項式 $h((x_{u,v})_{\set{u,v}\in E}) := \det(T)$ が恒等的にゼロかどうかを判定すればよい
  - 展開すると $n!$ 個の項が出てしまうので, 直接計算は非効率
  - **展開せずに**効率的にゼロかどうか判定できないか?
- この状況, どこかで見たような??? -> [多項式同一性判定問題](https://advanced-programming-2025.pages.dev/lec3/21?clicks=1) (第3回)

<div class="question" v-click>

$m$ 変数, 次数$n$ の多項式 $h(x_1,\dots,x_n)$ が **$h\equiv 0$** かどうか判定せよ.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 多項式同一性判定問題の乱択アルゴリズム (復習)

::content::

<div class="algorithm">

  アルゴリズム $A$
  1. ランダムな点 $r\in\Real^m$ を, 各成分を独立に $\{1,\dots,2n\}$ から一様ランダムに選ぶ
  2. もし$h(r)=0$ ならば, Yesを出力する. そうでなければNoを出力する

</div>

<div class="theorem">

上記のアルゴリズムは
  - $h\equiv 0$ ならば 確率$1$ でYesと出力
  - $h\not\equiv 0$ ならば 確率$\ge 1/2$ でNoと出力

</div>

<v-click>

-> これを使えば完全マッチングの判定問題が解ける!

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# Tutte行列に基づく乱択アルゴリズム

::content::

- 実際のアルゴリズムは以下の通り (繰り返し実行すれば成功確率を増幅できる)

<div class="algorithm">

  アルゴリズム $B$
  1. Tutte行列 $T$ を構成する
  2. 各変数 $x_{u,v}$ に対して, 独立に $\{1,\dots,2n\}$ から一様ランダムに値を割り当てる
  3. 行列式 $\det(T)$ を計算し, もし $\det(T)\ne 0$ ならば Yes を出力する. そうでなければ No を出力する

</div>

<div class="theorem" v-click>

上記のアルゴリズムは
  - $G$ が完全マッチングを持つならば 確率$\ge 1/2$ でYesと出力
  - $G$ が完全マッチングを持たないならば 確率$1$ でNoと出力

</div>


---
layout: top-title
color: amber-light
---

::title::
# 計算量と完全マッチングの探索

::content::

- 行列式は多項式時間で計算可能
  - 計算途中の数値は最大で $n^n$ くらいになりうるが, **桁数**は$n$に関して多項式オーダーなので問題ない
  - 桁数の増大を抑えるために, **ランダムな素数で割った余り**を計算する方法もある (高確率でうまくいく)

<div class="question">

完全マッチングが存在する場合に, どうやってその完全マッチングを一つ見つけるか?

</div>

- Tutte行列による完全マッチングの判定を活用して多項式時間で探索可能
- 各辺 $e$ について, **$e$を削除したグラフに完全マッチングが存在するか**を判定する
  - もし存在するならば, $e$をマッチングに追加し, その両端点を$G$から削除して繰り返す
  - 存在しないならば, $e$を削除して繰り返す

---
layout: section
color: amber-light
---

# 増加路に基づくアルゴリズム

---
layout: top-title
color: amber-light
---

::title::
# 最大マッチング問題

::content::

- そもそも最大マッチングを解けば, Tutte行列などを考えなくても完全マッチングの存在判定ができる

<div class="question">

与えられた二部グラフ $G=(L\sqcup R,E)$ に対して, **辺数最大のマッチングを求めよ**.

</div>

- グラフ理論的な視点に基づくアルゴリズムを紹介する
  - 乱択を用いない (特に, 確率の非自明な議論が不要)
  - 桁数の議論が不要
  - この知見は後の応用(最大流問題や最小費用流など)にも役立つ

<div class="topic-box">

アイデア: $M=\emptyset$から開始し, マッチングを**徐々に拡張**していく. 拡張できなくなったら最大マッチング.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 交互路と増加路

::content::

<div class="definition">

- 二部グラフ $G=(L\sqcup R,E)$ とそのマッチング $M\subseteq E$ に対して, **$M$に関する交互路** とは, 辺が**交互に**$M$に属する辺と属さない辺からなる路のことである.
- $M$に関する交互路であって, 両端点がどちらも$M$に属さない頂点であるものを **増加路** という.

</div>

<div class="topic-box">

交互路や増加路は記号$P$で表され, 辺部分集合 $P\subseteq E$ として扱う.

</div>

- マッチング$M$と$M$に関する交互路$P$に対して, 排他的論理和 **$M\oplus P := (M\setminus P) \cup (P\setminus M)$** を考える
  - これは常にマッチングとなっている
- 特に$P$が増加路ならば, 辺数が$1$増える

---
layout: top-title
color: amber-light
---

::title::
# 増加路に基づくマッチングの拡張

::content::

- マッチング$M$と増加路$P$に対して, **$M\oplus P$** は新しいマッチングとなり, 辺数が$1$増える

<AugmentingPathVisualizer />

<script setup>
import AugmentingPathVisualizer from './components/AugmentingPathVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# 最大マッチングと増加路

::content::

- 実は, XORによる拡張を繰り返すと最大マッチングが得られる!

<div class="theorem">

二部グラフ $G=(L\sqcup R,E)$ とそのマッチング $M\subseteq E$ に対して,
<div style="text-align: center;">

$M$ は最大マッチングである $\iff$ $G$ は $M$ に関する増加路を持たない

</div>

</div>

- $\Rightarrow$ の証明は簡単
  - 対偶を考える. 増加路$P$を持つならばより大きなマッチングが構成できるので$M$は最大でない
- $\Leftarrow$ の証明が非自明

---
layout: top-title
color: amber-light
---

::title::
# 最大マッチングと増加路

::content::

<div class="theorem">

二部グラフ $G=(L\sqcup R,E)$ とそのマッチング $M\subseteq E$ に対して,
<div style="text-align: center;">

$M$ は最大マッチングである $\iff$ $G$ は $M$ に関する増加路を持たない

</div>
</div>
<hr>

<div style="display: grid; grid-template-columns: 1.5fr 1fr; gap: 2em; margin: 2em 0; align-items: start;">
  <!-- 左側: 説明文 -->
  <div>

  - ($\Leftarrow$) の証明の概要: 対偶を考える
    - つまり, $M$が最大でない $\Rightarrow$ 増加路が存在
    - $M$が最大でないので, より大きなマッチング$M'$がある
    - $M\oplus M'$ からなるグラフにおいて, $M'$と$M$の辺を交互に辿れば増加路になる

  </div>
  
  <!-- 右側: 図 -->
  <img src="/images/two_matchings.svg" alt="二つのマッチングの図" style="max-width: 100%; height: auto;" />
</div>


---
layout: top-title
color: amber-light
---

::title::
# 増加路の有無判定/見つけ方

::content::

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; margin: 2em 0; align-items: start;">
  <!-- 左側: 説明文 -->
  <div>

  1. マッチングされている頂点を赤く塗る
  2. マッチング辺は$\leftarrow$, それ以外の辺は$\rightarrow$に向きつけ
  3. 新たに二頂点$s,t$を追加し, 以下の有向辺を追加
     - $s\to$「赤くない左頂点」
     - 「赤くない右頂点」$\to t$ の有向辺を追加する
  4. $st$路を求める
     - 求めた経路から$s,t$を除去すれば**増加路**
     - 例えば$st$最短路を求めればよい
     - 実は$O(|E|)$時間でできる


  </div>
  
  <!-- 右側: 図 -->
  <div style="text-align: center;">
    <img src="/images/find_aug.svg" alt="増加路の探索の図" style="max-width: 100%; height: auto;" />
  </div>
</div>

---
layout: top-title
color: amber-light
---

::title::
# 最大マッチングを求めるアルゴリズム

::content::

<div class="theorem">

重みなし二部グラフ$G=(L\sqcup R,E)$ が与えられたとき, $O(|V||E|)$時間で最大マッチングを求めることができる (ただし$V=L\sqcup R$).

</div>

<div class="algorithm">

1. $M=\emptyset$ で初期化
2. $M$に関する増加路$P$を見つけ, $M\leftarrow M\oplus P$ と更新
3. ステップ2を可能な限り繰り返す

</div>

  - 増加路の探索は$O(|E|)$時間
  - 一回の更新で$M$の要素数は$1$増える. 最大マッチングの本数は高々$|V|$なので, 更新回数は高々$|V|$
    - 実際には$\min\{|L|,|R|\}$で抑えられる


---
layout: section
color: amber-light
---

# LPに基づくアルゴリズム

---
layout: top-title
color: amber-light
---

::title::
# 最小重み完全マッチング

::content::

<div class="question">

$n$人の従業員に$n$個の仕事を割り振りたい.
人$i$に仕事$j$を割り当てると $W_{i,j}$円の費用が発生する.
また, 仕事の割り当ては1人ひとつずつでなければならない (1人が二つ以上の仕事をこなしてはならない).
費用を最小にするには, どのように割り当てればよいか?

</div>

<MinWeightMatchingVisualizer />

<script setup>
import MinWeightMatchingVisualizer from './components/MinWeightMatchingVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# 最小重み完全マッチング

::content::

- マッチングの言葉を使うと以下のように言い換えられる:

<div class="topic-box">

辺重みつきの二部グラフ $G=(L\sqcup R,E,w)$ を考える. マッチング $M\subseteq E$ の重みを辺重みの総和

$$
w(M) := \sum_{e\in E} w(e)
$$

と定義する. 完全マッチング$M$の中で$w(M)$が最小となるものを求めよ.
</div>

- $L$ = 従業員の集合
- $R$ = 割り当てたい仕事の集合
- 完全マッチング$M$ = 従業員への仕事の割り当て

---
layout: top-title
color: amber-light
---

::title::
# 最小重み完全マッチングのIP定式化

::content::

- **整数計画問題(IP)** として定式化する
  - 変数 $x_e \in \binset$ を, $e$をマッチングに含めるならば$x_e=1$, そうでなければ $x_e = 0$ とする
  - 選んだ辺集合がマッチング $\iff$ 全ての頂点に対して高々1本ずつ選んだ辺が接続している

  $$
  \boxed{
  \begin{align*}
  \text{min} \quad & \sum_{e\in E} w(e)\cdot x_e \\
  \text{s.t.} \quad & \sum_{e\in \delta(v)} x_e \le 1 & & (\forall v\in V) \\
                    & x_e \ge 0. & & (\forall e\in E)
  \end{align*}}
  $$



  
  
  