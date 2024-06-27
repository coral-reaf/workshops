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

当社の RHEL インプレース アップグレード自動化アプローチは、3 つのフェーズから成るワークフローに従います。:

![3 フェーズ ワークフロー: Analysis, Upgrade, Commit](images/ripu-workflow.svg)

> **注記**
>
> <sub>![arrow pointing down at server](images/playbook_icon.svg)</sub> は、Ansible Playbook によって自動化されるワークフロー ステップを示します。

#### 分析

分析フェーズでは、まだ変更は行われません。分析 Playbook が実行されると、Leapp ユーティリティを使用して、アップグレードの成功を妨げる可能性のある問題や障害がホストでスキャンされます。次に、見つかった潜在的なリスクをリストした詳細なレポートが生成されます。レポートには、報告された問題がアップグレードに影響を与える可能性を減らすために従うべき推奨アクションも含まれています。推奨される修復アクションが実行された場合は、リスクが解決されていることを確認するために分析スキャンを再度実行する必要があります。この反復は、レポートを確認する全員が残りの調査結果が許容できると確信するまで続けられます。

<!-- The following only applies when using the Ansible role being developed for LVM snapshots...

 In addition to upgrade risks that could impact the success of the upgrade, the report also indicates if there is enough free space to support the snapshot configuration required in case rolling back is required. If there is not enough free space, temporarily space should be made available, for example, by adding an additional virtual disk to the rootvg volume group. Removing /var/crash or other non-critical filesystems under the rootvg volume group is another option. It is strongly recommended to to make space so that a snapshot rollback is possible just in case.
-->

#### アップグレード

分析フェーズが完了し、レポートで許容できるリスクが示されたら、メンテナンス ウィンドウをスケジュールしてアップグレード フェーズを開始できます。このフェーズでは、ワークフロー ジョブ テンプレートを使用してアップグレード プレイブックが実行されます。最初のプレイブックは、アップグレードで問題が発生した場合にロールバックに使用できるスナップショットを作成します。スナップショットが作成されると、2 番目のプレイブックは Leapp ユーティリティを使用してアップグレードを実行し、RHEL OS を新しいメジャー バージョンに進めます。アップグレード中は、ホストにログインしたりアプリケーションにアクセスしたりすることはできません。アップグレードが完了すると、ホストは新しくアップグレードされた RHEL メジャー バージョンで再起動します。これで、運用チームとアプリケーション チームは、すべてのアプリケーション サービスが期待どおりに動作していることを確認し、アップグレードが成功したかどうかを評価できます。

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

- Click on the "AUTO / 01 Analysis" job template. This will display the Details tab of the job template:

  !["AUTO / 01 Analysis" job templates seen on AAP Web UI](images/analysis_template.svg)

- From here, we could use the "Edit" button if we wanted to make any changes to the job template. This job template is already configured, so we are ready to use it to submit a playbook job. To do this, use the "Launch" button which will bring up a series of prompts.

  > **Note**
  >
  > The prompts that each job template presents can be configured using the "Prompt on launch" checkboxes seen when editing a job template.

  ![Analysis job variables prompt on AAP Web UI](images/analysis_vars_prompt.svg)

- The first prompt as seen above allows for changing the default playbook variables or adding more variables. We don't need to do this at this time, so just click the "Next" button to move on.

  ![Analysis job survey prompt on AAP Web UI](images/analysis_survey_prompt.svg)

- Next we see the job template survey prompt. A survey is a customizable set of prompts that can be configured from the Survey tab of the job template. For this job template, the survey allows for choosing a group of hosts on which the job will execute the playbook. Choose the "ALL_rhel" option and click the "Next" button. This will bring you to a preview of the selected job options and variable settings.

  ![Analysis job preview on AAP Web UI](images/analysis_preview.svg)

- If you are satisfied with the job preview, use the "Launch" button to start the playbook job.

### Step 3 - Review the Playbook Job Output

After launching the analysis playbook job, the AAP Web UI will navigate automatically to the job output page for the job you just started.

- While the playbook job is running, you can monitor its progress by clicking the "Follow" button. When you are in follow mode, the output will scroll automatically as task results are streamed to the bottom of job output shown in the AAP Web UI.

- The analysis playbook will run the Leapp pre-upgrade scan. This will take about two or three minutes to complete. When it is done, you can find a "PLAY RECAP" at the end of the job output showing the success or failure status for the playbook runs executed on each host. A status of "failed=0" indicates a successful playbook run. Scroll to the bottom of the job output and you should see that your job summary looks like this example:

  ![Analysis job "PLAY RECAP" as seen at the end of the job output](images/analysis_job_recap.svg)

### Step 4 - Challenge Lab: Analysis Playbook

Let's take a closer look at the playbook we just ran.

> **Tip**
>
> Try looking at the configuration details of the "Project Leapp" project and the "AUTO / 01 Analysis" job template.

Can you find the upstream source repo and playbook code?

> **Warning**
>
> **Solution below\!**

- In the AAP Web UI, navigate to Resources > Projects > Project Leapp. Under the Details tab, you will see the "Source Control URL" setting that defines where job templates of this project will go to pull their playbooks. We see it is pointing to this git repo on GitHub: [https://github.com/redhat-partner-tech/leapp-project](https://github.com/redhat-partner-tech/leapp-project). Open this URL in a new browser tab.

- Go back to the AAP Web UI and now navigate to Resources > Templates > AUTO / 01 Analysis. Under the Details tab, you will see the "Playbook" setting with the name of the playbook this job template runs when it is used to submit a job. The playbook name is `analysis.yml`. In your GitHub browser tab, you can find `analysis.yml` listed in the files of the git repo. Click on it to see the playbook contents.

- Notice that the `Run RIPU preupg` task of the playbook is importing a role from the `infra.leapp` Ansible collection. By checking the `collections/requirements.yml` file in the git repo, we can discover that this role comes from another git repo at [https://github.com/redhat-cop/infra.leapp](https://github.com/redhat-cop/infra.leapp). It is the `analysis` role under this second git repo that provides all the automation tasks that ultimately runs the Leapp pre-upgrade scan and generates the report.

- Drill down to the `roles/analysis` directory in this git repo to review the README and yaml source files.

When you are ready to develop your own custom playbooks to run upgrades for your enterprise, you should consider using roles from the `infra.leapp` Ansible collection to make your job easier.

## Conclusion

In this exercise, we learned about the end-to-end workflow used by our automation approach for doing RHEL in-place upgrades. We used a job template in AAP to submit a playbook job that ran the Leapp pre-upgrade analysis on our pet application servers. In the challenge lab, we explored the playbook that we ran and how it includes a role from an upstream Ansible collection.

In the next exercise, we will review the pre-upgrade reports we just generated and take action to resolve any high-risk findings that were identified.

---

**Navigation**

[Previous Exercise](../1.1-setup/README.md) - [Next Exercise](../1.3-report/README.md)

[Home](../README.md)
