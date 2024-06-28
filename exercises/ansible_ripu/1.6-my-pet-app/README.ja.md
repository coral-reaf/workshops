# ワークショップ演習 - ペット アプリのデプロイ

## 目次

- [ワークショップ演習 - ペットアプリのデプロイ](#ワークショップ演習---ペットアプリのデプロイ)
  - [目次](#目次)
  - [オプション演習](#オプション演習)
  - [目標](#目標)
  - [ガイド](#ガイド)
    - [Step 1 - 従来のアプリケーション ライフサイクル](#step-1---the-traditional-application-lifecycle)
    - [Step 2 - 愛するペットアプリケーションのインストール](#step-2---installing-our-beloved-pet-application)
    - [Step 3 - ペットアプリケーションのテスト](#step-3---test-the-pet-application)
    - [Step 4 - 再起動時にアプリケーションが起動するように構成](#step-4---再起動時にアプリケーションが起動するように構成)
    - [Step 5 - 別のアップグレード前レポートを実行する](#step-5---別のアップグレード前レポートを実行する)
  - [まとめ](#まとめ)

## オプション演習

これはオプションの演習です。ワークショップを正常に完了するために必須ではありませんが、時間に余裕があれば試してみることをお勧めします。次のセクションに記載されている目標を確認して、この演習を行うか、それとも次の演習に進むかを決めてください。

* [演習 2.1 - OS アップグレード ジョブを実行する](../2.1-upgrade/README.ja.md)

## 目標

* 従来のサーバー環境でアプリケーションがどのように展開および維持されるかについて説明します
* サンプルのペット アプリケーションをインストールするか、独自のアプリケーションを用意します
* アプリケーションが期待どおりに機能しているかどうかをテストする方法を検討します

## ガイド

### ステップ 1 - 従来のアプリケーション ライフサイクル

少し立ち止まって、インプレース アップグレードを実行する理由について考えてみましょう。新しい RHEL バージョンで新しいサーバーまたは VM インスタンスを展開し、そこからアプリケーションの新規インストールを行うのがベスト プラクティスではないでしょうか。

- はい、でも...

  - アプリ チームがアプリの展開を自動化しておらず、代わりにすべてを手動でインストールして構成している場合はどうでしょうか。

  - 何年も前にアプリをインストールして以来、問題を解決したり、変化するビジネス要件に対応したりするために、アプリ環境に変更を加えてきた場合はどうでしょうか?

  - 蓄積されたドリフトや技術的負債をすべて把握できなくなり、一からやり直すのが非常に困難になった場合はどうでしょうか?

- 残念ながら、多くのアプリ チームがこのような状況に陥っています。従来のアプリ サーバーは、長年、愛するペットのように大切に扱われてきました。それを捨てて、すべてを最初からやり直すという考えは考えられません。

- アプリケーションに触れることなく、新しい RHEL バージョンに移行できるのであれば、それは非常に魅力的な選択肢です。そのため、インプレース アップグレードを行い、すべてを手動でインストールして再構成するという面倒な作業に悩まされることなく、企業のプラットフォーム ライフサイクル コンプライアンス レポートから姿を消すことを望んでいます。

- このオプションの演習では、アプリケーションをインストールして、RHEL インプレース アップグレードが何らかの影響を与えるかどうかを評価します。アップグレード後も、新しい RHEL バージョンでアプリが期待どおりに機能するかどうかを確認します。

### ステップ 2 - 愛するペットアプリケーションのインストール

このステップでは、サンプル アプリケーションをインストールします。インストールは昔ながらの方法で行います。つまり、混乱を招き、エラーが発生しやすい可能性がある従来のコマンドライン 手順に従って手動でインストールします。結局のところ、アプリケーションのデプロイメントがエンドツーエンドで自動化されていれば、インプレース アップグレードを行う必要はありません。

別のアプリケーション (たとえば、潜在的な影響をテストしたいエンタープライズ環境の実際のアプリケーション) をインストールすることもできます。以下の手順をスキップして、独自の冒険をしてください。アップグレードの前後でアプリケーションをテストするようにしてください。

- サンプル アプリケーションは、Java で記述された [Spring Pet Clinic サンプル アプリケーション](https://github.com/spring-projects/spring-petclinic) です。これは、Maven を使用して構築される Spring Boot アプリケーションです。起動時にサンプル データが入力される MySQL データベースに接続します。

> **注意**
>
> 以下の手順では、サンプル アプリケーションを手動でインストールして起動するために必要なコマンドを段階的に実行します。これは、企業内で従来の古いアプリケーションがどのようにデプロイされているかを模倣するために行われます。一連のコマンドを手動で実行する代わりにショートカットを使用する場合は、「PETS / App Install」ジョブ テンプレートを起動してサンプル アプリケーションをデプロイできます。
>
> アプリケーションのインストール ジョブが成功した場合は、[Step 3 - ペットアプリケーションのテスト](#step-3---ペットアプリケーションのテスト) に進んでください。
>

- アプリケーションのインストール手順の最初のステップは、Java JDK をインストールすることです。ここでは、楽しみのためにサードパーティの JDK を使用します。
> **警告**
>
> すべてのコマンドは、root ではなく ec2-user として実行する必要があります。root を必要とするコマンドでは、`sudo` が使用されます。

ペット アプリ サーバーにログインし、次のコマンドを実行します:

```
distver=$(sed -r 's/([^:]*:){4}//;s/(.).*/\1/' /etc/system-release-cpe)
sudo yum-config-manager --add-repo=https://packages.adoptium.net/artifactory/rpm/rhel/$distver/x86_64
sudo yum-config-manager --save --setopt=\*adoptium\*.gpgkey=https://packages.adoptium.net/artifactory/api/gpg/key/public
sudo yum install mariadb mariadb-server temurin-17-jdk
```

最後のコマンドからのプロンプトにはすべて `y` と答えます。

- temurin-17-jdk パッケージがインストールされていることを確認します:

```
rpm -q temurin-17-jdk
```

そうでない場合は、戻って何が間違っていたのかを調べます。

- 次に、ec2-user のホーム ディレクトリに Spring Pet Clinic サンプル アプリケーションをインストールします。次のコマンドを実行します:

```
cd ~
git clone https://github.com/spring-projects/spring-petclinic.git
```

これで、アプリケーション ファイルが `spring-petclinic` ディレクトリにインストールされていることがわかります。

<!-- ワークショップの EC2 インスタンスには、firewalld はありませんが、firewalld がある場所でこの手順を使用する場合は、次のコマンドを使用してファイアウォールを開きます:

```
sudo firewall-cmd --add-port=8080/tcp
sudo firewall-cmd --add-port=8080/tcp --permanent
```
-->
- データベース サーバーを起動し、アプリケーション用のデータベースを作成する必要があります。次のコマンドを使用して、データベースを有効にして起動します:

```
sudo systemctl enable --now mariadb
```

次に、`mysql` コマンドライン クライアントを使用して、データベース サーバーに接続します。例:

```
mysql --user root
```

`MariaDB [(none)]>` プロンプトが表示されます。このプロンプトで、次の SQL コマンドを入力します:

```
CREATE DATABASE IF NOT EXISTS petclinic;
ALTER DATABASE petclinic DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON petclinic.* TO 'petclinic'@'localhost' IDENTIFIED BY 'petclinic';
FLUSH PRIVILEGES;
quit
```

- これで、アプリケーション Web サービスを開始する準備ができました。 バックグラウンドで実行するには、次のコマンドを使用します:

```
echo 'cd $HOME/spring-petclinic && ./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql >> $HOME/app.log 2>&1' | at now
```

- アプリケーションの初回起動には数分かかります。 `app.log` ファイルをチェックして進行状況を追跡し、Web サービスが正常に開始されたことを確認します:

```
tailf ~/app.log
```

この例のように、ログ出力の下部にイベントがリストされている場合、アプリケーションが正常に開始され、テストの準備ができていることを意味します:

```
o.s.b.a.e.web.EndpointLinksResolver : ベース パス '/actuator' の下に 13 個のエンドポイントを公開しています
o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat がポート 8080 (http) でコンテキスト パス '' で開始されました
o.s.s.petclinic.PetClinicApplication : PetClinicApplication を 6.945 秒で開始しました (プロセスは 7.496 秒実行されました)
```

Ctrl-C を入力して `tailf` コマンドを終了します。

### Step 3 - Test the Pet Application

Now that we have installed our application and verified it is running, it's time to test how it works.

- Use this command to determine the application's external URL:

  ```
  echo "http://$(curl -s ifconfig.me):8080"
  ```

- Open a new web browser tab. Cut and paste the URL that was output by the command above. This should open the application web user interface. If the application is working correctly, you should see something like this:

  ![Pet Clinic application home page](images/petapp_home.svg)

- Try the different function tabs at the top of the web user interface. For example, navigate to "FIND OWNERS" and search for Davis. Click on one of the owner records to see their details.

  Use the "Edit Owner" and "Add New Pet" buttons to make changes and add new records.

  The "ERROR" tab in the top-right corner is a error handling test function. If you click it, the expected result is a "Something happened..." message will be displayed and a runtime exception and stack trace will be logged in the `app.log` file.

- Play with the application until you feel comfortable you understand its expected behavior. After the upgrade, we will test it again to verify it has not been impacted.

### Step 4 - Configure the Application to Start on Reboot

Right now, our application was started manually. We need to configure the app so it will start up automatically when our server is rebooted.

- Use this command to configure a reboot cron entry so the application will be started automatically after every reboot:

  ```
  echo '@reboot cd $HOME/spring-petclinic && ./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql >> $HOME/app.log 2>&1' | crontab -
  ```

  > **NOTE**
  >
  > If you deployed the example app using the "PETS / App Install" playbook job, you can skip the command above because the reboot cron entry was automatically created.
  >

- Now reboot the server to verify this works:

  ```
  sudo reboot
  ```

- Try refreshing the web browser tab where you have the Pet Clinic web app open. While the server is rebooting, you may see a timeout or connection refused error. After the reboot is finished, the web app should be working again.

  > **Note**
  >
  > Because the external IP addresses of the EC2 instances provisioned for the workshop are dynamically assigned (i.e., using DHCP), it is possible that the web user interface URL might change after a reboot. If you are unable to access the app after the reboot, run this command again to determine the new URL for the application web user interface:
  >
  > ```
  > echo "http://$(curl -s ifconfig.me):8080"
  > ```
  >

  FIXME: Shame on us for not using DNS!

### Step 5 - Run Another Pre-upgrade Report

Whenever changes are made to a server, its a good idea to rerun the Leapp pre-upgrade report to make sure those changes have not introduced any new risk findings.

- Launch the "AUTO / 01 Analysis" job template to generate a fresh pre-upgrade report. After the job finishes, review the report to see if there are any new findings. Refer to the steps in the previous exercises if you don't have them memorized by heart already.

- Did you notice that this high risk finding has popped up now?

  ![Packages not signed by Red Hat found on the system high risk finding](images/packages_not_signed_by_rh.svg)

- If we open the finding, we are presented with the following details:

  ![Packages not signed by Red Hat details view](images/packages_not_signed_details.svg)

- "Packages not signed by Red Hat" is just a fancy way of referring to 3rd-party packages and/or package that are built in-house by your app teams. In the case of this finding, the package that has been identified is `temurin-17-jdk`, the 3rd-party JDK runtime package we installed. The finding is warning that there is a risk of this package being removed during the upgrade if there are unresolvable dependencies.

  There is only one surefire way to know if the package will be removed or not. Let's run the upgrade and see what happens!

## Conclusion

In this exercise, we discussed the sorry state of traditional application maintenance, untracked drift and technical debt. We installed a 3rd-party Java runtime and then installed the Pet Clinic application on top of that. We made certain that our app is functioning as expected, but we also discovered a new "high risk" finding on our pre-upgrade report.

Congratulations on completing all the exercises in the first section of the workshop. It's time now to upgrade RHEL and see if there will be any application impact.

---

**Navigation**

[Previous Exercise](../1.5-custom-modules/README.md) - [Next Exercise](../2.1-upgrade/README.md)

[Home](../README.md)
