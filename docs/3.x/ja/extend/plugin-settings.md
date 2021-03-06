# プラグイン設定

## 設定モデル

グローバル設定を持つプラグインは、設定値の保存と検証のための責任を持つ「設定モデル」を定義する必要があります。

This “settings model” is responsible for storing and validating the setting values.

設定モデルは、他のいかなる [model](https://www.yiiframework.com/doc/guide/2.0/en/structure-models) とも全く同じです。 作成するには、プラグインのソースディレクトリ内に `models/` ディレクトリを作成し、その中に `Settings.php` ファイルを作成します。

```php
<?php

namespace ns\prefix\models;

use craft\base\Model;

class Settings extends Model
{
    public $foo = 'defaultFooValue';
    public $bar = 'defaultBarValue';

    public function rules()
    {
        return [
            [['foo', 'bar'], 'required'],
            // ...
        ];
    }
}
```

設定モデルと `createSettingsModel()` メソッドを適切に配置すると、メインプラグインクラスの `getSettings()` メソッド経由で、現在のプロジェクトにセットされたすべてのカスタム値が入った設定モデルにアクセスできるようになります。

```php
<?php

namespace mynamespace;

class Plugin extends \craft\base\Plugin
{
    public $hasCpSettings = true;

    protected function createSettingsModel()
    {
        return new \mynamespace\models\Settings();
    }

    protected function settingsHtml()
    {
        return \Craft::$app->getView()->renderTemplate(
            'my-plugin-handle/settings',
            [ 'settings' => $this->getSettings() ]
        );
    }

    // ...
}
```

::: warning
The config file cannot contain any keys that are not defined in the plugin’s settings model.
:::

## 設定へのアクセス

それは Craft の `settings/plugins/_settings` レイアウトに制限されず、テンプレート独自のテンプレートをレンダリングするよう選択できます。

```php
protected function createSettingsModel()
{
    return new \ns\prefix\models\Settings();
}
```

次に、メインプラグインクラスに設定モデルの新しいインスタンスを返す `createSettingsModel()` メソッドを追加します。

```php
// From your main plugin class:
$foo = $this->getSettings()->foo;

// From elsewhere:
$foo = \ns\prefix\Plugin::getInstance()->getSettings()->foo;
```

## 設定値の上書き

設定値は、プラグインハンドルの名前にちなんだプロジェクトの`config/` フォルダに含まれる PHP ファイルから、プロジェクトごとに上書きできます。 例えば、プラグインハンドルが `foo-bar` の場合、その設定は `config/foo-bar.php` ファイルから上書きできます。

ファイルは、上書きしたい値を持つ配列を返すだけです。

```php
<?php

return [
    'foo' => 'overriddenFooValue',
    'bar' => 'overriddenBarValue',
];
```

::: warning
設定ファイルは、プラグインの設定モデルで定義されていないキーを含めることはできません。
:::

```php
<?php

return [
    '*' => [
        'foo' => 'defaultValue',
    ],
    '.test' => [
        'foo' => 'devValue',
    ],
];
```

protected function settingsHtml()
    { return \Craft::$app-&gt;getView()-&gt;renderTemplate('plugin-handle/settings', [ 'settings' =&gt; $this-&gt;getSettings() ]); } // ...
:::

## 設定ページ

プラグインは、管理者がプラグインに依存する設定値を管理しやすくするために、コントロールパネルに設定ページを提供することもできます。

次に、メインプラグインクラス内で `$hasCpSettings` プロパティを `true` にセットし、新しいレンダリングテンプレートを返す `settingsHtml()` メソッドを定義します。

```twig
{% import "_includes/forms" as forms %}

{{ forms.textField({
    first: true,
    label: "Foo",
    name: 'foo',
    value: settings.foo
}) }}

{{ forms.textField({
    label: "Bar",
    name: 'bar',
    value: settings.bar
}) }}
```

Then, within your main plugin class, set the `$hasCpSettings` property to `true`, and define a `settingsHtml()` method that returns your new rendered template:

```php
<?php
namespace ns\prefix;

class Plugin extends \craft\base\Plugin
{
    protected function createSettingsModel()
    {
        return new \ns\prefix\models\Settings();
    }

    // ...
}
```

これで、プラグインは設定ページの独自アイコンと「設定 > プラグイン」ページの自身の行へ `/admin/settings/plugin-handle` にリンクする歯車アイコンを表示するようになります。

### 高度な設定ページ

コントロールパネル URL の `/admin/settings/plugin-handle` がリクエストされると、プラグインは最終的にレスポンスを担当します。 すなわち、プラグインの `getSettingsResponse()` メソッドです。 <craft3:craft\base\Plugin> のデフォルトの `getSettingsResponse()` 実装は、プラグインの `settingsHtml()` メソッドを呼び出します。 そして、Craft の（プラグイン設定ページのレイアウトテンプレートである）`settings/plugins/_settings` テンプレートをレンダリングし、`settingsHtml()` によって返された HTML を渡すようアクティブなコントローラーに伝えます。

プラグインが設定ページをもっとコントルールスル必要がある場合、`getSettingsResponse()` メソッドを上書きして、リクエストでしたいことを実行できます。

リクエストを全く別の URL にリダイレクトすることもできます。

```php
public function getSettingsResponse()
{
    $url = \craft\helpers\UrlHelper::cpUrl('plugin-handle/settings');

    return \Craft::$app->controller->redirect($url);
}
```

It can redirect the request to a completely different URL, too:

```php
public function getSettingsResponse()
{
    return \Craft::$app->controller->renderTemplate('plugin-handle/settings/template');
}
```

そこで返るものは、そこで正に起きている何かであるため、コントローラーアクションが返す何かとして筋が通っている必要があることに注意してください。 <craft3:craft\controllers\PluginsController::actionEditPluginSettings()> メソッドは、`getSettingsResponse()` の戻り値を直接返します。
