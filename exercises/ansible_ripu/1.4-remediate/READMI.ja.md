# ワークショップ演習 - 推奨される修復を実行する

## 目次

- [ワークショップ演習 - 推奨される修復を実行する](#workshop-exercise---perform-recommended-remediation)
  - [目次](#table-of-contents)
  - [目的](#objectives)
  - [ガイド](#guide)
    - [ステップ 1 - 阻害要因を解決するためのオプションを検討する](#step-1---explore-options-for-resolving-inhibitors)
    - [ステップ 2 - Leapp 回答ファイルの管理](#step-2---managing-the-leapp-answer-file)
    - [ステップ 3 - 修復プレイブックを使用して阻害要因を解決する](#step-3---resolving-inhibitors-using-a-remediation-playbook)
  - [結論](#conclusion)

## 目的

* 検討する阻害リスクの検出を解決するためのさまざまなオプション
* `analysis` ロールの `leapp_answerfile` 変数の使用方法を学習します
* 修復プレイブックを使用して、アップグレード前の準備を積極的に行います

## ガイド

### ステップ 1 - 阻害要因を解決するためのオプションを探ります

前の演習では、RHEL7 および RHEL8 ペット アプリケーション サーバー用に生成された Leapp アップグレード前レポートを確認しました。RHEL8 ホストでは阻害リスクの検出結果は報告されていなかったため、アップグレードを試行する準備は整っています。ただし、RHEL7 ホストでは阻害要因がいくつか報告されていました。これらのホストをアップグレードする前に、それらを解決するための措置を講じる必要があります。

これで、自動化アプローチ ワークフローの次の段階に進みました:

![推奨される修復を適用する手順が強調表示された自動化アプローチ ワークフロー図](images/ripu-workflow-hl-remediate.svg)

- まず、阻害要因の 1 つを分析してみましょう:

![回答ファイルで必要な回答が不足している詳細ビュー](images/missing_answers_dissected.svg)

<sub>![1.](images/circle_1.svg)</sub> 各検出結果には固有のタイトルがあります。

<sub>![2.](images/circle_2.svg)</sub> 各検出結果にはリスク要因が割り当てられますが、前の演習で説明したように、これは単純な高、中、低、または情報の評価で示されるよりも微妙な場合があります。

<sub>![3.](images/circle_3.svg)</sub> 概要には、リスクとソリューションの推奨事項の詳細な説明が記載されています。

<sub>![4.](images/circle_4.svg)</sub> 修復では、かなり規範的な推奨事項が提示されます。

<sub>![5.](images/circle_5.svg)</sub> 場合によっては、修復には、次のような正確なコマンドも含まれています。

- 上記の例のように修復コマンドが提示された場合、コマンドの実行方法として選択できるオプションがいくつかあります。もちろん、ホストのルート シェル プロンプトにアクセスしてコマンドを切り取って貼り付けるか、回答ファイルを手動で編集するという、手っ取り早い方法もあります。もちろん、その方法は人為的ミスが発生しやすく、拡張性も低くなります。別のオプションは、コマンドの上にある [修復の実行] ボタンを使用することです。このオプションを使用すると、RHEL Web コンソールがコマンドを実行します。この方法では人為的ミスが発生しにくくなりますが、この単一のホストでのみ実行されるため、拡張性は低くなります。

- 次のステップでは、Ansible Automation Platform (AAP) のスケールを活用して、大規模な RHEL 資産全体で一括して修復を実行する方法について説明します。

### ステップ 2 - Leapp 回答ファイルの管理

Leapp フレームワークは、ユーザー入力の選択を受け入れる手段として回答ファイルを使用します。これについては、Leapp 開発者ドキュメントの [ユーザーへの質問](https://leapp.readthedocs.io/en/latest/dialogs.html) セクションで詳しく説明されています。前のステップで分析した阻害要因の検出結果は、私たちに決定を求めています。より具体的には、RHEL アップグレード中に Leapp が pam_pkcs11 PAM モジュールを無効にすることを認識していることを確認するように求めています。

- [演習 1.2 - アップグレード前のジョブの実行](../1.2-preupg/README.md) では、`infra.leapp` Ansible コレクションの `analysis` ロールを使用してアップグレード前のレポートを実行するプレイブックを起動しました。 [このロールのドキュメント](https://github.com/redhat-c​​op/infra.leapp/blob/main/roles/analysis/README.md) をご覧ください。`leapp_answerfile` 入力変数がサポートされている場所がわかりますか。変数を設定すると、Leapp 応答ファイルが自動的に入力されます。

- この変数を定義して、アップグレード前のジョブを再度実行してみましょう。 [演習 1.2、ステップ 2](../1.2-preupg/README.md#step-2---use-aap-to-launch-an-analysis-playbook-job) で実行したのと同じように、"AUTO / 01 Analysis" ジョブ テンプレートを起動します。ただし、今回は、変数プロンプトが表示されたときに次の設定を追加します:

```json
"leapp_answerfile": "[remove_pam_pkcs11_module_check]\nconfirm = True\n",
```
例:

![`leapp_answerfile` 入力変数の設定](images/analysis_leapp_answerfile.svg)

上記のように変数を設定したら、[次へ] ボタンをクリックします。これにより、ジョブ テンプレートの調査プロンプトが表示されます。以前は、すべてのペット サーバーでアップグレード前を実行するために「ALL_rhel」オプションを使用しました。ただし、`leapp_answerfile` 設定は RHEL7 ホストに固有のため、今回は「rhel7」オプションを選択します。

![調査プロンプトで「rhel7」を選択](images/analysis_survey_rhel7_only.svg)

「次へ」ボタンをクリックしてプレビュー プロンプトに進みます。ジョブ プレビューに問題がなければ、「起動」ボタンを使用してジョブを開始します。

- 以前と同様に、ジョブを開始すると、AAP Web UI はジョブ出力ページに自動的に移動します。ジョブが完了するまでに数分かかり、ジョブ出力の最後に「PLAY RECAP」が表示されます。

- ここで、RHEL Web コンソールのブラウザー タブに戻り、RHEL7 ホストの 1 つのアップグレード前レポートに移動します。

> **注**
>
> 新しく生成されたレポートを表示するには、Ctrl + R を使用してブラウザーを更新する必要がある場合があります。

「応答ファイルに必要な回答がありません」という阻害要因の検出結果は報告されなくなったことがわかります。

例:

![応答ファイル阻害要因のない RHEL7 ホストのアップグレード前レポート](images/rhel7_answer_fixed.svg)

ただし、「ルート アカウントを使用したリモート ログインで問題が発生する可能性がある」という阻害要因はまだ残っているので、これを修正する必要があります。次にこれについて見てみましょう。

### Step 3 - Resolving Inhibitors Using a Remediation Playbook

In the previous step, we were able to resolve an inhibitor finding by simply setting the `leapp_answerfile` input variable supported by the `infra.leapp` Ansible collection `analysis` role. While that's a convenient way to resolve an answerfile inhibitor, our next inhibitor can't be resolved that way.

- Here is our other inhibitor finding:

  ![Details view of missing required answers in the answer file](../1.3-report/images/root_account_inhibitor.svg)

  Like the previous inhibitor finding, this one also provides a detailed summary and a a fairly prescriptive recommended remediation. However, it does not recommend an exact remediation command. Instead, the remediation recommends making edits to the `/etc/ssh/sshd_config` file.

- Of course, we're not going to just login to a root shell and `vi` the configuration file, are we? Right, let's make a playbook to automate the required remediations. Here's a task that should do the trick:

  ```yaml
  - name: Configure sshd
    ansible.builtin.lineinfile:
      path: "/etc/ssh/sshd_config"
      regex: "^(#)?{{ item.key }}"
      line: "{{ item.key }} {{ item.value }}"
      state: present
    loop:
      - {key: "PermitRootLogin", value: "prohibit-password"}
      - {key: "PasswordAuthentication", value: "no"}
    notify:
      - Restart sshd
  ```

  While we're at it, let's also add a task to take care of the answer file inhibitor using the `leapp answer` command. For example:

  ```yaml
  - name: Remove pam_pkcs11 module
    ansible.builtin.shell: |
      set -o pipefail
      leapp answer --section remove_pam_pkcs11_module_check.confirm=True
    args:
      executable: /bin/bash
  ```

- You will find the tasks above in the playbook [`remediate_rhel7.yml`](https://github.com/redhat-partner-tech/leapp-project/blob/main/remediate_rhel7.yml#L21-L38). There are a few more remediation task examples in this playbook as well. The "OS / Remediate" job template is already set up to execute this playbook, so let's use it to remediate our RHEL7 hosts.

- Return to your AAP Web UI browser tab. Navigate to Resources > Templates on the AAP Web UI and open the "OS / Remediate" job template. Click the "Launch" button to get started.

- This will bring you to the job template survey prompt. Again, choose the "rhel7" option at the "Select inventory group" prompt because our remediation playbook is specific to the pre-upgrade findings of our RHEL7 hosts. Then click the "Next" button. If you are satisfied with the job preview, use the "Launch" button to submit the job. This playbook includes only a small number of tasks and should run pretty quickly.

- When the "OS / Remediate" job is finished, launch the "AUTO / 01 Analysis" job template one more time again taking care to choose the "rhel7" option at the "Select inventory group" prompt. When the job completes, go back to the RHEL Web Console of your RHEL7 host and refresh the report. You should now see there are no inhibitors:

  ![Pre-upgrade report of RHEL7 host with no more inhibitors](images/rhel7_no_inhibitors.svg)

  With no inhibitors indicated on our RHEL7 and RHEL8 pet servers, we are ready to try the RHEL upgrade.

## Conclusion

In this exercise, we looked at the different ways we can resolve inhibitor risk findings. We learned how to use the `leapp_answerfile` variable of the `analysis` role to manage the Leapp answer file. Finally, we used an example remediation playbook to demonstrate how we could address pre-upgrade inhibitor findings at scale across our RHEL estate.

Now we are ready to try upgrading our RHEL pet app servers, but before we get to that, there are two more optional exercises in this section of the workshop:

- [Exercise 1.5 - Custom Pre-upgrade Checks](../1.5-custom-modules/README.md)
- [Exercise 1.6 - Deploy a Pet Application](../1.6-my-pet-app/README.md)

These exercises are not required to successfully complete the workshop, but we recommend doing them if time allows. If you can't wait and want skip ahead to upgrading your RHEL hosts, strap in for this exciting exercise:

- [Exercise 2.1 - Run the RHEL Upgrade Jobs](../2.1-upgrade/README.md)

---

**Navigation**

[Previous Exercise](../1.3-report/README.md) - [Next Exercise](../1.5-custom-modules/README.md)

[Home](../README.md)
