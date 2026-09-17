---
layout: section
color: amber-light
---

# Floyd--Warshall法

---
layout: top-title
color: amber-light
---

::title::
# 全点間最短経路問題

::content::

<div class="question">

重み付きグラフ $G=(V,E,w)$ が与えられる.
**全ての頂点組** $(u,v)\in V\times V$ について, $\dist(u,v)$ を求めよ.

</div>

<v-clicks>

- 単一始点最短経路問題を$\abs{V}$回解けば, 上の問題も解ける
- これより効率的に求めたい
  
<div class="question">

全ての始点について一々解き直すのではなく, 全部の距離を同時に計算できないか?

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# Floyd--Warshall法

::content::

<div class="theorem">

全点対最短経路問題は $O(\abs{V}^3)$ 時間で解ける.

</div>

<v-clicks>

- 頂点集合を $V=\{1,2,\ldots,n\}$ とし, 以下の値を考える:

<div class="topic-box">

  **$\dist_{\le k}(u,v)$** $=$ 頂点集合 $\{1,\ldots,k\}$ のみを中継頂点として用いる場合の, $u$ から $v$ への最短距離. <br>
  ただし, $\dist_{\le 0}(u,v)$ は, 辺 $(u,v)$ の重み $w(u,v)$ で定義する. 辺が存在しない場合は $\infty$ とする.

</div>

<img src="/images/WF.svg" alt="Floyd-Warshall algorithm illustration" style="max-width: 500px; display: block; margin: 1.5em auto;" />

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 計算例

::content::

<div class="topic-box">

  **$\dist_{\le k}(u,v)$** $=$ 頂点集合 $\{1,\ldots,k\}$ のみを中継頂点として用いる場合の, $u$ から $v$ への最短距離. <br>
  ただし, $\dist_{\le 0}(u,v)$ は, 辺 $(u,v)$ の重み $w(u,v)$ で定義する. 辺が存在しない場合は $\infty$ とする.

</div>

<img src="/images/WF1.svg" alt="Floyd-Warshall計算例" style="max-width: 300px; display: block; margin: 1.5em auto;" />
<div style="text-align: center; font-size: 0.9em; color: #555; margin-bottom: 1.5em;" markdown="1">

例1. $\dist_{\le 3}(4,10) = 3$

</div>

---
layout: top-title
color: amber-light
---

::title::
# 他の計算例

::content::

<div style="display: flex; gap: 2em; justify-content: center; align-items: flex-start; margin-bottom: 1.5em;">

  <div style="flex: 1 1 0; max-width: 320px; text-align: center;">
    <img src="/images/WF2.svg" alt="Floyd-Warshall計算例2" style="max-width: 100%; display: block; margin: 0 auto 0.7em auto;" />

  <div style="font-size: 0.9em; color: #555;" markdown="1">
    
  例2. $\dist_{\le 5}(10,7) = 4$

  </div>
  </div>

  <div style="flex: 1 1 0; max-width: 320px; text-align: center;">
    <img src="/images/WF3.svg" alt="Floyd-Warshall計算例3" style="max-width: 100%; display: block; margin: 0 auto 0.7em auto;" />
  <div style="font-size: 0.9em; color: #555;" markdown="1">
  
  例3. $\dist_{\le 3}(7,8) = \infty$

  </div>
  </div>

</div>

<div class="remark">

条件を満たす経路が存在しない場合は $\infty$ とする (右の例).

</div>

---
layout: top-title
color: amber-light
---

::title::
# Floyd--Warshall法の漸化式

::content::

<div class="topic-box">

$uv$間の距離は **$\dist_{\le n}(u,v)$** である. 従って, 全ての $\dist_{\le k}(u,v)$ を求めれば解けたことになる.

</div>

<v-clicks>

- $\dist_{\le k}(u,v)$ に寄与する$uv$-路は, 次の2通りに分類できる:

- ケース1: 中継頂点 $k$ を**通らない**場合
  -  この場合, 最短距離は $\dist_{\le k-1}(u,v)$ に等しい 

<img src="/images/WFcase1.svg" alt="Floyd-Warshallケース1" style="max-width: 700px; display: block; margin: 1.3em auto 1.3em auto;" />

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# Floyd--Warshall法の漸化式

::content::

<div class="topic-box">

$uv$間の距離は **$\dist_{\le n}(u,v)$** である. 従って, 全ての $\dist_{\le k}(u,v)$ を求めれば解けたことになる.

</div>

- $\dist_{\le k}(u,v)$ に寄与する$uv$-路は, 次の2通りに分類できる:


- ケース2: 中継頂点 $k$ を**通る**場合 (負閉路がないならば一度だけ通る)
  -  この場合, 最短距離は $\dist_{\le k-1}(u,k) + \dist_{\le k-1}(k,v)$ に等しい 

<v-clicks>

<img src="/images/WFcase2.svg" alt="Floyd-Warshallケース1" style="max-width: 700px; display: block; margin: 1.3em auto 1.3em auto;" />


<div class="topic-box">

二つの小さい方をとると, $\dist_{\le k}(u,v) = \min\{\dist_{\le k-1}(u,v), \dist_{\le k-1}(u,k) + \dist_{\le k-1}(k,v) \}$.

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# Floyd--Warshall法の漸化式

::content::

<div class="topic-box">

$uv$間の距離は **$\dist_{\le n}(u,v)$** である. 従って, 全ての $\dist_{\le k}(u,v)$ を求めれば解けたことになる.

</div>

- $k=0$ の場合:
  - $\dist_{\le 0}(u,v) = w(u,v)$ （辺 $(u,v)$ の重み. 辺が存在しない場合は $\infty$）で初期化
  - この値は中継頂点を使わない場合の最短距離に対応

<div class="lemma" v-click>

Floyd--Warshall法の漸化式:

$$
  \begin{align*}
    \dist_{\le k}(u,v) &=
    \begin{cases}
      w(u,v) & \text{if }k=0,\{u,v\}\in E, \\
      \infty & \text{if }k=0,\{u,v\}\notin E, \\[6pt]
      \min\{ \dist_{\le k-1}(u,v), \dist_{\le k-1}(u,k) + \dist_{\le k-1}(k,v) \} & \text{if }k \geq 1
    \end{cases}
  \end{align*}
$$

</div>

---
layout: top-title
color: amber-light
---

::title::
# Floyd--Warshall法の擬似コード

::content::

- 漸化式を使って $\dist_{\le k}(u,v)$ を全て計算し, $(\dist_{\le n}(u,v))_{u,v}$ を出力する

<div class="algorithm">

**入力:** 重み付きグラフ $G=(V,E,w)$ <br>

1. 初期化: 全ての $(u,v)\in V\times V$ について, $\dist_{\le 0}(u,v) \leftarrow w(u,v)$. 辺が存在しない場合は $\infty$ とする.
2. 各$k=1,\dots,\abs{V}$に対して:
   - 各$(u,v)\in V\times V$に対して:
      - $\dist_{\le k}(u,v) \leftarrow \min\{ \dist_{\le k-1}(u,v), \dist_{\le k-1}(u,k) + \dist_{\le k-1}(k,v) \}$
3. 出力: $(\dist_{\le \abs{V}}(u,v))_{u,v\in V}$ を出力して終了

</div>

- forループが3重になっているので, 計算量は $O(\abs{V}^3)$

---
layout: top-title
color: amber-light
---

::title::
# Floyd--Warshall法のまとめ

::content::

- 全点対最短経路問題を $O(\abs{V}^3)$ 時間で解くアルゴリズム
  - 全ての頂点ペアの最短経路長 $\dist(u,v)$ を同時に計算
- 概要: $\dist_{\le k}(u,v)$ という中継頂点を制限した最短距離を定義し, 漸化式で計算

<div class="remark" v-click>

補助的な数列 (今回は $\dist_{\le k}(u,v)$) を定義して漸化式を立て, 小さいインデックスから順に計算していく考え方は **動的計画法** と呼ばれるアルゴリズムの基本設計方針の一つ.

動的計画法は非常に色んな場面で使われるアイデアである.

</div>
