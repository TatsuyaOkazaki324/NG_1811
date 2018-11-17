# Eterm - Best Terminal for Education

[![Thumbnail](thumb.png)](https://vimeo.com/296239145)

## 製品概要

### 黒い画面 x Tech

### 背景（製品開発のきっかけ、課題等）
黒い画面（CUI）は非常に便利かつ、ITエンジニアには必須なものです。   
しかし、その操作は初めての人には敷居が高く、学習は容易ではありません。  
そこで、プログラミング環境の[Scratch](https://scratch.mit.edu)のようなインターフェースを持つターミナルがあればよいのではないかと考えました。

### 製品説明（具体的な製品の説明）

### 特長

#### 1. 独自のインターフェース
どこに何を入力すればよいのかが一目で分かる、グラフィカルなインターフェースを採用しています。
コマンドやオプションの選択を、キーボードだけでなくマウスでも行えます。
クロスプラットフォーム対応を想定し、Electronをベースにしています。

#### 2. コマンドの推薦アルゴリズム
木構造のデータを用いた推薦機能を搭載しています。
専用のデータは、[モデルフォーマット](model/README.md)に従い作成できます。

### 解決出来ること今回機械学習に使用したデータセットは、開発期間中に用意した単純なものであるため、より大きなデータセット（コマンド履歴）を用いたコマンド推薦を行いたい。  
CUIのハードルを下げ、学習を容易にする

### 今後の展望
リリースを目指し改善を進める。
また、利用するコマンドのデータセットを充実させる必要がある。

## 開発内容・開発技術
### 活用した技術
#### API・データ
ありません。

#### フレームワーク・ライブラリ・モジュール

クロスプラットフォーム対応も想定し、Electronをベースとしているため
* [Vue](https://github.com/vuejs/vue)
* [Electron](https://github.com/electron/electron)

#### デバイス
ありません。

### 研究内容・事前開発プロダクト（任意）
ありません。

### 独自開発技術（Hack Dayで開発したもの）
* コマンド推薦アルゴリズムおよびモデル
* 独自のインターフェースを持ったデスクトップアプリケーション

