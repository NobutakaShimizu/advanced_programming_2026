---
theme: neversink
layout: cover
title: プログラミング応用第7回
routerMode: hash
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

# プログラミング応用 第7回: <br> なんか「うまくいく」アルゴリズム

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

1. [理論と実用のギャップ](/3)
2. [k-means法](/6)
   - [平滑化解析](/16)
3. [局所探索と焼きなまし法](/17)
   - [最適解への収束](27)
   - [ボルツマン分布](/30)


---
layout: section
color: amber-light
---

# 理論と実用のギャップ


---
layout: top-title
color: amber-light
---

::title::
# 理論と実用のギャップ

::content::

<div class="question">

NP困難であっても実用上高速なアルゴリズムが多く知られている. なぜ?

</div>

<v-clicks>

- LPに対する単体法 (最悪の場合は指数時間かかるが多くの場合は高速)
- 充足可能性判定問題に対するSATソルバー
- 整数計画問題(IP)に対するIPソルバー
- 焼きなまし法
- k-means法
- ニューラルネットによる学習 (汎化誤差が小さい)

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# アルゴリズムの理論保証

::content::

- これまで扱ってきた効率性の指標: **最悪時**時間計算量
  - 計算量 $\le T(n)$ $\iff$ **全て** のサイズ $n$ の入力に対する計算ステップ数が $\le T(n)$
  
<v-clicks>

<div class="topic-box">

「いじわるな」入力が存在すれば, 最悪時時間計算量はその入力に引っ張られる.

</div>

- 「いじわるな」入力は**実用ではまず現れない**こともある
  - このようなとき, 「実用的な計算量」$\ll$「最悪時時間計算量」
  
<div class="question">

「実用的な計算量」はどのように定義すべきか?

</div>

- 専門的な内容ではあるが, 知っておく価値はとても高いので紹介したい

</v-clicks>

---
layout: section
color: amber-light
---

# k-means法

---
layout: top-title
color: amber-light
---

::title::
# 背景

::content::

- あらゆる分野で**高次元の数値データ**が得られる  
  - 顧客行動ログ, 画像埋め込み, 文書ベクトル, etc
- これらのデータから学習して**将来予測**に活用したい
  - 購買の予想, 画像の生成(拡散モデル), 次の文の推測(Transformer), etc

<img src="/images/keiba.png" alt="クラスタリング例" style="max-width: 25%; display: block; margin: 32px auto 20px;" />

---
layout: top-title
color: amber-light
---

::title::
# 背景

::content::

- しかし多くの場合, **ラベル（正解）** が存在しない -> **教師なし学習**
  - 画像データはあってもその画像が何なのか説明されることは少ない
  
<v-click>  
  
- 似た性質を持つデータをまとめて潜在的なカテゴリや代表的な振る舞いや特徴を抽出する  
  - この発想を **クラスタリング (clustering)** という

<img src="/images/clustering.png" alt="クラスタリング例" style="max-width: 40%; display: block; margin: 32px auto 20px;" />

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 解きたい問題

::content::

- たくさんの点(**データ点**)を受け取り, 近い点同士でグループ化したい

<div class="question">

$n$ 個の点 $x_1,\dots,x_n \in \mathbb{R}^d$ (**データ点**) を受け取り, $k$ 個の点 $C=\set{c_1,\dots,c_k} \subseteq \Real^d$ (**中心点**と呼ぶ) を出力せよ.
ただし, 以下の目的関数を最小化したい:

$$
\Phi(C) = \sum_{i=1}^n \min_{j\in[k]} \|x_i - c_j\|^2
$$

</div>

- 各データ点を最も近い中心点に割り当てたときの**二乗誤差の総和**を最小化
- $k$も入力で与えられる (もしくは定数, 例えば$k=3$で固定)

---
layout: top-title
color: amber-light
---

::title::
# 目的関数の意味

::content::

<div class="topic-box">

$C=\{c_1,\dots,c_k\} \subseteq \Real^d$に対し,

$$
\Phi(C) = \sum_{i=1}^n \min_{j\in[k]} \|x_i - c_j\|^2
$$

</div>

<KMeansObjectiveVisualizer />

<script setup>
import KMeansObjectiveVisualizer from './components/KMeansObjectiveVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# Lloydのアルゴリズム

::content::

<div class="algorithm">

1. **初期化**: $k$個の中心点 $C=\{c_1,\dots,c_k\}$ をランダムに初期化
2. **割り当てステップ**: 各データ点$x_i$を, 最も近い中心点に割り当てる (候補が複数ある場合は任意に一つ選ぶ)
3. **更新ステップ**: 各$c_j\in C$ について, $c_j$に割り当てられたデータ点の平均座標を計算し, $c_j$ をその平均に更新する
5. ステップ2と3を収束するまで（または所定の回数まで）繰り返す

</div>

<v-clicks>

- 初期点の選び方にも工夫の余地がある -> **k-means++**
- データ点の次元が大きい場合, 前処理で工夫して高速化の余地がある
  - PCA (主成分分析): 大きい固有値に対応する固有空間への射影
  - [Johnson-Lindenstraussの次元削減](https://advanced-programming-2025.pages.dev/lec3/34): ランダムな射影
  
</v-clicks>  

---
layout: top-title
color: amber-light
---

::title::
# 実行例

::content::

<KMeansVisualizer />

<script setup>
import KMeansVisualizer from './components/KMeansVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# Lloydのアルゴリズム

::content::

<div class="question">

- 最適解が得られる?
- アルゴリズムの反復回数は?

</div>

<v-clicks>

<div class="theorem">

<div style="display: grid; grid-template-columns: 7fr 3fr; gap: 2rem; align-items: center;">

  - $\Phi(C)$を最小化する$C$の計算は**NP困難**
  - 反復回数が**指数時間かかる**データ点配置の例が存在

  <div style="display: flex; justify-content: center; align-items: center;">
    <img src="/images/fusagikomu.png" alt="クラスタリング例" style="max-width: 40%;" />
  </div>
</div>
</div>

- 「いじわるな」データ点配置が存在
- 最悪時計算量の意味では困難

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最悪時以外の計算量解析

::content::

<div class="topic-box">

最悪時計算量と実用性とのギャップをなんとかしたい.

</div>

<v-clicks>

- **平均時計算量** (Average-Case Complexity)
  - **入力が何らかの分布に従って生成される**と仮定 (ランダムな入力を考える)
  - 「平均的な」計算量を解析 (例えば期待値) -> 「いじわるな」入力例には引っ張られない
  - 最適化の多くの論文はランダムに生成した入力上で計算機実験している

<div class="topic-box">

特定の分布でうまくいくことが保証されるに過ぎない.
多くの場合は入力分布が不明なので, アルゴリズムの実用性を捉えるかは疑問が残る.

</div>

<div class="remark">

暗号分野では非常に重要視されている (RSA暗号では**ランダムな自然数**の素因数分解の困難性に依拠).

</div>
  
</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 最悪時以外の計算量解析

::content::


<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2rem; align-items: start;">

<div>

- **平滑化解析** (Smoothed Analysis)
  - いじわるな入力に **小さい「ノイズ」** を加えて得られた入力上での計算量を評価
  - 各入力の周辺だけで期待値をとる

<v-clicks>

- ノイズの分散が小さい -> 最悪時計算量
- ノイズの分散が大きい -> 平均時計算量

</v-clicks>

</div>

<div>

<SmoothedAnalysisVisualizer />

</div>

</div>

<script setup>
import SmoothedAnalysisVisualizer from './components/SmoothedAnalysisVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# 平滑化解析 (Spielman & Teng, 2001)

::content::

- 最悪時だと指数時間であっても, 平滑化では多項式時間になる
  - [単体法](https://dl.acm.org/doi/10.1145/990308.990310) (LP)
  - [k-means](https://dl.acm.org/doi/10.1145/2027216.2027217) (クラスタリング)
  - [最大カットに対する局所探索](https://dl.acm.org/doi/10.1145/3011870)

<div class="topic-box" v-click>

- 現状, 平滑化解析が成功している例はまだ少ない (研究の余地あり)
- 平滑化解析の意味で多項式時間であることは示されていても, 多項式のオーダーはとても大きい ($O(n^{30})$時間など)

</div>

---
layout: section
color: amber-light
---

# 局所探索と焼きなまし法



---
layout: top-title
color: amber-light
---

::title::
# 局所探索

::content::

<div class="topic-box">

初期解からスタートし, 少しずつ解を更新して最適解に近づいていく方針に基づくアルゴリズムを総称して **局所探索** と呼ぶ.

</div>

<v-clicks>

<div style="display: grid; grid-template-columns: 1fr 1.2fr; gap: 2em; align-items: center;">

  <div>
  
  - 勾配法
  - ポテンシャルの更新に基づく最短経路問題の解法
  - k-means法
  
  <div class="topic-box">
  
  k-meansのように, 必ずしも最適解にしなくても「良い」解が得られることもある.
  
  </div>
  
  </div>

  <div>
    <video src="/images/GD1d.mp4" controls style="max-width: 100%; width: 100%; border: 1px solid #ccc; border-radius: 8px; margin: 1em 0 1em 0;"></video>
  </div>
</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 近似アルゴリズムとヒューリスティクス

::content::

<div class="topic-box">

実用的には, 最適解でなくとも**目的関数値が十分良い解で事足りる**場面が多い.

</div>

<v-clicks>

- 例えば, ナップザック問題や巡回セールスマン問題では, 必ずしも最適解でなくとも, 「十分良い」解があればよい場合が多い
- このような場合, **局所探索**は一つの有力なアプローチとなる
  - 好きなタイミングで停止できるので, 十分反復を繰り返したあとの解を出力すればよい

<div class="remark">

最適解を求めることがNP困難であっても, 「十分良い」解は効率的に求められるかもしれない
  - 出力した解の質に理論的な保証があるアルゴリズム -> **近似アルゴリズム**
  - 理論的な保証はないが, 実用上よく機能するアルゴリズム -> **ヒューリスティクス (発見的解法)**

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# 巡回セールスマン問題に対する局所探索(2-opt法)

::content::

<div class="question">

**入力**: 距離関数 $w\colon V\times V\to \Real_{\ge 0}$ (ただし$w(u,u)=0$)

**出力**: 全ての頂点を一度ずつ訪れる巡回路 $P$ であって, 総距離 **$w(P):=\sum_{(u,v)\in P} w(u,v)$** を最小化

</div>

<v-click>

- 次のような局所探索アルゴリズム(2-opt法)が知られている:

<div class="algorithm">

1. 初期巡回路 $P$ を適当に構成
2. $P$ の2つの辺を順序つきで選んでそれぞれ$(a,b),(c,d)$とする. $P$から辺$\{a,b\},\{c,d\}$を削除し, 新たに辺 $\{a,d\},\{c,b\}$を追加して得られる巡回路 $P'$ を考える (ただし$P'$も順回路になるようにする)
3. もし **$w(P') < w(P)$** ならば, $P \leftarrow P'$ と更新し, ステップ2に戻る
4. 更新ができなくなったら終了し, $P$ を出力

</div>

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 巡回セールスマン問題に対する局所探索(2-opt法)

::content::

<img src="/images/2-opt.svg" alt="2-opt法の図" style="display: block; margin: 2em auto; max-width: 500px; width: 100%;" />
<figcaption style="text-align: center; margin-top: 1em; color: #666; font-size: 0.95em;">

2-opt法による2本の辺の入れ替え操作. 代わりに $\{a,c\},\{b,d\}$ を追加してしまうと二つの巡回路に分かれてしまう.

</figcaption>

---
layout: top-title
color: amber-light
---

::title::
# 2-opt法の可視化

::content::

<TSP2OptVisualizer />

<script setup>
import TSP2OptVisualizer from './components/TSP2OptVisualizer.vue'
</script>


---
layout: top-title
color: amber-light
---

::title::
# 局所最適解と大域最適解

::content::

<div style="display: grid; grid-template-columns: 1.2fr 1fr; gap: 2em; align-items: start;">

<div>

- 2-opt法の反復を繰り返すと, これ以上改善できない巡回路 $P$ に到達する
  - これを**局所最適解**という
- 対して最適解を**大域最適解**という
- 局所最適解が**必ずしも最適解とは限らない**ことに注意

</div>

<div>

<LocalGlobalOptimumVisualizer />

</div>

</div>

<script setup>
import LocalGlobalOptimumVisualizer from './components/LocalGlobalOptimumVisualizer.vue'
</script>

<div class="topic-box" v-click>

局所最適解から脱出するには一旦「山を登る」(=目的関数値が悪化する操作を行う) 必要がある

</div>

---
layout: top-title
color: amber-light
---

::title::
# 焼きなまし法 (Simulated Annealing)

::content::

- 焼きなまし法: 目的関数値が悪化する操作も**確率的に**受け入れる
  - **確率$\theta$** で $P\leftarrow P'$ と更新
  - ただし $\theta$ は **現在時刻 $t$** と **改善幅 $w(P')-w(P)$** に依存する
- 時間とともに 許容確率 $\theta$ を小さくしていく
  - 時間が経つにつれて改善操作のみを受け入れるようになっていく

<div class="topic-box">

2-opt法とは異なり, 悪くなる操作も受け入れることで, 局所最適解から脱出できる可能性がある.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 焼きなまし法の詳細

::content::

- TSPの例では, 次のようになる:

<div class="algorithm">

1. 初期巡回路 $P$ を適当に構成し, 時刻 $t \leftarrow 0$ で初期化
2. $P$ の2つの辺を**ランダムに**選び, それらの一つの端点を入れ替えることで得られる巡回路 $P'$ を考える
3. 確率 $\min\left\{1, \exp\left(-\frac{w(P') - w(P)}{\textcolor{c2185b}{T(t)}}\right) \right\}$ で $P \leftarrow P'$ と更新
4. $t\leftarrow t+1$ とし, ステップ2に戻る

</div>

- $T(t)$ は **温度関数** と呼ばれ, 時刻 $t$ とともに減少する関数
  - 例えば, $T(t) = \frac{T_0}{\log(2+t)}$ など ($T_0$は大きな定数)

---
layout: top-title
color: amber-light
---

::title::
# 更新確率の可視化

::content::

<div style="display: grid; grid-template-columns: 1.2fr 1fr; gap: 2em; align-items: start;">

<div>

- 更新確率 **$\theta = \min\left\{1,\exp\left(-\frac{\Delta w}{T}\right)\right\}$**
  - $\Delta w = w(P') - w(P)$ を固定値として設定可能
  - パラメータ $T$ を横軸、$\theta$ を縦軸としてプロット
- $T$ が大きいほど $\theta$ が大きくなる（悪化操作も受け入れやすい）
- $T$ が小さいほど $\theta$ が小さくなる（改善操作のみ受け入れる傾向）
- $\Delta w < 0$ ならば $\theta = 1$ -> 必ず更新

</div>

<div>

<SimulatedAnnealingProbabilityVisualizer />

</div>

</div>

<script setup>
import SimulatedAnnealingProbabilityVisualizer from './components/SimulatedAnnealingProbabilityVisualizer.vue'
import BoltzmannDistributionVisualizer from './components/BoltzmannDistributionVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# 2-opt vs. 焼きなまし法

::content::

<TSPComparisonVisualizer />

<script setup>
import TSPComparisonVisualizer from './components/TSPComparisonVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# 2-opt vs. 焼きなまし法

::content::

- 時刻$t$が小さいところでは, 2-optの方が改善して焼きなまし法は改悪しやすい
- $t$を大きくすると焼きなまし法の方が良い解に収束することが多い (反復回数を大きくすることが大事)

- 実は焼きなまし法は **最適解** への収束が保証されてる

<div class="theorem">

焼きなまし法は**十分にゆっくり温度を下げれば**, 最適解に収束する.
具体的には, 十分大きな$T_0>0$に対し


$$
  \begin{align*}
    T(t) = \frac{T_0}{\log(2+t)}
  \end{align*}
$$

とすれば, $t\to\infty$のとき, 確率$1$で最適解に収束する.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 直感的な説明

::content::

- TSPの局所探索は**全ての順回路$P$全体の集合を頂点集合とみなした**メタ的なグラフ上の探索とみなせる
  - メタグラフの頂点数は $(n-1)!/2$ 個 (巡回路の数)
  - 二つの頂点 $P,P'$ が隣接 $\iff$ 2-optの更新で行き来できる

<img src="/images/metagraph.svg" alt="メタグラフの図" style="display: block; margin: 2em auto; max-width: 400px; width: 100%;" />


---
layout: top-title
color: amber-light
---

::title::
# 直感的な説明

::content::

- 焼きなまし法はメタグラフ上の**ランダムウォーク**とみなせる
  - ランダムウォーク: 各ステップで隣接頂点にランダムに移動する乱択探索
  - 焼きなまし法: ランダムな2辺を選択 -> 受理確率に基づき遷移
  
 
<v-click>
  
- メタグラフの各頂点(=順回路) $P$ に対し以下の値を考える:
  
  $$
    \begin{align*}
      \mu_t(P) = \frac{\text{\textcolor{c2185b}{ランダムウォークが時刻 $t$ までに頂点 $P$ を訪問した回数}}}{t}
    \end{align*}
  $$

  - $\mu_t(P)$が一定の場合大きい $\Rightarrow$ ランダムウォークが$P$を訪問しやすい

<div class="remark">

$\mu_t(P)\ge 0$ かつ $\sum_P \mu_t(P) = 1$ なので, 各$\mu_t$ は順回路全体の集合上の分布を定める

</div>  

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# ランダムウォークのシミュレーション

::content::

<RandomWalkVisualizer />

<script setup>
import RandomWalkVisualizer from './components/RandomWalkVisualizer.vue'
</script>

---
layout: top-title
color: amber-light
---

::title::
# ランダムウォークの定常分布

::content::

<div class="theorem">

温度$T$が一定の場合, **ある分布 $\pi$ (定常分布) が存在して**, 焼きなまし法の任意の初期巡回路 $P_0$ に対し,

$$
  \begin{align*}
    \lim_{t\to\infty} \mu_t(P) = \pi(P).
  \end{align*}
$$

</div>

<v-click>

- メタグラフが連結ならランダムウォークは収束する
- **温度を固定して**焼きなまし法を動かすと, 十分に時間が経った後の巡回路 $P$ の分布はほぼ $\pi$
  - 実際には温度を非常に**ゆっくり**動かすので, $T$を固定した状況とほぼ同じ
- 定常分布 $\pi$ は以下を満たす (**ボルツマン分布**という):

$$
  \begin{align*}
    \pi(P) \propto \textcolor{c2185b}{\exp\left(-\frac{w(P)}{T}\right)}
  \end{align*}
$$

</v-click>

---
layout: top-title
color: amber-light
---

::title::
# 焼きなまし法の定常分布

::content::

- 焼きなまし法では

$$
  \begin{align*}
    \mu_t(P) \to \pi(P) \propto \textcolor{c2185b}{\exp\left(-\frac{w(P)}{T}\right)}
  \end{align*}
$$

- つまり, 順回路 $P$ の訪問回数は $\exp\left(-\frac{w(P)}{T}\right)$ に比例
  - $w(P)$が小さいほど訪問回数が多い
  - $T\to \infty$のとき, 一様分布に収束 (温度が高い状態) 

<div class="question">

$T\to 0$のとき, 定常分布 $\pi$ はどうなる?

</div>


---
layout: top-title
color: amber-light
---

::title::
# 焼きなまし法の定常分布

::content::

<BoltzmannDistributionVisualizer />

- $w(P)$が小さいほど, $\pi(P)$は大きい. 温度 $T$ が小さいほど, その差が顕著になる

---
layout: top-title
color: amber-light
---

::title::
# 焼きなまし法の定常分布

::content::

- $P$が最適解じゃないならば, $\pi(P)\to 0$
  - よって, $T\to 0$の極限では定常分布 $\pi(P)$ は**最適解上の一様分布**となる

- ただし, $T=0$ としてしまうと2-opt法と一致
  - ランダムウォークの収束性が失われる
  - 実際, **初期温度が十分高くないと**最適解への収束が保証できない
  
<div class="remark">

実際には **$T$を動かしながらランダムウォークを行う** が, その変化は非常にゆっくりなので, 温度$T$での定常分布 $\pi_T$ に収束する.
しかしその証明はとても難しい.

</div>

---
layout: top-title
color: amber-light
---

::title::
# 焼きなまし法のまとめ

::content::

- 焼きなまし法では, 温度パラメータによって二つの性質を緩やかに変化させられる
  - 高温状態 $\approx$ メタグラフ上の一様ランダムな隣接点への遷移
  - 低音状態 $\approx$ 2-opt法
- 温度変化がゆっくりだと, それぞれの温度帯で訪問回数が定常分布(ボルツマン分布)に収束
  - 低音状態では, 最適解上に確率が集中する

<div class="remark">

最適解への収束性は保証されているが, その収束のスピードは非常に遅いため, 実用では**温度は早く下げる**ことが多い (それでも経験上, 良い解が得られることが多い).
- 理論保証: $T(t) = \frac{T_0}{\log(2+t)}$ ($T_0$は十分大きな値)
- 現実: $T(t) = \frac{T_0}{t}$

</div>

---
layout: top-title
color: amber-light
---

::title::
# メタヒューリスティクス

::content::

- 焼きなまし法はTSP以外にも様々な問題に対して応用可能
  - 原理的には「メタグラフ」上での遷移が定義できれば実装可能

<div class="remark">

このような, 多くの問題に汎用的に適用できる発見的解法を **メタヒューリスティクス** と呼ぶ.

</div>

- 他にも**遺伝的アルゴリズム**, **タブーサーチ**, **アリコロニー最適化**, **粒子群最適化**など
- これらは全て, 局所探索の枠組みを拡張したものであるが, **理論保証がない**
- 個人的には, 少なくとも研究の場面では, 複雑で凄そうなアルゴリズムを使うよりも, まずは単純な局所探索や焼きなまし法で試すことをお勧めする
  - 「これじゃなきゃいけない」理由を説明できないアルゴリズムは避けるべき

---
layout: top-title
color: amber-light
---

::title::
# 今日のまとめ

::content::

1. **理論と実用のギャップ**
   - 最悪時計算量は「いじわるな」入力に引っ張られる
   - 実用的な計算量を捉えるための考え方: 平均時計算量, 平滑化解析

2. **k-means法**
   - Lloydのアルゴリズムは最悪時では指数時間かかるが、平滑化解析の意味では多項式時間
   - 平滑化解析: いじわるな入力にノイズを加えた上での計算量を評価

1. **局所探索と焼きなまし法**
   - 局所探索: 初期解から少しずつ解を更新して最適解に近づく
   - 焼きなまし法: 悪化する操作も確率的に受け入れることで局所最適解から脱出
   - 温度を十分ゆっくり下げれば最適解に収束することが保証される (ボルツマン分布への収束)

