# AWS上のSingle Node Openshift環境にMAS/MANAGEを導入する手順書

## 目次
- [はじめに](#はじめに)
- [00_構成と前提](00_architecture/index.md)
- [01_事前準備](01_prereqs/index.md)
- [02_AWS準備](02_aws_prepare/index.md)
- [03_MAS/MANAGEインストール](03_manageinstall/index.md)
- [04_管理者ユーザーの作成](04_maxadmin/index.md)
- [05_導入後環境の確認](05_confirm/index.md)
## はじめに
本資料は、AWS環境にSingle Node Openshiftを構築し、IBM Maximo Application Suite(MAS) 8.9 / MAS Manage 8.5 を導入する手順書です。  
AWS上に テストやPoC利用を目的とした 最小構成の MAS Manage 環境を構築することを想定しております。

本手順は下記のURLを参考に導入しています。

* Single Node OpenShiftにMASを導入する手順  

	https://ibm-mas-manage.github.io/sno/

* Ansible collection

	https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=installing-ansible-collection

Maximo Application Suite の環境別の導入方法の詳細については以下をご覧ください。  
MAS Manage のみ導入する場合も以下をご参照ください。

* Supported installation paths

	https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=suite-supported-installation-paths


## 注意事項
- 当手順は2023年3月に、Maximo Application Suite 8.9/Manage8.5の構築・アクティベーションをAWSのSingle Node Openshift環境にて実施した際の作業ログをベースに記述しております。
- 本資料の記載内容は、正式な IBM のテストやレビューを受けておりません。内容について、できる限り正確を期すよう努めておりますが、いかなる明示または暗黙の保証も責任も負いかねます。
- 本資料の情報は、使用先の責任において使用されるべきものであることを、あらかじめご了承ください。
- 掲載内容は不定期に変更されることもあります。他のメディア等に無断で転載する事はご遠慮ください。
- 本資料の著作権は日本アイ・ビー・エムにあります。非営利目的の個人利用の場合において、自由に使用してもかまいませんが、営利目的の使用は禁止させていただきます。
- 本資料は日本アイ・ビー・エム株式会社ならびに日本アイ・ビー・エム・システムズ・エンジニアリング株式会社の正式なレビューを受けておりません。当資料は正式なマニュアルをはじめとするドキュメントの補完資料として参照して下さい。
- 本資料は、製品の特定バージョンを使ってテストをした結果をもとに記述しています。今後のバージョンおよびメンテナンスリリース、Feature Packなどの適用により動作が当資料に記述された内容とは異なってくる可能性がありますのでご了承下さい。
- 本資料は、「本資料にて導入する環境」記載の環境でのみ稼働確認を行っており、他の環境での動作確認は実施しておりません。他の環境での手順につきましては、マニュアルを参照ください。
- AWSの課金が発生することをご留意ください。
- step by stepの手順ではありません。前提知識を必要とします。
- 資料中、以下の略称を使用する場合があります。

| 略称     | 正式名称                       |
| -------- | ------------------------------ |
| MAS      | IBM Maximo Application Suite   |
| AWS      | Amazon Web Services            |
| OCP      | OpenShift Container Platform   |
| UDS      | IBM User Data Service          |
| SLS      | Suite License Service          |
| DB2      | IBM Db2                        |
| Manage   | IBM Maximo Manage              |
| Pipeline | Red Hat Openshift Pipelines    |

## 本資料にて導入する環境
本資料にて利用する各種ソフトウェアバージョンは以下の通りです。


| 略称                                | バージョン |
| ----------------------------------- | ---------- |
| Red Hat OpenShift                   | 4.10.52    |
| Red Hat OpenShift CLI               | 4.10.52    |
| IBM Maximo Application Suite        | 8.9.0      |
| IBM Maximo Manage                   | 8.5.0      |
| IBM Cloud Pack foundational Service | 3.23.0     |
| UDS                                 | 2.0.9      |
| SLS                                 | 3.5.0      |

## 参考.IBM Maximo Application Suite 8.9 前提ソフトウェア
Supported software versions

https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=suite-supported-software-versions

Compatibility matrix

https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=suite-compatibility-matrix

各ソフトウェアの採用にあたり詳細な注意事項がある場合があります。
サポート対象となるハードウェア/ソフトウェアバージョンに関しての最新情報や、今回のガイドで用いない構成に関わるソフトウェアのサポート状況に関しては製品システム要件をご確認ください。

IBM Maximo Application Requirements and capacity planning  
https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=suite-requirements-capacity-planning

IBM Maximo Manage system requirements  
https://www.ibm.com/docs/en/maximo-manage/continuous-delivery?topic=deploy-system-requirements


## 構築手順時間の目安
| ステップ                  | 所要目安時間                       |
| ------------------------- | ---------------------------------- |
| 01_事前準備               | 1時間                              |
| 02_AWS準備                | 30分                               |
| 03_MAS/MANAGEインストール | アクティベート完了まで6時間程度     |
| 04_管理者ユーザーの作成   | 30分                               |
| 05_導入後環境の確認       | 15分                               |


## 参考リンク

* Maximo Application Suite 8.9/Manage8.5の導入手順書
  
    https://github.com/IBM/MAS-install-JP

* IBM Documentation：MAS8.8 and later
  
	https://www.ibm.com/docs/en/mas-cd/continuous-delivery


* IBM Documentation：Manage 8.4 and later
  
	https://www.ibm.com/docs/en/maximo-manage/continuous-delivery

* IBM Maximo Application Suite CLI Utility
  	
	https://ibm-mas.github.io/cli/


* 参考サイト
  
    Maximo Programming

	https://maximopro.tumblr.com/

	MAS MS Data Sheet

	https://www.ibm.com/software/reports/compatibility/clarity-reports/report/html/softwareReqsForProduct?deliverableId=3F0E2B305E7111EABE1C939145D7672E

	Deployment Guide for Maximo Application Suite

	https://mam-hol.eu-gb.mybluemix.net/


	MAS お客様向けライセンス・ガイド @Seismic

	https://ibm.seismic.com/Link/Content/DC8fdwbEQVi0-ClDMMN66Elw


	MAS Pricing Guide @Seismic

	https://ibm.seismic.com/Link/Content/DCV7TR4HYCgUaD2mrkzrxzAQ

	以下、IBM社員のみ

	MAS MS Wiki 

	https://github.ibm.com/maximoappsuite/tracker-masms/wiki/Blueline-Assessment#as-a-service-capabilities

	Maximo Performance Wiki

	https://pages.github.ibm.com/maximo/performance-wiki/


### 次項
- [00_構成と前提](./00_architecture/index.md)
