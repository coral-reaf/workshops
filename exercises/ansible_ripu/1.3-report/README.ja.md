# ワークショップ演習 - アップグレード前レポートのレビュー

## 目次

- [ワークショップ演習 - アップグレード前レポートのレビュー](#ワークショップ演習---アップグレード前レポートのレビュー)
  - [目次](#目次)
  - [目標](#目標)
  - [ガイド](#ガイド)
    - [Step 1 - Leapp アップグレード前結果の管理](#step-1---leapp-アップグレード前結果の管理)
    - [Step 2 - RHEL Web コンソールの操作](#step-2---rhel-web-コンソールの操作)
    - [Step 3 - RHEL8 ホストの Leapp アップグレード前レポートのレビュー](#step-3---RHEL8-ホストの-Leapp-アップグレード前レポートのレビュー)
    - [Step 4 - RHEL7 ホストの Leapp アップグレード前レポートのレビュー](#step-4---RHEL7-ホストの-Leapp-アップグレード前レポートのレビュー)
    - [チャレンジラボ: 多数のハイレベルの検出結果を無視するのはどうでしょうか?](#チャレンジラボ-多数のハイレベルの検出結果を無視するのはどうでしょうか)
  - [まとめ](#まとめ)

## 目標

* Leapp のアップグレード前レポートを管理するためのさまざまなオプションを理解する
* RHEL Web コンソールを使用して、生成したレポートを確認する
* アップグレード前レポートのエントリをフィルター処理する方法を学ぶ
* 失敗を受け入れよう!

## ガイド

### Step 1 - Leapp アップグレード前結果の管理

前の演習では、プレイブック ジョブ テンプレートを使用して、各ペット アプリ サーバーで Leapp のアップグレード前レポートを生成しました。次に、それらのレポートにリストされている結果を確認する必要があります。レポートにアクセスする方法はいくつかあります。これらを確認し、長所と短所を検討してみましょう。:

- Leapp フレームワークを使用して単一の RHEL ホストのみを手動でアップグレードする場合は、ホストでシェルプロンプトにアクセスして、ローカルレポートファイルの出力を確認するだけで済みます。 [Exercise 1.1, Step 2](../1.1-setup/README.ja.md#step-2---ターミナルセッションを開く) では、ペット アプリ サーバーの 1 つへの ssh セッションを開く方法を学習しました。これらの手順に従い、ログイン後、次のコマンドを使用してローカルの Leapp アップグレード前レポート ファイルを確認します:

  ```
  less /var/log/leapp/leapp-report.txt
  ```

  これはレポートを確認するための "手っ取り早い" 方法ですが、多数のホストのレポートを確認する必要がある場合には適していません。

  > **注記**
  >
  > 上下矢印キーを使用してファイルをスクロールし、`less` コマンドを終了する準備が出来たら `q` と入力します。

- RHEL ホストが [Red Hat Insights](https://www.redhat.com/en/technologies/management/insights) に登録されている場合、Insights コンソールで Leapp のアップグレード前レポートを確認できます。このワークショップで利用する、ペットアプリサーバーは Insights に登録されていないため、この演習で試すことはできませんがブログ記事 [Take the unknowns out of RHEL upgrades with Red Hat Insights](https://www.redhat.com/en/blog/take-unknowns-out-rhel-upgrades-red-hat-insights) を読んで、Insights を使用して Leapp のアップグレード前を確認および管理する方法の例を確認してください。

- RHEL には [Cockpit](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/managing_systems_using_the_rhel_8_web_console/index#what-is-the-RHEL-web-console_getting-started-with-the-rhel-8-web-console) に基づくオプションの管理 Web コンソールが含まれており、これを RHEL Web コンソールと呼んでいます。この演習の次のステップでは、RHEL Web コンソールを使用して Leapp のアップグレード前レポートを確認する方法について説明します。
 
- Leapp は、プレーン テキストの `leapp-report.txt` ファイルを書き込むだけでなく、JSON 形式の `leapp-report.json` ファイルも生成します。このファイルには、プレーンテキストファイルと同じレポート結果が含まれますが、Elastic/Kibana や Splunk などのログ管理ツールで取り込むのに最適な JSON 形式です。多くの大企業は、アップグレード前のレポート データをこれらのツールの 1 つにプッシュして、環境 (開発/テスト/本番など)、場所、アプリケーション ID、所有チームなどでレポートをフィルターできる独自のカスタム ダッシュボードを開発します。 <!-- FIXME: add Splunk example here when https://issues.redhat.com/browse/RIPU-35 gets done. -->

### Step 2 - RHEL Web コンソールの操作

このワークショップでは、RHEL Web コンソールを使用して、生成した Leapp アップグレード前レポートにアクセスします。

- [Exercise 1.1, Step 4](../1.1-setup/README.ja.md#step-4---rhel-web-コンソールへのアクセス) で開いた RHEL Web コンソールのブラウザー タブに戻ります。これは AAP コントローラー ホストの RHEL Web コンソールですが、アップグレード前レポートを表示するには、ペットアプリケーションサーバーホストにアクセスする必要があります。これを行うには、RHEL Web コンソールの左上隅にある "student&#8203;@&#8203;ansible-1.example.com" ボックスをクリックして、リモートホストメニューを表示します。例:

  ![Remote host menu listing all pet app servers](images/remote_host_menu_with_pets.svg)

- リモートホストメニューを使用して、各ペットアプリサーバーの Web コンソールに移動できます。今すぐペットサーバーの 1 つを選択してみてください。RHEL Web コンソールのシステム概要ページに、インストールされているオペレーティングシステムのバージョンが表示されます。たとえば、このペットアプリ サーバーは RHEL8 を実行しています:

  ![upward-moray running Red Hat Enterprise Linux 8.7 (Ootpa)](images/rhel8_os.svg)

  RHEL7 を実行している例を次に示します:

  ![Operating System Red Hat Enterprise Linux Server 7.9 (Maipo)](images/rhel7_os.svg)

- RHEL Web コンソールで別のホストに移動する場合、"limited access mode" の警告に注意してください:

  ![Web console is running in limited access mode](images/limited_access.svg)

  この警告が表示された場合は、続行する前にボタンを使用して管理アクセス モードに切り替えてください。 次のような確認が表示されます:

  ![You now have administrative access](images/administrative_access.svg)

- さまざまなペットアプリサーバーの RHEL Web コンソールで使用できるナビゲーション メニューを少し調べてください。コンソールの操作とホストの切り替えに慣れたら、次のステップに進み、最初のアップグレード前レポートを確認します。

### Step 3 - RHEL8 ホストの Leapp アップグレード前レポートのレビュー

これで、RHEL Web コンソールを使用して Leapp アップグレード前レポートを表示する準備ができました。まずは RHEL8 ホストの 1 つを見てから、次のステップで RHEL7 ホストの 1 つを見てみましょう。

RHEL7 または RHEL8 のみのアップグレードについて学習したいかもしれませんが、両方の演習手順に従うことをお勧めします。このワークショップでは、アップグレードする OS バージョンに関係なく知っておく必要のあるさまざまなトピックを網羅した RHEL7 と RHEL8 の例を使用して、必要なスキルを紹介します。

今、オートメーションアプローチワークフローのここにいます:

![Automation approach workflow diagram with review report step highlighted](images/ripu-workflow-hl-review.svg)

- RHEL Web コンソールのリモート ホスト メニューに移動し、RHEL8 ペット アプリ サーバーの 1 つのホスト名をクリックします。前のステップで学習したように、システム概要ページで RHEL バージョンを確認できることを覚えておいてください。また、前の手順で説明したように、管理アクセスが有効になっていることを確認してください。

- RHEL8 ペットアプリサーバーの 1 つを参照していることを確認したら、メインメニューから ツール > アップグレードレポート に移動します。これにより、選択したホスト用に生成された Leapp アップグレード前レポートが表示されます。たとえば、レポートは次のようになります。:

  ![Example pre-upgrade report of RHEL8 host](images/rhel8_report.svg)

  > **注記**
  >
  > このワークショップの作成以降にリリースされた Leapp フレームワーク、またはラボ環境における RHEL パッケージの更新により、レポートの内容が上記の例と異なる場合があります。恐らく皆さんの環境で表示されているものは上記とは異なるリスク一覧となっているのではと思いますが、以下、柔軟に自身のラボ環境で表示されているリスクに読み替えて確認いただければと思います。重要なポイントは、RHEL 8 では、阻害要因となるリスクは無い、つまりこのままアップデートを行っても良いと判断できるということです。それでも、ワークショップの演習の理解を大幅に妨げるクリティカルな違いが見つかった場合は、 [here](https://github.com/ansible/workshops/issues/new) で問題を提起してお知らせください。  

- アップグレード前レポートが生成されると、Leapp フレームワークはシステム データを収集し、多数のチェックに基づいてアップグレード可能性を評価します。これらのチェックのいずれかで潜在的なリスクが明らかになった場合、レポートに検出結果が記録されます。これらの検出結果は、リスクの高いものから低いものの順にリストされます。上記のレポートでは、リスクの高い検出結果が 3 つ（もしくは 4つ）あります。それぞれ確認してみましょう。

- 最初にリストされている検出結果のタイトルは "Leapp could not identify where GRUB core is located." です。リスト内の検出結果をクリックすると、その詳細を表示できます。たとえば、最初の検出結果をクリックすると、次の詳細が表示されます。:

  ![Details view of grub core finding](images/grub_core_finding.svg)

  この検出結果は、ワークショップ用にデプロイされた EC2 インスタンスに個別の /boot パーティションがないため報告されています。今のところは無視しますが、後の演習でチャレンジ ラボでこれを再度検討する可能性があるため、覚えておいてください。 <!-- We'll talk about fixing this in the commit playbook. -->

- 次の検出結果は "Remote root logins globally allowed using password." です。詳細を表示するには、以下をクリックしてください。:

  ![Details view of remote root logins finding](images/remote_root_logins_finding.svg)

  この検出結果は、RHEL9 で導入されたデフォルトのルート ログイン設定の変更について認識を高めることを目的としています。誰もがすでにベストプラクティスに従って、ルートユーザーとして直接ログインしないようになっているため、この検出結果は無視しても問題ありません。
  
- これで、最後の高リスクの検出結果に至りました。これは、実際には Leapp フレームワークの既知のバグであるため、少し恥ずかしいものです。

  ![Details view usage of deprecated model bug finding](images/leapp_bug_finding.svg)

  幸いなことに、これは完全に無害であるため、無視しても問題ありません。このバグは、まもなくリリースされる予定の Leapp フレームワークの更新で修正されます。 <!-- FIXME: remove this after the bug fix gets released. Also remove from RHEL7 report step further down. -->

- 幸いなことに、RHEL8 ホストの検出結果はどれも最も深刻な "inhibitor（阻害要因）" ではありませんでした。inhibitor が検出されると、RHEL のアップグレードはブロックされ、阻害要因のリスク検出の原因を修正する措置を最初に講じない限り、先に進めません。

- リスク レベル、対象者などに応じて表示される検出結果を制限するために使用できるフィルター オプションがいくつかあります。この機能を試すには、 "Filters" ボタンをクリックします。たとえば、 "Is inhibitor?" フィルターチェックボックスをクリックすると、該当となるものがないため、検出結果は表示されません。

- 次に、RHEL7 ホストの 1 つに関するアップグレード前のレポートに移りましょう。ネタバレ注意: このホストでは、inhibitor の検出結果に対処する必要があります！

### Step 4 - RHEL7 ホストの Leapp アップグレード前レポートのレビュー

前の手順では、RHEL8 ホストの 1 つについてアップグレード前のレポートを確認しました。次に、RHEL7 ホストの 1 つからのレポートを見てみましょう。

- RHEL Web コンソールのリモート ホスト メニューに移動し、RHEL7 ペットアプリサーバーの 1 つのホスト名をクリックします。選択したホストが RHEL7 であることを確認します。次に、メイン メニューを使用して Tools > に移動します。これにより、選択したホストの Leapp アップグレード前レポートが表示されます。たとえば、レポートは次のようになります。:

  ![Example pre-upgrade report of RHEL7 host](images/rhel7_report.svg)

  > **注記**
  >
  > このワークショップの作成後にリリースされた Leapp フレームワークおよびその他の RHEL パッケージの更新により、レポートの内容が上記の例と異なる場合があります。このワークショップの演習の流れを著しく妨げる相違点を発見した場合は、 [here](https://github.com/ansible/workshops/issues/new) で問題を提起してお知らせください。

- 上記の RHEL7 ペット アプリ サーバーのレポートには、6 つ（環境によって 7つの場合など違いがあります）のハイリスクの検出結果があり、そのうち 2 つは阻害要因の検出結果です。阻害要因ではない高リスクの検出結果を確認することから始めましょう。

- "GRUB core will be updated during upgrade" という検出結果は、RHEL8 のアップグレード前レポートで学んだ同じタイトルの検出結果と違いがないため、今のところは無視します。
 
- "Usage of deprecated Model" というハイリスクの検出結果は、前に説明した Leapp フレームワークのバグが原因です。これは煩わしいものですが、問題ないで無視します。

- 次に、RHEL7 のアップグレード前レポートでのみ確認できる新しい検出結果を見てみましょう。リストの一番上には、"Packages available in excluded repositories will not be installed" という結果があります。この結果をクリックして詳細ビューを表示すると、次のようになります:

  ![Details view of packages available in excluded repositories will not be installed](images/excluded_repos_finding.svg)

  この結果は、パッケージ python3-pyxattr と rpcgen は "アップグレード中に使用されるリポジトリのリストから意図的に除外されたターゲットシステムリポジトリでのみ利用可能," であるためアップグレードされないという警告ですが、詳細については "Excluded target system repositories" を確認するように促されています。左ペインに戻ってそのタイトルをクリックすると、詳細が表示されます:

  ![Details view of excluded target system repositories information finding](images/enablerepo_info_finding.svg)

  ここでは、修復のヒントで、`--enablerepo` オプションを指定して `leapp` ユーティリティを実行するように提案されています。しかし、ちょっと待ってください。これは、`leapp` コマンドを手動で実行していることを前提としています。心配しないでください。次の演習では、アップグレード Playbook ジョブを送信するときに変数を設定することでこのオプションを指定する方法を説明します。お楽しみに!

- リストの次のハイリスクエントリは、"Difference in Python versions and support in RHEL8" の検出結果です。:

  ![Details view of Difference in Python versions and support in RHEL8 finding](images/python_finding.svg)

  この検出結果は、システム提供の Python インタープリターを使用しているペット サーバー上のアプリがある場合に問題になる可能性があります。そのようなアプリがない場合は、この検出結果を無視できます。
  
- では、2 つの阻害要因を見てみましょう。1 つ目は、"Possible problems with remote login using root account" という検出結果です。結果をクリックすると詳細が表示されます: 

  ![Details view of possible problems with remote login using root account inhibitor finding](images/root_account_inhibitor.svg)

  阻害要因が検出された場合、この問題を解決しない限り Leapp フレームワークを利用した RHEL インプレースアップグレードが出来ないことに注意してください。

- もう 1 つの阻害要因は、"Missing required answers in the answer file" という検出です。これに関する詳細は次のとおりです。:

  ![Details view of missing required answers in the answer file](images/missing_answers_inhibitor.svg)

  ここでも、この検出を修正するための措置を講じる必要があります。慌てないでください。次の演習では、必要な修正措置と推奨事項を自動化するためのさまざまなオプションについて説明します。
  
### チャレンジラボ: 多数のハイレベルの検出結果を無視するのはどうでしょうか?

"Inhibitor" だけケアすればよいのか？と疑問に思うかもしれません。レポートに赤で表示されるその他のハイリスクの検出結果について・・・。 赤は危険を意味しますよね！レポートのすべての検出結果を解決せずにアップグレードを試行する理由は何でしょうか? ごもっともな質問です。

> **ヒント**
>
> ワークショップの冒頭で紹介した 4 つの主要な機能を思い出してください。

リスクの軽減に役立つ特定の機能はありますか?

> **警告**
>
> **解決策は以下\!**

もちろん、答えは自動スナップショット/ロールバック機能です。

- アップグレード前のレポートにリストされている高リスクの検出結果のいずれかが最終的にアップグレードの失敗につながったり、アプリケーションの互換性に影響を及ぼしたりした場合は、ロールバックすることですぐに元の状態に戻すことができます。

- *IEEE Software* に掲載された有名な記事  [Fail Fast](http://www.martinfowler.com/ieeeSoftware/failFast.pdf) に非常によく説明されている概念があります。この記事は 2004 年にさかのぼるので、これはまったく新しい概念ではありません。残念ながら、失敗には不名誉がつきもので、リスクを過度に回避する行動につながる可能性があります。自動スナップショットの最も重要な利点は、障害をすばやく元に戻せることです。これにより、安全に fail fast と fail smart のマントラを採用できます。

- もちろん、リスクを軽減するたのベストプラクティスはたくさんあります。最初に開発環境やテスト環境でアップグレードを試して、アプリケーションへの影響をテストすること。開発サーバーとテスト サーバーで問題が解決されていれば、運用環境でそれらの問題の発生が回避可能です。

- Leapp のアップグレード前レポートで報告されたハイリスクの調査結果は、潜在的な障害モードを認知させるためのものですが、経験上、多くの場合は問題にならないことがわかっています。レポートに赤い結果が表示されても、怖がらないことです。やってみて失敗したら元に戻す。まずは、恐れず、早めに頻繁にアップグレードすることを心がけてください。

## まとめ

この演習では、Leapp のアップグレード前レポートを管理するためのさまざまなオプションについて学習しました。RHEL Web コンソールを使用して、前の演習で生成したレポートを確認し、報告された結果のいくつかを確認しました。チャレンジ ラボでは、スナップショットの重要性について検討し、失敗を受け入れることを学びました。

次の演習では、阻害要因の解決に必要な修復アクションを自動化する方法を学びます。

---

**ナビゲーション**

[前の演習](../1.2-preupg/README.ja.md) - [次の演習](../1.4-remediate/README.ja.md)

[ホーム](../README.ja.md)
