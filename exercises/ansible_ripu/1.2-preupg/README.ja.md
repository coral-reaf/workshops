# ワークショップ演習 - プリアップグレードジョブの実行

## 目次

- [ワークショップ演習 - プリアップグレードジョブの実行](#ワークショップ演習---プリアップグレードジョブの実行)
  - [目次](#目次)
  - [目標](#目的)
  - [ガイド](#ガイド)
    - [Step 1 - RHEL インプレースアップグレードオートメーションワークフロー](#step-1---rhel-インプレースアップグレードオートメーションワークフロー)
      - [分析](#分析)
      - [アップグレード](#upgrade)
      - [コミット](#コミット)
      - [始めましょう](#始めましょう)
    - [Step 2 - AAP を使った分析 Playbook ジョブの起動](#step-2---aap-を使った分析-playbook-ジョブの起動)
    - [Step 3 - Playbook ジョブ出力のレビュー](#step-3---playbook-ジョブ出力のレビュー)
    - [Step 4 - チャレンジ ラボ: 分析 Playbook](#step-4---チャレンジラボ-分析-playbook)
  - [結論](#結論)


## 目標

* エンドツーエンドの RHEL インプレース アップグレード ワークフローを理解する
* AAP ジョブ テンプレートを使用して Ansible プレイブックを実行する方法を理解する
* アップグレード前の分析ジョブを実行する

## ガイド

### Step 1 - RHEL インプレースアップグレードオートメーションワークフロー

Red Hat Enterprise Linux (RHEL) には、Leapp ユーティリティが付属しています。これは、自動化アプローチでオペレーティング システムを次のメジャー バージョンにアップグレードするために使用する基盤となるフレームワークです。[Leapp documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/upgrading_from_rhel_7_to_rhel_8/index) では、Leapp フレームワークを使用して RHEL ホストを手動でアップグレードする方法について説明しています。アップグレードする RHEL ホストが数台しかない場合はこれで問題ありませんが、数万台の RHEL ホストがある大企業の場合はどうでしょうか。手動プロセスは拡張できません。自動化を使用すると、RHEL ホストのアップグレードのエンドツーエンドのプロセスは数日に短縮され、実際のアップグレードに必要なダウンタイムの合計は数時間以内になります。

当社の RHEL インプレース アップグレードオートメーションのアプローチは、3 つのフェーズから成るワークフローに従います。:

![3 フェーズ ワークフロー: Analysis, Upgrade, Commit](images/ripu-workflow.svg)

> **注記**
>
> <sub>![arrow pointing down at server](images/playbook_icon.svg)</sub> は、Ansible Playbook によって自動化されるワークフロー ステップを示します。

#### 分析

Analysis Phase では、まだ変更は行われません。分析 Playbook が実行されると、Leapp ユーティリティを使用して、アップグレードの成功を妨げる可能性のある問題や障害がホストでスキャンされます。次に、見つかった潜在的なリスクをリストした詳細なレポートが生成されます。レポートには、報告された問題がアップグレードに影響を与える可能性を減らすために従うべき推奨アクションも含まれています。推奨される修復アクションが実行された場合は、リスクが解決されていることを確認するために分析スキャンを再度実行する必要があります。この反復は、レポートを確認する全員が残りの調査結果が許容できると確信するまで続けられます。

<!-- The following only applies when using the Ansible role being developed for LVM snapshots...

 In addition to upgrade risks that could impact the success of the upgrade, the report also indicates if there is enough free space to support the snapshot configuration required in case rolling back is required. If there is not enough free space, temporarily space should be made available, for example, by adding an additional virtual disk to the rootvg volume group. Removing /var/crash or other non-critical filesystems under the rootvg volume group is another option. It is strongly recommended to to make space so that a snapshot rollback is possible just in case.
-->

#### アップグレード

Analysis Phase が完了し、レポートが許容可能なリスク内に収まったら、メンテナンスウィンドウをスケジュールして Upgrade Phase を開始することができます。このフェーズでは、ワークフロージョブ テンプレートを使用してアップグレード Playbook が実行されます。最初の Playbook は、アップグレードで問題が発生した場合にロールバックに使用できるスナップショットを作成します。スナップショットが作成されると、2 番目のプレイブックは Leapp ユーティリティを使用してアップグレードを実行し、RHEL OS を新しいメジャー バージョンに進めます。アップグレード中は、ホストにログインしたりアプリケーションにアクセスしたりすることはできません。アップグレードが完了すると、ホストは新しくアップグレードされた RHEL メジャー バージョンで再起動します。これで、運用チームとアプリケーション チームは、すべてのアプリケーション サービスが期待どおりに動作していることを確認し、アップグレードが成功したかどうかを評価できます。

#### コミット

スケジュールされたメンテナンス ウィンドウ内で簡単に修正できないアプリケーションへの影響が見つかった場合は、スナップショットをロールバックしてアップグレードを元に戻す決定を下すことができます。これにより、すべての変更が元に戻り、ホストが以前の RHEL バージョンに戻ります。すぐに問題が見つからない場合は、コミット フェーズが開始されます。コミット フェーズでは、後で問題が見つかった場合に備えてスナップショットを保持しながら、ホストを通常の操作に戻すことができます。 <!-- This is LVM specific: However, while the snapshots are kept, regular disk writes to the rootvg volume group will continue to consume the free space allocated to the snapshots. The amount of time this takes will depend on the amount of free space initially available and the volume of write i/o activity to the rootvg volume group. Before the snapshot space is exhausted, the snapshots must be deleted and then there is no turning back. --> アップグレードされたホストが問題ないことが確認されたら、スナップショットを削除します。これで RHEL インプレースアップグレードが完了となります。

#### 始めましょう

RHEL インプレースアップグレードオートメーションアプローチを実現するワークフローは、インプレースアップグレードと新しい RHEL ホストの展開に伴うリスクを軽減するように設計されています。分析フェーズとアップグレードフェーズの最後にある決定ポイントにより、レポートチェックと実際のアップグレード結果から学んだ教訓を活用して、プロセスをロールバックして再開できます。もちろん、本番環境への影響や停止を回避するためのベストプラクティスは、本番環境ホストに移行する前に、適切に構成された開発環境とテスト環境でアップグレードを進めることです。


### Step 2 - AAP を使った分析 Playbook ジョブの起動

ワークショップを進めるにつれて、この図を参照して、自動化アプローチ ワークフローのどこにいるかを追跡します。現在は、以下の強調表示されたブロックから開始しています。:

![Automation approach workflow diagram with analysis step highlighted](images/ripu-workflow-hl-analysis.svg)

ペット アプリ ホストをアップグレードする最初のステップは、分析プレイブックを実行して、各ホストの Leapp アップグレード前レポートを生成することです。これを行うには、ワークショップ ラボ環境で事前構成されている Ansible Automation Platform (AAP) 自動化コントローラー ホストを使用します。

- 前の演習の手順 3 で開いた AAP Web UI ブラウザー タブに戻ります。ナビゲーション メニューの "リソース" グループの下にある "テンプレート" をクリックして、リソース > テンプレートに移動します。これにより、ターゲット ホストで Playbook ジョブを実行するために使用できるジョブ テンプレートのリストが表示されます。:

  ![Job templates listed on AAP Web UI](images/aap_templates.svg)

- "AUTO / 01 Analysis" ジョブテンプレートをクリックします。これにより、ジョブテンプレートの詳細タブが表示されます。:

  !["AUTO / 01 Analysis" job templates seen on AAP Web UI](images/analysis_template.svg)

- ここから、ジョブ テンプレートに変更を加える場合は "編集" ボタンを使用できます。このジョブ テンプレートはすでに構成されているため、これを使用してプレイブック ジョブを送信する準備が整いました。これを行うには、"起動" ボタンを使用して一連のプロンプトを表示します。

  > **注記**
  >
  > 各ジョブ テンプレートに表示されるプロンプトは、ジョブ テンプレートの編集時に表示される "起動時にプロンプ​​トを表示" チェックボックスを使用して構成できます。

  ![Analysis job variables prompt on AAP Web UI](images/analysis_vars_prompt.svg)

- 上記の最初のプロンプトでは、デフォルトのプレイブック変数を変更したり、変数を追加したりできます。この時点でこれを行う必要はありません。"次へ" ボタンをクリックして先に進みます。

  ![Analysis job survey prompt on AAP Web UI](images/analysis_survey_prompt.svg)

- 次に、ジョブ テンプレート サーベイ プロンプトが表示されます。サーベイは、ジョブ テンプレートの サーベイタブから構成できるカスタマイズ可能な一連のプロンプトです。このジョブ テンプレートでは、ジョブがプレイブックを実行するホストのグループを調査で選択できます。"ALL_rhel" オプションを選択し、"次へ" ボタンをクリックします。選択したジョブ オプションと変数設定のプレビューが表示されます。

  ![Analysis job preview on AAP Web UI](images/analysis_preview.svg)

- ジョブ プレビューに問題がなければ、"起動" ボタンを使用してプレイブック ジョブを開始します。

### Step 3 - Playbook ジョブ出力のレビュー

分析 Playbook ジョブを起動すると、AAP Web UI は、開始したジョブのジョブ出力ページに自動的に移動します。

- Playbook ジョブの実行中は、"フォロー" ボタンをクリックして進行状況を監視できます。フォローモードの場合、タスクの結果が AAP Web UI に表示されるジョブ出力の下部にストリーミングされると、出力が自動的にスクロールします。

- 分析 Playbook は、Leapp のアップグレード前スキャンを実行します。完了するまでに約 2 ～ 3 分かかります。完了すると、ジョブ出力の最後に、各ホストで実行された Playbook 実行の成功または失敗のステータスを示す "PLAY RECAP" が表示されます。ステータスが "failed=0" の場合、Playbook の実行が成功したことを示します。ジョブ出力の一番下までスクロールすると、ジョブの概要が次の例のようになっていることがわかります。:

  ![Analysis job "PLAY RECAP" as seen at the end of the job output](images/analysis_job_recap.svg)

### Step 4 - チャレンジ ラボ: 分析 Playbook

実行したばかりの Playbook を詳しく見てみましょう。

> **ヒント**
>
> "Project Leapp" プロジェクトと"AUTO / 01 Analysis"ジョブテンプレートの構成の詳細を確認してみてください。

アップストリーム ソース リポジトリと Playbook コードを見つけることができますか?

> **警告**
>
> **解決策は以下\!**

- AAP Web UI で、"リソース" > "プロジェクト" > "Project Leapp" に移動します。"詳細" タブの下に、"ソース コントロール URL" 設定が表示されます。この設定では、このプロジェクトのジョブ テンプレートが Playbook をプルする場所が定義されています。 GitHub のこの Git リポジトリを指していることがわかります: [https://github.com/redhat-partner-tech/leapp-project](https://github.com/redhat-partner-tech/leapp-project). この URL を新しいブラウザ タブで開きます。それをクリックすると、Playbook の内容が表示されます。

- AAP Web UI に戻り、リソース > テンプレート > AUTO / 01 分析 に移動します。"詳細" タブの下に、ジョブの送信に使用されるときにこのジョブ テンプレートが実行する Playbook の名前を含む " Playbook" 設定が表示されます。 Playbookの名前は `analysis.yml` です。GitHub ブラウザ タブで、git リポジトリのファイルにリストされている `analysis.yml` を見つけることが出来ます。

- Playbookの `Run RIPU preupg` が `infra.leapp` Ansible コレクションからロールをインポートしていることに注意してください。Git リポジトリの `collections/requirements.yml` ファイルを確認すると、このロールが [https://github.com/redhat-cop/infra.leapp](https://github.com/redhat-cop/infra.leapp)  にある別の Git リポジトリから来ていることがわかります。この 2 番目の Git リポジトリの `analysis` ロールは、最終的に Leapp のアップグレード前スキャンを実行してレポートを生成するすべての自動化タスクを提供します。

- この Git リポジトリの `roles/analysis` ディレクトリにドリルダウンして、README および yaml ソース ファイルを確認します。

エンタープライズのアップグレードを実行するための独自のカスタム  Playbookを開発する準備ができたら、作業を容易にするために `infra.leapp` Ansible コレクションのロールの使用を検討する必要があります。

## まとめ

この演習では、RHEL インプレース アップグレードを実行するための自動化アプローチで使用されるエンドツーエンドのワークフローについて学習しました。 AAP のジョブ テンプレートを使用して、ペット アプリケーション サーバーで Leapp のアップグレード前分析を実行する Playbook ジョブを送信しました。チャレンジ ラボでは、実行した Playbookと、その Playbookにアップストリーム Ansible コレクションのロールがどのように含まれているかを調べました。

次の演習では、生成したアップグレード前レポートを確認し、特定された高リスクの検出結果を解決するための措置を講じます。

---

**ナビゲーション**

[前の演習](../1.1-setup/README.ja.md) - [次の演習](../1.3-report/README.ja.md)

[ホーム](../README.ja.md)
