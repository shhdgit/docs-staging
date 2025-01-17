---
title: Organization SSO Authentication
summary: Learn how to log in to the TiDB Cloud console via your customized organization authentication.
---

# 組織のSSO認証 {#organization-sso-authentication}

シングル サインオン (SSO) は、 TiDB Cloud[組織](/tidb-cloud/tidb-cloud-glossary.md#organization)のメンバーが、電子メール アドレスとパスワードの代わりにアイデンティティ プロバイダー (IdP) からのアイデンティティを使用してTiDB Cloudにログインできるようにする認証スキームです。

TiDB Cloud は、次の 2 種類の SSO 認証をサポートしています。

-   [基本的な SSO](/tidb-cloud/tidb-cloud-sso-authentication.md) : メンバーは、GitHub、Google、または Microsoft の認証方法を使用して[TiDB Cloudコンソール](https://tidbcloud.com/)にログインできます。基本 SSO は、 TiDB Cloudのすべての組織に対してデフォルトで有効になっています。

-   クラウド組織 SSO: メンバーは、組織が指定した認証方法を使用して、 TiDB Cloudのカスタム ログイン ページにログインできます。クラウド組織 SSO はデフォルトでは無効になっています。

基本的な SSO と比較して、クラウド組織 SSO は柔軟性とカスタマイズ性が高いため、組織のセキュリティとコンプライアンスの要件をより適切に満たすことができます。たとえば、ログイン ページに表示する認証方法を指定したり、ログインを許可する電子メール アドレス ドメインを制限したり、 [OpenID Connect (OIDC)](https://openid.net/connect/)または[Securityアサーション マークアップ言語 (SAML)](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) ID プロトコルを使用する ID プロバイダー (IdP) を使用してメンバーがTiDB Cloudにログインできるようにしたりできます。 。

このドキュメントでは、組織の認証スキームを基本 SSO からクラウド組織 SSO に移行する方法について説明します。

> **注記：**
>
> -   クラウド組織 SSO 機能はベータ版であり、リクエストがあった場合にのみ利用可能です。この機能をリクエストするには、 **「?」**をクリックしてください。 [TiDB Cloudコンソール](https://tidbcloud.com)の右下隅にある**[サポートをリクエスト]**をクリックします。次に、「**説明」**フィールドに「クラウド組織 SSO の申請」と入力し、 **「送信」**をクリックします。
> -   現在の TiDB ログイン URL が`https://tidbcloud.com/enterprise/`で始まる場合は、クラウド組織 SSO が組織ですでに有効になっていることを意味します。

## あなたが始める前に {#before-you-begin}

クラウド組織 SSO に移行する前に、組織に関するこのセクションの項目を確認してください。

> **注記：**
>
> -   クラウド組織 SSO を一度有効にすると、無効にすることはできません。
> -   クラウド組織 SSO を有効にするには、 TiDB Cloud組織の`Organization Owner`ロールに所属する必要があります。役割の詳細については、 [ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

### 組織のTiDB Cloudログイン ページのカスタム URL を決定します。 {#decide-a-custom-url-for-the-tidb-cloud-login-page-of-your-organization}

クラウド組織 SSO が有効になっている場合、メンバーは、パブリック ログイン URL ( `https://tidbcloud.com` ) の代わりにカスタム URL を使用してTiDB Cloudにログインする必要があります。

カスタム URL は有効化後に変更できないため、どの URL を使用するかを事前に決定する必要があります。

カスタム URL の形式は`https://tidbcloud.com/enterprise/signin/your-company-name`で、会社名をカスタマイズできます。

### 組織メンバーの認証方法を決定する {#decide-authentication-methods-for-your-organization-members}

TiDB Cloud は、組織 SSO 用に次の認証方法を提供します。

-   グーグル
-   GitHub
-   マイクロソフト
-   OIDC
-   SAML

クラウド組織 SSO を有効にすると、最初の 3 つの方法がデフォルトで有効になります。

有効な認証方法はすべてカスタムTiDB Cloudログイン ページに表示されるため、どの認証方法を有効または無効にするかを事前に決定する必要があります。

### 自動プロビジョニングを有効にするかどうかを決定する {#decide-whether-to-enable-auto-provision}

自動プロビジョニングは、メンバーが`Organization Owner`または`Project Owner`からの招待を必要とせずに自動的に組織に参加できるようにする機能です。 TiDB Cloudでは、サポートされているすべての認証方法に対してデフォルトで無効になっています。

-   認証方法の自動プロビジョニングが無効になっている場合、 `Organization Owner`または`Project Owner`によって招待されたユーザーのみがカスタム URL にログインできます。
-   認証方法で自動プロビジョニングが有効になっている場合、この認証方法を使用するすべてのユーザーがカスタム URL にログインできます。ログイン後、組織内のデフォルトの**メンバー**役割が自動的に割り当てられます。

セキュリティを考慮して、自動プロビジョニングを有効にすることを選択した場合は、 [認証方法の詳細を構成する](#step-2-configure-authentication-methods)時に許可される電子メール ドメインを制限することをお勧めします。

### クラウド組織の SSO 移行計画についてメンバーに通知する {#notify-your-members-about-the-cloud-organization-sso-migration-plan}

クラウド組織 SSO を有効にする前に、次のことをメンバーに必ず通知してください。

-   TiDB Cloudのカスタム ログイン URL
-   ログインに`https://tidbcloud.com`の代わりにカスタム ログイン URL の使用を開始する時期
-   利用可能な認証方法
-   メンバーがカスタム URL にログインするための招待が必要かどうか

## ステップ 1. クラウド組織 SSO を有効にする {#step-1-enable-cloud-organization-sso}

クラウド組織 SSO を有効にするには、次の手順を実行します。

1.  `Organization Owner`役割を持つユーザーとして[TiDB Cloudコンソール](https://tidbcloud.com)にログインします。

2.  TiDB Cloudコンソールの左下隅にある をクリックします。<mdsvgicon name="icon-top-organization">をクリックし、 **[組織の設定]**をクリックします。</mdsvgicon>

3.  **[組織の設定]**ページで、 **[認証]**タブをクリックし、 **[有効にする]**をクリックします。

4.  ダイアログで、組織のカスタム URL を入力します。これはTiDB Cloud内で一意である必要があります。

    > **注記：**
    >
    > クラウド組織 SSO を有効にすると、URL を変更できなくなります。組織内のメンバーは、カスタム URL を使用してのみTiDB Cloudにログインできます。構成された URL を後で変更する必要がある場合は、 [TiDB Cloudのサポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

5.  **[理解して確認します**] チェック ボックスをクリックし、 **[有効にする]**をクリックします。

    > **注記：**
    >
    > ダイアログにクラウド組織 SSO に再招待および再参加するユーザーのリストが含まれている場合、クラウド組織 SSO を有効にした後、 TiDB Cloudはそれらのユーザーに招待メールを自動的に送信します。招待メールを受信した後、各ユーザーはメール内のリンクをクリックして身元を確認する必要があり、カスタム ログイン ページが表示されます。

## ステップ 2. 認証方法を構成する {#step-2-configure-authentication-methods}

TiDB Cloudで認証方法を有効にすると、その方法を使用するメンバーがカスタム URL を使用してTiDB Cloudにログインできるようになります。

### Google、GitHub、または Microsoft の認証方法を構成する {#configure-google-github-or-microsoft-authentication-methods}

Cloud Organization Cloud を有効にした後、次のように Google、GitHub、または Microsoft 認証方法を構成できます。

1.  **[組織の設定]**ページで、必要に応じて Google、GitHub、または Microsoft の認証方法を有効または無効にします。

2.  有効な認証方法については、 <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 20H21M3.00003 20H4.67457C5.16376 20 5.40835 20 5.63852 19.9447C5.84259 19.8957 6.03768 19.8149 6.21663 19.7053C6.41846 19.5816 6.59141 19.4086 6.93732 19.0627L19.5001 6.49998C20.3285 5.67156 20.3285 4.32841 19.5001 3.49998C18.6716 2.67156 17.3285 2.67156 16.5001 3.49998L3.93729 16.0627C3.59139 16.4086 3.41843 16.5816 3.29475 16.7834C3.18509 16.9624 3.10428 17.1574 3.05529 17.3615C3.00003 17.5917 3.00003 17.8363 3.00003 18.3255V20Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg>メソッドの詳細を設定します。

3.  メソッドの詳細では、以下を設定できます。

    -   [**アカウントの自動プロビジョニング**](#decide-whether-to-enable-auto-provision)

        デフォルトでは無効になっています。必要に応じて有効にすることができます。セキュリティを考慮して、自動プロビジョニングを有効にする場合は、認証に許可される電子メール ドメインを制限することをお勧めします。

    -   **許可された電子メールドメイン**

        このフィールドを構成すると、この認証方法で指定された電子メール ドメインのみがカスタム URL を使用してTiDB Cloudにログインできるようになります。ドメイン名を入力する場合は、 `@`記号を除外し、カンマで区切る必要があります。たとえば、 `company1.com,company2.com` 。

        > **注記：**
        >
        > 電子メール ドメインを構成している場合は、 TiDB Cloudによってロックアウトされないように、設定を保存する前に、現在ログインに使用している電子メール ドメインを必ず追加してください。

4.  **「保存」**をクリックします。

### OIDC 認証方法を構成する {#configure-the-oidc-authentication-method}

OIDC アイデンティティ プロトコルを使用するアイデンティティ プロバイダーがある場合は、 TiDB Cloudログインの OIDC 認証方法を有効にすることができます。

TiDB Cloudでは、OIDC 認証方法はデフォルトで無効になっています。 Cloud Organization Cloud を有効にした後、次のように OIDC 認証方法を有効にして構成できます。

1.  TiDB Cloud Organization SSO のアイデンティティ プロバイダーから次の情報を取得します。

    -   発行者のURL
    -   クライアントID
    -   クライアントシークレット

2.  **[組織の設定]**ページで、 **[認証]**タブをクリックし、 **[認証方法]**領域で OIDC の行を見つけて、[認証] をクリックします。 <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 20H21M3.00003 20H4.67457C5.16376 20 5.40835 20 5.63852 19.9447C5.84259 19.8957 6.03768 19.8149 6.21663 19.7053C6.41846 19.5816 6.59141 19.4086 6.93732 19.0627L19.5001 6.49998C20.3285 5.67156 20.3285 4.32841 19.5001 3.49998C18.6716 2.67156 17.3285 2.67156 16.5001 3.49998L3.93729 16.0627C3.59139 16.4086 3.41843 16.5816 3.29475 16.7834C3.18509 16.9624 3.10428 17.1574 3.05529 17.3615C3.00003 17.5917 3.00003 17.8363 3.00003 18.3255V20Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg> OIDC メソッドの詳細を表示します。

3.  メソッドの詳細では、以下を設定できます。

    -   **名前**

        カスタム ログイン ページに表示される OIDC 認証方法の名前を指定します。

    -   **発行者 URL** 、**クライアント ID** 、および**クライアント シークレット**

        IdP から取得した対応する値を貼り付けます。

    -   [**アカウントの自動プロビジョニング**](#decide-whether-to-enable-auto-provision)

        デフォルトでは無効になっています。必要に応じて有効にすることができます。セキュリティを考慮して、自動プロビジョニングを有効にする場合は、認証に許可される電子メール ドメインを制限することをお勧めします。

    -   **許可された電子メールドメイン**

        このフィールドを構成すると、この認証方法で指定された電子メール ドメインのみがカスタム URL を使用してTiDB Cloudにログインできるようになります。ドメイン名を入力する場合は、 `@`記号を除外し、カンマで区切る必要があります。たとえば、 `company1.com,company2.com` 。

        > **注記：**
        >
        > 電子メール ドメインを構成している場合は、 TiDB Cloudによってロックアウトされないように、設定を保存する前に、現在ログインに使用している電子メール ドメインを必ず追加してください。

4.  **「保存」**をクリックします。

### SAML 認証方法を構成する {#configure-the-saml-authentication-method}

SAML ID プロトコルを使用する ID プロバイダーがある場合は、 TiDB Cloudログインの SAML 認証方法を有効にすることができます。

TiDB Cloudでは、SAML 認証方法はデフォルトで無効になっています。 Cloud Organization Cloud を有効にした後、次のように SAML 認証方法を有効にして構成できます。

1.  TiDB Cloud Organization SSO のアイデンティティ プロバイダーから次の情報を取得します。

    -   サインオン URL
    -   署名証明書

2.  **[組織の設定]**ページで、 **[認証]**タブをクリックし、 **[認証方法]**領域で SAML の行を見つけて、[認証] をクリックします。 <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 20H21M3.00003 20H4.67457C5.16376 20 5.40835 20 5.63852 19.9447C5.84259 19.8957 6.03768 19.8149 6.21663 19.7053C6.41846 19.5816 6.59141 19.4086 6.93732 19.0627L19.5001 6.49998C20.3285 5.67156 20.3285 4.32841 19.5001 3.49998C18.6716 2.67156 17.3285 2.67156 16.5001 3.49998L3.93729 16.0627C3.59139 16.4086 3.41843 16.5816 3.29475 16.7834C3.18509 16.9624 3.10428 17.1574 3.05529 17.3615C3.00003 17.5917 3.00003 17.8363 3.00003 18.3255V20Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg> SAML メソッドの詳細を表示します。

3.  メソッドの詳細では、以下を設定できます。

    -   **名前**

        カスタム ログイン ページに表示される SAML 認証方法の名前を指定します。

    -   **サインオン URL**

        IdP から取得した URL を貼り付けます。

    -   **署名証明書**

        開始行`---begin certificate---`と終了行`---end certificate---`を含む、IdP からの署名証明書全体を貼り付けます。

    -   [**アカウントの自動プロビジョニング**](#decide-whether-to-enable-auto-provision)

        デフォルトでは無効になっています。必要に応じて有効にすることができます。セキュリティを考慮して、自動プロビジョニングを有効にする場合は、認証に許可される電子メール ドメインを制限することをお勧めします。

    -   **許可された電子メールドメイン**

        このフィールドを構成すると、この認証方法で指定された電子メール ドメインのみがカスタム URL を使用してTiDB Cloudにログインできるようになります。ドメイン名を入力する場合は、 `@`記号を除外し、カンマで区切る必要があります。たとえば、 `company1.com,company2.com` 。

        > **注記：**
        >
        > 電子メール ドメインを構成している場合は、 TiDB Cloudによってロックアウトされないように、設定を保存する前に、現在ログインに使用している電子メール ドメインを必ず追加してください。

4.  **「保存」**をクリックします。
