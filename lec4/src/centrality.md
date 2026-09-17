---
layout: section
color: amber-light
---

# 応用: ネットワーク中心性

---
layout: top-title
color: amber-light
---

::title::
# ネットワーク解析

::content::

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 1.5em; align-items: center;">
  <div>

  - 現実社会の多様なネットワーク
    - ソーシャルネットワーク
    - 交通ネットワーク
    - 感染症拡大モデル
    - 情報・物流ネットワーク

  </div>
  <div>
    <div style="display: flex; gap: 1em; justify-content: center; align-items: center;">
      <img src="/images/BA.png" alt="ネットワーク例 (BA)" style="max-width: 48%; border-radius: 8px;" />
      <img src="/images/RGG.png" alt="ネットワーク例 (RGG)" style="max-width: 48%; border-radius: 8px;" />
    </div>
  </div>
</div>

<div class="question">

- 最も影響力のある人物は誰か? (販促してもらう)
- 最も渋滞が起こりやすい頂点/辺はどこか? (渋滞緩和)
- 最も感染が拡大しやすそうな場所はどこか?

</div>

---
layout: top-title
color: amber-light
---

::title::
# ネットワーク解析

::content::

<div class="topic-box">

ネットワーク(グラフ)の様々な性質を定量的に評価するための様々な指標が存在する:
- クラスタ係数
- 直径
- 中心性

これらの定義の理解には**グラフの用語の知識**が必要であり, その効率的な計算には**グラフアルゴリズム**の知見が必要.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 中心性

::content::

- 各頂点 $v$ がどれくらい「中心的か」を表す指標の総称
  - 例えば検索結果の表示順は**中心度が高いページから**順に表示される
  - Webページを作成したとき, 検索結果の上位にくるように工夫することがある(SEO)

<v-click>

- 様々な指標が存在する
  - 媒介中心性, 近接中心性, ページランク, 次数中心性, ...

<div class="topic-box">

今回の講義は**最短経路**に基づく中心性として以下の二つを紹介:
- 媒介中心性 ([Freeman, 1970](https://www.jstor.org/stable/3033543?origin=crossref&seq=1)) 
- 近接中心性 ([Bavelas, 1950](https://pubs.aip.org/asa/jasa/article-abstract/22/6/725/646415/Communication-Patterns-in-Task-Oriented-Groups?redirectedFrom=fulltext))

</div>

- 話を単純にするため, 重みなしグラフを考える

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 媒介中心性

::content::

<div class="definition">

  グラフ $G=(V,E)$ および三つの頂点$u,v,i\in V$に対し
  
  $$
    \begin{align*}
      &N_{u,v} = \text{$uv$-最短路の個数} \\
      &N_{u,v}(i) = \text{$uv$-最短路であって$i$を経由するものの個数}
    \end{align*}$$
  
  とする. 頂点 $i$ の**媒介中心性** $\mathrm{BC}(i)\in\Real$ を

  $$
    \begin{align*}
      \mathrm{BC}(i) = \sum_{u,v\in V\setminus\{i\}} \frac{N_{u,v}(i)}{N_{u,v}}
    \end{align*}  $$
  で定める.

</div>

<div class="topic-box">

直感: $\mathrm{BC}(u)$ は「$i$を通る最短路の割合」. これが大きいと「よく通過する」ので中心的である.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 媒介中心性の例 (ランダム幾何グラフとBAモデル)

::content::

<div style="display: flex; justify-content: center; align-items: center; gap: 2em;">
  <img src="/images/RGG_BC.png" alt="媒介中心性の例 (RGG)" style="max-width: 38%; height: auto; box-shadow: 0 2px 12px rgba(0,0,0,0.08); border-radius: 8px; margin: 1.5em 0;" />
  <img src="/images/BA_BC.png" alt="媒介中心性の例 (BA)" style="max-width: 38%; height: auto; box-shadow: 0 2px 12px rgba(0,0,0,0.08); border-radius: 8px; margin: 1.5em 0;" />
</div>

- 頂点の重要度を相対的に評価できる


---
layout: top-title
color: amber-light
---

::title::
# 近接中心性

::content::

<div class="definition">

  グラフ $G=(V,E)$ に対し, $\dist(u,v)$を頂点$uv$間の距離とする.
  各頂点$i\in V$ に対し,

  $$
    \begin{align*}
      \mathrm{CC}(i) = \frac{|V|}{\sum_{u\in V}\dist(i,u)}
    \end{align*}  $$
  
  を頂点 $i$ の**近接中心性** と呼ぶ.

</div>

- 分子の$|V|$は頂点数で正規化

<div class="topic-box">

直感: $\mathrm{CC}(i)$が大きい $\iff$ 他頂点との距離が平均的に小さい

つまり, 他の頂点にすぐ辿り着けるグラフが「中心的である」

</div>