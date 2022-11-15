# aws-cdk-ec2-private-link

## AWS CDK プロジェクトの初期化

```
$ mkdir [MakeEmptyDir]
$ cd [MakeEmptyDir]
$ npx cdk init app --language typescript
```

## Nodejs の設定

### インストール

```
sudo amazon-linux-extras install nginx1 -y
```

インストール確認
```
nginx -v
```

### 起動

```
sudo systemctl start nginx
sudo systemctl status nginx
sudo systemctl enable nginx
systemctl is-enabled nginx
```

起動確認
```
curl localhost
```

- 参考: [【AWS EC2】Amazon Linux2にnginxをインストールする方法](https://qiita.com/tamorieeeen/items/07743216a3662cfca890)


## PrivateLink の設定

```

```

料金:

- PrivateLink: 
  - 0.01 USD/h
  - 0.01 USD/GB
- NLB:
  - 0.0225USD/h
  - 0.006USD/h

- 合計: 0.0325 USD/h -> (0.0325*140*24*30=) 3276 円/月

- 参考:
  - [VPC間のAWS PrivateLinkを試してみた](https://dev.classmethod.jp/articles/tried-aws-privatelink-between-vpcs/)
  - [【初心者】AWS PrivateLink を使ってみる](https://qiita.com/mksamba/items/20903940b8b256ef2487)


## Transit Gateway

- Transit Gateway:
  - 0.05 USD/h -> (0.05*140*24*30=) 5040 円/月

- 参考:
  - [Transit Gatewayを利用してVPC間で通信してみた](https://dev.classmethod.jp/articles/transit-gateway-vpc/)

