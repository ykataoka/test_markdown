分析タスクを実施したNTT i3のレポジトリ。
Step2(2018.10- 2018.01) 及び Step1(2018.05 - 2018.09)を含む

## Anomaly Analytics Code (Step2 - Task1)
### anomaly_modules.py
 - 異常値スコアを求めるためのモジュール群
 - Influxから元データへのアクセス (要：DBのエンドポイントはパラメータ調整)
 - InfluxDBへの書き込み(要：DBのエンドポイントはパラメータ調整)
 - 特徴量の算出、代表ベクトル、対象ベクトルの生成
 - 追い越し/追い越され判定データの取得
 - 異常度スコアの算出、正規化や重み付けを実施 (要：データ列について、個別に重み付けの更新が必要)

### create_represent_feature.py
 -  ```python:create_represent_feature.py
    $ python create_represent_feature.py 'column_name' 'LapID1' 'LAPID2' ...
    $ (e.g. representation of cfrp) python create_represent_feature.py angleA 20181018_NBR_6 20181018_NBR_7 20181023_NBR_2 20181023_NBR_4 20181023_NBR_5 
    ```
 - 代表ベクトルを生成するためのスクリプト。
 - 分析したい項目（操舵角とか車速とか）、代表ベクトル生成の元ネタとなるLAP_ID（複数指定可）を入力とする。
 - LapIDの付与基準は、YYYYMMDD_NBR_N (YYYYMMDD: 走行日、N: その日の中での周回数)
 - (2018.10月の計測)LapIDはTanakaさんがまとめたexcel表を参照する。

### calculate_anomaly.py
 -  ```python:calculate_anomaly.py
    $ python calculate_anomaly.py 'column_name' 'LapID1' 'LAPID2'
    $ (e.g. anomaly of Cayman) python calculate_anomaly.py angleA 20181016_NBR_2 20181016_NBR_3 20181016_NBR_4 20181016_NBR_5 20181016_NBR_6
    ```
 - 対象ベクトルを生成し、予め作っておいた代表ベクトルとの比較を行った上でAnomaly値を各セクション毎に計算する。
 - 分析したい項目（操舵角とか車速とか）、対象ベクトル生成の元ネタとなるLAP_ID（複数指定可）を入力とする。
 - スクリプトを実行すると、インタラクティブにInfluxDBに登録する際に使うLAP indexを聞かれるのでYYYYMMDD_NBR_NNNN(YYYYMMDD: 任意日付、NNNN: 他と重複しないような任意の数字) を入力する。
 
### make_anomaly_plot.py
 -  ```python:make_anomaly_plot.py
    $ python make_anomaly_plot.py 'column_name' [--replap LapID1 [LapID2 ...]] [--tarlap LapID1 [LapID2 ...]]
    $ (e.g.) python make_anomaly_plot.py angleA --replap 20181018_NBR_6 20181018_NBR_7 --tarlap 20181016_NBR_2 20181016_NBR_3
    ```
 - Anomaly値のヒストグラムや代表ベクトルや対象ベクトルのプロットを作成するスクリプト。
 - Anomaly値を10階級に分け、それぞれの階級に属するSectionを３つランダムにチョイスし、一覧としてプロットする。
 - 他、高AnomalyなSectionについては個別にプロットを行う(しきい値はPLOT_THRESHにて指定可)。
 - また、オプションとしてLapIDを指定した場合、そのLap、そのSectionにおける実測値のプロットも行う。
 - 各グラフはpngファイルとして、スクリプト実行フォルダの1つ上に作成されるanomaly_figフォルダ配下に出力される。
   - ヒストグラム: histogram_of_Anomaly.png
   - ベクトル一覧プロット: Overview_of_anomaly.png
   - セクション個別プロット: Anomaly_N.NNNN_Section_id_NNN.png (N: 0-9)
     - LapIDを引数に与えた場合、ベクトル値グラフと実測値グラフの２つが表示される。
     - 与えなかった場合、ベクトル値グラフのみが表示される。

### norm_expo_dist.py
 - 正規化をするためのモジュール 及び 予備実験のためのコード

### comp_var_by_secID.py
 - cayman, cfrp, rx8の3シートを"平均操舵挙動"と"安定度"の2つの観点で比較。　
 -  ```python:comp_var_by_secID.py
    $ python comp_var_by_secID.py
     ```
 - 対象データはpickle化されており、追い越され/追い越し地点の除外した特徴量となっている。


## Data Analytics / Automoation Code (Step1)

### vbox_analytics.py
 - vboxデータをパースして、前処理
 - vboxデータはGPS付きとそうでないのがあるので、それぞれで対応

### acc_analytics.py
 - 加速度データをパースして、前処理

### point_analytics.py
 - モーションデータをパースして、前処理

### cross_correlation.py
 - 二つの計測データ間の⊿Tを相互相関で算出し、プロット
 - 現在、実験的にvboxとaccの二つのデータを、”速度”で合わせている。
 - 元データのそれぞれの差分データで同期。100[msec]ずれくらいにおさまる。目標は20[msec]だけど
 - 同期信号が入ればこの問題は軽くクリアできるので、あまり深入りはしない。

### gps_telemetry_motion_analytics.py
 - 操舵角を分析するためのパイプライン
 - 地点インデックスを付与

### data_merge_vbox_motion_acc.py
 - vbox, motion, accの三計測システムを自動でマージする。
 - 時刻合わせや地点インデックス付与も行う。

### data_merge_gps_motion.py
 - 簡易GPSとそのテレメトリー, motionの二計測システムを自動でマージする。
 - こちらは、step1の分析タスクのみで使用するため、時間合わせはハードコーディング。
 - 時刻合わせのデータは、田中さんからもらっており、時間合わせ.xlsxと時間合わせIRにある。
 - 実行時、第一引数に (name)_(seattype)を指定
 - (name): 'T', 'I', 'R'
 - (seattype): 'R', 'CO', 'M', 'CF'

### cross_correlation.py
 - 相互相関を用いて２つの信号差が最も少なくなる補正値を求める。同期信号がない場合に、計測機器間の時間ずれの補正量を求める。

### auto_process.sh
 ```sh:auto_process.sh
 $ ./auto_process.sh (please make sure the execution authority is activated by 'chmod')
 ```
- data_merge_gps_motion.pyの実行パターン全12パターンをすべてを実行
 - dumpされたデータを使って、解析を行う
 
### analytics_diff_sheet.py
 - step1の、シート違い分析、可視化を実行
 - 実行は、$ python data_merge_gps_motion.py (name)
 - data_merge_gps_motion.pyでdumpされたデータを用いる。

### analytics_driver_anomaly.py
 - step1において、速度や角度の異常地点を可視化。
 - 可視化を作り、世界観を伝えるためのコード。
 - アルゴリズムの改善は別途必要

## Experiment Code
### analyze_steering.py
 - ステアリング角を様々な方法で検証

### gmplot_test.py
 - google map上で可視化するテストコード

### plot_steering.py
 - モーションセンサからステアリング角を算出したデータのプロット

### gauss.py
 - 波形差を求める際に別のアルゴリズムを用いる。
 - 確率をベースにする手法のテストコード