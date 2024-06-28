# ワークショップ演習 - カスタムモジュール

## 目次

- [ワークショップ演習 - カスタムモジュール](#ワークショップ演習---カスタムモジュール)
  - [目次](#目次)
  - [オプション演習](#オプション演習)
  - [目標](#目標)
  - [ガイド](#ガイド)
    - [ステップ 1 - カスタムモジュールとは](#step-1---カスタムモジュールとは)
    - [ステップ 2 - Leapp カスタムアクターをインストールする](#step-2---leapp-カスタムアクターをインストールする)
    - [ステップ 3 - 新しいアップグレード前レポートを生成する](#step-3---新しいアップグレード前レポートを生成する)
    - [ステップ 4 - インプレースアップグレードのカスタマイズについてさらに学ぶ](#step-4---インプレースアップグレードのカスタマイズについてさらに学ぶ)
  - [結論](#conclusion)

## オプション演習

これはオプションの演習です。ワークショップを正常に完了するために必須ではありませんが、時間に余裕がある場合は実行することをお勧めします。次のセクションに記載されている目標を確認して、この演習を実行するか、次の演習に進むかを決定します。

* [演習 1.6 - (オプション) ペットアプリケーションのデプロイ](../1.6-my-pet-app/README.ja.md)
* [演習 2.1 - RHEL アップグレードジョブの実行](../2.1-upgrade/README.ja.md)

## 目標

* カスタム モジュールとは何かを学習する
* 追加のアップグレード前チェックを実装するために Leapp カスタム アクターをインストールする
* 独自のカスタム アクターの作成についてさらに学習するための場所を確認する

## ガイド

### ステップ 1 - カスタム モジュールとは

カスタム モジュールとは、企業固有の特別な要件を満たすために RHEL インプレース アップグレード自動化アプローチに重ねることができるカスタム機能を指す一般的な用語です。

- たとえば、組織では、システム メンテナンスを行う前に従う必要がある標準のポリシー セットを確立している場合があります。その場合、ポリシーに準拠していない条件が検出されると、アップグレード前のインヒビターを発生させる Leapp カスタム アクターを実装できます。

- カスタム モジュールのもう 1 つの使用例は、標準の RHEL サーバー ビルドに含めるサードパーティのツールとエージェントを処理することです。たとえば、構成の収束に Chef を使用する場合、アップグレード プレイブックに追加のタスクを実装して、Chef クライアント パッケージを更新し、新しい OS バージョンで必要となる Chef ノード属性と実行リストに必要な変更を加えることができます。

- さまざまな組織の多様な要件に合わせてロジックと自動化を設計および実装する方法についての単一の青写真がないため、"カスタム モジュール" という一般的な用語を使用します。カスタム モジュールの要件では、Leapp カスタム アクター、カスタム Ansible 自動化、または両方を使用した統合設計が必要になる場合があります。

- カスタム Leapp アクターは、必要な追加のアップグレード前チェックを実装するのに最適です。これらのチェックの結果は、Leapp フレームワークによって生成されるアップグレード前レポートにシームレスに含まれるためです。カスタム Leapp アクターは、Leapp アップグレードの中間システム (initrd) フェーズ中に実行する必要がある自動化タスクにも使用する必要があります。

- Leapp カスタム アクターの開発は、プレイブックにタスクを追加するだけの場合ほど簡単ではありません。そのため、ほとんどのカスタム自動化要件は、Ansible を使用して実現するのが最適です。タスクは、RHEL OS アップグレードを実際に実行するために から `infra.leapp` コレクション `upgrade` ロールをインポートするタスクの前後にアップグレード プレイブックに含めることができます。

### Step 2 - Install a Leapp Custom Actor

There is a collection of example custom actor at the GitHub repo [oamg/leapp-supplements](https://github.com/oamg/leapp-supplements). We will use one of these to demonstrate adding a custom check to our pre-upgrade reports.

- The example custom actor we are going to install implements checks for compliance with an imaginary organization's "reboot hygiene" policy. The actor will block the upgrade by reporting inhibitor risk if any of the following conditions are detected:

  - The host uptime is greater than the maximum defined by the policy.

  - The running kernel version does not match the default kernel version configured in the bootloader.

  - The /boot directory has any files that have been modified since the last reboot.

- Go to your VS Code browser tab and open a terminal session. Refer back to [Exercise 1.1, Step 2](https://github.com/swapdisk/workshops/blob/devel/exercises/ansible_ripu/1.1-setup/README.md#step-2---open-a-terminal-session) if you need a reminder of how we did that.

- Login to one of your pet app servers using the `ssh` command. For example:

  ```
  ssh tidy-bengal
  ```

- Now we will install the RPM package that provides our custom actor. Run the following command on your pet app server:

  ```
  sudo yum -y --enablerepo=leapp-supplements install leapp-upgrade-\*-supplements
  ```

  > **Note**
  >
  > We are installing the package manually just for the purpose of demonstrating this custom actor. If we were ready to roll out custom actors at enterprise scale, we would include the package installation at the beginning of our analysis playbook.

- This is an example of the output you should expect to see if the package is installed successfully:

  ```
  Resolving Dependencies
  --> Running transaction check
  ---> Package leapp-upgrade-el7toel8-supplements.noarch 0:1.0.0-47.demo.el7 will be installed
  --> Finished Dependency Resolution

  Dependencies Resolved

  ==========================================================================================
   Package                             Arch    Version             Repository          Size
  ==========================================================================================
  Installing:
   leapp-upgrade-el7toel8-supplements  noarch  1.0.0-47.demo.el7   leapp-supplements   12 k

  Transaction Summary
  ==========================================================================================
  Install  1 Package

  Total download size: 12 k
  Installed size: 18 k
  Downloading packages:
  leapp-upgrade-el7toel8-supplements-1.0.0-47.demo.el7.noarch.rpm    |  12 kB  00:00:00
  Running transaction check
  Running transaction test
  Transaction test succeeded
  Running transaction
    Installing : leapp-upgrade-el7toel8-supplements-1.0.0-47.demo.el7.noarch            1/1
    Verifying  : leapp-upgrade-el7toel8-supplements-1.0.0-47.demo.el7.noarch            1/1

  Installed:
    leapp-upgrade-el7toel8-supplements.noarch 0:1.0.0-47.demo.el7

  Complete!
  ```

- To demonstrate the custom actor at work, let's create a condition that violates our policy so that an inhibitor finding will be reported. Use this command:

  ```
  sudo touch /boot/policy-violation
  ```

  With this command, we just created a file under /boot with a timestamp later than the last reboot. This host is now out of compliance with our reboot hygiene policy!

### Step 3 - Generate a New Pre-upgrade Report

We are now ready to try running a pre-upgrade report including the checks from our custom actor.

- Return to your AAP Web UI browser tab. Navigate to Resources > Templates and open the "AUTO / 01 Analysis" job template. Launch the job choosing the "ALL_rhel" option at the "Select inventory group" prompt.

- When the job completes, go back to the RHEL Web Console and use the remote host menu to navigate to the pet app server where you installed the custom actor package. Refresh the pre-upgrade report. You should now see there is a new inhibitor finding. For example:

  ![Pre-upgrade report showing inhibitor finding from custom actor](images/reboot_hygiene.svg)

- Click on the finding to open the detail view. Here we see the summary with an explanation of the finding and the remediation hint which politely says please reboot:

  ![Finding details reported by reboot hygiene custom actor](images/reboot_hygiene_finding.svg)

- Reboot the host to resolve the inhibitor finding. For example:

  ```
  sudo reboot
  ```

- Now generate another pre-upgrade report after rebooting. Verify that this inhibitor finding has disappeared with the new report.

### Step 4 - Learn More About Customizing the In-place Upgrade

Read the knowledge article [Customizing your Red Hat Enterprise Linux in-place upgrade](https://access.redhat.com/articles/4977891) to understand best practices for handling the upgrade of third-party packages using custom repositories for an in-place upgrade or custom actors.

The gritty details of developing Leapp custom actors are beyond the scope of this workshop. Here are some resources you can check out to learn more on your own:

  - [Developer Documentation for Leapp](https://leapp.readthedocs.io/en/latest/): this documentation covers the internal workflow architecture of the Leapp framework and how to develop and test your own custom actors.

  - [Leapp Dashboard](https://oamg.github.io/leapp-dashboard/#/): dig around here to make sure the custom actor functionality you are considering doesn't already exist in the mainstream Leapp framework.

  - [oamg/leapp-supplements](https://github.com/oamg/leapp-supplements): GitHub repo where you can find example custom actors and contribute your own. It also has the `Makefile` for custom actor RPM packaging.

## Conclusion

In this exercise, we learned that custom modules can be Leapp custom actors or simply custom tasks added to your upgrade playbook. We demonstrated installing an RPM package that provides an example custom actor with additional pre-upgrade checks and generated a new pre-upgrade report to see it in action.

---

**Navigation**

[Previous Exercise](../1.4-remediate/README.md) - [Next Exercise](../1.6-my-pet-app/README.md)

[Home](../README.md)
