---
theme: neversink
layout: cover
title: ガイダンス
author: 清水 伸高
mdc: true
routerMode: hash
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
2026年 10月6日
</div>


---
layout: top-title
color: amber-light
---

::title::

# 概要

::content::

- 内容: アルゴリズムの理論について
- 講義: 小テスト(30分) + 授業(70分)
- 演習: 毎週演習の開始時にレポート課題を配布

- **小テスト: 講義中に実施. 前回出題したレポートと同様の問題**
  - レポート課題だけだと生成AIで機械的に解けてしまうので、今年から導入

- 評価基準:
  - 小テスト 70%
  - 課題 30%


---
layout: top-title
color: amber-light
---

::title::

# 課題

::content::

- 演習の開始時に配布
  - 締切は次の週の**昼の13時** (日本標準時)
  - LMSで提出 (pdfで提出)
  - word, latex, ノート (markdown形式のノート、Obsidian, Notionなど)を強く推奨
  - 手書きでも良いが、**字が汚くて内容が分からない回答は採点されません** (昨年はかなり多かった)
    - 採点者側は書いてある内容を把握する努力はしない
  - 課題に関する質問やヒントなどの相談は **常時(講義以外の時間も)** 受け付けます
    - 手厚く対応しますので、積極的に尋ねてください
    - 締切の直前すぎると対応が間に合わないかもしれないので、留意してください

---
layout: top-title
color: amber-light
---

::title::

# 小テスト

::content::

- 講義の開始時に問題を配布し、30分後に回収
- 持ち込みやPCの利用はNG
- 問題自体は前回に出題した演習とほぼ同じ

---
layout: top-title
color: amber-light
---

::title::

# 受講に必要な環境

::content::

講義の受講にあたっては以下の環境が**必須**:

- 大学のSlackアカウント
  - 手続きについては[こちら](https://portal.isct.ac.jp/ja/sys/slack/guide.html#sign-in)を参照
  - 講義に関する私への連絡は**必ずSlackのDM**で行ってください
  - メールでの連絡は見逃す可能性が非常に高い
  
- LMSへのログイン
  - 課題の提出に利用する

---
layout: top-title
color: amber-light
---
::title::
# 生成AIの利用について
::content::

<div class="topic-box">

  **学習の補助として**の利用は推奨。しかし生成AIの出力を理解せずそのまま書くのはNG。

</div>

- 明らかに理解していない提出は、該当部分は減点
  - 例えば講義で用いていない専門用語を説明なしで使うなど
  - 自身の提出が減点されないか不安な箇所があれば、個人的に相談してください


---
layout: top-title
color: amber-light
---

::title::

# 講義日程

::content::

- 10月6日(初回)
- 11月10日は出張のため休講
- 11月24（火）が最終回


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
