# Linux Forensics

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も**高度な**コミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>**htARTE（HackTricks AWS Red Team Expert）**で**ゼロからヒーローまでのAWSハッキング**を学びましょう！</strong></summary>

**HackTricksをサポートする他の方法：HackTricksで企業を宣伝したい場合やHackTricksをPDFでダウンロードしたい場合は、**[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)**をチェックしてください！**[**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)**を入手してください**[**The PEASS Family**](https://opensea.io/collection/the-peass-family)**を発見し、独占的な**[**NFTs**](https://opensea.io/collection/the-peass-family)**のコレクションを見つけてください💬** [**Discordグループ**](https://discord.gg/hRep4RUj7f)**または**[**telegramグループ**](https://t.me/peass)**に参加するか、Twitter 🐦** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。HackTricksと**[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)**のgithubリポジトリにPRを提出して、あなたのハッキングトリックを共有してください。**

</details>

**初期情報収集基本情報まず最初に、USBに良く知られたバイナリとライブラリが含まれていることが推奨されます（単にUbuntuを取得して、\_ /bin\_、\_ /sbin\_、\_ /lib\_、および\_ /lib64\_のフォルダをコピーできます）。次にUSBをマウントし、環境変数を変更してこれらのバイナリを使用します：**export PATH=/mnt/usb/bin:/mnt/usb/sbinexport LD\_LIBRARY\_PATH=/mnt/usb/lib:/mnt/usb/lib64**一度システムを良いものや既知のバイナリを使用するように設定したら、基本情報を抽出することができます：**date #Date and time (Clock may be skewed, Might be at a different timezone)uname -a #OS infoifconfig -a || ip a #Network interfaces (promiscuous mode?)ps -ef #Running processesnetstat -anp #Proccess and portslsof -V #Open filesnetstat -rn; route #Routing tabledf; mount #Free space and mounted devicesfree #Meam and swap spacew #Who is connectedlast -Faiwx #Loginslsmod #What is loadedcat /etc/passwd #Unexpected data?cat /etc/shadow #Unexpected data?find /directory -type f -mtime -1 -print #Find modified files during the last minute in the directory**疑わしい情報基本情報を取得する際に、次のような奇妙な点をチェックする必要があります:Rootプロセス は通常、低いPIDで実行されます。そのため、大きなPIDで実行されているRootプロセスが見つかった場合は疑うべきです`/etc/passwd` 内にシェルを持たないユーザーの登録されたログイン を確認する`/etc/shadow` 内にシェルを持たないユーザーのパスワードハッシュ を確認するメモリーダンプ実行中のシステムのメモリを取得するには、**[**LiME**](https://github.com/504ensicsLabs/LiME) **を使用することをお勧めします。**\
**コンパイル するには、被害者マシンが使用している同じカーネル を使用する必要があります。被害者マシンに LiME やその他の何かをインストールすることはできない ことを覚えておいてください。それにより、複数の変更が加えられますしたがって、Ubuntuの同一バージョンがある場合は、`apt-get install lime-forensics-dkms` を使用できます**\
**それ以外の場合は、**[**LiME**](https://github.com/504ensicsLabs/LiME) **をgithub からダウンロードし、正しいカーネルヘッダーを使用してコンパイルする必要があります。被害者マシンの正確なカーネルヘッダーを取得する には、単に`/lib/modules/<kernel version>` ディレクトリをあなたのマシンにコピー し、それを使用して LiME をコンパイル します:**make -C /lib/modules/\<kernel version>/build M=$PWDsudo insmod lime.ko "path=/home/sansforensics/Desktop/mem\_dump.bin format=lime"**LiMEは3つのフォーマットをサポートしています：Raw（すべてのセグメントが連結されたもの）Padded（Rawと同じですが、右ビットにゼロが入っています）Lime（メタデータを含む推奨フォーマット）LiMEは、`path=tcp:4444`のような方法を使用して、ダンプをネットワーク経由で送信することもできます。ディスクイメージングシャットダウンまず、システムをシャットダウンする必要があります。これは常に選択肢とは限りません。なぜなら、システムが企業がシャットダウンする余裕のない本番サーバーである場合があるからです。**\
**システムをシャットダウンする方法には、通常のシャットダウンと\*\*「プラグを抜く」シャットダウンの2つがあります。前者はプロセスが通常通り終了し、ファイルシステムが同期されることを可能にしますが、悪意のあるソフトウェアが証拠を破壊する可能性もあります。後者の「プラグを抜く」アプローチは、一部の情報が失われる可能性があります（メモリのイメージをすでに取得しているため、失われる情報はほとんどありません）し、悪意のあるソフトウェアが何もできなくなります。したがって、悪意のあるソフトウェアが疑われる場合は、システムで`sync`\*\* コマンドを実行してプラグを抜いてください。ディスクのイメージを取得するケースに関連する何かにコンピュータを接続する前に、情報を変更しないように読み取り専用でマウントされることを確認する必要があります。**#Create a raw copy of the diskdd if=\<subject device> of=\<image file> bs=512#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)dcfldd if=\<subject device> of=\<image file> bs=512 hash=\<algorithm> hashwindow=\<chunk size> hashlog=\<hash file>dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes**ディスクイメージの事前分析データがない状態でディスクイメージを作成します。**#Find out if it's a disk image using "file" commandfile disk.imgdisk.img: Linux rev 1.0 ext4 filesystem data, UUID=59e7a736-9c90-4fab-ae35-1d6a28e5de27 (extents) (64bit) (large files) (huge files)#Check which type of disk image it'simg\_stat -t evidence.imgraw#You can list supported types withimg\_stat -i listSupported image format types:raw (Single or split raw file (dd))aff (Advanced Forensic Format)afd (AFF Multiple File)afm (AFF with external metadata)afflib (All AFFLIB image formats (including beta ones))ewf (Expert Witness Format (EnCase))#Data of the imagefsstat -i raw -f ext4 disk.imgFILE SYSTEM INFORMATION--------------------------------------------File System Type: Ext4Volume Name:Volume ID: 162850f203fd75afab4f1e4736a7e776Last Written at: 2020-02-06 06:22:48 (UTC)Last Checked at: 2020-02-06 06:15:09 (UTC)Last Mounted at: 2020-02-06 06:15:18 (UTC)Unmounted properlyLast mounted on: /mnt/disk0Source OS: Linux\[...]#ls inside the imagefls -i raw -f ext4 disk.imgd/d 11: lost+foundd/d 12: Documentsd/d 8193:       folder1d/d 8194:       folder2V/V 65537:      $OrphanFiles#ls inside folderfls -i raw -f ext4 disk.img 12r/r 16: secret.txt#cat file inside imageicat -i raw -f ext4 disk.img 16ThisisTheMasterSecret\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)**を使用して、世界で最も先進的なコミュニティツールによって強化されたワークフローを簡単に構築および自動化します。**\
**今すぐアクセスしてください：既知のマルウェアを検索変更されたシステムファイルLinuxには、潜在的に問題のあるファイルを見つけるために重要なシステムコンポーネントの整合性を確認するためのツールが用意されています。RedHatベースのシステム：包括的なチェックには`rpm -Va`を使用します。Debianベースのシステム：初期検証には`dpkg --verify`を使用し、その後`debsums | grep -v "OK$"`（`apt-get install debsums`を使用して`debsums`をインストールした後）を使用して問題を特定します。マルウェア/ルートキット検出ツールマルウェアを見つけるのに役立つツールについては、次のページを参照してください：インストールされたプログラムを検索DebianとRedHatの両方のシステムでインストールされたプログラムを効果的に検索するには、システムログやデータベースを活用し、一般的なディレクトリでの手動チェックを検討してください。Debianの場合、パッケージのインストールに関する詳細を取得するために、\_`/var/lib/dpkg/status`**_**と**_**`/var/log/dpkg.log`\_を調査し、`grep`を使用して特定の情報をフィルタリングします。RedHatユーザーは、`rpm -qa --root=/mntpath/var/lib/rpm`を使用してRPMデータベースをクエリし、インストールされたパッケージをリストします。これらのパッケージマネージャーの外で手動でインストールされたソフトウェアを特定するには、**_**`/usr/local`**_**、**_**`/opt`**_**、**_**`/usr/sbin`**_**、**_**`/usr/bin`**_**、**_**`/bin`**_**、\_`/sbin`\_などのディレクトリを調査します。ディレクトリリストをシステム固有のコマンドと組み合わせて使用し、既知のパッケージに関連付けられていない実行可能ファイルを特定することで、すべてのインストールされたプログラムを検索を強化します。**# Debian package and log detailscat /var/lib/dpkg/status | grep -E "Package:|Status:"cat /var/log/dpkg.log | grep installed# RedHat RPM database queryrpm -qa --root=/mntpath/var/lib/rpm# Listing directories for manual installationsls /usr/sbin /usr/bin /bin /sbin# Identifying non-package executables (Debian)find /sbin/ -exec dpkg -S {} \\; | grep "no path found"# Identifying non-package executables (RedHat)find /sbin/ –exec rpm -qf {} \\; | grep "is not"# Find exacuable filesfind / -type f -executable | grep \<something>\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)**を使用して、世界で最も高度なコミュニティツールによって強化されたワークフローを簡単に構築および自動化します。**\
**今すぐアクセスしてください：削除された実行中のバイナリの回復/tmp/exec から実行され、その後削除されたプロセスを想像してください。それを抽出することが可能です**cd /proc/3746/ #PID with the exec file deletedhead -1 maps #Get address of the file. It was 08048000-08049000dd if=mem bs=1 skip=08048000 count=1000 of=/tmp/exec2 #Recorver it**オートスタートの場所を調査するスケジュールされたタスク**cat /var/spool/cron/crontabs/\*  \\/var/spool/cron/atjobs \\/var/spool/anacron \\/etc/cron\* \\/etc/at\* \\/etc/anacrontab \\/etc/incron.d/\* \\/var/spool/incron/\* \\#MacOSls -l /usr/lib/cron/tabs/ /Library/LaunchAgents/ /Library/LaunchDaemons/ \~/Library/LaunchAgents/**サービスマルウェアがインストールされている可能性のあるサービスのパス：/etc/inittab：rc.sysinitなどの初期化スクリプトを呼び出し、さらに起動スクリプトに誘導します。/etc/rc.d/ および /etc/rc.boot/：サービスの起動スクリプトが含まれており、後者は古いLinuxバージョンに見られます。/etc/init.d/：Debianなどの特定のLinuxバージョンで起動スクリプトを保存するために使用されます。サービスは、Linuxのバリアントに応じて /etc/inetd.conf または /etc/xinetd/ からも起動される可能性があります。/etc/systemd/system：システムおよびサービスマネージャースクリプト用のディレクトリ。/etc/systemd/system/multi-user.target.wants/：マルチユーザーランレベルで起動する必要があるサービスへのリンクが含まれています。/usr/local/etc/rc.d/：カスタムまたはサードパーティのサービス用。\~/.config/autostart/：ユーザー固有の自動起動アプリケーション用であり、ユーザー向けのマルウェアの隠れた場所となる可能性があります。/lib/systemd/system/：インストールされたパッケージによって提供されるシステム全体のデフォルトユニットファイル。カーネルモジュールマルウェアによってルートキットコンポーネントとして利用されるLinuxカーネルモジュールは、システム起動時にロードされます。これらのモジュールにとって重要なディレクトリとファイルは次のとおりです：/lib/modules/$(uname -r)：実行中のカーネルバージョン用のモジュールを保持します。/etc/modprobe.d：モジュールのロードを制御する構成ファイルが含まれています。/etc/modprobe および /etc/modprobe.conf：グローバルモジュール設定用のファイル。その他の自動起動場所Linuxは、ユーザーログイン時に自動的にプログラムを実行するためのさまざまなファイルを使用しており、これらは潜在的にマルウェアを隠す可能性があります：/etc/profile.d/\*、/etc/profile、および /etc/bash.bashrc：すべてのユーザーログイン時に実行されます。\~/.bashrc、\~/.bash\_profile、\~/.profile、および \~/.config/autostart：ユーザー固有のファイルで、ユーザーのログイン時に実行されます。/etc/rc.local：すべてのシステムサービスが起動した後に実行され、マルチユーザー環境への移行の終了を示します。ログの調査Linuxシステムは、さまざまなログファイルを介してユーザーのアクティビティやシステムイベントを追跡します。これらのログは、不正アクセス、マルウェア感染、およびその他のセキュリティインシデントを特定するために重要です。主要なログファイルには次のものがあります：/var/log/syslog（Debian）または /var/log/messages（RedHat）：システム全体のメッセージとアクティビティをキャプチャします。/var/log/auth.log（Debian）または /var/log/secure（RedHat）：認証試行、成功および失敗したログインを記録します。`grep -iE "session opened for|accepted password|new session|not in sudoers" /var/log/auth.log` を使用して関連する認証イベントをフィルタリングします。/var/log/boot.log：システムの起動メッセージが含まれています。/var/log/maillog または /var/log/mail.log：メールサーバーのアクティビティを記録し、メール関連サービスの追跡に役立ちます。/var/log/kern.log：エラーや警告を含むカーネルメッセージを保存します。/var/log/dmesg：デバイスドライバーメッセージを保持します。/var/log/faillog：失敗したログイン試行を記録し、セキュリティ侵害の調査を支援します。/var/log/cron：cronジョブの実行を記録します。/var/log/daemon.log：バックグラウンドサービスのアクティビティを追跡します。/var/log/btmp：失敗したログイン試行を文書化します。/var/log/httpd/：Apache HTTPDのエラーおよびアクセスログが含まれています。/var/log/mysqld.log または /var/log/mysql.log：MySQLデータベースのアクティビティを記録します。/var/log/xferlog：FTPファイル転送を記録します。/var/log/：ここで予期しないログを常にチェックしてください。Linuxシステムのログと監査サブシステムは、侵入やマルウェアのインシデントで無効化または削除される可能性があります。Linuxシステムのログは一般的に悪意のある活動に関する最も有用な情報のいくつかを含んでいるため、侵入者は定期的にそれらを削除します。したがって、利用可能なログファイルを調査する際には、削除や改ざんの兆候となる欠落や順序の乱れを探すことが重要です。Linuxは各ユーザーのコマンド履歴を維持しており、以下に保存されています：\~/.bash\_history\~/.zsh\_history\~/.zsh\_sessions/\*\~/.python\_history\~/.\*\_historyさらに、`last -Faiwx` コマンドを使用してユーザーログインのリストを取得できます。未知または予期しないログインがあるかどうかを確認してください。追加の特権を付与できるファイルをチェックしてください：予期しないユーザー特権が付与されている可能性があるかどうかを確認するために、`/etc/sudoers` を確認してください。予期しないユーザー特権が付与されている可能性があるかどうかを確認するために、`/etc/sudoers.d/` を確認してください。異常なグループメンバーシップや権限を特定するために、`/etc/groups` を調べてください。異常なグループメンバーシップや権限を特定するために、`/etc/passwd` を調べてください。一部のアプリケーションは独自のログを生成することがあります：SSH：**_**\~/.ssh/authorized\_keys**_** および **_**\~/.ssh/known\_hosts**_** を調べて、許可されていないリモート接続を見つけます。Gnomeデスクトップ：Gnomeアプリケーションを介して最近アクセスされたファイルを示す **_**\~/.recently-used.xbel**_** を調べてください。Firefox/Chrome：怪しい活動を示すために、**_**\~/.mozilla/firefox**_** または **_**\~/.config/google-chrome**_** でブラウザの履歴とダウンロードをチェックしてください。VIM：アクセスされたファイルパスや検索履歴などの使用詳細を示す **_**\~/.viminfo**_** を確認してください。Open Office：侵害されたファイルを示す可能性のある最近のドキュメントアクセスをチェックしてください。FTP/SFTP：許可されていないファイル転送を示す **_**\~/.ftp\_history**_** または **_**\~/.sftp\_history**_** のログを確認してください。MySQL：実行されたMySQLクエリを示す **_**\~/.mysql\_history**_** を調査して、許可されていないデータベースアクティビティを明らかにすることができます。Less：表示されたファイルや実行されたコマンドなどの使用履歴を分析する **_**\~/.lesshst**_** を確認してください。Git：リポジトリへの変更を示す **_**\~/.gitconfig**_** およびプロジェクト **_**.git/logs**_** を調べてください。USBログ**[**usbrip**](https://github.com/snovvcrash/usbrip) **は、Linuxログファイル（ディストリビューションに応じて `/var/log/syslog*` または `/var/log/messages*`）を解析してUSBイベント履歴テーブルを作成する、Python 3で書かれた小さなソフトウェアです。使用されたすべてのUSBデバイスを把握することは重要であり、許可されたUSBデバイスのリストを持っていると、そのリストに含まれていないUSBデバイスの使用を見つけるのに役立ちます。インストール**pip3 install usbripusbrip ids download #Download USB ID database**例**usbrip events history #Get USB history of your curent linux machineusbrip events history --pid 0002 --vid 0e0f --user kali #Search by pid OR vid OR user#Search for vid and/or pidusbrip ids download #Downlaod databaseusbrip ids search --pid 0002 --vid 0e0f #Search for pid AND vid**更多示例和信息请查看github:** [**https://github.com/snovvcrash/usbrip**](https://github.com/snovvcrash/usbrip)**使用**[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) **可轻松构建和自动化工作流程，使用全球最先进的社区工具。**\
**立即获取访问权限:检查用户帐户和登录活动检查 **_**/etc/passwd**_**、**_**/etc/shadow**_** 和安全日志，查找异常名称或在已知未经授权事件附近创建或使用的帐户。还要检查可能的sudo暴力攻击。**\
**此外，检查文件如 **_**/etc/sudoers**_** 和 **_**/etc/groups**_**，查看是否给用户授予了意外的特权。**\
**最后，查找没有密码或易于猜测密码的帐户。检查文件系统在恶意软件调查中分析文件系统结构在调查恶意软件事件时，文件系统的结构是信息的重要来源，可以揭示事件序列和恶意软件的内容。然而，恶意软件作者正在开发技术来阻碍此分析，例如修改文件时间戳或避免使用文件系统进行数据存储。为了对抗这些反取证方法，重要的是：使用工具如Autopsy进行彻底的时间线分析，可视化事件时间线，或使用Sleuth Kit的`mactime`获取详细的时间线数据。检查系统的$PATH中的意外脚本，可能包括攻击者使用的shell或PHP脚本。检查`/dev`中的非典型文件，通常包含特殊文件，但可能包含与恶意软件相关的文件。搜索具有像".. "（点 点 空格）或"..^G"（点 点 控制-G）等名称的隐藏文件或目录，可能隐藏恶意内容。使用命令查找具有root权限的setuid文件: `find / -user root -perm -04000 -print`，这会找到具有提升权限的文件，可能会被攻击者滥用。检查inode表中的删除时间戳，以发现大量文件删除，可能表明存在rootkit或特洛伊木马。在识别一个恶意文件后，检查相邻的inode，因为它们可能被放在一起。检查常见的二进制目录（**_**/bin**_**、**_**/sbin**_**）中最近修改的文件，因为这些文件可能被恶意软件更改。**# List recent files in a directory:ls -laR --sort=time /bin\`\`\`# Sort files in a directory by inode:ls -lai /bin | sort -n\`\`\`**攻撃者は時間を変更してファイルを正規に見せることができますが、inodeを変更することはできません。同じフォルダ内の他のファイルと同じ時間に作成および変更されたことを示すファイルが見つかった場合、inodeが予期しないほど大きい場合、そのファイルのタイムスタンプが変更されたことになります。異なるファイルシステムバージョンのファイルを比較ファイルシステムバージョン比較の要約ファイルシステムバージョンを比較し変更点を特定するために、簡略化された`git diff`コマンドを使用します：新しいファイルを見つけるには、2つのディレクトリを比較します：**git diff --no-index --diff-filter=A path/to/old\_version/ path/to/new\_version/**変更されたコンテンツについては、特定の行を無視して変更点をリストアップします。**git diff --no-index --diff-filter=M path/to/old\_version/ path/to/new\_version/ | grep -E "^\\+" | grep -v "Installed-Time"**削除されたファイルを検出するために:**git diff --no-index --diff-filter=D path/to/old\_version/ path/to/new\_version/**フィルターオプション (`--diff-filter`) は、追加された (`A`)、削除された (`D`)、または変更された (`M`) ファイルなど、特定の変更を絞り込むのに役立ちます。`A`: 追加されたファイル`C`: コピーされたファイル`D`: 削除されたファイル`M`: 変更されたファイル`R`: 名前が変更されたファイル`T`: タイプの変更（例：ファイルからシンボリックリンクへ）`U`: マージされていないファイル`X`: 不明なファイル`B`: 破損したファイル参考文献**[**https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf**](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)[**https://www.plesk.com/blog/featured/linux-logs-explained/**](https://www.plesk.com/blog/featured/linux-logs-explained/)[**https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203**](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203)**書籍: Malware Forensics Field Guide for Linux Systems: Digital Forensics Field Guides**\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)**を使用して、世界で最も高度なコミュニティツールによって強化されたワークフローを簡単に構築および自動化できます。**\
**今すぐアクセスしてください：**