# アセットフィールド

アセットフィールドでは、[アセット](assets.md)を他のエレメントに関連付けることができます。

## 設定

アセットフィールドの設定は、次の通りです。

- **アップロードを単一のフォルダに限定しますか？** – ファイルのアップロード / 関連付けを単一のフォルダに制限するかどうか。

  有効にすると、次の設定が表示されます。

  - **Upload Location** – The location that files dragged directly onto the field should be saved in.

  If disabled, the following settings will be visible:

  - **Sources** – Which asset volumes (or other asset index sources) the field should be able to relate assets from.
  - **Default Upload Location** – The default location that files dragged directly onto the field should be saved in.

- **許可されるファイルの種類を制限しますか？** 特定の種類のファイルだけをアップロード / 関連付けできるフィールドにするかどうか。
- **Limit** – The maximum number of assets that can be related with the field at once. (Default is no limit.)
- **View Mode** – How the field should appear for authors.
- **Selection Label** – The label that should be used on the field’s selection button.

### マルチサイト設定

マルチサイトがインストールされている場合、次の設定も有効になります。（「高度」のトグルボタンで表示されます）

- **特定のサイトから アセット を関連付けますか?** – 特定のサイトのアセットとの関連付けのみを許可するかどうか。

  有効にすると、サイトを選択するための新しい設定が表示されます。

  無効にすると、関連付けられたアセットは常に現在のサイトから取得されます。

- **サイトごとにリレーションを管理** – それぞれのサイトが関連付けられたアセットの独自のセットを取得するかどうか。

### 動的なサブフォルダパス

「ロケーションをアップロードする」と「既定のアップロードロケーション」設定に定義されるサブフォルダには、オプションで Twig タグ（例：`news/{{ slug }}`）を含めることができます。

ソースエレメント（アセットフィールドを持つエレメント）でサポートされているすべてのプロパティは、ここで使用できます。

::: tip
[行列フィールド](matrix-fields.md)の中にアセットフィールドを作成する場合、ソースエレメントは作成された行列フィールドのエレメント _ではなく_ 行列ブロックになります。

そのため、行列フィールドがエントリに紐づけられていて、動的なサブフォルダパスにエントリ ID を出力したい場合、`id` ではなく `owner.id` を使用します。
:::

## フィールド

アセットフィールドには、現在関連付けられているすべてのアセットのリストと、新しいアセットを追加するためのボタンがあります。

「アセットを追加」ボタンをクリックすると、新しいアセットのアップロードはもちろん、すでに追加されているアセットの検索や選択ができるモーダルウィンドウが表示されます。

### インラインのアセット編集

関連付けられたアセットをダブルクリックすると、アセットのタイトルやカスタムフィールドを編集したり、（画像の場合）イメージエディタを起動できる HUD を表示します。

::: tip
アセットで使用するカスタムフィールドは、「設定 > アセット > [ボリューム名] > フィールドレイアウト」から選択できます。
:::

## テンプレート記法

### アセットフィールドによるエレメントの照会

アセットフィールドを持つ[エレメントを照会](dev/element-queries/README.md)する場合、フィールドのハンドルにちなんで名付けられたクエリパラメータを使用して、アセットフィールドのデータに基づいた結果をフィルタできます。

利用可能な値には、次のものが含まれます。

| 値              | 取得するエレメント               |
| -------------- | ----------------------- |
| `':empty:'`    | 関連付けられたアセットを持たない。       |
| `':notempty:'` | 少なくとも1つの関連付けられたアセットを持つ。 |

```twig
{# Fetch entries with a related asset #}
{% set entries = craft.entries()
    .<FieldHandle>(':notempty:')
    .all() %}
```

### アセットフィールドデータの操作

テンプレート内でアセットフィールドのエレメントを取得する場合、アセットフィールドのハンドルを利用して関連付けられたアセットにアクセスできます。

```twig
{% set query = entry.<FieldHandle> %}
```

これは、所定のフィールドで関連付けられたすべてのアセットを出力するよう準備された[アセットクエリ](dev/element-queries/asset-queries.md)を提供します。

関連付けられたすべてのアセットをループするには、[all()](api3:craft\db\Query::all()) を呼び出して、結果をループ処理します。

```twig
{% set relatedAssets = entry.<FieldHandle>.all() %}
{% if relatedAssets|length %}
    <ul>
        {% for rel in relatedAssets %}
            <li><a href="{{ rel.url }}">{{ rel.filename }}</a></li>
        {% endfor %}
    </ul>
{% endif %}
```

関連付けられた最初のアセットだけが欲しい場合、代わりに [one()](api3:craft\db\Query::one()) を呼び出して、何かが返されていることを確認します。

```twig
{% set rel = entry.<FieldHandle>.one() %}
{% if rel %}
    <p><a href="{{ rel.url }}">{{ rel.filename }}</a></p>
{% endif %}
```

（取得する必要はなく）いずれかの関連付けられたアセットがあるかを確認したい場合、[exists()](api3:craft\db\Query::exists()) を呼び出すことができます。

```twig
{% if entry.<FieldHandle>.exists() %}
    <p>There are related assets!</p>
{% endif %}
```

アセットクエリで[パラメータ](dev/element-queries/asset-queries.md#parameters)をセットすることもできます。例えば、画像だけが返されることを保証するために、[kind](dev/element-queries/asset-queries.md#kind) パラメータをセットできます。

```twig
{% set relatedAssets = clone(entry.<FieldHandle>)
    .kind('image')
    .all() %}
```

### フロントエンドの投稿フォームからのファイルアップロード

フロントエンドの[投稿フォーム](dev/examples/entry-form.md)から、アセットフィールドへのファイルアップロードをユーザーに許可するには、2つの調整が必要です。

まず、`<form>` タグに `enctype="multipart/form-data"` 属性があることを確認して、ファイルをアップロードできるようにします。

```markup
<form method="post" accept-charset="UTF-8" enctype="multipart/form-data">
```

次に、ファイル入力欄をフォームに追加します。

```markup
<input type="file" name="fields[<FieldHandle>]">
```

::: tip
`<FieldHandle>` を実際のフィールドハンドルに置き換えます。例えば、フィールドハンドルが “heroImage” の場合、input 名は `fields[heroImage]` になります。
:::

複数ファイルをアップロードできるようにする場合、`multiple` 属性を追加し、input 名の末尾に `[]` を追加します。

```markup
<input type="file" name="fields[<FieldHanlde>][]" multiple>
```

## 関連項目

* [アセットクエリ](dev/element-queries/asset-queries.md)
* <api3:craft\elements\Asset>
* [リレーション](relations.md)