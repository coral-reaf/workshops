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

- 上記の例のように修復コマンドが提示された場合、コマンドの実行方法として選択できるオプションがいくつかあります。もちろん、ホストのルート シェル プロンプトにアクセスしてコマンドを切り取って貼り付けるか、回答ファイルを手動で編集するという、手っ取り早い方法もあります。もちろん、その方法は人為的ミスが発生しやすく、拡張性も低くなります。別のオプションは、コマンドの上にある "Run Remediation" ボタンを使用することです。このオプションを使用すると、RHEL Web コンソールがコマンドを実行します。この方法では人為的ミスが発生しにくくなりますが、この単一のホストでのみ実行されるため、拡張性は低くなります。

- 次のステップでは、Ansible Automation Platform (AAP) のスケールを活用して、大規模な RHEL 資産全体で一括して修復を実行する方法について説明します。

### ステップ 2 - Leapp 回答ファイルの管理

Leapp フレームワークは、ユーザー入力の選択を受け入れる手段として回答ファイルを使用します。これについては、Leapp 開発者ドキュメントの [Asking user questions](https://leapp.readthedocs.io/en/latest/dialogs.html) セクションで詳しく説明されています。前のステップで分析した阻害要因の検出結果は、私たちに決定を求めています。より具体的には、RHEL アップグレード中に Leapp が pam_pkcs11 PAM モジュールを無効にすることを認識していることを確認するように求めています。

- [演習 1.2 - アップグレード前のジョブの実行](../1.2-preupg/README.ja.md) では、`infra.leapp` Ansible コレクションの `analysis` Role を使用してアップグレード前のレポートを実行する Playbook を起動しました。 [このロールのドキュメント](https://github.com/redhat-c​​op/infra.leapp/blob/main/roles/analysis/README.md)  [role のドキュメント](https://github.com/redhat-cop/infra.leapp/blob/main/roles/analysis/README.md) をご覧ください。`leapp_answerfile` 入力変数がサポートされている場所がわかりますか。変数を設定すると、Leapp 応答ファイルが自動的に入力されます。




 
- この変数を定義して、アップグレード前のジョブを再度実行してみましょう。 [演習 1.2、ステップ 2](../1.2-preupg/README.ja.md#step-2---aap-を使った分析-playbook-ジョブの起動) で実行したのと同じように、"AUTO / 01 Analysis" ジョブ テンプレートを起動します。ただし、今回は、変数プロンプトが表示されたときに次の設定を追加します:

```json
"leapp_answerfile": "[remove_pam_pkcs11_module_check]\nconfirm = True\n",
```
例:

![`leapp_answerfile` 入力変数の設定](images/analysis_leapp_answerfile.svg)

上記のように変数を設定したら、"次へ" ボタンをクリックします。これにより、ジョブ テンプレートの調査プロンプトが表示されます。以前は、すべてのペット サーバーでアップグレード前を実行するために "ALL_rhel" オプションを使用しました。ただし、`leapp_answerfile` 設定は RHEL7 ホストに固有のため、今回は "rhel7" オプションを選択します。

![調査プロンプトで「rhel7」を選択](images/analysis_survey_rhel7_only.svg)

"次へ" ボタンをクリックしてプレビュー プロンプトに進みます。ジョブ プレビューに問題がなければ、"起動" ボタンを使用してジョブを開始します。

- 以前と同様に、ジョブを開始すると、AAP Web UI はジョブ出力ページに自動的に移動します。ジョブが完了するまでに数分かかり、ジョブ出力の最後に「PLAY RECAP」が表示されます。

- ここで、RHEL Web コンソールのブラウザー タブに戻り、RHEL7 ホストの 1 つのアップグレード前レポートに移動します。

> **注**
>
> 新しく生成されたレポートを表示するには、Ctrl + R を使用してブラウザーを更新する必要がある場合があります。

"応答ファイルに必要な回答がありません" という阻害要因の検出結果は報告されなくなったことがわかります。

例:

![応答ファイル阻害要因のない RHEL7 ホストのアップグレード前レポート](images/rhel7_answer_fixed.svg)

ただし、"ルート アカウントを使用したリモート ログインで問題が発生する可能性がある" という阻害要因はまだ残っているので、これを修正する必要があります。次にこれについて見てみましょう。

### ステップ 3 - 修復 Playbook を使用して阻害要因を解決する

前のステップでは、`infra.leapp` Ansible コレクション `analysis` ロールでサポートされている `leapp_answerfile` 入力変数を設定するだけで阻害要因の検出を解決できました。これは回答ファイルの阻害要因を解決する便利な方法ですが、次の阻害要因はその方法では解決できません。

- これがもう 1 つの阻害要因の検出です:

![回答ファイルで欠落している必須回答の詳細ビュー](../1.3-report/images/root_account_inhibitor.svg)

前の阻害要因の検出と同様に、この検出でも詳細な概要とかなり規範的な推奨修復が提供されます。ただし、正確な修復コマンドは推奨されません。代わりに、修復では `/etc/ssh/sshd_config` ファイルを編集することが推奨されます。

- もちろん、ルート シェルにログインして設定ファイルを `vi` するだけではありませんよね。では、必要な修復を自動化する Playbook を作成しましょう。次のタスクでうまくいきます:

```yaml
- name: sshd を構成する
ansible.builtin.lineinfile:
path: "/etc/ssh/sshd_config"
regex: "^(#)?{{ item.key }}"
line: "{{ item.key }} {{ item.value }}"
state: present
loop:
- {key: "PermitRootLogin", value: "prohibit-password"}
- {key: "PasswordAuthentication", value: "no"}
notification:
- sshd を再起動
```

ついでに、`leapp answer` コマンドを使用して、応答ファイル インヒビターを処理するタスクも追加しましょう。例:

```yaml
- name: Remove pam_pkcs11 module
ansible.builtin.shell: |
set -o pipefail
leapp answer --section remove_pam_pkcs11_module_check.confirm=True
args:
executable: /bin/bash
```

- 上記のタスクは、Playbook  [`remediate_rhel7.yml`](https://github.com/redhat-partner-tech/leapp-project/blob/main/remediate_rhel7.yml#L21-L38) にあります。この Playbook には、さらにいくつかの修復タスクの例もあります。この Playbook を実行するために、"OS / Remediate" ジョブ テンプレートがすでに設定されているので、それを使用して RHEL7 ホストを修復しましょう。

- AAP Web UI ブラウザー タブに戻ります。 AAP Web UI で リソース > テンプレート に移動し、"OS / Remediate" ジョブ テンプレートを開きます。開始するには、"起動" ボタンをクリックします。

- これにより、ジョブ テンプレートの調査プロンプトが表示されます。ここでも、"インベントリ グループの選択" プロンプトで "rhel7" オプションを選択します。これは、修復 Playbook が RHEL7 ホストのアップグレード前の調査結果に固有のものであるためです。次に、"次へ" ボタンをクリックします。ジョブのプレビューに問題がなければ、"起動" ボタンを使用してジョブを送信します。この Playbook には少数のタスクしか含まれていないため、すぐに実行できます。

- "OS / Remediate" ジョブが終了したら、"インベントリ グループの選択" プロンプトで "rhel7" オプションを選択するように注意しながら、"AUTO / 01 Analysis" ジョブ テンプレートをもう一度起動します。ジョブが完了したら、RHEL7 ホストの RHEL Web コンソールに戻り、レポートを更新します。これで、阻害要因がなくなったことがわかります:

![阻害要因がなくなった RHEL7 ホストのアップグレード前レポート](images/rhel7_no_inhibitors.svg)

RHEL7 および RHEL8 ペット サーバーに阻害要因が表示されなくなったので、RHEL のアップグレードを試す準備ができました。

## まとめ

この演習では、阻害リスクの検出を解決するさまざまな方法を確認しました。`analysis` ロールの `leapp_answerfile` 変数を使用して Leapp 回答ファイルを管理する方法を学びました。最後に、修復プレイブックの例を使用して、アップグレード前の阻害の検出に RHEL 資産全体で大規模に対処する方法を示しました。

これで、RHEL ペットアプリサーバーのアップグレードを試す準備ができましたが、その前に、2 つのオプションの演習があります。

- [演習 1.5 - カスタムのアップグレード前チェック](../1.5-custom-modules/README.ja.md)
- [演習 1.6 - ペットアプリケーションのデプロイ](../1.6-my-pet-app/README.ja.md)

これらの演習はワークショップは必須ではありませんが、時間に余裕がある場合は実行することをお勧めします。待ちきれず、RHEL ホストのアップグレードに進みたい場合は、このエキサイティングな演習に取り組んでください:

- [演習 2.1 - RHEL アップグレード ジョブを実行する](../2.1-upgrade/README.ja.md)

---

**ナビゲーション**

[前の演習](../1.3-report/README.ja.md) - [次の演習](../1.5-custom-modules/README.ja.md)

[ホーム](../README.ja.md)
