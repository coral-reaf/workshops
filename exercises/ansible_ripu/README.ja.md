# RHEL In-place Upgrade Automation ワークショップ

このワークショップでは、Red Hat Enterprise Linux (RHEL) のインプレース・アップグレードを自動化する包括的なアプローチを紹介します。このソリューションでは、Ansible Automation Platform (AAP) を使用して、大規模な RHEL ホスト全体でエンタープライズ規模のアップグレードを実行します。このワークショップでは、RHEL7 から RHEL8 へのアップグレード、および RHEL8 から RHEL9 へのアップグレードを実行するために、このアプローチの例を使用する方法を示します。また、このソリューションを企業環境の特殊な要件に合わせてカスタマイズする方法についても学びます。

このソリューション・アプローチには、規模を拡大して成功を収めるために推奨される4つの重要な特徴があります:

![Automate Everything, Snapshot/rollback, Custom Modules, Reporting Dashboard](images/ripu_key_features.svg)

このワークショップを進めるにつれ、これらの機能の重要性と、企業での実装方法のさまざまなオプションについて、より詳しく学ぶことができます。このワークショップでは、Ansible Automation Platformの使用経験、およびAnsible playbookとロールの使用経験があることを前提としています。Ansibleを初めて使用する場合は、まず次のワークショップを完了することをご検討ください。 [Ansible for Red Hat Enterprise Linux](https://aap2.demoredhat.com/exercises/ansible_rhel/README.ja.html).

## 目次

- [RHEL In-place Upgrade Automation ワークショップ](#rhel-in-place-upgrade-automation-workshop)
  - [目次](#目次)
  - [プレゼンテーション](#プレゼンテーション)
  - [Time Planning](#time-planning)
  - [Lab Diagram](#lab-diagram)
  - [Workshop Exercises](#workshop-exercises)
    - [Section 1 - Pre-upgrade Analysis](#section-1---pre-upgrade-analysis)
    - [Section 2 - RHEL OS Upgrade](#section-2---rhel-os-upgrade)
    - [Section 3 - Rolling Back](#section-3---rolling-back)
    - [Supplemental Exercises](#supplemental-exercises)
  - [Workshop Navigation](#workshop-navigation)

## プレゼンテーション

この演習は、RHEL のインプレースアップグレードを自動化するためのすべてのフェーズに対して、参加者をガイドします。すべてのコンセプトは、紹介されながら説明されます。

このワークショップで示されたアプローチの利点に関する追加情報をまとめたプレゼンテーション・デッキをオプションで用意しています。:
[RHEL In-place Upgrade Automation](../../decks/ansible_ripu.pdf)

## Time Planning

The time required to complete the workshop depends on the number of participants and how familiar they are with Linux and Ansible. The exercises themselves should take a minimum of 4 hours. The introduction in the optional presentation adds another hour. There are also some optional exercises which can be skipped, but are recommended if time allows. There are also supplemental exercises at the end of the workshop to allow for open-ended experimentation and exploring customizations that may apply to your specific environment and requirements. The lab environment provisioned could even be used for a multi-day deep dive workshop, but that is beyond the scope of this guide.

## Lab Diagram

The lab environment provisioned for the workshop includes a number of RHEL cloud instances. One instance is dedicated to hosting AAP and is used to run playbook and workflow jobs. The jobs are executed against the remaining hosts which will be upgraded in-place to the next RHEL major version. The automation uses LVM to manage the snapshot/rollback capability.

![RHEL In-place Upgrade Automation Workshop lab diagram](images/ripu_lab_diagram.svg)

## Workshop Exercises

The workshop is composed of three sections each of which includes a number of exercises. Each exercise builds upon the steps performed and concepts learned in the previous exercises, so it is important to do them in the prescribed order.

### Section 1 - Pre-upgrade Analysis

* [Exercise 1.1 - Workshop Lab Environment](1.1-setup/README.md)
* [Exercise 1.2 - Run Pre-upgrade Jobs](1.2-preupg/README.md)
* [Exercise 1.3 - Review Pre-upgrade Reports](1.3-report/README.md)
* [Exercise 1.4 - Perform Recommended Remediation](1.4-remediate/README.md)
* [Exercise 1.5 - (Optional) Custom Pre-upgrade Checks](1.5-custom-modules/README.md)
* [Exercise 1.6 - (Optional) Deploy a Pet App](1.6-my-pet-app/README.md)

### Section 2 - RHEL OS Upgrade

* [Exercise 2.1 - Run OS Upgrade Jobs](2.1-upgrade/README.md)
* [Exercise 2.2 - Let's Talk About Snapshots](2.2-snapshots/README.md)
* [Exercise 2.3 - Check if the Upgrade Worked](2.3-check-upg/README.md)
* [Exercise 2.4 - (Optional) How is the Pet App Doing?](2.4-check-pet-app/README.md)

### Section 3 - Rolling Back

* [Exercise 3.1 - (Optional) Trash the Instance](3.1-rm-rf/README.md)
* [Exercise 3.2 - Run Rollback Job](3.2-rollback/README.md)
* [Exercise 3.3 - Check if Upgrade Undone](3.3-check-undo/README.md)
* [Exercise 3.4 - Rinse and Repeat](3.4-conclusion/README.md)

## Workshop Navigation

Your will find links to the previous and next exercises at the bottom of each exercise page. Click the link below to get started.

---

**Navigation**

[Next Exercise](1.1-setup/README.md)
