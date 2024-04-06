# SmbExec/ScExec

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)をフォローする。
* **HackTricks**および**HackTricks Cloud**のGitHubリポジトリにPRを提出して、あなたのハッキングテクニックを共有してください。

</details>

## 動作原理

**Smbexec**は、Windowsシステムでリモートコマンドを実行するためのツールであり、**Psexec**に類似していますが、ターゲットシステムに悪意のあるファイルを配置することを避けます。

### **SMBExec**に関する主なポイント

- ターゲットマシン上で一時的なサービス（たとえば、「BTOBTO」）を作成して、バイナリをドロップせずにcmd.exe（%COMSPEC%）を介してコマンドを実行します。
- ステルス的なアプローチにもかかわらず、実行された各コマンドに対してイベントログを生成し、非対話的な「シェル」を提供します。
- **Smbexec**を使用して接続するコマンドは次のようになります：
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### バイナリなしでコマンドを実行する

- **Smbexec** は、サービスの binPaths を介して直接コマンドを実行できるようにし、ターゲット上で物理的なバイナリが必要なくなります。
- この方法は、Windows ターゲット上で一度だけコマンドを実行するのに便利です。たとえば、Metasploit の `web_delivery` モジュールと組み合わせることで、PowerShell をターゲットとした逆接続 Meterpreter ペイロードを実行できます。
- 攻撃者のマシンで binPath を設定して提供されたコマンドを cmd.exe を介して実行するリモートサービスを作成することで、サービスの応答エラーが発生した場合でも、Metasploit リスナーを使用してコールバックとペイロードの実行を成功させることができます。

### コマンドの例

以下のコマンドを使用して、サービスの作成と開始を行うことができます：
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
さらなる詳細については、[https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)を参照してください。


## 参考文献
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を使って、ゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)で**フォロー**する
* **HackTricks**および**HackTricks Cloud**のGitHubリポジトリにPRを提出して、**ハッキングトリックを共有**する

</details>