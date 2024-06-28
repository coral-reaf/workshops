# ワークショップ演習 - スナップショットについてお話しましょう

## 目次

- [ワークショップ演習 - スナップショットについてお話しましょう](#workshop-exercise---スナップショットについてお話しましょう)
  - [目次](#目次)
  - [目的](#目的)
  - [ガイド](#ガイド)
    - [Step 1 - スナップショットとは何か、またそうではないものは何か](#step-1---スナップショットとは何か、またそうではないものは何か)
    - [Step 2 - さまざまなスナップショットソリューションの評価](#step-2---さまざまなスナップショットソリューションの評価)
      - [LVM](#lvm)
      - [VMware](#vmware)
      - [Amazon EBS](#amazon-ebs)
      - [休憩ミラー](#break-mirror)
      - [ReaR](#rear)
    - [Step 3 - スナップショットの範囲](#step-3---snapshot-scope)
    - [Step 4 - 最適なスナップショットソリューションの選択](#step-4---choosing-the-best-snapshot-solution)
  - [結論](#conclusion)

## 目標

* バックアップとスナップショットの違いを理解する
* スナップショットを実行するさまざまな方法について学習する
* スナップショットの自動化で遭遇する可能性のある課題や障壁に備える
* 組織に適したスナップショットの範囲を検討する

## ガイド

前回の演習では、ペット アプリケーション サーバーの RHEL インプレース アップグレードを開始するための自動化を開始しました。アップグレード ワークフロー テンプレートの最初のステップは、アップグレードする各 RHEL インスタンスのスナップショットを作成することです。アップグレードで問題が発生した場合、スナップショットによりアップグレードをすばやく元に戻すことができます。

スナップショットの自動化は、RHEL インプレース アップグレード ソリューション アプローチの最も難しい機能の 1 つです。この演習では、企業が直面するいくつかの課題を検討し、それらを克服するための戦略を検討します。

まず、スナップショットについて話すときに何を意味するのかを正確に定義することから始めましょう。

### ステップ 1 - スナップショットとは何か、またそうでないものは何か

成熟した従来のコンピューティング環境を持つほとんどの組織では、バックアップを実行するための標準とツールが実装されています。通常、バックアップは定期的なスケジュールで実行されます。より重要なデータやより動的なデータは、ほとんどが静的なデータよりも頻繁にバックアップされる可能性があります。多くの場合、完全バックアップはたまにしか実行せず、変更されたファイルをより頻繁に保存するために増分バックアップを使用する戦略があります。

バックアップを実行する理由は、何らかの理由で失われたデータを回復できるようにするためです。操作上の問題やソフトウェアの欠陥、または誤って削除されたためにデータが破損した場合、バックアップを使用すると時計を戻して失われたデータを簡単に復元できます。

ただし、サーバー全体が失われた場合、バックアップを使用して回復するのはより困難です。バックアップから何かを復元するには、まず新しいオペレーティング システムをインストールする必要があります。データは完全バックアップと複数の増分バックアップに分散される可能性があり、完全なサーバー回復にかかる時間がさらに長くなります。ほとんどの組織では、バックアップ ソリューションを個々のファイルやディレクトリの復元にのみ使用していますが、サーバー上のすべてのものを復元する準備は整っていません。たとえ整っていたとしても、そのような復元には長い時間がかかります。

スナップショットは、個々のファイルをバックアップおよび復元しないという点で異なります。代わりに、バックアップはストレージ デバイス レベルで動作し、論理ボリュームまたは仮想ディスク全体の内容を即座に保存します。バックアップとは異なり、スナップショットはバックアップされるデータのコピーを作成するのではなく、変更されたすべてのデータのコピーがコピーされる時点をマークします。このため、スナップショットに使用される基礎技術は、「コピー オン ライト」または COW と呼ばれることがよくあります。

COW スナップショットは、従来の完全バックアップや増分バックアップの代替にはなりませんが、いくつかの利点があります。最も重要な利点は、従来のバックアップと復元では数時間以上かかるのに対し、スナップショットの作成とロールバックはほぼ瞬時に行われることです。サーバー全体を迅速に過去に戻すことができるため、スナップショットは RHEL インプレース アップグレードを実行するリスクを軽減するのに最適です。

### Step 2 - Assessing Different Snapshot Solutions

選択できるスナップショット ソリューションにはさまざまな種類があります。それぞれに利点と欠点があり、次の表にまとめられています。:

| スナップショットの種類 | 機能 | 利点 | 欠点 |
| ------------- | --------- | -------- | -------- |
| LVM |<ul><li>ベアメタル</li><li>オンプレミス VM</li><li>クラウド*</li></ul>|<ul><li>外部 API アクセスは不要</li><li>スコープは OS のみまたはすべて</li></ul>|<ul><li>ボリューム グループに空き領域が必要</li>サイズが適切でない場合、スナップショットの領域が不足する可能性があります</li><li>自動化では /boot を個別にバックアップおよび復元する必要があります</ul>|
| VMware |<ul><li>オンプレミス VM (ESX)</li></ul>|<ul><li>シンプルで信頼性が高い</li><li>スコープにすべてが含まれる</li></ul>|<ul><li>ベアメタルなどはサポートされていません</li><li>VMware スナップショットを 3 日以上使用することは推奨されません</li><li>API アクセスの取得が困難な場合があります</li><li>オーバーコミットメントのため、データストアに空き領域がありません</li><li>すべてをスコープにすると大きすぎる場合があります</li></ul>|
| Amazon EBS |<ul><li>Amazon EC2</li></ul>|<ul><li>シンプルで信頼性が高い</li><li>無制限のストレージ容量</li><li>スコープは OS のみ、またはすべてにすることができます</li></ul>|<ul><li>AWS でのみ機能します</li></ul>|
|ミラーの解除 |<ul><li>ベア メタル</li></ul>|<ul><li>ハードウェア RAID 搭載サーバー用の LVM の代替</li></ul>|<ul><li>開発とテストに多大な労力が必要</li><li>RAID および Redfish API 標準は、ベンダーやハードウェア モデルによって異なります</li></ul>|
| ReaR |<ul><li>ベア メタル</li><li>オンプレミスの VM</li></ul>|<ul><li>スナップショット オプションが機能しない場合の最後の手段</li></ul>|<ul><li>実際にはスナップショットではありませんが、ブート ISO の完全回復機能を提供します</li></ul>|

次のセクションでは、長所と短所について詳しく説明します。

#### LVM

The Logical Volume Manager (LVM) is a set of tools included in RHEL that provide a way to create and manage virtual block devices known as logical volumes. LVM logical volumes are typically used as the block devices from which RHEL OS filesystems are mounted. The LVM tools support creating and rolling back logical volume snapshots. Automating these actions from an Ansible playbook is relatively simple.

> **Note**
>
> The snapshot and rollback automation capability implemented for our workshop lab environment creates LVM snapshots managed using Ansible roles from the [`infra.lvm_snapshots`](https://github.com/swapdisk/infra.lvm_snapshots#readme) collection.

Logical volumes are contained in a storage pool known as a volume group. The storage available in a volume group comes from one or more physical volumes, that is, block devices underlying actual disks or disk partitions. Typically, the logical volumes where the RHEL OS is installed will be in a "rootvg" volume group. If best practices are followed, applications and app data will be isolated in their own logicial volumes either in the same volume group or a separate volume group, "appvg" for example.

To create logical volume snapshots, there must be free space in the volume group. That is, the total size of the logical volumes in the volume group must be less than the total size of the volume group. The `vgs` command can be used query volume group free space. For example:

```
# vgs
  VG         #PV #LV #SN Attr   VSize  VFree
  VolGroup00   1   7   0 wz--n- 29.53g 9.53g
```

In the example above, the `VolGroup00` volume group total size is 29.53 GiB and there is 9.53 GiB of free space in the volume group. This should be enough free space to support rolling back a RHEL upgrade.

If there is not enough free space in the volume group, there are a few ways we can make space available:

- Adding another physical volume to the volume group (i.e., `pvcreate` and `vgextend`). For a VM, you would first configure an additional virtual disk.
- Temporarily remove a logical volume you don't need. For example, on bare metal servers, there is often a large /var/crash empty filesystem. Removing this filesystem from `/etc/fstab` and then using `lvremove` to remove the logical volume from which it was mounted will free up space in the volume group.
- Reducing the size of one or more logical volumes. This is tricky because first the filesystem in the logical volume needs to be shrunk. XFS filesystems do not support shrinking. EXT filesystems do support shrinking, but not while the filesystem is mounted. Until recently, this way of freeing up volume group space was considered a last resort to be attempted by only the most skilled Linux admin, but it now possible to safely automate shrinking logical volumes using the [`shrink_lv`](https://github.com/swapdisk/infra.lvm_snapshots/tree/main/roles/shrink_lv#readme) role of the aforementioned `infra.lvm_snapshots` collection.

After a snapshot is created, COW data will start to utilize the free space of the snapshot logical volume as blocks are written to the origin logical volume. Unless the snapshot is create with the same size as the origin, there is a chance that the snapshot could fill up and become invalid. Testing should be performed during the development of the LVM snapshot automation to determine snapshot sizings with enough cushion to prevent this. The `snapshot_autoextend_percent` and `snapshot_autoextend_threshold` settings in lvm.conf can also be used to reduce the risk of snapshots running out of space. The [`lvm_snapshots`](https://github.com/swapdisk/infra.lvm_snapshots/tree/main/roles/lvm_snapshots#readme) role of the `infra.lvm_snapshots` collection supports variables that may be used to automatically configure the autoextend settings.

Unless you have the luxury of creating snapshots with the same size as their origin volumes, LVM snapshot sizing needs to be thoroughly tested and free space usage carefully monitored. However, if that challenge can be met, LVM snapshots offer a reliable snapshot solution without the headache of depending on external infrastructure such as VMware.

#### VMware

A VMware snapshot preserves the state and data of a VM at a specific point in time. Because VMware snapshots operate at the hypervisor level, they are completely independent of the guest OS. This makes them foolproof to anything that can go wrong during a RHEL upgrade. Even if an upgrade fails so badly that the OS can't even be booted up again, reverting the VMware snapshot will still save the day. For these reasons, VMware snapshots appear to be a very compelling snapshot option.

VMware snapshots can be manually created and reverted using the vSphere management UI. To create or revert a VMware snapshot automatically from an Ansible playbook, access permissions to the required vSphere API calls must be authorized for the AAP control node.

In our experience, having this access granted can be extremely challenging. The team that controls the VMware environment in most organizations is deeply invested in the "ClickOps" model of doing everything manually using the vSphere management UI. They may also be hesitant to trust that automation developed outside of their team can be trusted to perform the operations they would do manually to create a VMware snapshot, including checking for sufficient free space in the VMFS data store where the snapshot will be created.

The VMware team may resist supporting snapshots because of limited storage space. While standard VMDK files are fixed in size, COW snapshots will grow over time and require careful monitoring with data stores in VMware environments often running tight on capacity.

Another justification for pushing back on supporting automated snapshots will be the VMware vendor recommendation that snapshots should never be used for more than 72 hours (see KB article [Best practices for using VMware snapshots in the vSphere environment](https://kb.vmware.com/s/article/1025279)). Unfortunately, app teams usually need more than 3 days of soak time before they are comfortable that no impact to their apps has resulted from a RHEL upgrade.

VMware snapshots work great when they can be automated. If you are considering this option, engage early with the team that controls the VMware environment for your organization and be prepared for potential resistance.

#### Amazon EBS

Amazon Elastic Block Store (Amazon EBS) provides the block storage volumes used for the virtual disks attached to AWS EC2 instances. When a snapshot is created for an EBS volume, the COW data is written to Amazon S3 object storage.

While EBS snapshots operate independently from the guest OS running on the EC2 instance, the similarity to VMware snapshots ends there. An EBS snapshot saves the data of the source EBS volume, but does not save the state or memory of the EC2 instance to which the volume is attached. Also unlike with VMware, EBS snapshots can be created for an OS volume only while leaving any separate application volumes as is.

Automating EBS snapshot creation and rollback is fairly straightforward assuming your playbooks can access the required AWS APIs. The tricky bit of the automation is identifying the EC2 instance and attached EBS volume that corresponds to the target host in the Ansible inventory managed by AAP, but this can be solved by setting identifying tags on your EC2 instances.

#### Break Mirror

This method is an alternative to LVM that can be used with bare metal servers where the root disk is on a hardware RAID mirror set. Technically speaking, it is not a snapshot, but it still provides a near instantaneous rollback capability.

Instead of creating a snapshot just before starting the upgrade, the automation reconfigures the RAID controller to break the mirror set of the root disk so then it's just two JBOD disks. One of the JBOD disks is used going forward with the upgrade while the other is left untouched. To perform a rollback, the mirror set is reconstructed from the untouched JBOD.

Most bare metal servers support out-of-band management and those manufactured in the last decade will support APIs based on the [Redfish](https://www.dmtf.org/standards/redfish) standard. These APIs can be used by automation to break and reconstruct the mirror set, but be prepared for a significant development and testing effort because the API implementations are not always the same across different vendors and server models.

#### ReaR

ReaR (Relax and Recover) is a backup and recovery tool that is included with RHEL. ReaR doesn't use snapshots, but it does make it very easy to perform a full backup and restore of your RHEL server. When taking a full backup, ReaR creates a bootable ISO image with the current state of the server. To use a ReaR backup to revert an in-place upgrade, we simply boot the server from the ISO image and then choose the "Automatic Recover" option from the menu.

While ReaR backup and recovery is not instantaneous like rolling back a snapshot, it is remarkably fast compared to recovery tools that require you to perform a fresh OS install and then manually recover at a file level.

Read the article [ReaR: Backup and recover your Linux server with confidence](https://www.redhat.com/sysadmin/rear-backup-and-recover) to learn more.

### Step 3 - Snapshot Scope

The best practice for allocating the local storage of a RHEL servers is to configure volumes that separate the OS from the apps and app data. For example, the OS filesystems would be under a "rootvg" volume group while the apps and app data would be in an "appvg" volume group or at least in their own dedicated logical volumes. This separation helps isolate the storage usage requirements of these two groups so they can be manged based on their individual requirements and are less likely to impact each other. For example, the backup profile for the OS is likely different than for the apps and app data.

This practice helps to enforce a key tenet of the RHEL in-place upgrade approach: that is that the OS upgrade should leave the applications untouched with the expectation that system library forward compatibility and middleware runtime abstraction reduces the risk of the RHEL upgrade impacting app functionality.

With these concepts in mind, let's consider if we want to include the apps and app data in what gets rolled back if we need to revert the RHEL upgrade:

| Snapshot scope | Benefits | Drawbacks |
| -------------- | -------- | --------- |
| OS only |<ul><li>Simplifies storage requirements</li><li>Preserves isolation of OS changes from apps and app data</li><li>Reduces risk of rolling back impacting external apps</li></ul>|<ul><li>Probably not possible with VMware snapshots</li><li>Discipline required to avoid temptation of trying quick app changes to fix impacts</li></ul>|
| OS and apps/data |<ul><li>Reduces risk if trying to fix app impact during maintenance window</li><li>Helpful if app impact could lead to app data corruption</li></ul>|<ul><li>More storage space required</li><li>Rolling back app data could impact external systems</li></ul>|

When snapshots only include the upgraded OS volumes, the best practice of isolating OS changes from app changes is followed. In this case, it is important to resist the temptation to make some heroic app changes in an attempt to avoid rolling back in the face of application impact after a RHEL upgrade. For the sake of safety and soundness, gather the evidence required to help understand what caused any app impact, but then do a rollback. Don't make any app changes that could be difficult to untangle after rolling back the OS.

Unfortunately, a VMware snapshot saves the full state of a VM instance including all virtual disks irrespective of whether they contain OS or app data. This can prove challenging for a couple reasons. First, more storage space will be required for the snapshots and it is more difficult to anticipate how much snapshot growth will result because of app data activity. The other problem is that rolling back app data may result in the app state becoming out of sync with external systems leading to unpredictable issues. When rolling back app data for any reason, be aware of the potential headaches that may result.

### Step 4 - Choosing the Best Snapshot Solution

There are a number of factors you should consider when deciding which method of snapshot/rollback will work best in your environment.

- What is your mix of bare metal servers versus VMware or cloud instances?
- Where can free space most readily be made available?
- Can you get unfettered access to your VMware inventory and vSphere APIs?
- What is the appropriate snapshot scope for your organization?
- Which snapshot solution can you most easily make fully automated?

Consider a belt and suspenders approach, that is, offer support for more than one method. Maybe it makes sense to recommend one method for bare metal and another for VMs.

Whatever your decision, remember that an effective snapshot/rollback capability integrated with your end-to-end automation is the most important feature of any RHEL in-place upgrade solution.

## Conclusion

In this exercise, we learned about the pros and cons of a number of different methods of achieving an automated snapshot/rollback capability. We also considered the risks that can happen because of rolling back app data that isn't isolated from OS changes. With this knowledge, you are ready to make more informed decisions when designing your snapshot/rollback automation approach.

In the next exercise, we'll go back to look at how the RHEL in-place upgrades are progressing on our pet application servers.

---

**Navigation**

[Previous Exercise](../2.1-upgrade/README.md) - [Next Exercise](../2.3-check-upg/README.md)

[Home](../README.md)
