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

### ipの確認
`# ip netns`  
```
qdhcp-94931237-7351-4690-9bfe-2a973739d631
qdhcp-df3a58b5-74f7-4403-93ee-3699413fa976
qdhcp-3e41c24b-d84a-46ed-9fa7-d5849ad7010c
qrouter-3f9d3fae-7193-4af0-8e2d-d1a2ff356c1a
```
`# ip netns exec qdhcp-df3a58b5-74f7-4403-93ee-3699413fa976 ifconfig `  
***ip nets execでコマンドを打つことが可能***
```
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

tapb5cfa271-71: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.50.14.51  netmask 255.255.255.0  broadcast 10.50.14.255			★10.50.14.51がDHCP ServerのIP
        inet6 fe80::f816:3eff:fe90:58fa  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:90:58:fa  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
### eno1をvswitchに接続
`# /etc/init.d/network stop`  
`# cat /etc/sysconfig/network-scripts/ifcfg-eno1`  
```
TYPE=OVSPort
DEVICETYPE=ovs
OVS_BRIDGE=br-ex
NM_CONTROLLED=no
BOOTPROTO=none
ONBOOT=yes
DEVICE=eno1
PEERDNS=no
```
`# cat /etc/sysconfig/network-scripts/ifcfg-br-ex`  
```
DEVICE=br-ex
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=static
IPADDR=10.50.14.50
NETMASK=255.255.255.0
GATEWAY=10.50.14.1
ONBOOT=yes
```
`# /etc/init.d/network start`  
#### 各種サービスのリスタート  
`# service neutron-server restart`  
`# service neutron-dhcp-agent restart`  
`# service neutron-l3-agent restart`  
`# service neutron-metadata-agent restart`  
`# service neutron-openvswitch-agent restart`  
`# ovs-vsctl show`  
```
ad8bb742-5fd4-43cb-b277-dba16e9a6b2e
    Bridge br-tun
        fail_mode: secure
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port br-tun
            Interface br-tun
                type: internal
    Bridge br-ex
        Port br-ex
            Interface br-ex		★
                type: internal
        Port "qg-48cdaf68-67"		★
            Interface "qg-48cdaf68-67"
                type: internal
        Port "eno1"			★
            Interface "eno1"
    Bridge br-int
        fail_mode: secure
        Port br-int
            Interface br-int
                type: internal
        Port "qvoa7b1762b-3b"
            tag: 1
            Interface "qvoa7b1762b-3b"
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port "qr-610bdae7-c1"
            tag: 1
            Interface "qr-610bdae7-c1"
                type: internal
        Port "qvo4e992338-59"
            tag: 1
            Interface "qvo4e992338-59"
        Port "tap754104f3-90"
            tag: 3
            Interface "tap754104f3-90"
                type: internal
        Port "tapb5cfa271-71"
            tag: 2
            Interface "tapb5cfa271-71"
                type: internal
        Port "tap565d3477-22"
            tag: 1
            Interface "tap565d3477-22"
                type: internal
        Port "qvo22a56b8e-12"
            tag: 1
            Interface "qvo22a56b8e-12"
        Port "qvofa8a9472-6f"
            tag: 1
            Interface "qvofa8a9472-6f"
    ovs_version: "2.5.0"
 ```
 `# ip netns exec qrouter-4f44407a-6679-401f-8b7a-30c89f8749cf ifconfig`  
 ```
 lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 1  bytes 112 (112.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1  bytes 112 (112.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

qg-48cdaf68-67: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.50.14.57  netmask 255.255.255.0  broadcast 10.50.14.255			★
        inet6 fe80::f816:3eff:feb2:d8f0  prefixlen 64  scopeid 0x20<link>
        inet6 2400:4010:423:22:f816:3eff:feb2:d8f0  prefixlen 64  scopeid 0x0<global>
        ether fa:16:3e:b2:d8:f0  txqueuelen 0  (Ethernet)
        RX packets 437  bytes 34287 (33.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 23  bytes 1754 (1.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

qr-610bdae7-c1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.1  netmask 255.255.255.0  broadcast 192.168.10.255		★
        inet6 fe80::f816:3eff:fea7:2400  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:a7:24:00  txqueuelen 0  (Ethernet)
        RX packets 470  bytes 47931 (46.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 415  bytes 49191 (48.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
 ```
 #### openvswitchの確認  
 `# ovs-vsctl show`  
 ```
 ad8bb742-5fd4-43cb-b277-dba16e9a6b2e
    Bridge br-tun
        fail_mode: secure
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port br-tun
            Interface br-tun
                type: internal
    Bridge br-ex
        Port "qg-ba0a66c4-66"
            Interface "qg-ba0a66c4-66"
                type: internal
        Port br-ex
            Interface br-ex
                type: internal
    Bridge br-int
        fail_mode: secure
        Port br-int
            Interface br-int
                type: internal
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port "qvof534b306-e5"
            tag: 3
            Interface "qvof534b306-e5"
        Port "tap754104f3-90"
            tag: 1
            Interface "tap754104f3-90"
                type: internal
        Port "tapb5cfa271-71"			
            tag: 6
            Interface "tapb5cfa271-71"			★10.50.14.0/24に繋がるポート
                type: internal
        Port "qr-b89a8bbc-72"
            tag: 1
            Interface "qr-b89a8bbc-72"
                type: internal
        Port "qvobd0738c2-57"
            tag: 3
            Interface "qvobd0738c2-57"
        Port "tap565d3477-22"
            tag: 7
            Interface "tap565d3477-22"			▲192.168.10.0/24に繋がるポート
                type: internal
        Port "qvo8130fa4c-a8"
            tag: 3
            Interface "qvo8130fa4c-a8"
    ovs_version: "2.5.0"
```
`# neutron router-port-list  a30bc5a4-d69a-4e25-9b8b-59ae24ab4db4`  
```
+--------------------------------------+------+-------------------+----------------------------------------------+
| id                                   | name | mac_address       | fixed_ips                                    |
+--------------------------------------+------+-------------------+----------------------------------------------+
| 709be681-e961-4396-a762-435dc0c5b40f |      | fa:16:3e:71:8a:c6 | {"subnet_id": "f0af004f-45c6-4399-9acb-      |
|                                      |      |                   | 8a33413dbc6d", "ip_address": "192.168.10.1"} |
| e3a96c2e-3870-4a40-9735-e7cb9b583f0d |      | fa:16:3e:7b:f6:09 | {"subnet_id": "dc39b715-ef37-480c-           |
|                                      |      |                   | 98a9-ec3f1123ba82", "ip_address":            |
|                                      |      |                   | "10.50.14.52"}                               |
+--------------------------------------+------+-------------------+----------------------------------------------+
```
#### 参考
<http://transparent-to-radiation.blogspot.jp/2014/01/openstack-neutronnetwork-interface.html>  
<http://www.usupi.org/sysad/260.html>  

