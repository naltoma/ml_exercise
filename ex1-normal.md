# 1章：入力1次元・連続値出力に対するフィッティング（回帰）
## 目次
- 1章：入力1次元・連続値出力に対するフィッティング（回帰）
  - データセットの構築（dataset1.py）
    - <a href="#ex1.1">演習1.1：真の関数の準備</a>
    - <a href="#ex1.2">演習1.2：観測点と真値の準備</a>
    - <a href="#ex1.3">演習1.3：ノイズを付与した観測値の準備</a>
    - <a href="#ex1.4">演習1.4：データセットのファイル出力</a>
    - <a href="#ex1.5">演習1.5：データセットのファイル読み込み</a>
    - <a href="#ex1.6">演習1.6：演習1.1〜演習1.5をモジュールとして整理</a>
  - モデルへのフィッティング（learn1.py）
    - <a href="#ex1.7">演習1.7：演習1.6で作成したモジュールによりデータセットを読み込む</a>
    - <a href="#ex1.8">演習1.8：線形回帰モデルの準備</a>
    - <a href="#ex1.9">演習1.9：フィッティング</a>
    - <a href="#ex1.10">演習1.10：テスト用データセットに対する予測</a>
    - <a href="#ex1.11">演習1.11：モデルの可視化</a>
    - <a href="#ex1.12">演習1.12：質の評価</a>
    - <a href="#ex1.13">演習1.13：モデルの保存</a>
    - <a href="#ex1.14">演習1.14：モデルのリカバリ</a>
    - <a href="#ex1.15">演習1.15：演習1.7〜1.14をモジュールとして整理</a>

<hr>

## データセットの構築（dataset1.py）
- <a name="ex1.1">演習1.1：真の関数の準備</a>
  - 背景
    - 機械学習とは、対象となる事象の本質や仕組みを表現する数式もしくは計算方法を獲得する手段のことを指す。例えば、``y = 2x + 1`` という数式を、xを入力とした場合にyを出力する数式は、一つの事象を表しており、x=1のときはy=3、x=-3のときはy=-5という出力を求めることができる。端的に述べると、機械学習とは、少ない観測値（データセット）から、事象を表すパラメータ付き数式におけるパラメータを推定することといえる。
    - 対象となる事象のことを「真の関数」と呼ぶことにしよう。「真の」という形容詞を付加した理由は、本来ならば知ることができない関数だからである。例えば台風の進路であったり、人口の流動であったり、様々な事象が世の中には存在し、可能な限り観測しようとする手段もあるが、後述するバイアスにより真の値を知ることは困難である。それでもここでは演習ということで真の値を出力できるような関数を自前で用意しよう。
  - 課題内容
    - 真の関数として ``y = sin(pi * x * 0.8) * 10`` の結果を返す関数を定義せよ。関数名は ``true_function`` とする。
    - 関数への入出力においてNumpyのブロードキャストを利用できるようにすること。すなわち、入力xや出力yは np.array型を利用しよう。
      - 参考
        - Numpy公式: [Broadcasting](https://numpy.org/doc/stable/user/basics.broadcasting.html)
        - [NumPy♪ブロードキャストを雰囲気で理解していませんか？](https://snowtree-injune.com/2020/06/14/broadcast-z007/)
    - x=0のときy=0であることを確認せよ。
    - plotlyにより、定義域``-1 <= x <= 1`` における事象を線グラフにより描け。なお、凡例も掲載すること。

<hr>

- <a name="ex1.2">演習1.2：観測点と真値の準備</a>
  - 課題内容
    - 観測点（x座標）を乱数により設定しよう。ここでは、定義域``-1 <= x <= 1``の範囲内でシード値（デフォルト値0）を指定した上で、ランダムにn個（デフォルト値20）の観測点を用意せよ。
    - 次に、各観測点に対応する真の値を、真の関数により求めよ。
    - 上記2手順により、観測点とそれに対応した真値が揃ったはずだ。このサンプル集合を pandas.DataFrame型（20行2列）として設定せよ。列の順番は問わないが、各々「観測点」「真値」として列名を設定せよ。
    - 演習1.1の線グラフ上に、サンプル集合をプロットせよ。

<hr>

- <a name="ex1.3">演習1.3：ノイズを付与した観測値の準備</a>
  - 背景
    - 真の関数は本来は知ることができない。用意できるデータセットには、<a href="https://developers.google.com/machine-learning/crash-course/fairness/video-lecture">reporting bias, selection biasなど</a>様々なバイアスが混入したサンプルが含まれる。このような「バイアスが混入したサンプル」を作るため、真の関数にノイズを付与してみよう。
  - 課題内容
    - サンプルに対応する真の値は true_function により得られる。その値に付与するノイズを[正規分布（ガウス分布）](https://ja.wikipedia.org/wiki/正規分布)により用意しよう。20個のサンプルに対して、平均値を0.0、分散を2.0としたときの正規分布に従うノイズ（ホワイトノイズ）を求め、それをさらに半分にした値をノイズとして付与せよ。付与した値を、演習1.2で用意したDataFrameにおける新たな列として挿入せよ。列名を「観測値」とする。
    - 演習1.2のグラフ内に、観測値をプロットせよ。

<hr>

- <a name="ex1.4">演習1.4：線形回帰モデルの準備</a>
  - 背景
    - 演習1.2で観測できない真の関数を用意し、演習1.3でノイズを付与する形でデータセットを構築した。機械学習の目指すゴールは、ノイズを含む少数のサンプルからなるデータセットから、本来は知り得るととができない真の関数を推測することである。「少数」とは、本来の事象である母集団と比べると相対的に少ないということを指した表現である。例えば新型コロナに罹患した全患者について把握するというようなことは、見落とし・人的金銭的限界・誤診等の様々な要因が影響するため現実的ではない。新型コロナに限らず一般的に母集団を直接対象とすることはそもそも困難であり、何らかの方法で観測して収集した誤りを含む少数サンプルから母集団の特徴を推定するように取り組むこととなる。
    - 今回は真の関数を[Linear Models](https://scikit-learn.org/stable/modules/linear_model.html)の線形回帰モデル（LinearRegression）により推測しよう。基本的な使い方は殆どのモデルで共通しており、(1)モジュールを読み込む、(2)モデルを用意する、(3)モデルに対してデータセットをフィッティングさせる、という3つの手順により適用することができる。
  - 課題内容
    - [Linear Models](https://scikit-learn.org/stable/modules/linear_model.html)を参考に、線形回帰モデル（LinearRegression）のモデルを用意せよ。
  - Tips
    - この時点ではモデルを用意するだけで良い。学習は次の演習で行う。

<hr>

- <a name="ex1.5">演習1.5：フィッティング（学習）</a>
  - 背景
    - フィッティングするだけならデータセット全てをモデルに与えても構わないが、一般的には「モデルの質」を評価するためにデータセットを学習用とテスト用に分けることが多い。この場合、学習用データセットをモデルに与えて予測可能なモデル（predictive model）を構築し、構築したモデルに対してテスト用データセットの入力（観測値）だけを与えた場合の出力（予測結果）を獲得する。この予測結果が、テスト用データセットにおける観測値とどれだけ近いかを判定材料とすることで、構築したモデルの質を評価する。
    - ここで構築前のモデルとは、パラメータを持つ数式により入出力関係を定義しているもの、例えば``y = ax + b`` のように、入力xを与えられるとパラメータa,bにより出力yを求めることができるようなものを想像しよう。このパラメータa,bが未定義の状態ではまだ出力yを求めることができない。
    - 演習1.2〜1.3で用意したデータセットは「ある入力xが与えられたときにはyと出力して欲しい」というサンプルの集合で構成されている。このデータセットをうまく表現するためのパラメータa,bを推測することが機械学習における「学習」であり、フィッティングとも呼ばれている。
  - 課題内容
    - [tarin_test_split](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html)を参考に、演習1.3で用意したデータセットに対し、8割のサンプルを学習用として用い、残り2割をテスト用に分割せよ。ただしデータセット分割時にはシャッフルすること。
    - 学習データに線形回帰モデルを適用（フィッティング）せよ。

<hr>

- <a name="ex1.6">演習1.6：テスト用データセットに対する予測</a>
  - 背景
    - 学習用データセットから構築したモデルは、真の関数をどれだけ適切に表現できているだろうか。未知のデータ（テスト用データセット）に対する予測結果から確認してみよう。
  - 課題内容
    - 構築したモデルに対し、テスト用データセット（残り2割）の入力データを与えた際の結果（＝予測結果）を出力せよ。

- <a name="ex1.7">演習1.7：モデルの可視化</a>
  - 背景
    - 構築したモデルは、真の関数をどれだけ適切に表現できているだろうか。今回は入力1次元、出力1次元のため、入力をx軸、出力をy軸としたxy座標空間における線グラフとして描くことで観察してみよう。
  - 課題内容
    - 演習1.5で構築したモデルを定義域``-1 <= x <= 1``にて線グラフとして描け。なお、凡例も掲載すること。
    - 描画したグラフを観察し、構築したモデルが真の関数をどれだけ適切に表現できているかについて、考察せよ。（自由記述）

<hr>

- <a name="ex1.8">演習1.8：質の評価</a>
  - 背景
    - モデルの可視化（グラフ描画）による評価は観察しやすい部分もあるが、その一方で、主観的な観点が強すぎるため客観性に欠けている。より客観性の高い評価方法として、観測値とのズレを絶対誤差により求め、1サンプルあたりの[平均絶対誤差（MAE, Mean absolute error）](https://scikit-learn.org/stable/modules/model_evaluation.html#mean-absolute-error)を求めてみよう。
  - 課題内容
    - 2割のテスト用データセットにおける観測値とその予測結果からMEAを求めよ。
    - MAEの値はどのように解釈したら良いだろうか？　例えばMAE=10.0とはどのような意味があるだろうか？　解釈について考察せよ。（自由記述）

<hr>

- <a name="ex1.9">（おまけ）演習1.9：モデルの保存</a>
  - 背景
    - 今回はデータセットが小さく、モデルも簡易なために学習に要するコスト（計算機資源、時間）もほぼ無視できる。これに対しコストが無視できない場合、例えば学習に1ヶ月時間がかかるといった場合には、何か調整する都度1ヶ月待たないと結果が得られず、また、プログラム終了時には学習結果が失われてしまうため、後日再利用したくても同じだけのコストを毎回支払う必要が出てくる。
    - このような手間を避けるため、一般的には学習済みモデルを何らかの方法で保存しておき、再利用する際にはそこから読み込むことで対応することが多い。
  - 課題内容
    - [Persistence example](https://scikit-learn.org/stable/modules/model_persistence.html?highlight=pickle)を参考に、学習済みモデルをpickleを用いてファイルに保存せよ。ファイル名は model.pkl とする。
  - Tips
    - [pickle](https://docs.python.org/ja/3/library/pickle.html)自体はPython公式モジュールであり、ほとんどのオブジェクトを直接化（シリアライズ化、pickle化）したり、そこから復元することが可能。
    - 多くの場合、問題にならないはずですが、[pickleには4GB制限](https://stackoverflow.com/questions/31468117/python-3-can-pickle-handle-byte-objects-larger-than-4gb)があるようです。

<hr>

- <a name="ex1.10">（おまけ）演習1.10：モデルのリカバリ</a>
  - 背景
    - Web公開された学習済みモデル（ここでは前述のmodel.pkl）を利用する状況を想定しよう。利用者は、どのようなデータセットを準備し、どのようにフィッティングさせたのかは分からないが、公開された学習済みモデルを利用することができないと困るはずだ。
  - 課題内容
    - [Persistence example](https://scikit-learn.org/stable/modules/model_persistence.html?highlight=pickle)を参考に、model.pklを読み込んで学習済みモデルを復元せよ。
    - 復元した学習済みモデルを用いて、演習1.6〜演習1.8を試行し、再現できることを確認せよ。
