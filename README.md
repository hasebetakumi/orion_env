# ORION開発環境構築

# 目次
[前提条件](#pre)<br>
[WSLインストール](#wsl)<br>
[Ubuntuインストール](#ubuntu)<br>
[WSLのバージョン変更](#wsl-v)<br>
[WSL2上でsystemctlを利用できるようにする](#pid)<br>
[SSH接続の準備](#pre-ssh)<br>
[TeratermインストールとSSH接続](#ssh)<br>
[Dockerインストール](#docker)<br>

<a id="pre"></a>
# 前提条件

参照リンク：<br>
[WSL を使用して Windows に Linux をインストールする](https://learn.microsoft.com/ja-jp/windows/wsl/install)<br>

Windows 10 バージョン 2004 以降

(Windouw11なら)<br>
Microsoft Store版のWSLを利用することで、PID1でのsystemdが公式に採用された。(2022/9/21)
(https://forest.watch.impress.co.jp/docs/news/1441775.html)
また、インボックス版のWSLをインストールする際は、wsl --installコマンド一つで、WSL2がインストールされUbuntuの起動まで自動で行われるそう。

<a id="wsl"></a>
# WSLインストール

参照リンク：<br>
[WSL2のインストールと分かりやすく解説【Windows10/11】](https://chigusa-web.com/blog/wsl2-win11/)<br>
[WSL(Windows10)にLinuxのUbuntuインストール](https://reffect.co.jp/windows/wsl-windows-ubuntu-install#Ubuntu-2)

1. PowerShellを管理者として実行
2. WSLをインストール
```
wsl --install
```
3. PC再起動
4. 「Windows の機能の有効化または無効化」を開く
5. 「Linux 用 Windows サブシステム」と「仮想マシン プラットフォーム」が有効になっていることを確認

<a id="ubuntu"></a>
# Ubuntuインストール

参照リンク：<br>
[WSL(Windows10)にLinuxのUbuntuインストール](https://reffect.co.jp/windows/wsl-windows-ubuntu-install#Ubuntu-2)

1. Microsoft StoreでUbuntuをインストール
2. 「複数のデバイスで使用する」は「必要ありません」
3. インストール完了後、「開くor起動」
4. 任意のユーザー名とパスワードを設定→Teratermからのパスワード認証で用いる
5. Ubuntuバージョン確認
```
more /etc/os-release
```
6. デフォルトのリポジトリを日本に変更
```
cd /etc/apt
```
```
sudo sed -i.bak -e "s%http://archive.ubuntu.com/ubuntu/%http://jp.archive.ubuntu.com/ubuntu/%g" sources.list
```
7. 最新情報取得
```
sudo apt update
```
8. パッケージの更新
```
sudo apt upgrade
```
9. 日本語化パッケージインストール
```
sudo apt install language-pack-ja
```
10. 日本語設定をjp_JP.UTF-8に設定
```
sudo update-locale LANG=ja_JP.UTF-8
```
11. 日本語反映のためにログアウト
```
exit
```
12. Ubuntu起動
13. dateコマンドを実行し、日本語になっているか確認
```
date
```

<a id="wsl-v"></a>
# WSLのバージョン変更

参照リンク：<br>
[WSLのバージョン確認・変更方法を現役エンジニアが解説](https://www.radical-dreamer.com/programming/wsl-version/)

1. wsl installでwsl2がインストールされていない恐れあり。PowerShellから、以下のコマンドでバージョン確認
```
wsl -l -v
```
2. VERSIONが「1」なら以下のコマンドで、バージョンを「2」にする。（結構時間かかりました）
```
wsl --set-version Ubuntu 2
```
  ※WSLのインストールの際に、「Linux 用 Windows サブシステム」と「仮想マシン プラットフォーム」を有効化したにも関わらず、エラーが出る場合がある。
  →BIOSから仮想化を有効化する必要がある可能性。メーカーや型番によって操作が異なるため、型番を検索しメーカーのマニュアルに従ってください。

3. 再度バージョン確認
```
wsl -l -v
```

<a id="pid"></a>
# WSL2上でsystemctlを利用できるようにする

参照リンク：<br>
[Windows 10 or 11 （WSL2）のUbuntuでsystemctlを利用する方法（systemdをPID1で動作させる方法）](https://snowsystem.net/other/windows/wsl2-ubuntu-systemctl/)<br>
[Install the .NET SDK or the .NET Runtime on Ubuntu](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu#2204)

1. Ubuntuコンソールからsystemctlが利用できるか確認する
```
systemctl
```
2. PIDのプロセス状態確認→PID1をsystemdに利用させる必要
```
ps aux
```
3. genieを利用するとsystemdをPID1で動作させることができる（https://github.com/arkane-systems/genie）
4. dotnet-runtime-6.0をインストール
```
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
```
```
sudo dpkg -i packages-microsoft-prod.deb
```
```
rm packages-microsoft-prod.deb
```

```
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-6.0
```

5. 残りの依存モジュールをインストール
```
sudo apt install -y daemonize dbus gawk libc6 libstdc++6 policykit-1 systemd systemd-container
```

6. wsl-transdebianのリポジトリ設定
```
sudo apt install apt-transport-https
```
```
sudo wget -O /etc/apt/trusted.gpg.d/wsl-transdebian.gpg https://arkane-systems.github.io/wsl-transdebian/apt/wsl-transdebian.gpg
```
```
sudo chmod a+r /etc/apt/trusted.gpg.d/wsl-transdebian.gpg
```
```
sudo cat << EOF > /etc/apt/sources.list.d/wsl-transdebian.list
deb https://arkane-systems.github.io/wsl-transdebian/apt/ focal main
deb-src https://arkane-systems.github.io/wsl-transdebian/apt/ focal main
EOF
```
（catコマンドで書き込めない場合は、viコマンドで直接書き込み）
```
sudo apt update
```

7. genieインストール
```
sudo apt install -y systemd-genie
```

8. genieを実行しsystemdをPID1にする
```
genie -s
```

9. systemdがPID1で稼働しているか確認する
```
ps aux
```

10. systemctlが実行できるか確認する
```
systemctl
```

11. 今後もgenieを起動していないと、systemctlコマンドは使用できない。以下のリンクにgenieを自動で起動する設定方法があるが、自分は試していない。

[Windows 10 or 11 （WSL2）のUbuntuでsystemctlを利用する方法（systemdをPID1で動作させる方法）](https://snowsystem.net/other/windows/wsl2-ubuntu-systemctl/)

<a id="pre-ssh"></a>
# SSH接続の準備

参照リンク：<br>
[Ubuntu 20.04 - SSHのインストールと接続方法](https://codechacha.com/ja/ubuntu-install-openssh/)<br>
[WSL上のUbuntuにTeraTermでログインしてみる](https://qiita.com/nnydtmg157/items/fc8bb2808ab318105116)

1. Open SSH Serverのインストール
```
sudo apt update
```
```
sudo apt install openssh-server
```

2. SSH鍵の生成
```
sudo ssh-keygen -A
```

3. パスワード認証の許可
```
sudo vi /etc/ssh/sshd_config
```
  PasswordAuthenticationの設定をyesに変更

4. SSH Serverの起動とその確認
```
sudo systemctl start ssh
```
```
sudo systemctl status ssh
```

<a id="ssh"></a>
# TeratermインストールとSSH接続

参照リンク：<br>
[【ゼロからわかる】Teratermのインストールと使い方](https://eng-entrance.com/teraterm-install)

1. IPアドレス確認。Ubuntuコンソール上で以下のコマンド実行。eth0: のinetを確認
```
ip a
```

2. Teratermダウンロード→こだわりなければ何も変更せずインストール
  https://ja.osdn.net/projects/ttssh2/releases/
  
3. デスクトップにアイコンが生成されればそこから起動。されなければ解凍されたフォルダから「ttermpro.exe」を起動

4. ホスト名に先ほど確認したIPアドレスを入れ、ポートは22のまま「OK」

5. 最初の接続時にはホスト認証の確認がある。「続行」をクリック

6. Ubuntuインストールの4. で設定したユーザー名とパスフレーズを入力し「OK」

7. setup→general→言語をJapaneseに、言語UIをJapanese.Ingに

7. 設定→その他の設定→ログからログの保存設定を行う

8. 標準ログファイル名「%Y%m%d_%H%M%S_&h.log」、標準ログ保存先フォルダはドキュメント下等の好きな場所を指定、「自動的にログ採取を開始する」にチェック、オプションは「追記」「プレーンテキスト」「タイムスタンプ」にチェックを入れて「OK」クリック

9. 設定→設定の保存→TERATERM.INIの上書きを行うことで、次回起動時からは設定が自動で呼ばれる

<a id="docker"></a>
# Dockerインストール

参照リンク：<br>
[WSL2のUbuntu 22.04でSystemdでDockerを起動させる](https://level69.net/archives/31296)<br>
[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

1. 公式ドキュメント通りにインストール
```
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
  
  恐らくここでdocker-ceの処理中にエラー。インストール自体はされているため、次からの手順を続ける<br>
  
  (2022年10月19日追記)<br>
  上記のエラーはインストール時ではなく、Dockerの起動時に発生。
  パッケージ関連のエラーであると推測する。
  https://qiita.com/Crow314/items/794cabf5603cc5938855
  以下のコマンドでパッケージエラーを調べられる。出力がなければエラーなし。
  ```
  sudo dpkg --audit
  ```
  パッケージエラー解消のために、以下のコマンドを実行。
  ```
  sudo dpkg --configure docker-ce
  ```
  
2. iptableをレガシーに設定
```
update-alternatives --set iptables /usr/sbin/iptables-legacy
```
```
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```
```
update-alternatives --config iptables
```
  type selection number: は「1」

3. Docker起動とその確認
```
sudo systemctl start docker
```
```
sudo systemctl status docker
```
```
docker ps
```

4. root権限を使用せずDockerを利用できるようにする。その後、再ログイン
```
sudo usermod -aG docker <your-user>
```
```
exit
```

5. 試しにnginx起動
```
docker run nginx
```

6. Orion学習コンテンツで利用するmongoDBを起動
```
docker run --name mongodb -d mongo:4.4
```

7. 起動しない場合はwindowsの設定→ネットワークとインターネット→プロキシ→自動プロキシセットアップの「設定を自動的に検出する」をオフに

