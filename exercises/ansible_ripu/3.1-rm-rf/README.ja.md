# ワークショップ演習 - インスタンスを破棄

## 目次

- [ワークショップ演習 - インスタンスを破棄](#workshop-exercise---trash-the-instance)
  - [目次](#table-of-contents)
  - [オプション演習](#オプション演習)
  - [目標](#目標)
  - [ガイド](#ガイド)
    - [Step 1 - ペットアプリ サーバーの選択](#step-1---ペットアプリ サーバーの選択)
    - [Step 2 - アプリケーションを壊す](#step-2---アプリケーションを壊す)
        - すべてを削除
        - glibc をアンインストール
        - アプリケーションを破棄
        - ブートレコードを消去
        - ディスクをいっぱいにする
        - フォーク爆弾を発射する
  - [まとめ](#まとめ)

## オプションの演習

これはオプションの演習です。ワークショップを正常に完了するために必須ではありませんが、RHEL アップグレードのロールバックの有効性を示すのに役立ちます。次のセクションに記載されている目標を確認して、この演習を行うか、次の演習に進むかを決めてください。

* [演習 3.2 - ロールバックジョブを実行する](3.2-rollback/README.ja.md)

## 目標

* 失敗した OS アップグレードまたはアプリケーションへの影響をシミュレートする
* スナップショットのロールバックの範囲を示す

## ガイド

RHEL ホストで `rm -rf /*` を実行して何が起こるか試してみたいと思ったことはありませんか?あるいは、同じように破壊的な再帰コマンドを誤って実行し、その結果をすでに知っているかもしれません。この演習では、ペット アプリ サーバーの 1 つを意図的に台無しにして、ロールバックによって事態を打開する方法を説明します。

では、始めましょう!

### Step 1 - ペット アプリ サーバーの選択

次の演習では、サーバーの 1 つで RHEL アップグレードをロールバックします。

- アプリ サーバーを選択します。現在 RHEL8 になっている RHEL7 インスタンスの 1 つ、または RHEL9 にアップグレードされた RHEL8 インスタンスの 1 つを選択できます。

- [演習 1.1: Step 2](../1.1-setup/README.ja.md#step-2---ターミナルセッションを開く) で使用した手順に従って、ロールバックするアプリ サーバーでターミナル セッションを開きます。

- シェル プロンプトで、`sudo -i` コマンドを使用して、ルート ユーザーに切り替えます。たとえば:

```
[ec2-user@cute-bedbug ~]$ sudo -i
[root@cute-bedbug ~]#
```

上記の例のようなルートプロンプトが表示されることを確認します。

### Step 2 - アプリケーションを壊す

- [演習 1.6: Step 5](../1.6-my-pet-app/README.ja.md#step-5---別のアップグレード前レポートの実行) では、解決できない依存関係がある場合にアップグレード中に `temurin-17-jdk` サードパーティ JDK ランタイム パッケージが削除される可能性があるという潜在的なリスクを警告するアップグレード前の調査結果を確認しました。もちろん、ペットアプリは引き続き完全に動作しているため、このようなことは発生していないことはわかっています。

しかし、このパッケージが削除されたらどうなるでしょうか。ペットアプリが機能するには JDK ランタイムが必要です。これがないと、アプリケーションは中断されます。これをシミュレートするには、次のようにパッケージを手動で削除します:

```
dnf -y remove temurin-17-jdk
```

ここで、アプリケーション サーバーを `reboot` すると、ペット アプリケーションは起動せず、`~/app.log` ファイルの末尾に次のエラーが表示されます:

```
...
which: no javac in (/home/ec2-user/.local/bin:/home/ec2-user/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
Error: JAVA_HOME が正しく定義されていません。
実行できません
```

これは、アップグレードをロールバックすることで元に戻せるアプリケーションへの影響の現実的な例です。

## まとめ

おめでとうございます。アプリケーションサーバーの 1 つが動かなくなりました。面白かったですか?

次の演習では、アップグレードをロールバックして元の状態に戻します。

---

**ナビゲーション**

[前の演習](../2.4-check-pet-app/README.ja.md) - [次の演習](../3.2-rollback/README.ja.md)

[ホーム](../README.ja.md)