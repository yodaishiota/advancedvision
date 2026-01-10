# advancedvison
千葉工業大学大学院未来ロボティクス専攻 2025年度 アドバンスドビジョン課題用リポジトリ．

## 概要
本モデルは28×28のグレースケール画像を入力とし，中間層でわずか4次元まで情報を圧縮した後，元の画像を復元するオートエンコーダモデルである．

本モデルの性能評価としてMNISTデータセットを使用して10-epoch学習をした結果，784次元の情報を約1/200の4次元に圧縮しても，数字の概形を復元可能であることを確認した．

## ネットワーク構造

### ブロック図
graph TD
    subgraph Input
    A[Input Image<br/>(Flatten)] -->|784 dim| B
    end

    subgraph Encoder
    B[Linear + ReLU] -->|128 dim| C[Linear + ReLU]
    C -->|64 dim| D[Linear + ReLU]
    D -->|12 dim| E[Linear]
    end

    subgraph Latent
    E -->|4 dim| F((Latent Variable<br/>z))
    end

    subgraph Decoder
    F -->|4 dim| G[Linear + ReLU]
    G -->|12 dim| H[Linear + ReLU]
    H -->|64 dim| I[Linear + ReLU]
    end

    subgraph Output
    I -->|128 dim| J[Linear + Sigmoid]
    J -->|784 dim| K[Output Image<br/>(Reshape)]
    end

    style F fill:#f9f,stroke:#333,stroke-width:4px

## ライセンス
- このソフトウェアパッケージは，3条項BSDライセンスの下，再頒布および使用が許可されます．
- © 2025 Yodai Shiota