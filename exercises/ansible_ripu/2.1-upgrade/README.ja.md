# ワークショップ演習 - OS アップグレード ジョブの実行

## 目次

- [ワークショップ演習 - OS アップグレードジョブの実行](#workshop-exercise---os-アップグレードジョブの実行)
  - [目次](#目次)
  - [目標](#目標)
  - [ガイド](#ガイド)
    - [Step 1 - アップグレードワークフロージョブテンプレートの起動](#step-1---アップグレードワークフロージョブテンプレートの起動)
    - [Step 2 - Leapp の詳細の確認](#step-2---leapp-の詳細の確認)
  - [まとめ](#まとめ)

## 目的

* ワークフロージョブ テンプレートを使用してスナップショットを作成し、アップグレードを開始する
* Leapp フレームワークが RHEL OS をアップグレードする方法について学習する

## ガイド

RHEL インプレースアップグレードオートメーションフェーズを開始する準備ができました:

![自動化アプローチ ワークフロー図アップグレード手順が強調表示されています](images/ripu-workflow-hl-upgrade.svg)

このフェーズでは、ワークフロー ジョブ テンプレートを使用してアップグレード プレイブックが実行されます。最初のプレイブックは、アップグレードで問題が発生した場合にロールバックするために使用できるスナップショットを作成します。スナップショットが作成されると、2 番目のプレイブックは Leapp ユーティリティを使用して、ホストを新しい RHEL メジャー バージョンにアップグレードするアップグレードを実行します。

### Step 1 - アップグレード ワークフロー ジョブ テンプレートを起動する

ペットアプリケーションサーバーの RHEL インプレース アップグレードを開始します。アップグレード中は、ホストにログインしたりアプリケーションにアクセスしたりすることはできません。アップグレードが完了すると、ホストは新しくアップグレードされた RHEL メジャー バージョンで再起動します。

アップグレードには通常 1 時間もかかりませんが、シャットダウンが遅いアプリケーションや、再起動サイクルが長いベア メタル ホストがある場合は、さらに時間がかかることがあります。ワークショップ ラボ環境用にプロビジョニングされたクラウド インスタンスは、従来のエンタープライズ アプリケーション サーバーに比べて非常に軽量であるため、かなり迅速にアップグレードされます。

- Web ブラウザの AAP Web UI タブに戻ります。リソース > テンプレート に移動して、"AUTO / 02 アップグレード" ジョブ テンプレートを開きます。次のようになります。

![アップグレード ジョブ テンプレートの詳細ビューを表示する AAP Web UI](images/upgrade_template.svg)

- "起動" ボタンをクリックすると、変数プロンプトから始まるジョブを送信するためのプロンプトが表示されます。

![AAP Web UI のアップグレード ジョブ変数プロンプト](images/upgrade_vars_prompt.svg)

- 変数設定を変更する必要はないので、[次へ] ボタンをクリックして先に進みます。

![AAP Web UI のアップグレード ジョブ サーベイ プロンプト](images/upgrade_survey_prompt.svg)

- 次に、インベントリ グループを選択するように求めるジョブ テンプレート サーベイ プロンプトが表示されます。 1 つのジョブを使用してすべてのペット アプリ ホストをアップグレードするため、"ALL_rhel" オプションを選択し、"次へ" ボタンをクリックします。これにより、選択したジョブ オプションと変数設定のプレビューが表示されます。

![AAP Web UI でのアップグレード ジョブのプレビュー](images/upgrade_preview.svg)

- ジョブ プレビューに問題がなければ、「起動」ボタンを使用してジョブを開始します。

### Step 2 - Learn More About Leapp

After launching the upgrade job, the AAP Web UI will navigate automatically to the workflow job output page of the job we just started. This job will take up to 20 minutes to finish, so let's take this time to learn a little more about how the Leapp framework upgrades your OS to next RHEL major version.

- Keep in mind that the Leapp framework is responsible only for upgrading the RHEL OS packages. Additional tasks required for upgrading your standard agents, tools, middleware, etc., need to be included in the upgrade playbooks you develop to deal with the specific requirements of your organization's environment.

- The Leapp framework performs the RHEL in-place upgrade by following a sequence of phases. These phases are represented in the following diagram:

  ![Leapp upgrade flow diagram](images/inplace-upgrade-workflow-gbg.svg)

- The steps of the RHEL in-place upgrade are implemented in modules known as Leapp actors. The Leapp framework is message-driven. The execution of actors is dependent on the data produced by other actors running before them. Actors running in the early phases scan the system to produce messages that add findings to the pre-upgrade report as well as messages that later actors use to make decisions or apply changes during the upgrade.

- Each phase includes three defined stages: before, main, and after. Before and after stages are used to further refine when an actor will be run in relation to any other actors in the phase. Actors are tagged to define the phase and stage during which they are to run.

- There are three groups of phases: Old System, Interim System, and New System. Phases under the Old System group run under the existing RHEL installed version. The Interim System phases starts after the InitRamStart phase reboots the host to an upgrade initramfs environment under which the network and other services are not started. It is at this time that all RHEL packages can be upgraded. Once all the packages are upgraded, another reboot brings the host up under the new RHEL major version and the FirstBoot phase starts. This final phase runs a few post-upgrade actors that require network access and then the upgrade is done.

- Being aware of these phases helps if you need to troubleshoot an issue during the Leapp upgrade and is especially important if you are planning to develop any Leapp custom actors. You can learn more about the Leapp framework architecture and internal design by reading the upstream [Leapp Developer Documentation](https://leapp.readthedocs.io/en/latest/index.html).

## Conclusion

In this exercise, we launched a workflow job template to create snapshots and start the upgrades of our pet app servers. We learned more about the Leapp framework to better understand what is happening as the RHEL OS is being upgraded.

In the next exercise, we'll learn more about how snapshots work.

---

**Navigation**

[Previous Exercise](../1.6-my-pet-app/README.md) - [Next Exercise](../2.2-snapshots/README.md)

[Home](../README.md)
