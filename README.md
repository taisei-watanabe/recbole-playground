# RecBole survey

## このリポジトリの目的

このリポジトリの目的はリコメンデーションタスクに関連する研究開発を支援するライブラリである [RecBole](https://recbole.io/docs/index.html) の利用方法について調査または検証することです。

## 確認済み動作環境

以下の環境で動作を確認しております。

- MacOS 11.3
- Python 3.9.9

## 環境構築

以下のコマンドを実行して依存パッケージをインストールしてください。

```shell
pip install -r requirements.txt
```

## データセットの置き場所

`dataset/example` に以下のファイルが格納されています。

- `example.user`
  - ユーザーID とそれに対応する特徴量を記述します。
- `example.item`
  - アイテムID とそれに対応する特徴量を記述します。
- `example.inter`
  - インタラクションが起こったユーザー ID とアイテム ID とそれに対応する特徴量を記述します。

RecBole ではこれらのデータセットを [Atomic Files](https://recbole.io/docs/user_guide/data/atomic_files.html) というフォーマットで記述します。
特徴量の間のセパレーターは**タブ区切り**であることに注意してください。以下は `example.user` の中身を示しています。例えば `user_id:token` と `feature1:token` の間は　`user_id:token\tfeature1:token\t` となっています。数値も同様です。

```
user_id:token	feature1:token	feature2:token
1	286	130
2	491	3
3	342	32
```

## xDeepFM の訓練評価ループの実行

用途に合わせて以下のコマンドを実行してください。リコメンデーションモデルの1種である [xDeepFM](https://recbole.io/docs/user_guide/model/context/xdeepfm.html#xdeepfm) の訓練が実行されます。

### RecBole から提供されているデータセットとモデルを使った訓練

取り急ぎ RecBole から提供されているデータセットとモデルで実行したい場合には、以下のコマンドを実行してください。
バッチサイズや学習率などの訓練評価全体に関わるような設定は `configs/basic.yaml` から変更できます。
コンフィグファイルの書式についての詳細は [Config Introduction](https://recbole.io/docs/user_guide/config_settings.html) をご参考ください。

```shell
python run.py --dataset_name example --config_file configs/basic.yaml
```

### ご自身で用意したデータセットやモデルを使った訓練

ご自身で作成したデータセットやモデルを使って訓練したい場合には、こちらを基にスクリプトを変更するとよいでしょう。
以下のコマンドを実行することで xDeepFM の訓練評価ループが開始します。
xDeepFM のハイパーパラメータは `configs/models/xdeepfm.yaml` から変更できます。

```shell
python run_manually.py
```

### ハイパーパラメータの自動調整を含めた訓練

ハイパーパラメータの自動調整をしたい場合はこちらを基にスクリプトを変更するとよいでしょう。
以下のコマンドを実行することで xDeepFM の訓練評価ループをハイパーパラメータの調整を含めて実行することができます。
ハイパーパラメータの探索範囲は `configs/hyperparams/xdeepfm.hyper` から変更できます。
xDeepFM の設定可能なハイパーパラメータは [Tuning Hyper Parameters](https://recbole.io/docs/user_guide/model/context/xdeepfm.html#tuning-hyper-parameters) をご参考ください。

```shell
python run_with_hparam_tuning.py
```

ハイパーパラメータの探索範囲は `configs/hyperparams/xdeepfm.hyper` のように設定します。
.hyper ファイルの記法は [Parameter Tuning](https://recbole.io/docs/v1.0.0/user_guide/usage/parameter_tuning.html) を参考ください。

NOTE: `HyperTuning` を使うと最初のトライアルで生成されたログファイルにすべてのトライアルのログが記録されます。その他のトライアルで生成されたログファイルには何も書き込まれず、空のファイルが生成されることに注意してください。

## 新たな特徴量を追加する

以下のように `.user`, `.item`, `.inter` に新たな特徴量を追加することができます。

### .user と .inter にそれぞれ特徴量を追加する例

以下では `example.user` と `example.inter` に新たな特徴量を追記する例を示します。

1. `dataset/example/example.user` に `new_feature` を追記してください。

     NOTE: [Atomic Files](https://recbole.io/docs/user_guide/data/atomic_files.html) 形式で記述されていることに注意してください。

     ```
     user_id:token	feature1:token	feature2:token	new_feture:token
     1	286	130	987
     2	491	3	876
     3	342	32	765
     ```

2. `dataset/example/example.inter` に `weather_id`, `precipitation` を追記してください。

     ```
     user_id:token	item_id:token	timestamp:float	weather_id:token	precipitation:float
     1	1	1630461974	1	1
     2	2	1630462246	2	2
     3	2	1630462432	3	1
     1	2	1630462532	1	5
     1	3	1630462632	2	10
     2	1	1630462732	3	3
     2	2	1630462832	1	8
     2	3	1630462932	2	7
     3	1	1630463032	3	3
     3	2	1630463132	1	6
     3	3	1630463232	2	9
     ```

3. これに合わせて `configs.basic.yaml` の `load_col` を以下のように修正してください。

     ```yaml
     load_col:
          inter: [user_id, item_id, timestamp, weather_id, precipitation]
          user: [user_id, feature1, feature2, new_feature]
          item: [item_id, item_name, item_category_id]
     ```
