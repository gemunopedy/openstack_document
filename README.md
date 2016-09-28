# OpenStak 構築
## 構築環境
- OS : Centos7
- Openstack : mitaka

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
***インストールは非常に時間がかかる***
### netutron
### 外部通信可能なネットワークの作り方  
`# neutron net-create net-ext --provider:network_type=local  --router:external=true --shared`  
以下のような結果が表示される。    
```
+---------------------------+--------------------------------------+  
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2016-07-26T03:38:38                  |
| description               |                                      |
| id                        | df3a58b5-74f7-4403-93ee-3699413fa976 |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| is_default                | False                                |
| mtu                       | 1500                                 |
| name                      | net-ext                              |
| provider:network_type     | local                                |
| provider:physical_network |                                      |
| provider:segmentation_id  |                                      |
| router:external           | True                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | 2ed991629f1e49a4b7e3ca74434f8ee9     |
| updated_at                | 2016-07-26T03:38:38                  |
+---------------------------+--------------------------------------+
```
`#neutron subnet-create net-ext 10.50.14.0/24 --name=sub-ext  --allocation_pool start=x.x.x.51,end=x.x.x.57`  
以下のような結果が表示される。  
```
+-------------------+------------------------------------------------+
| Field             | Value                                          |
+-------------------+------------------------------------------------+
| allocation_pools  | {"start": "x.x.x.51", "end": "x.x.x.57"}       |
| cidr              | x.x.x.0/24                                     |
| created_at        | 2016-07-26T03:44:55                            |
| description       |                                                |
| dns_nameservers   |                                                |
| enable_dhcp       | True                                           |
| gateway_ip        | 10.50.14.1                                     |
| host_routes       |                                                |
| id                | dc39b715-ef37-480c-98a9-ec3f1123ba82           |
| ip_version        | 4                                              |
| ipv6_address_mode |                                                |
| ipv6_ra_mode      |                                                |
| name              | sub-ext                                        |
| network_id        | df3a58b5-74f7-4403-93ee-3699413fa976           |
| subnetpool_id     |                                                |
| tenant_id         | 2ed991629f1e49a4b7e3ca74434f8ee9               |
| updated_at        | 2016-07-26T03:44:55                            |
+-------------------+------------------------------------------------+
```

### 内部ネットワークの設定
`# neutron net-create net-int --provider:network_type=local   --shared`  
以下の結果が表示される。  
```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2016-07-26T03:48:16                  |
| description               |                                      |
| id                        | 94931237-7351-4690-9bfe-2a973739d631 |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| mtu                       | 1500                                 |
| name                      | net-int                              |
| provider:network_type     | local                                |
| provider:physical_network |                                      |
| provider:segmentation_id  |                                      |
| router:external           | False                                |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | 2ed991629f1e49a4b7e3ca74434f8ee9     |
| updated_at                | 2016-07-26T03:48:16                  |
+---------------------------+--------------------------------------+
```
`#neutron subnet-create net-int 192.168.10.0/24 --name=sub-int  --allocation_pool start=192.168.10.10,end=192.168.10.50`  
以下の結果が表示される。  
```
+-------------------+----------------------------------------------------+
| Field             | Value                                              |
+-------------------+----------------------------------------------------+
| allocation_pools  | {"start": "192.168.10.10", "end": "192.168.10.50"} |
| cidr              | 192.168.10.0/24                                    |
| created_at        | 2016-07-26T03:49:57                                |
| description       |                                                    |
| dns_nameservers   |                                                    |
| enable_dhcp       | True                                               |
| gateway_ip        | 192.168.10.1                                       |
| host_routes       |                                                    |
| id                | f0af004f-45c6-4399-9acb-8a33413dbc6d               |
| ip_version        | 4                                                  |
| ipv6_address_mode |                                                    |
| ipv6_ra_mode      |                                                    |
| name              | sub-int                                            |
| network_id        | 94931237-7351-4690-9bfe-2a973739d631               |
| subnetpool_id     |                                                    |
| tenant_id         | 2ed991629f1e49a4b7e3ca74434f8ee9                   |
| updated_at        | 2016-07-26T03:49:57                                |
+-------------------+----------------------------------------------------+
```
### ルータの設定
`# neutron router-create router1`  
以下の結果が表示される。   
```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | True                                 |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   |                                      |
| ha                      | False                                |
| id                      | a30bc5a4-d69a-4e25-9b8b-59ae24ab4db4 |
| name                    | router1                              |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tenant_id               | 2ed991629f1e49a4b7e3ca74434f8ee9     |
+-------------------------+--------------------------------------+
```
1. routerとgatewayをひもづける。  
`# neutron router-gateway-set a30bc5a4-d69a-4e25-9b8b-59ae24ab4db4 df3a58b5-74f7-4403-93ee-3699413fa976`
2. routerとinterfaceをひもづける。  
`# neutron router-interface-add  a30bc5a4-d69a-4e25-9b8b-59ae24ab4db4 f0af004f-45c6-4399-9acb-8a33413dbc6d`

