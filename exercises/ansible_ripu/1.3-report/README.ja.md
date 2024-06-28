# ワークショップ演習 - アップグレード前レポートのレビュー

## 目次

- [ワークショップ演習 - アップグレード前レポートのレビュー](#ワークショップ演習---アップグレード前レポートのレビュー)
  - [目次](#目次)
  - [目標](#目標)
  - [ガイド](#ガイド)
    - [Step 1 - Leapp アップグレード前結果の管理](#step-1---managing-leapp-pre-upgrade-results)
    - [Step 2 - RHEL Web コンソールの操作](#step-2---navigating-the-rhel-web-console)
    - [Step 3 - RHEL8 ホストの Leapp アップグレード前レポートのレビュー](#step-3---RHEL8-ホストの-Leapp-アップグレード前レポートのレビュー)
    - [Step 4 - RHEL7 ホストの Leapp アップグレード前レポートのレビュー](#step-4---RHEL7-ホストの-Leapp-アップグレード前レポートのレビュー)
    - [チャレンジラボ: 多数の高レベルの検出結果を無視するのはどうでしょうか?](#チャレンジラボ-多数の高レベルの検出結果を無視するのはどうでしょうか)
  - [まとめ](#まとめ)

## 目標

* Leapp のアップグレード前レポートを管理するためのさまざまなオプションを理解する
* RHEL Web コンソールを使用して、生成したレポートを確認する
* アップグレード前レポートのエントリをフィルター処理する方法を学ぶ
* 失敗を受け入れよう!

## ガイド

### Step 1 - Leapp アップグレード前結果の管理

前の演習では、プレイブック ジョブ テンプレートを使用して、各ペット アプリ サーバーで Leapp のアップグレード前レポートを生成しました。次に、それらのレポートにリストされている結果を確認する必要があります。レポートにアクセスする方法はいくつかあります。これらを確認し、長所と短所を検討してみましょう。:

- Leapp フレームワークを使用して単一の RHEL ホストのみを手動でアップグレードする場合は、ホストでシェルプロンプトにアクセスして、ローカルレポートファイルの出力を確認するだけで済みます。 [Exercise 1.1, Step 2](../1.1-setup/README.md#step-2---open-a-terminal-session) では、ペット アプリ サーバーの 1 つへの ssh セッションを開く方法を学習しました。これらの手順に従い、ログイン後、次のコマンドを使用してローカルの Leapp アップグレード前レポート ファイルを確認します:

  ```
  less /var/log/leapp/leapp-report.txt
  ```

  これはレポートを確認するための "手っ取り早い" 方法ですが、多数のホストのレポートを確認する必要がある場合には適していません。

  > **注記**
  >
  > 上下矢印キーを使用してファイルをスクロールし、`less` コマンドを終了する準備が出来たら `q` と入力します。

- RHEL ホストが [Red Hat Insights](https://www.redhat.com/en/technologies/management/insights) に登録されている場合、Insights コンソールで Leapp のアップグレード前レポートを確認できます。このワークショップ用にプロビジョニングされたペット アプリ サーバーは Insights に登録されていないため、ここでは説明を割愛しますがブログ記事 [Take the unknowns out of RHEL upgrades with Red Hat Insights](https://www.redhat.com/en/blog/take-unknowns-out-rhel-upgrades-red-hat-insights) を読んで、Insights を使用して Leapp のアップグレード前を確認および管理する方法の例を確認してください。

- RHEL には [Cockpit](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/managing_systems_using_the_rhel_8_web_console/index#what-is-the-RHEL-web-console_getting-started-with-the-rhel-8-web-console) に基づくオプションの管理 Web コンソールが含まれており、これを RHEL Web コンソールと呼んでいます。この演習の次のステップでは、RHEL Web コンソールを使用して Leapp のアップグレード前レポートを確認する方法について説明します。
 
- Leapp は、プレーン テキストの `leapp-report.txt` ファイルを書き込むだけでなく、JSON 形式の `leapp-report.json` ファイルも生成します。このファイルには、プレーンテキストファイルと同じレポート結果が含まれますが、Elastic/Kibana や Splunk などのログ管理ツールで取り込むのに最適な JSON 形式です。多くの大企業は、アップグレード前のレポート データをこれらのツールの 1 つにプッシュして、環境 (開発/テスト/本番など)、場所、アプリケーション ID、所有チームなどでレポートをフィルターできる独自のカスタム ダッシュボードを開発します。 <!-- FIXME: add Splunk example here when https://issues.redhat.com/browse/RIPU-35 gets done. -->

### Step 2 - RHEL Web コンソールのナビゲート

このワークショップでは、RHEL Web コンソールを使用して、生成した Leapp アップグレード前レポートにアクセスします。

- [Exercise 1.1, Step 4](../1.1-setup/README.md#step-4---access-the-rhel-web-console) で開いた RHEL Web コンソールのブラウザー タブに戻ります。これは AAP コントローラー ホストの RHEL Web コンソールですが、アップグレード前レポートを表示するには、ペット アプリケーション サーバー ホストにアクセスする必要があります。これを行うには、RHEL Web コンソールの左上隅にある "student&#8203;@&#8203;ansible-1.example.com" ボックスをクリックして、リモートホストメニューを表示します。例:

  ![Remote host menu listing all pet app servers](images/remote_host_menu_with_pets.svg)

- リモートホストメニューを使用して、各ペットアプリサーバーの Web コンソールに移動できます。今すぐペットサーバーの 1 つを選択してみてください。RHEL Web コンソールのシステム概要ページに、インストールされているオペレーティングシステムのバージョンが表示されます。たとえば、このペットアプリ サーバーは RHEL8 を実行しています:

  ![upward-moray running Red Hat Enterprise Linux 8.7 (Ootpa)](images/rhel8_os.svg)

  RHEL7 を実行している例を次に示します:

  ![Operating System Red Hat Enterprise Linux Server 7.9 (Maipo)](images/rhel7_os.svg)

- RHEL Web コンソールで別のホストに移動する場合、"limited access mode" の警告に注意してください:

  ![Web console is running in limited access mode](images/limited_access.svg)

  この警告が表示された場合は、続行する前にボタンを使用して管理アクセス モードに切り替えてください。 次のような確認が表示されます:

  ![You now have administrative access](images/administrative_access.svg)

- さまざまなペット アプリ サーバーの RHEL Web コンソールで使用できるナビゲーション メニューを少し調べてください。コンソールの操作とホストの切り替えに慣れたら、次のステップに進み、最初のアップグレード前レポートを確認します。

### Step 3 - Review Leapp Pre-upgrade Report of RHEL8 Host

Now we are ready to use the RHEL Web Console to see the Leapp pre-upgrade reports. Let's start by looking at one of the RHEL8 hosts and then we'll look at one of the RHEL7 hosts in the next step.

While you might be interested in learning about upgrading only RHEL7 or RHEL8, we recommend following the exercise steps for both. This workshop presents the skills you need with the RHEL7 and RHEL8 examples covering different topics you must know irrespective of the OS version being upgraded.

We are now here in the automation approach workflow:

![Automation approach workflow diagram with review report step highlighted](images/ripu-workflow-hl-review.svg)

- Navigate to the RHEL Web Console remote host menu and click on the hostname of one of your RHEL8 pet app servers. Remember as we learned in the previous step, you can confirm the RHEL version on the system overview page. Also make sure you enabled administrative access as explained in the previous step.

- Having verified you are looking at one of the RHEL8 pet app servers, use the main menu to navigate to Tools > Upgrade Report. This will display the Leapp pre-upgrade report that was generated for the selected host. For example, the report might look like this:

  ![Example pre-upgrade report of RHEL8 host](images/rhel8_report.svg)

  > **Note**
  >
  > The contents of your report may differ from the example above because of updates made to the Leapp framework and other RHEL packages released over time since this workshop was written. If you discover any differences that materially break the flow of the exercises in the workshop, kindly let us know by raising an issue [here](https://github.com/ansible/workshops/issues/new).

- When the pre-upgrade report is generated, the Leapp framework collects system data and assesses upgradeability based on a large collection of checks. When any of these checks uncovers a potential risk, it is recorded as a finding in the report. These findings are listed in order from highest risk to lowest. In the report above, we see there are three high risk findings. Let's review each of these.

- The first finding we see listed has the title "Leapp could not identify where GRUB core is located." You can see additional details for any finding by clicking on it in the list. For example, click on the first finding and you will see these details:

  ![Details view of grub core finding](images/grub_core_finding.svg)

  This finding is being reported because the EC2 instances deployed for the workshop do not have a separate /boot partition. We'll ignore this one for now, but make a mental note as we may revisit this with a Challenge Lab in a later exercise. <!-- We'll talk about fixing this in the commit playbook. -->

- The next finding is titled "Remote root logins globally allowed using password." Click on it to see the details:

  ![Details view of remote root logins finding](images/remote_root_logins_finding.svg)

  This finding is meant to raise awareness of a change to the default root login settings introduced with RHEL9. We can safely ignore this finding because surely everybody already follows best practices by never logging in directly as the root user.

- That brings us to the final high risk finding. This one is a little embarrassing because it's actually a known bug in the Leapp framework.

  ![Details view usage of deprecated model bug finding](images/leapp_bug_finding.svg)

  Luckily, it is completely benign and we can safely ignore it. This bug will be fixed with an update to the Leapp framework expected to be released soon. <!-- FIXME: remove this after the bug fix gets released. Also remove from RHEL7 report step further down. -->

- The good news is that none of the findings with our RHEL8 host were the most severe "inhibitor" classification. When any inhibitor findings are reported, the RHEL upgrade is blocked and can't proceed without first taking action to correct the cause of the inhibitor risk finding.

- There are a number of filtering options you can use to limit the findings that are displayed according to risk level, audience, etc. Click on the "Filters" button to experiment with this feature. For example, if you click the "Is inhibitor?" filter checkbox, you will see no findings displayed because there were no inhibitors.

- Let's now move on to the pre-upgrade report for one of our RHEL7 hosts. Spoiler alert: we will have to deal with some inhibitor findings with this one!

### Step 4 - Review Leapp Pre-upgrade Report of RHEL7 Host

In the previous step, we reviewed the pre-upgrade report for one of our RHEL8 hosts. Now let's take a look at the report from one of our RHEL7 hosts.

- Navigate to the RHEL Web Console remote host menu and click on the hostname of one of your RHEL7 pet app servers. Verify the host you have chosen is RHEL7. Then use the main menu to navigate to Tools > Upgrade Report. This will bring up the Leapp pre-upgrade report for the selected host. For example, the report might look like this:

  ![Example pre-upgrade report of RHEL7 host](images/rhel7_report.svg)

  > **Note**
  >
  > The contents of your report may differ from the example above because of updates made to the Leapp framework and other RHEL packages released over time since this workshop was written. If you discover any differences that materially break the flow of the exercises in this workshop, kindly let us know by raising an issue [here](https://github.com/ansible/workshops/issues/new).

- In the report for our RHEL7 pet app server above, we see there are six high risk findings and two of those are inhibitor findings. Let's start by reviewing the high risk findings that are not inhibitors.

- The "GRUB core will be updated during upgrade" finding is no different than the finding with the same title we learned about in the RHEL8 pre-upgrade report, so we'll ignore this for now.

- The high risk finding "Usage of deprecated Model" is again because of the Leapp framework bug we talked about before. It's annoying but benign and we can ignore it.

- Now let's look at the new findings we are seeing only on our RHEL7 pre-upgrade report. At the top of the list we see the "Packages available in excluded repositories will not be installed" finding. Clicking on the finding to bring up the detailed view, we see this:

  ![Details view of packages available in excluded repositories will not be installed](images/excluded_repos_finding.svg)

  This finding is warning that packages python3-pyxattr and rpcgen will not be upgraded because "they are available only in target system repositories that are intentionally excluded from the list of repositories used during the upgrade," but then refers to an informational finding titled "Excluded target system repositories" for more information. Scroll down and click on that finding to show its details:

  ![Details view of excluded target system repositories information finding](images/enablerepo_info_finding.svg)

  Here we see the remediation hint suggests to run the `leapp` utility with the `--enablerepo` option. But wait, that's assuming we are manually running the `leapp` command. Don't worry, in an upcoming exercise, we'll explore how this option can be given by setting a variable when submitting the upgrade playbook job. Stay tuned!

- The next high risk entry on the list is the "Difference in Python versions and support in RHEL8" finding:

  ![Details view of Difference in Python versions and support in RHEL8 finding](images/python_finding.svg)

  This finding could be a concern if we have any apps on our pet server that are using the system-provided Python interpreter. Let's assume we don't have any of those in which case we can blissfully ignore this finding.

- That leaves us with our two inhibitor findings. The first is the "Possible problems with remote login using root account" finding. You know the drill; click on the finding to review the details:

  ![Details view of possible problems with remote login using root account inhibitor finding](images/root_account_inhibitor.svg)

  Remember that with inhibitor findings, if we don't take action to resolve the inhibitor, the Leapp framework will block the RHEL in-place upgrade from going forward.

- The other inhibitor is the "Missing required answers in the answer file" finding. Here are the details for this one:

  ![Details view of missing required answers in the answer file](images/missing_answers_inhibitor.svg)

  Here again, we will need to take action to remediate this finding. Don't panic! In the next exercise, we will explore different options for automating the required remediation actions and recommendations.

### Challenge Lab: What About Ignoring So Many High Findings?

You may be wondering why are we only worrying about the inhibitor findings. What about all the other high risk findings showing up in red on the report? Red means danger! Why would we be going forward with attempting an upgrade without first resolving all the findings on the report? It's a fair question.

> **Tip**
>
> Think back to the four key features that we introduced at the beginning of the workshop.

Is there a specific feature that helps with reducing risk?

> **Warning**
>
> **Solution below\!**

Of course, the answer is our automated snapshot/rollback capability.

- If any of the high risk findings listed in the pre-upgrade report ultimately leads to the upgrade failing or results in application compatibility impact, we can quickly get back to where we started by rolling back the snapshot. Before rolling back, we can debug the root cause and use the experience to understand the best way to eliminate the risk of that failure or impact happening in the future.

- There is a concept explained quite well in the famous article [Fail Fast](http://www.martinfowler.com/ieeeSoftware/failFast.pdf) published in *IEEE Software*. The article dates back to 2004, so this is hardly a new concept. Unfortunately, there is a stigma associated with failure that can lead to excessively risk-averse behavior. The most important benefit of having automated snapshots is being able to quickly revert failures. That allows us to safely adopt a fail fast and fail smart mantra.

- Of course, there are many best practices we can follow to reduce risk. Obviously, test for application impacts by trying upgrades in your lower environments first. Any issues that can be worked out with Dev and Test servers will help you be prepared to avoid those issues in production.

- The high risk findings reported by the Leapp pre-upgrade report are there to make us aware of potential failure modes, but experience has shown that they are not a problem in many cases. Don't become petrified when you see those red findings on the report. Upgrade early and often!

## Conclusion

In this exercise, we learned about the different options for managing Leapp pre-upgrade reports. We used the RHEL Web Console to look at the reports we generated in the previous exercise and reviewed a number of the reported findings. In the challenge lab, we explored the importance of snapshots and learned to embrace failure.

In the next exercise, we are going to look at how to automate the remediation actions required to resolve our inhibitor findings.

---

**Navigation**

[Previous Exercise](../1.2-preupg/README.md) - [Next Exercise](../1.4-remediate/README.md)

[Home](../README.md)
