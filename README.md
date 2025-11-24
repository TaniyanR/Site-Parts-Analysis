## 🧾 新しい共通パーツ用 README（コピーしてOK）

````md
# THK Analytics Core（ヘッドレスアクセス解析エンジン）

このプロジェクトは、THK Analytics 1.2.4 をベースにして、

- ログイン機能
- 管理画面（グラフなどのUI）
- サイトの枠・レイアウト・テーマ
- 画面上のクレジット表示

をすべて省き、**純粋なアクセス解析エンジン部分だけ**を取り出した
「ヘッドレスアクセス解析ライブラリ」です。

## 🎯 目的

- PHPで作った別のツール・サイトから
  - JavaScriptタグ
  - PHP関数呼び出し  
  で共通のアクセスログを記録できるようにする
- 管理画面やログイン認証は持たず、  
  **「ログを受け取ってデータベースに保存する」ことだけに特化** する
- 後から好きな UI / ダッシュボードを自作できるようにする

> ⚠️ 注意（ライセンス）
> - ソースコード内の著作権表記・ライセンス表記（ヘッダコメント）は **GPL的に残す必要があります**。
> - 画面に表示されるロゴやフッターのクレジットは、この Core ではそもそも UI を持たないため出ません。

---

## 🏗 構成（ファイル/ディレクトリ）

Core 版では、THK Analytics 本体から以下の部分だけを利用します。

```text
_core/                         … 解析エンジン本体
  system/ThkInclude.php        … 各種パス定義・共通読み込み
  application/config/          … 各種設定・一覧
  application/libs/Track.php   … 解析処理の中枢（Trackクラス）
  application/models/Log.php   … ログ保存ロジック
  application/models/Site.php  … サイト情報
  （その他、Track/Logが必要とするヘルパー/モデル）

thk-core/                      … Web から叩く入口だけを残したディレクトリ
  initialize.php               … Coreの初期化
  script.php                   … JavaScriptコード配信
  track.php                    … JSからのアクセス記録
  phptrack.php                 … PHPからのアクセス記録（_thkTrack）
  setting/
    path.php                   … _core へのパス設定
    database.php               … DB接続情報
    siteurl.php                … サイトURL設定（必要に応じて）
````

元の THK Analytics にあった以下のものは **Core では使いません**：

* `_core/application/controllers/*`（Admin / Login / Research / Install など）
* `_core/application/views/*`
* `thk/view/*`
* `thk/index.php`（管理画面のエントリ）
* ログイン画面・管理画面向けの JS / CSS 各種

---

## ⚙ インストール / セットアップ

### 1. データベース作成

THK Analytics 用の MySQL データベースを作成し、
元パッケージ付属の SQL（テーブル作成スクリプト）を流します。

* THK Analytics オリジナルのテーブル構成（`log_*` など）をそのまま利用します。
* 文字コードは `utf8` / `utf8mb4` を推奨します。

### 2. ファイルを配置

1. `_core/` ディレクトリをサーバーにアップロードします。
2. `thk-core/` ディレクトリとして、以下のファイルを置きます（元の `thk/` から必要なものだけコピー）:

   * `initialize.php`
   * `script.php`
   * `track.php`
   * `phptrack.php`
   * `setting/` ディレクトリとその中の `path.php`, `database.php`, `siteurl.php` など

> ※ 元の `thk/` ディレクトリから **管理画面用の `view/` や `index.php` などはコピーしません。**
> 解析の入口だけを残します。

### 3. 設定ファイルの調整

`thk-core/setting/` 配下のファイルを編集します。

* `path.php`
  `_core/` ディレクトリまでのパスを設定します（元の `CORE_PATH` と同じ考え方）。

* `database.php`
  データベースの接続情報（ホスト名 / DB名 / ユーザー / パスワードなど）を設定します。

* `siteurl.php`（必要な場合）
  サイトのベースURLを設定します。

---

## 📡 トラッキングの方法

THK Analytics Core は、**2種類の方法でアクセスを記録**できます。

### 1. JavaScript タグによる解析

JavaScript を使ったリッチな解析（クリック・画面サイズなど）を行う方法です。

#### 1-1. JSコードの読み込み

解析したいページの `<head>` または `<body>` 終了前に、次のようなコードを追加します。

```html
<script src="https://your-domain.example.com/thk-core/script.php"></script>
```

※ `script.php` は、元の THK Analytics と同様に、必要な JavaScript を動的に生成します。
　JS側で `track.php` に対してアクセス情報を送信します。

#### 1-2. 取得できる情報（オリジナル準拠）

* ページURL
* リファラー（リンク元）
* ホスト名 / IP アドレス
* OS / ブラウザ / デバイス種別
* JavaScript / Cookie の ON/OFF
* 画面解像度 / 色数
* クリック / ボタンクリック / 各種イベント
  （元の THK Analytics の `Track` クラスが持っている機能に準拠）

### 2. PHP 関数呼び出しによる解析（JavaScriptなし）

PHP から直接 `_thkTrack()` を呼び出すことで、
JavaScript を使わずにアクセスを記録する方法です。

#### 2-1. 共通読み込み

解析したい PHP ファイル内で、`phptrack.php` を読み込みます。

```php
<?php
require_once '/path/to/thk-core/phptrack.php';
```

#### 2-2. ページ表示時に記録する

ページタイトル等を渡してアクセスを記録します。

```php
<?php
require_once '/path/to/thk-core/phptrack.php';

$title = 'トップページ';
_thkTrack($title);
```

### 2-3. WordPress テンプレートなどでの例

```php
<?php
require_once get_template_directory() . '/thk-core/phptrack.php';

// タイトルを WordPress から取得して渡す
_thkTrack( wp_title('-', false, 'right') );
```

> ⚠️ PHPトラッキングのみを使う場合は、JavaScriptタグ（`script.php`）との **併用はしない** 想定です（オリジナルの仕様準拠）。

---

## 🔩 内部構造（どのコードが解析しているか）

Core 版で主に使うクラスは以下の通りです。

* `_core/application/libs/Track.php`

  * `Track::doScript()` … `script.php` から呼び出され、クライアント側 JS を出力
  * `Track::doTrack()` … `track.php` から呼び出され、アクセス情報を記録
  * `Track::doPhpTrack($title)` … `_thkTrack()` から呼び出され、PHPからのアクセスを記録

* `_core/application/models/Log.php`

  * 実際に各種テーブルへログを書き込む処理を担当

* `_core/system/ThkInclude.php`

  * 設定ファイル / モデル / ライブラリの読み込み
  * パス定義（`THK_CORE_DIR`, `THK_LIBS_DIR`, `THK_MODEL_DIR` など）

この Core では、**ログインや管理画面に関するコントローラ・ビューには一切触れません。**
今後必要に応じて、別プロジェクトとして可視化用UIを作成できます。

---

## 🧪 他のPHPツールからの使い方イメージ

あなたの側の PHP ツールでは、次のように使います。

### 例1：単純なPHPアプリに JS 解析を入れる

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>サンプルアプリ</title>
  <script src="https://analytics.example.com/thk-core/script.php"></script>
</head>
<body>
  ...
</body>
</html>
```

### 例2：PHPから直接トラッキング（JSなし）

```php
<?php
require_once '/var/www/analytics/thk-core/phptrack.php';

_thkTrack('マイツールのダッシュボード画面');
```

---

## 🚧 今後の拡張のためのメモ

* **この Core プロジェクトは「記録専用」**です。
* 集計・グラフ表示・管理者ログインなどは別プロジェクトとして実装してください。
* ログテーブルに対しては、任意の BI ツール / 自作ダッシュボード / SQL などで参照できます。

将来的なアイデア：

* Laravel / Slim 等で別途「閲覧専用API + フロントエンド」を作る
* BigQueryや別DBへのエクスポートバッチを追加する
* event ベースでのトラッキングAPI（`trackEvent()`相当）を薄くラップする

---

````

---

## 🧠 Copilot に作業させるときの一言テンプレ

GitHub上で作業するときは、例えばこんな感じで Copilot に指示すればOKです（そのままコピペで使えます）👇

```text
このリポジトリに入っている THK Analytics 1.2.4 から、
ログイン機能や管理画面、ビューなどを取り除いて、
純粋なアクセス解析エンジン部分だけを「thk-core」というディレクトリに整理したいです。

やりたいこと：
- _core/application/libs/Track.php と、Trackが依存しているモデル/ヘルパー/設定だけを使う
- thk/initialize.php, thk/track.php, thk/script.php, thk/phptrack.php と setting/ を
  新しい thk-core/ ディレクトリに移し、パス等を調整する
- _core/application/controllers や _core/application/views、thk/view など
  ログイン・管理画面関連のコードには依存しない構成にする

この README.md に書かれている方針を前提に、
必要なファイルをコピー・修正して「THK Analytics Core」として動くようにしてください。
````


みたいな拡張もしたくなったら、それ前提の v2 用 README も一緒に作ります！
