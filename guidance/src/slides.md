---
theme: neversink
layout: cover
title: ガイダンス
author: 清水 伸高
mdc: true
githubPages:
  ogp: true
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


---

# プログラミング応用 ガイダンス

[清水 伸高](https://sites.google.com/view/nobutaka-shimizu/home) (塩浦研 助教)

<div style="position: absolute; bottom: 20px; font-size: 0.8em; width: 100%; text-align: center;">
2025年 10月7日
</div>


---
layout: top-title
color: amber-light
---

::title::

# 概要

::content::

<div class="topic-box">

- **講義**: アルゴリズムの理論的側面と様々な最適化アルゴリズムを学ぶ.
- **演習**: 講義で紹介したアルゴリズムや理論に関する問題を解く.
- 基本的に**数学**の授業です.

</div>

<v-click>

  - 演習課題 (毎週): 実装課題 or レポート課題
    - 実装課題: LMS上で配布
    - レポート課題: 演習開始時に問題用紙を配布
    - 締め切り: 一週間後の日本時間13時
    - 提出方法: LMSにアップロード (レポートはpdf形式のみ受け付ける)
  - 最終回後に期末課題を出題

</v-click>

---
layout: top-title
color: amber-light
---

::title::

# 受講に必要な環境

::content::

講義の受講にあたっては以下の環境が**必須**:

<v-clicks>

- 大学のSlackアカウント
  - 手続きについては[こちら](https://portal.isct.ac.jp/ja/sys/slack/guide.html#sign-in)を参照
  - 講義に関する私への連絡は**必ずSlackのDM**で行ってください
  
- LMSへのログイン
  - 課題の提出に利用する
- Pythonの実行環境
  - 個人のGoogleアカウントがあれば, [Google Colab](https://colab.research.google.com/?hl=ja)を利用できる (特に何もインストールしなくてよい).
  - 手元でPythonの環境が整っているならば, それを利用してもよい.

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::
# (個人的に)オススメな環境
::content::

将来的にプログラミングに携わる可能性があるならば, 以下のソフトやスキルを身につけておくとよい:

- **VSCode** (Visual Studio Code)
  - エディタ(プログラムを書くソフト)のデファクトスタンダード
  - 拡張機能が豊富で, Pythonの実行環境も整えやすい
  - 生成AI (GitHub Copilot) を用いたコーディングも可能
    - 最近は生成AIを用いたコーディングに特化したエディタ**Cursor**も人気 (私はこれを使っています)
    
<v-click>    

- **GitHub** (バージョン管理システム)
  - ソースコードの「セーブデータ」の履歴を管理できる (例えば学位論文の執筆で便利)
  - GitHub Pagesを使ってウェブサイトを作成できる
  - VSCodeにはGitHub用の拡張機能がある
  - 踏み込んだ使い方をしようとすると学習コストが高い
  
</v-click>

---
layout: top-title
color: amber-light
---
::title::
# (個人的に)オススメな環境
::content::

- **Markdown形式**
  - テキストファイルの書き方の一つで, 拡張子は`.md`
  - 簡単に言うと「htmlの簡易版」(可読性が高い)
  - Wordで書くよりもAIに学習させやすい
  - このスライドもマークダウン形式で書いている

<v-click>

- **Notion**または**Obsidian**
  - Markdown形式でドキュメントの作成・管理に便利 (勉強ノートや研究ノートの作成に便利)
    - 去年はNotionで[講義資料](https://nobunote.notion.site/112b125551f780cfb468e1542fecd175?v=112b125551f78147a287000cb91ef0c2&pvs=74)を作っています  
  - Notionは初心者でも直感的で使いやすく, Obsidianは拡張機能が豊富で細かいカスタマイズが可能
  - LaTeXで数式の入力も可能

</v-click>

---
layout: top-title
color: amber-light
---
::title::
# 生成AIの利用について
::content::

<div class="topic-box">

  **内容を理解していれば**生成AIを使ったCodingは推奨.

</div>

- 例えば, GitHub Copilotを使うと, コードの自動補完やコードの説明をしてくれる
  - Google ColabではGeminiが使える
  - 現在ではコーディングのみならず, 数学の証明や論理的な説明においても非常に有用
- ただし, 間違える可能性があるので, 出力されたコードをそのまま使うのではなく, 自分で確認すること
- 演習の時間中も利用はOK (おまかせします)

---
layout: top-title
color: amber-light
---

::title::

# Google Colabについて

::content::

<div class="remark">

以降はGoogle Colabの説明なのでPythonの実行環境が整っている方は聞かなくても構いません.

</div>

<v-clicks>

- 正式名称は**Google Colaboratory** (略称: Colab).
- Googleが提供する**Jupyter Notebook**のクラウド上での実行環境です.
  - Jupyter Notebook: 端的に言うと, プログラミング実行環境と文書をまとめたもの.
    - コードと文章を同じファイルに書けるので, 課題の提出に適している.
    - 拡張子は`.ipynb`
  - 三つの言語(**Ju**lia, **Pyt**hon, **R**)に対応している.
  - ブラウザでGoogle Colabのページにアクセスするだけで利用できる.

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::

# Google Colabの使い方

::content::

アクセスすると以下のような画面が表示される:

<div class="image-container" style="width: 80%; margin: auto;">
  <img src="./images/colab1.png" alt="Google Colabの画面" />
</div>

---
layout: top-title
color: amber-light
---

::title::
# Google Colabの使い方
::content::

右下の「キャンセル」をクリックするとポップアップ画面が消え, 以下のような画面になる:
<div class="image-container" style="width: 80%; margin: auto;">
  <img src="./images/colab2.png" alt="Google Colabの画面" />
</div>

---
layout: top-title
color: amber-light
---
::title::
# Google Colabの使い方
::content::

左上の「ファイル → ノートブックをアップロード」でダウンロードしたipynbファイルをアップロードして開ける:
<div class="image-container" style="width: 80%; margin: auto;">
  <img src="./images/colab3.png" alt="Google Colabの画面" />
</div>

---
layout: top-title
color: amber-light
---
::title::
# Google Colabの使い方
::content::
ダウンロードした課題ファイルを開くと, 以下のような画面になる:
<div class="image-container" style="width: 60%; margin: auto;">
  <img src="./images/colab4.png" alt="Google Colabの画面" />
</div>

コードが書いてある欄(セルと呼ばれる)をクリックして, その中にコードを書いていく.
セル内のコードの実行は, セルを選択してから上部の「▶︎」ボタンをクリック.

<div class="image-container" style="width: 60%; margin: auto;">
  <img src="./images/colab5.png" alt="Google Colabの画面" />
</div>

---
layout: top-title
color: amber-light
---
::title::
# Google Colabの使い方
::content::
書き終わったら, 上部の「ファイル → ダウンロード → .ipynb」を選択して, ipynbファイルをダウンロードし, これをLMSを通じて提出すればよい.
<div class="image-container" style="width: 60%; margin: auto;">
  <img src="./images/colab3.png" alt="Google Colabの画面" />
</div>