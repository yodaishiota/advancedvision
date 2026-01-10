# advancedvison
千葉工業大学大学院未来ロボティクス専攻 2025年度 アドバンスドビジョン課題用リポジトリ．

## 概要
本モデルは28×28のグレースケール画像を入力とし，中間層でわずか4次元まで情報を圧縮した後，元の画像を復元するオートエンコーダモデルである．

本モデルの性能評価としてMNISTデータセットを使用して10-epoch学習をした結果，784次元の情報を約1/200の4次元に圧縮しても，数字の概形を復元可能であることを確認した．

## ネットワーク構造
本モデルは全結合層Affine層のみで構成されるシンプルなオートエンコーダである．
Encoder部は入力画像(784次元)を段階的に削減して4次元の潜在変数に変換し，Decoder部はそれを元の784次元に復元する．
活性化関数には中間層にReLU，出力層にSigmoid関数を使用している．

以下に本モデルのネットワーク構造の数式を示す.

### 数式

#### 1. 入力データの定義
MNIST画像は $28 \times 28$ の2次元データだが，本モデルでは全結合層に入力するため，1次元の列ベクトル $\boldsymbol{x}$ に平坦化（Flatten）して扱う．

$$
\boldsymbol{x} \in \mathbb{R}^{784} \quad (784 = 28 \times 28)
$$

#### 2. Encoder（符号化器）
Encoderは4層の全結合層から構成され，次元数を $784 \to 128 \to 64 \to 12 \to 4$ と段階的に圧縮する．
第 $l$ 層の重み行列を $\boldsymbol{W}^{(l)}_E$，バイアスベクトルを $\boldsymbol{b}^{(l)}_E$ と定義する．

例えば，第1層（$784 \to 128$）の計算は，ReLU関数 $\phi(\cdot) = \max(0, \cdot)$ を用いて次のように表される．

$$
\boldsymbol{h}^{(1)} = \phi(\boldsymbol{W}^{(1)}_E \boldsymbol{x} + \boldsymbol{b}^{(1)}_E)
$$

各パラメータの次元数は以下の通り．
* $\boldsymbol{W}^{(1)}_E \in \mathbb{R}^{128 \times 784}$
* $\boldsymbol{b}^{(1)}_E \in \mathbb{R}^{128}$
* $\boldsymbol{h}^{(1)} \in \mathbb{R}^{128}$

同様の計算を繰り返し，最終的に **4次元** の潜在変数ベクトル $\boldsymbol{z} \in \mathbb{R}^{4}$ を得る．

#### 3. Decoder（復号化器）
DecoderはEncoderと逆の構造を持ち，潜在変数 $\boldsymbol{z}$ から元の次元数（今回は784）への復元を行う．
出力層以外の活性化関数にはReLUを使用するが，最終出力層では画素値を $[0, 1]$ の範囲に収めるため，**Sigmoid関数** $\sigma(\cdot)$ を使用する．

$$
\sigma(u) = \frac{1}{1 + e^{-u}}
$$

復元画像ベクトル $\boldsymbol{y}$ の計算は以下の通り．

$$
\boldsymbol{y} = \sigma(\boldsymbol{W}_{out} \boldsymbol{h}_{last} + \boldsymbol{b}_{out})
$$

#### 4. 損失関数（Loss Function）
学習には，**平均二乗誤差（Mean Squared Error: MSE）** を使用する．
入力 $\boldsymbol{x}$ の第 $i$ 成分（画素）を $x_i$，出力 $\boldsymbol{y}$ の第 $i$ 成分を $y_i$ とすると，損失 $\mathcal{L}$ は次式で定義される．

$$
\mathcal{L} = \frac{1}{N} \sum_{i=1}^{N} (x_i - y_i)^2
$$

## 実行方法
1. 以下のボタンから，Google Colab上でノートブックを直接開く．
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/yodaishiota/advancedvision/main.py)
2. ランタイムのタイプを「Python 3」に変更し，ハードウェアアクセラレータをGPUに変更する．(なお，使用できなければcpuのままでも問題ありません)
3. 上から順にセルを実行する.

## 動作環境
- 実行環境：Google Colab
- Python：3.12.12
- PyTorch：2.9.0+cpu
- torchvision
- Matplotlib
- NumPy

## 性能評価
本モデルの性能評価としてMNISTデータセットを使用し検証を行った．
以下に実際の学習推移と出力結果を示す．

### 学習推移

<p align="center">
  <img src="./README_ima/loss_curve.png" width="600">
</p>ews2d3

### 結果
以下の画像はテスト用データに対する出力結果である．
上段の画像はオリジナル画像，下段は出力結果の画像となっている．

<p align="center">
  <img src="./README_ima/result_images.png" width="800">
</p>

### 考察
画像を比較すると，全体的に軽くぼやけており，細かい筆跡は失われてしまっている．特に，「４」や「９」といった数字は区別がつきにくくなっている．しかし，「０」や「１」といった数字は大まかな形状は保持されていることが分かる．
これにより，手書き数字の主要な特徴量は4次元空間内に概ね埋め込み可能であることが分かる．

## 参考文献
本プログラムの実装に当たり，以下の文献を参考にしました．
- [Deep Insider (ITmedia): PyTorchでオートエンコーダーによる画像生成をしてみよう](https://atmarkit.itmedia.co.jp/ait/articles/2007/10/news024.html)
- [CUBE SUGAR CONTAINER: PyTorch で AutoEncoder を書いてみる](https://blog.amedama.jp/entry/pytorch-mnist-autoencoder)

### 講義資料
また上田教授の下記スライド群のlesson3を参考にさせていただきました．
    https://github.com/ryuichiueda/slides_marp/tree/master/advanced_vision

## ライセンス
- このソフトウェアパッケージは，3条項BSDライセンスの下，再頒布および使用が許可されます．
- © 2026 Yodai Shiota