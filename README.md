# MyApp CI/CD (GitHub → CodePipeline → CodeBuild → Manual Approval → CodeDeploy)

## 概要
このリポジトリは、AWS CodePipeline/CodeBuild/CodeDeploy を用いた EC2 デプロイのひな形です。
アーティファクトは CodePipeline 経由で S3 に格納され、CodeDeploy が EC2 にデプロイします。

本サンプルは「Python + systemd」で 8080 ポートに簡易 Web サーバを起動し、
/health エンドポイントで疎通確認を行います。

## 重要なパス/設定
- デプロイ先ディレクトリ: /opt/myapp
- 公開コンテンツ: /opt/myapp/public
- サービス名: myapp (systemd unit: myapp.service)
- listen: 0.0.0.0:8080
- health: http://localhost:8080/health

## 事前条件（EC2）
- CodeDeploy Agent が稼働していること
- systemd が利用できること
- ポート 8080 が必要に応じて到達可能であること（SG/NACL）

## デプロイの流れ（CodeDeploy hooks）
1. BeforeInstall: 既存サービス停止、ディレクトリ準備
2. AfterInstall: Python3/依存の確認、systemd unit 配置、daemon-reload
3. ApplicationStart: サービス有効化/起動
4. ValidateService: /health をリトライしつつ確認

## カスタマイズ
- アプリが Node/Java 等の場合:
  - app/ 配下を実アプリ構成へ置換
  - scripts/after_install.sh の依存インストール、systemd unit、起動コマンドを変更
  - scripts/validate_service.sh のヘルスチェックを変更

## ログ
- CodeDeploy hook ログ: /var/log/codedeploy-hooks/*.log
- systemd ログ: journalctl -u myapp -e

## 権限（IAM）
- CodeBuild ロール: Logs / S3 / SSM / Secrets (必要ならKMS Decrypt)
- EC2 インスタンスロール: S3 GetObject (アーティファクト取得)
