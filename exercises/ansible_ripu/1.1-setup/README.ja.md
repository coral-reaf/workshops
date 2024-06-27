# ワークショップ演習 - Lab 環境

## 目次

- [ワークショップ演習 - Lab 環境](#workshop-exercise---your-lab-environment)
  - [目次](#目次)
  - [目的](#目的)
  - [ガイド](#ガイド)
    - [Lab 環境](#lab-環境)
    - [Step 1 - 環境へのアクセス](#step-1---環境へのアクセス)
    - [Step 2 - ターミナルセッションを開く](#step-2---ターミナルセッションを開く)
    - [Step 3 - AAP Web UI へのアクセス](#step-3---aap-web-ui-へのアクセス)
    - [Step 4 - RHEL Web コンソールへのアクセス](#step-4---rhel-web-コンソールへのアクセス)
    - [Step 5 - チャレンジラボ](#step-5---チャレンジラボ)
  - [まとめ](#まとめ)

## 目的

* ラボのトポロジと環境へのアクセス方法を理解する
* ワークショップ演習の実施方法を理解する
* チャレンジラボを理解する

## ガイド

### Lab 環境

ワークショップでは、事前構成されたラボ環境がプロビジョニングされます。Ansible Automation Platform (AAP) でデプロイされたホストにアクセスして、RHEL インプレース アップグレード ワークフロー ステップを自動化するプレイブックとワークフロー ジョブを制御します。また、RHEL7 の 2 つと RHEL8 の 2 つの「ペット」アプリケーションホストにもアクセスします。これらは、RHEL オペレーティング システム (OS) をアップグレードするホストです。

| Role                 | Inventory name |
| ---------------------| ---------------|
| AAP Control Host     | ansible-1      |
| RHEL7 pet app host 1 | tidy-bengal    |
| RHEL7 pet app host 2 | strong-hyena   |
| RHEL8 pet app host 1 | more-calf      |
| RHEL8 pet app host 2 | upward-moray   |

> **注記**
>
> ペット アプリ ホストのインベントリ名は、上記の例とは異なるランダムなペット名になります。 <!-- FIXME: The workshop launch page provided by your instructor will list the names actually provisioned with your workshop instance. --> ランダムな名前を使用する理由については、後の演習で詳しく説明します。

### Step 1 - 環境へのアクセス

Visual Studio Code (VS Code) を使用します。これは、Web ブラウザーを使用してファイルを編集したり、ターミナル セッションにアクセスしたりするための便利で直感的な方法を提供するためです。コマンド ラインに精通していて、VS Code が気に入らない場合は、直接 SSH アクセスを利用できます。さらに明確にする必要がある場合は、YouTube の短い説明ビデオがあります: <a href="https://youtu.be/Y_Gx4ZBfcuk">Ansible Workshops - Accessing your workbench environment</a>.

- 講師から提供されたワークショップ起動ページの「VS Code アクセス」の下にある「WebUI」リンクを使用して、Web ブラウザーで VS Code を開くことができます。パスワードはリンクの下に記載されています。例:

  ![Example link to VS Code WebUI](images/vscode_link.png)

- リンクを開いたら、提供されたパスワードを入力して VS Code のインスタンスにアクセスします。

> **注記**
>
> VS Code ユーザー エクスペリエンスの構成をガイドする Welcome ウィザードが表示される場合があります。このワークショップではデフォルト設定で問題なく機能するため、これはオプションです。自由にウィザードを操作して VS Code のさまざまな機能を確認したり、スキップしたりしてください。

### Step 2 - ターミナルセッションを開く

ターミナル セッションでは、RHEL コマンドとユーティリティにアクセスできるため、自動化による RHEL インプレース アップグレードが行われているときに「舞台裏で」何が起こっているかを把握するのに役立ちます。

- VS Code を使用してターミナル セッションを開きます。例:

  ![Example of how to open a terminal session in VS Code](images/new_term.svg)

- このターミナル セッションは、AAP コントロール ホスト `ansible-1` で実行されます。 `cat /etc/hosts` コマンドを使用して、ペット アプリ ホストのホスト名を確認します。次に、 `ssh` コマンドを使ってペットアプリホストの 1 つにログインします。最後に、強調表示されたコマンドを使用して、インストールされている RHEL OS バージョンとカーネル バージョンを確認します。

  例:

  ![Example ssh login to pet app host](images/ssh_login.svg)

- 上記の例では、 `ssh tidy-bengal` コマンドは、指定されたペット アプリ ホスト上の新しいセッションに接続します。次に、 `cat /etc/redhat-release` と `uname -r` を使ってそのホストからOSのリリース情報 `Red Hat Enterprise Linux Server release 7.9 (Maipo)` とカーネルバージョン `3.10.0-1160.88.1.el7.x86_64` を出力しています。

### Step 3 - AAP Web UI へのアクセス

AAP Web UI は、RHEL インプレースアップグレードワークフローを自動化するために使用する Ansible Playbook ジョブの送信とステータスの確認を行う場所です。

- "Automation controller" の "Console" の URL リンクをクリックし AAP の Web UI を開きます。例:

  ![Example link to AAP Web UI](images/aap_link.png)

- ユーザー名 `admin` と表示されているパスワードを使ってログインします。これにより、次の例のように AAP Web UI ダッシュボードが表示されます。:

  ![Example AAP Web UI dashboard](images/aap_console_example.svg)

- 次の演習では、AAP Web UI の使用方法について詳しく学習します。

### Step 4 - RHEL Web コンソールへのアクセス

RHEL Web コンソールを使用して、ペット アプリ サーバー用に生成した Leapp アップグレード前レポートの結果を確認します。

- "RHEL Web Console" へのリンクをクリックし、新しい Web ブラーザータブを開きます。例:

  ![Example link to RHEL Web Console](images/cockpit_link.png)

- ユーザー名 `student` と表示されているパスワードを使ってログインします。これにより、次の例のような RHEL Web コンソールの概要ページが表示されます。:

  ![Example RHEL Web Console](images/cockpit_example.svg)

- 今後の演習でアップグレード前のレポートを確認する準備ができたら、RHEL Web コンソールを再度確認します。

### Step 5 - チャレンジラボ

ワークショップの多くの演習には "Challenge Lab" ステップがあることにすぐに気づくでしょう。これらのラボは、これまでに学習した内容を使用して解決する小さなタスクを提供することを目的としています。タスクの解決方法は、警告サインの下に表示されます。

## まとめ

この演習では、ワークショップの演習を続けるために使用するラボ環境について学習しました。Web ブラウザーで VS Code を使用でき、そこからターミナル セッションを開くことができることを確認しました。また、RHEL インプレース アップグレード自動化ワークフローの次の手順を実行するために使用する「セルフサービス ポータル」となる AAP Web UI にアクセスできることも確認しました。最後に、RHEL Web コンソールに接続し、すぐにアップグレード前のレポートを確認します。

次の演習に進むには、以下のリンクを使用してください。

---

**ナビゲーション**

[Next Exercise](../1.2-preupg/README.ja.md)

[Home](../README.ja.md)
