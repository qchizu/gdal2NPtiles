# gdal2NPtiles

Modified gdal2tiles.py script capable of producing Numerical PNG Tiles.

gdal2tiles.pyを拡張して、数値PNGタイルを生成できるようにしたものです。

本レポジトリは、数値PNGタイルの普及を目的に公開するものです。

# 数値PNGタイルについて

数値PNGタイルは、標高値を始めとした各種データをWebブラウザで使用するためのフォーマットで、産業技術総合研究所地質調査総合センター（GSJ）が考案したものです。

数値PNGタイルの画素のRGB値（R, G, B = 0～255）から数値hを取得する方法

x = 2<sup>16 </sup>R + 2<sup>8</sup>G + B

数値の分解能をuとして:

- x < 2<sup>23</sup>の場合: h = xu
- x = 2<sup>23</sup>の場合: h = NA
- x > 2<sup>23</sup>の場合: h = (x-2<sup>24</sup>)u

※無効値は (R, G, B) = (128, 0, 0)。

標高値を記録する場合、標高分解能u = 0.01mで、-83,886.07mから+83,886.07mまでの範囲を表すことができ、エベレスト（8,849m）やマリアナ海溝チャレンジャー海淵（-10,920m）などの標高を十分に表現できます。

詳細な情報は以下をご覧ください。

https://www.jstage.jst.go.jp/article/geoinformatics/26/4/26_155/_article/-char/ja

## 主な機能

Float32のTIFファイル（VRTファイル含む）から高品質な数値PNGタイル(RGBA/RGB)を生成することができます。

デフォルトではRGBAフォーマットのタイルを生成し、無効値を(128,0,0,0)として作成します。

## 使用環境

GDALが使える環境が必要です。

また、numpy、struct、zoomが必要です。

## 使用方法

基本的な使用方法は以下のとおりです。

python gdal2nptiles.py --numerical input_dem.tif output_folder

### 主なオプション：

* --numerical: 数値PNGタイルの生成を有効にする
* --numerical-resolution: 数値解像度を設定（デフォルト: 0.01）
* --numerical-base-tile-resampling: ベースタイルのリサンプリング方法（デフォルト: bilinear）
* --numerical-overview-tile-resampling: オーバービュータイルのリサンプリング方法（デフォルト: average）
* --numerical-rgb-only: RGBAの代わりにRGBタイルを出力（デフォルト: RGBA出力）

## 動作の仕組みとgdal2tiles.pyのコードからの主な変更点

数値→リサンプリング→RGBへの変換を行ってbase tileを作成します。

その後、RGB→数値→リサンプリング→RGBへ変換を行ってoverview tileを作成します。

（RGBのままリサンプリングを行うと、正しくリサンプリングできないため、数値の状態でリサンプリングを行うようにしています。）

また、EPSG:3857への投影変換（reproject_dataset）の際に、作成するタイルの解像度に合わせてリサンプリングを行っています。

（当初は、もとのコードと同様、create_base_tile関数内でリサンプリングを行うようにしていましたが、タイルの品質面で問題があったため、reproject_datasetでリサンプリングまで終わらせるようにしました。）

### GDAL2Tilesクラス:

open_inputメソッド: 数値PNGタイル用の入力チェックとオプション設定を追加

generate_base_tilesメソッド: 数値タイル用の設定を追加

### worker_tile_details関数:

数値PNGタイル用のオプション設定を追加

### create_base_tile関数:

数値データの読み込みと処理を追加

numerical_to_rgb関数を使用してRGB変換を実装

### create_overview_tile関数:

数値タイル用の処理を追加

rgb_to_numerical関数を使用して数値データに戻す処理を実装

### reproject_dataset関数:

base tileの解像度にあわせてリサンプリングする機能を追加

### 新規関数:

numerical_to_rgb: 数値データをRGB形式に変換(create_base_tile、create_overview_tileで使用)

rgb_to_numerical: RGB形式のデータを数値データに変換(create_overview_tileで使用)

## 生成されるタイルの検証

以下を参照してください。

なお、gdal2tilesには多数のオプションが用意されていますが、以下のファイル記載以外の動作の検証は行えていません。

[gdal2NPtiles.pyの検証.pdf](./gdal2NPtiles.pyの検証.pdf)
