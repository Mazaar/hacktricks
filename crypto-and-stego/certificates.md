# 証明書

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong>を使って、**ゼロからヒーローまでAWSハッキングを学びましょう**！</summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見る
- 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)をフォローする
- ハッキングテクニックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出する

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## 証明書とは

**公開鍵証明書**は、暗号化で使用されるデジタルIDであり、誰かが公開鍵を所有していることを証明するために使用されます。これには、鍵の詳細、所有者の身元（サブジェクト）、および信頼された機関（発行者）からのデジタル署名が含まれます。ソフトウェアが発行者を信頼し、署名が有効であれば、鍵の所有者との安全な通信が可能です。

証明書は、主に[証明書機関](https://en.wikipedia.org/wiki/Certificate\_authority)（CAs）によって[公開鍵インフラストラクチャ](https://en.wikipedia.org/wiki/Public-key\_infrastructure)（PKI）のセットアップで発行されます。別の方法は、[信頼のウェブ](https://en.wikipedia.org/wiki/Web\_of\_trust)であり、ユーザーが直接お互いの鍵を検証します。証明書の一般的な形式は[X.509](https://en.wikipedia.org/wiki/X.509)であり、RFC 5280で詳細に説明されている特定のニーズに適応できます。

## x509共通フィールド

### **x509証明書の共通フィールド**

x509証明書では、いくつかの**フィールド**が証明書の有効性とセキュリティを確保するために重要な役割を果たします。これらのフィールドの詳細は次のとおりです：

- **バージョン番号**はx509形式のバージョンを示します。
- **シリアル番号**は、主に取り消し追跡のために証明書機関（CA）システム内で証明書を一意に識別します。
- **サブジェクト**フィールドは、証明書の所有者を表し、機械、個人、または組織である可能性があります。以下の詳細な識別情報が含まれます：
  - **Common Name (CN)**: 証明書でカバーされるドメイン。
  - **Country (C)**、**Locality (L)**、**State or Province (ST, S, or P)**、**Organization (O)**、および**Organizational Unit (OU)**は地理的および組織的な詳細を提供します。
  - **Distinguished Name (DN)**は完全なサブジェクト識別をカプセル化します。
- **発行者**は、CAのためのサブジェクトと同様のサブフィールドを含め、誰が証明書を検証して署名したかを示します。
- **有効期間**は、**Not Before**および**Not After**のタイムスタンプによってマークされ、証明書が特定の日付前または後に使用されないようにします。
- 証明書のセキュリティに重要な**公開鍵**セクションは、公開鍵のアルゴリズム、サイズ、およびその他の技術的詳細を指定します。
- **x509v3拡張**は、証明書の機能性を向上させ、**Key Usage**、**Extended Key Usage**、**Subject Alternative Name**などのプロパティを指定して証明書のアプリケーションを微調整します。

#### **Key Usageと拡張**

- **Key Usage**は、公開鍵の暗号化アプリケーションを識別します（デジタル署名や鍵の暗号化など）。
- **Extended Key Usage**は、証明書の使用ケースをさらに絞り込みます（例：TLSサーバー認証用）。
- **Subject Alternative Name**および**Basic Constraint**は、証明書でカバーされる追加のホスト名を定義し、それがCA証明書かエンドエンティティ証明書かを示します。
- **Subject Key Identifier**および**Authority Key Identifier**などの識別子は、鍵の一意性と追跡可能性を確保します。
- **Authority Information Access**および**CRL Distribution Points**は、発行CAを検証し、証明書の取り消し状態を確認するためのパスを提供します。
- **CT Precertificate SCTs**は、証明書への公共の信頼に不可欠な透明性ログを提供します。
```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```
### **OCSPとCRL配布ポイントの違い**

**OCSP**（**RFC 2560**）は、デジタル公開鍵証明書が取り消されたかどうかを確認するために、クライアントとレスポンダーが協力して作業する方法であり、完全な**CRL**をダウンロードする必要がない。この方法は、取り消された証明書のシリアル番号のリストを提供しますが、潜在的に大きなファイルをダウンロードする必要がある従来の**CRL**よりも効率的です。CRLには最大512エントリを含めることができます。詳細は[こちら](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm)で入手できます。

### **証明書透明性とは**

証明書透明性は、SSL証明書の発行と存在がドメイン所有者、CA、およびユーザーに可視であることを確認することで、証明書関連の脅威に対抗します。その目的は次のとおりです：

* ドメイン所有者の知識なしにドメインのためにSSL証明書を発行するCAを防ぐこと。
* 誤ってまたは悪意を持って発行された証明書を追跡するためのオープンな監査システムを確立すること。
* ユーザーを詐欺的な証明書から保護すること。

#### **証明書ログ**

証明書ログは、ネットワークサービスによって維持される証明書の公開監査可能な追加専用レコードであり、監査目的のための暗号的証拠を提供します。発行機関と一般の両方がこれらのログに証明書を提出したり、検証のためにクエリを送信したりできます。ログサーバーの正確な数は固定されていませんが、グローバルで1000未満であると予想されています。これらのサーバーは、CA、ISP、または関心を持つ任意のエンティティによって独立して管理される可能性があります。

#### **クエリ**

任意のドメインの証明書透明性ログを調べるには、[https://crt.sh/](https://crt.sh)を訪れてください。

## **フォーマット**

### **PEMフォーマット**

* 証明書に最も広く使用されているフォーマット。
* 証明書と秘密鍵用の別々のファイルが必要で、Base64 ASCIIでエンコードされています。
* 一般的な拡張子：.cer、.crt、.pem、.key。
* 主にApacheおよび類似のサーバーで使用されます。

### **DERフォーマット**

* 証明書のバイナリフォーマット。
* PEMファイルに見られる「BEGIN/END CERTIFICATE」ステートメントが欠けています。
* 一般的な拡張子：.cer、.der。
* Javaプラットフォームでよく使用されます。

### **P7B/PKCS#7フォーマット**

* Base64 ASCIIで保存され、拡張子は.p7bまたは.p7cです。
* 秘密鍵を除く証明書とチェーン証明書のみを含みます。
* Microsoft WindowsおよびJava Tomcatでサポートされています。

### **PFX/P12/PKCS#12フォーマット**

* サーバー証明書、中間証明書、および秘密鍵を1つのファイルにカプセル化するバイナリフォーマット。
* 拡張子：.pfx、.p12。
* 証明書のインポートおよびエクスポートにWindowsで主に使用されます。

### **フォーマットの変換**

互換性のために**PEM変換**が重要です：

* **x509からPEMへ**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEMからDERへ**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DERからPEMへ**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEMからP7Bへ**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7をPEM形式に変換する**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX変換**はWindowsで証明書を管理するために重要です：

* **PFXからPEMへ**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFXをPKCS#8に変換する**には、2つのステップが必要です：
1. PFXをPEMに変換します
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. PEMをPKCS8に変換します。
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7BをPFXに** 変換するには、2つのコマンドが必要です：
1. P7BをCERに変換します。
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. CERおよびプライベートキーをPFXに変換します。
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks)を使用して、世界で最も先進的なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>で**ゼロからヒーローまでAWSハッキングを学ぶ**</summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝**したい場合や**HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手してください
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つけてください
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**をフォロー**してください。
* **HackTricks**および**HackTricks Cloud**のgithubリポジトリにPRを提出して、あなたのハッキングトリックを共有してください。

</details>