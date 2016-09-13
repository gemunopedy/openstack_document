# OpenStak 構築
## 構築環境
- OS : Centos7
- Server : hoge
- Kernel : hoge

## 構築手順
### Openstack インストール
1. Netwokmanagerの停止及び使用不可  
`#systemctl stop NetworkManager`  
`#sytemctl disable NetworkManager`  
2. Openstack-packstackのインストール  
`yum update -y`  
`yum install -y https://rdoproject.org/repos/rdo-release.rpm`  
`yum install -y openstack-packstack`  
3. Openstackのインストール  
answer-fileを適宜編集し、必要なコンポーネントをインストールする。  
`packstack --gen-answer-file answer-file.txt`  
`vi answer-file.txttyum `  
以下のように、anserwer-fileを編集する。  

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

`packstack --answer-file answer-file.txt`  
*** インストールは非常に時間がかかる。***
### netutron
### 外部通信可能なネットワークの作り方  
`# neutron net-create net-ext --provider:network_type=local  --router:external=true --shared`  
>+---------------------------+--------------------------------------+  
>| Field                     | Value                                |  
>+---------------------------+--------------------------------------+  
>| admin_state_up            | True                                 |  
>| availability_zone_hints   |                                      |  
>| availability_zones        |                                      |  
>| created_at                | 2016-07-26T03:38:38                  |  
>| description               |                                      |  
>| id                        | df3a58b5-74f7-4403-93ee-3699413fa976 |  
>| ipv4_address_scope        |                                      |  
>| ipv6_address_scope        |                                      |  
>| is_default                | False                                |  
>| mtu                       | 1500                                 |  
>| name                      | net-ext                              |  
>| provider:network_type     | local                                |  
>| provider:physical_network |                                      |  
>| provider:segmentation_id  |                                      |  
>| router:external           | True                                 |  
>| shared                    | True                                 |  
>| status                    | ACTIVE                               |  
>| subnets                   |                                      |  
>| tags                      |                                      |  
>| tenant_id                 | 2ed991629f1e49a4b7e3ca74434f8ee9     |  
>| updated_at                | 2016-07-26T03:38:38                  |  
>+---------------------------+--------------------------------------+  

### 内部ネットワークの設定
`# neutron net-create net-int --provider:network_type=local   --shared`  
### ルータの設定
`# neutron router-create router1`  



