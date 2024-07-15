# エンドツーエンドのビデオベースのアイトラッキングに向けて

ECCV 2020で発表された論文とデータセット、EVEに付随するコードです。

* 著者: [Seonwook Park](https://ait.ethz.ch/people/spark/)、[Emre Aksan](https://ait.ethz.ch/people/eaksan/)、[Xucong Zhang](https://ait.ethz.ch/people/zhang/)、[Otmar Hilliges](https://ait.ethz.ch/people/hilliges/)
* プロジェクトページ: https://ait.ethz.ch/eve
* Codalab（テストセット評価とパブリックリーダーボード）: https://competitions.codalab.org/competitions/28954


## セットアップ

このリポジトリのセットアップには、Dockerイメージや仮想環境（[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/install.html)推奨）を使用することを推奨します。このコードベースは以下の環境でテストされています：
* Ubuntu 18.04 / Linuxベースのクラスターシステム（CentOS 7.8）
* Python 3.6 / Python 3.7
* PyTorch 1.5.1

リポジトリをクローンし、以下のコマンドを実行してください：

    git clone git@github.com:swook/EVE
    cd EVE/

その後、リポジトリのベースディレクトリから、以下のコマンドで全ての依存関係をインストールしてください：

    pip install -r requirements.txt

`torch`および`torchvision`パッケージのセットアップについては、[PyTorchの公式インストールガイド](https://pytorch.org/get-started/locally/)を参照してください。

また、ビデオデコードのために**ffmpeg**をセットアップする必要があります。Linuxでは、ディストリビューション固有のパッケージ（通常は`ffmpeg`）をインストールすることを推奨します。必要に応じて、[公式ダウンロードページ](https://ffmpeg.org/download.html)や[コンパイル手順](https://trac.ffmpeg.org/wiki/CompilationGuide)を参照してください。


## 使用方法

### コードフレームワークに関する情報

#### 設定ファイルシステム

全ての利用可能な設定パラメータは、`src/core/config_default.py`に定義されています。

デフォルト値を上書きするには、以下の方法があります：

1. `train.py`または`inference.py`にコマンドラインパラメータとしてパラメータを渡す。この場合、全ての`_`文字を`-`に置き換えてください。例えば、設定パラメータ`refine_net_enabled`は`--refine-net-enabled 1`となります。ブールパラメータは`0/no/false`または`1/yes/true`のいずれかで渡すことができます。
2. `src/configs/eye_net.json`や`src/configs/refine_net.json`のようなJSONファイルを作成する。

適用の順序は次の通りです：
1. デフォルトパラメータ
2. JSONファイルで提供されるパラメータ。JSONファイルの宣言順に適用されます。例えば、コマンド`python train.py config1.json config2.json`では、重複するエントリがあれば`config2.json`が`config1.json`を上書きします。
3. CLIで提供されるパラメータ。

#### Google Sheetsへの自動ログ記録

このフレームワークは、全てのパラメータ、損失項、およびメトリクスをGoogle Sheetsドキュメントに自動的に記録するコードを実装しています。これは`gspread`ライブラリによって行われます。この機能を有効にするには、以下の手順に従ってください：

1. https://gspread.readthedocs.io/en/latest/oauth2.html#for-end-users-using-oauth-client-id の手順に従ってください。
2. `--gsheet-secrets-json-file`に資格情報JSONファイルへのパスを、`--gsheet-workbook-key`にドキュメントキーを設定してください。このキーは`https://docs.google.com/spreadsheets/d/`の後、クエリやハッシュパラメータの前にある部分です。

サンプルの設定JSONファイルは、`src/configs/sample_gsheet.json`にあります。

### モデルのトレーニング

モデルをトレーニングするには、`src/`ディレクトリから`python train.py`を適切な設定変更で実行してください（「設定ファイルシステム」を参照）。

既存のモデルのトレーニングを再開するには、`--resume-from`引数で出力フォルダへのパスを指定する必要があります。

また、`train.py`の各実行時に一意の識別子が生成され、`outputs/EVE/`に一意の出力フォルダが生成されます。したがって、モデルの追跡にはGoogle Sheetsのログ記録機能（「Google Sheetsへの自動ログ記録」を参照）を使用することを推奨します。

### 推論の実行

`src/inference.py`のシングルサンプル推論スクリプトは、`train.py`と同じ引数を受け取りますが、特に以下の2つの引数を期待します：

* `--input-path` は、EVEデータセット内に存在する `basler.mp4`、`webcam_l.mp4`、`webcam_c.mp4`、または `webcam_r.mp4` へのパスです。
* `--output-path` は、望む出力場所へのパス（`.mp4`で終わる）です。

このスクリプトは、トレーニング、検証、およびテストサンプルの両方に対応しており、利用可能な場合は参照点の視線真実値を表示します。

## 引用
このコードベースおよび/またはEVEデータセットを研究に使用する場合は、以下の論文を引用してください：

    @inproceedings{Park2020ECCV,
      author    = {Seonwook Park and Emre Aksan and Xucong Zhang and Otmar Hilliges},
      title     = {Towards End-to-end Video-based Eye-Tracking},
      year      = {2020},
      booktitle = {European Conference on Computer Vision (ECCV)}
    }

## Q&A

**Q: このコードを画面ベースのアイトラッキングに使用するにはどうすれば良いですか？**

A: このコードは実際のアイトラッキングを提供するものではありません。むしろ、元の論文に記載されているビデオベースの視線推定方法のベンチマークを行います。このコードを画面ベースのアイトラッキングに対応する使いやすいソフトウェアに拡張するのは、カメラのキャリブレーション（内部パラメータ、外部パラメータ）や、正確かつ安定したリアルタイムの目または顔のパッチ抽出のための効率的なパイプラインの要件があるため、やや非自明です。したがって、このコードリポジトリの範囲外と考えています。

**Q: テストセットのラベルはどこにありますか？**

A: 公共評価サーバーとリーダーボードは、https://competitions.codalab.org/competitions/28954 にホストされています。これにより、テストセットの評価が一貫して信頼性があり、ビデオベースの視線推定分野での競争を促進します。Codalabによる評価のパフォーマンスは、元の論文の結果と厳密には比較できないことに注意してください。これは、フルテストセットの大部分でのみ評価を行うためです。更新されたパフォーマンスの数値は、[リーダーボード](https://competitions.codalab.org/competitions/28954#results)から取得することをお勧めします。