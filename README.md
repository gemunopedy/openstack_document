# openstack_document
## enviroment
OS : Centos7
## 初期インストール
`#systemctl stop NetworkManager`  
`#sytemctl disable NetworkManager`  
`yum update -y`  
`yum install -y https://rdoproject.org/repos/rdo-release.rpm`  
`yum install -y openstack-packstack`  
`packstack --gen-answer-file answer-file.txt`  
`vi answer-file.txttyum `  
>CONFIG_HEAT_INSTALL=y
>...オーケストレーション機能を有効化
>CONFIG_SAHARA_INSTALL=y 
>...データ処理サービスを有効化
>CONFIG_LBAAS_INSTALL=y 
>...ロードバランスサービスを有効化
>CONFIG_NEUTRON_FWAAS=y 
>...ファイアウォールサービスを有効化
>CONFIG_TROVE_INSTALL=y 
>...データベースサービスを有効化
>CONFIG_HORIZON_SSL=y 
>...ダッシュボードアクセスをhttpsに変更
>CONFIG_KEYSTONE_ADMIN_PW=centos 
>...パスワードをcentosに変更
>CONFIG_NTP_SERVERS=133.243.238.244 
>...NTPサーバのアドレスを設定(ntp.nict.jp)
>CONFIG_PROVISION_DEMO=n
>...デモ機能の無効化
>CONFIG_HEAT_CLOUDWATCH_INSTALL=y
>CONFIG_HEAT_CFN_INSTALL=y
>CONFIG_CINDER_VOLUMES_SIZE=20G
>CONFIG_SWIFT_STORAGE_SIZE=2G
## netutron
## cinder


