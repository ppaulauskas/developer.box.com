---
rank: 3
type: guide
hide_step_number: false
related_endpoints: []
related_guides:
  - authentication/select
  - authentication/oauth2
  - applications/custom-apps/app-approval
required_guides:
  - authentication/select
  - applications/custom-apps
related_resources: []
alias_paths:
  - /guides/applications/limited-access-apps
category_id: applications
subcategory_id: applications/custom-apps
is_index: false
id: applications/custom-apps/app-token-setup
total_steps: 4
sibling_id: applications/custom-apps
parent_id: applications/custom-apps
next_page_id: applications/custom-apps/app-approval
previous_page_id: applications/custom-apps/jwt-setup
source_url: >-
  https://github.com/box/developer.box.com/blob/main/content/guides/applications/custom-apps/app-token-setup.md
fullyTranslated: true
---
# アプリトークンを使用した設定

カスタムアプリは、認証にサーバー側の[アプリトークン][app-token]を使用するよう設定できます。

<CTA to="g://authentication/app-token">

アプリトークン認証のしくみを確認する

</CTA>

## 前提条件

To set up a Custom App using server-side authentication, you will need to ensure you have access the [Developer Console][devconsole] from your Box enterprise account. Alternatively, you may sign up for a [developer account][devaccount].

## App creation steps

### 1. 開発者コンソールにログインする

Log into Box and navigate to the [Developer Console][devconsole]. Select **Create New App**.

### 2. カスタムアプリを作成する

Select **Limited Access App** from the list of application types. A modal will appear to prompt the next step.

<ImageFrame border>

![アプリケーションの選択画面](../images/select-app-type.png)

</ImageFrame>

### 3. Select an app name

Finally, select a unique name for your application and click **Create App**.

<ImageFrame border width="600" center>

![アプリ名のフォーム](../images/limited-access-naming.png)

</ImageFrame>

## アプリの承認

Once a keypair is successfully added to your application your Box enterprise Admin needs to authorize the application within the Box Admin Console.

Navigate to the **General Settings** tab for your application within the [Developer console][devconsole] and scroll down to the **App Authorization** section.

<ImageFrame border width="400" center>

![キーの追加と管理](../images/app-authorization.png)

</ImageFrame>

Click **Submit and Review** to send an email to your Box enterprise Admin for approval. More information on this process is available in our [support article for app authorization][app-auth].

## 基本的な構成

アプリケーションを使用するには、事前にいくつかの基本的な追加構成が必要になる場合があります。

### プライマリおよびセカンダリアプリトークン

Authentication with Limited Access Apps is done through preconfigured [App Tokens][app-token]. To configure an app token, navigate to the **Configuration** tab for your application within the [Developer console][devconsole].

Scroll down to the **Primary Access Token** section and click the **Generate Key** button.

<ImageFrame border width="600" center>

![アプリトークンの作成](../images/app-generate-key.png)

</ImageFrame>

App tokens can be configured to automatically expire or be valid indefinitely. After creation, the key can be used to make [API calls][api-calls].

<Message warning>

# アプリの承認

App Tokens can not be generated until the application is successfully authorized within the Box Admin Console.

</Message>

### CORSドメイン

If your application makes API calls from front-end browser code in Javascript, the domain that these calls are made from will need to be added to an allow-list due to [Cross Origin Resource Sharing][cors], also known as CORS. If all requests will be made from server-side code, you may skip this section.

To add the full URI(s) to the allow-list, navigate to the **CORS Domain** section at the bottom of the **Configuration** tab in the [Developer console][devconsole].

<ImageFrame border>

![アプリ名のフォーム](../images/app-cors.png)

</ImageFrame>

[devconsole]: https://app.box.com/developers/console

[devaccount]: https://account.box.com/signup/n/developer

[devtoken]: g://authentication/access-tokens/developer-tokens

[scopes]: g://api-calls/permissions-and-errors/scopes

[cors]: https://en.wikipedia.org/wiki/Cross-origin_resource_sharing

[app-token]: g://authentication/app-token

[api-calls]: g://api-calls

[app-auth]: https://community.box.com/t5/Managing-Developer-Sandboxes/Authorizing-Apps-in-the-Box-App-Approval-Process/ta-p/77293
