# AWS Transit Gateway を AWS Management Console からインストールする手順

このドキュメントでは，Transit Gateway を AWS Management Console で構成する手順を説明する

## 初期構成

下記のコマンドを実行すると，表記の環境が deploy される．

```console
cd ./infra
npx npm i
npx cdk deloy
```

- VPC1
  - EC2: web サーバへの接続検証用
- VPC2
  - EC2: 検証用の web サーバ (nginx が可動)

## Transit Gateway の構成手順

1. Transit Gateway の作成
   VPC を接続するために，Transit Gateway を作成する．
   1. 作成画面に移動する
      [AWS Management Console]
      → [VPC] で検索
      → [(サイドバーの) Transit Gateway]
      → [Transit Gateway を作成]
      の順にクリック
   2. 項目の入力
      - 名前タグ: example_2022_11
      - 説明: ―― (空欄)
      - Transit Gateway を設定
        - Amazon 側の自律システム番号 (ASN) : ―― (空欄)
        - DNS サポート: ✅
        - VPN ECMP サポート: ✅
        - デフォルトルートテーブルの関連付け: ✅
        - デフォルトルートテーブル伝播: ✅
        - マルチキャストサポート: □
      - クロスアカウント共有オプションの設定
        - 共有アタッチメントを自動承諾: □
      - Transit Gateway CIDR ブロック
        - CIDR - オプション: ―― (空欄)
<!--
手動で紐づけようとすると，AWS のバグ？で紐づけができない．このため，デフォルトでルートテーブルの関連付けをする．
2. ルートテーブルの作成
   Transit Gateway の接続の経路定義用にルートテーブルを作成する．
   1. 作成画面に移動する
      [AWS Management Console]
      → [VPC] で検索
      → [(サイドバーの) Transit Gateway ルートテーブル]
      → [Transit Gateway ルートテーブルを作成]
      の順にクリック
   2. 項目の入力
      - 名前: example_2022_11
      - Transit Gateway ID: ＜先程作成した Transit Gateway の IDを入力する＞
-->
3. アタッチメント付与
   各 VPC に Transit Gateway 接続用のアタッチメント付与する．
   1. 作成画面に移動する
       [AWS Management Console]
       → [VPC] で検索
       → [(サイドバーの) Transit Gateway アタッチメント]
       → [Transit Gateway アタッチメントを作成]
       の順にクリック
   2. 項目の入力 (VPC1)
      - 名前: example_2022_11_for_vpc1
      - Transit Gateway ID: ＜先程作成した Transit Gateway の IDを入力する＞
      - アタッチメントタイプ: VPC
        ※ Peering Connection を利用すると，他の AWS アカウントと接続できる．
      - VPC アタッチメント:
        - DNS サポート: ✅
        - IPv6 サポート: □
        - VPC ID: InfraStack/vpc1
   3. 項目の入力 (VPC2)
      - 名前: example_2022_11_for_vpc2
      - Transit Gateway ID: ＜先程作成した Transit Gateway の IDを入力する＞
      - アタッチメントタイプ: VPC
        ※ Peering Connection を利用すると，他の AWS アカウントと接続できる．
      - VPC アタッチメント:
        - DNS サポート: ✅
        - IPv6 サポート: □
        - VPC ID: InfraStack/vpc2
4. Transit Gateway のルートテーブルの設定
   1. アソシエーション (関連付け) の設定
      ルートテーブルに作成したアタッチメントを登録します
      1. 作成画面に移動する
         [AWS Management Console]
         → [VPC] で検索
         → [(サイドバーの) Transit Gateway ルートテーブル]
         → ＜作成したルートテーブル名＞
         → [関連付け (タブ)]
         → [関連付けを作成]
         の順にクリック
      2. 項目の入力 (VPC1)
         - 詳細
           - 関連付けるアタッチメントを選択: example_2022_11_for_vpc1
      3. 項目の入力 (VPC2)
         - 詳細
           - 関連付けるアタッチメントを選択: example_2022_11_for_vpc2
   2. プロパゲーション (伝搬) の設定
      ルートテーブルに伝搬ルールをを登録します
      1. 作成画面に移動する
         [AWS Management Console]
         → [VPC]
         → [(サイドバーの) Transit Gateway ルートテーブル]
         → ＜作成したルートテーブル名＞
         → [伝搬 (タブ)]
         → [伝搬を作成]
         の順にクリック
      2. 項目の入力 (VPC1)
         - 詳細
           - 関連付けるアタッチメントを選択: example_2022_11_for_vpc1
      3. 項目の入力 (VPC2)
         - 詳細
           - 関連付けるアタッチメントを選択: example_2022_11_for_vpc2
5. 各 VPC の設定
   各 VPC のルートテーブル，セキュリティグループ（と設定している場合は ACL）で，VPC1 から VPC2 へのトラフィックを許可する．
   ルートテーブルはサブネットごとに存在しており，VPC の画面からでは，EC2 に割り当てられているサブネットを判別するのは難しい．このため，EC2 の画面から，EC2 に付与されたサブネットを選択して，設定していく．
   1. 各サブネットのルートテーブルの設定
      1. 作成画面に移動する
         [AWS Management Console]
         → [EC2] で検索
         → [(サイドバーの) インスタンス]
         → ＜InfraStack/general_purpose_ec2_on_vpc1＞または＜InfraStack/general_purpose_ec2_on_vpc2＞
         → [サブネットID]
         → ＜表示されたサブネット＞
         → [ルート (タブ)]
         → [ルートを編集]
         → [ルートを追加]
         の順にクリック
      2. 項目の入力 (VPC1)
         作業画面に移動する際に「＜InfraStack/general_purpose_ec2_on_vpc1＞」を選択した後，以下の設定を付与する．
         - 送信先: 10.1.0.0/16 (VPC2 のアドレス範囲を指定する)
         - ターゲット: Transit Gateway → ＜作成した Transit Gateway＞
      3. 項目の入力 (VPC2)
         作業画面に移動する際に「＜InfraStack/general_purpose_ec2_on_vpc2＞」を選択した後，以下の設定を付与する．
         - 送信先: 10.0.0.0/16 (VPC1 のアドレス範囲を指定する)
         - ターゲット: Transit Gateway → ＜作成した Transit Gateway＞
   2. 各サブネットのセキュリティグループの設定
      ここでは，EC2 にアタッチしているセキュリティグループを編集する．セキュリティグループは分類としては VPC だが，探すのが大変なので EC2 の画面から遷移する．ここでは，VPC1 から VPC2 にある web サーバへの問い合わせのみ許可するので，VPC2 にのみインバウンドルールを設定する
      1. 作成画面に移動する
         [AWS Management Console]
         → [EC2] で検索
         → [(サイドバーの) インスタンス]
         → ＜InfraStack/general_purpose_ec2_on_vpc2＞
         → [セキュリティ (タブ)]
         → [セキュリティグループ]
         → [インバウンドルールの編集]
         → [ルールを追加]
         の順にクリック
      2. 項目の入力 (VPC2)
         - タイプ: すべてのトラフィック
         - ソース: 10.0.0.0/16 (VPC1 のアドレス範囲を指定する)
6. 疎通の確認
   1. 接続先 EC2 の IP アドレス確認
      1. 作成画面に移動する
         [AWS Management Console]
         → [EC2] で検索
         → [(サイドバーの) インスタンス]
         → ＜InfraStack/general_purpose_ec2_on_vpc2＞
         → [接続]
         → [セッションマネージャー (タブ)]
         の順にクリック
      2. 「プライベート IPv4 アドレス」を確認する
         ここでは，10.1.0.100 と仮定して進める
   2. VPC1 の EC2 にログイン
      [AWS Management Console]
      → [EC2]
      → [(サイドバーの) インスタンス]
      → ＜InfraStack/general_purpose_ec2_on_vpc1＞
      → [接続]
      → [セッションマネージャー (タブ)]
      の順にクリック
   3. ping による確認
      ```
      ping -c 3 10.1.0.100
      ```
   4. web サーバの動作確認
      ```
      curl 10.1.0.100
      ```

## デバッグ

疎通しない場合は，リーチャビリティアナライザでデバッグする．
始めはなるべく緩い条件でエラーを確認していき，疎通するごとに条件を絞る．

1. 作成画面に移動する
   [AWS Management Console]
   → [Reachablility Analyzer] で検索
   → [パスの作成と分析]
   の順にクリック
2. パスの作成と分析
   - 名前: example_2022_11
   - 送信元: 
     - 送信元タイプ: Instance
     - 送信元: ＜InfraStack/general_purpose_ec2_on_vpc1＞
   - 送信先: 
     - 送信先タイプ: Instance
     - 送信先: ＜InfraStack/general_purpose_ec2_on_vpc2＞

## 参考

- [Transit Gatewayを利用してVPC間で通信してみた](https://dev.classmethod.jp/articles/transit-gateway-vpc/)
