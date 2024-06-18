# 敏感内容審査

Dify にはシステムに内蔵された内容審査タイプの他に、ユーザーが独自にカスタマイズした内容審査ルールを拡張する機能もあります。この方法は、プライベートデプロイ環境での開発者向けにカスタム開発が可能です。例えば、企業内部のカスタマーサポートにおいて、ユーザーが問い合わせを行う際やカスタマーサポートが返信する際に、暴力、性、違法行為などの関連用語を入力してはならないといった規定に加え、企業が独自に定めた禁則語や内部の審査ロジックに違反する内容も含めないようにすることができます。このような場合、開発者はプライベートデプロイ環境の Dify コード層で独自の内容審査ルールを拡張できます。

## クイックスタート

ここでは、`クラウドサービス` 内容審査拡張を例にとって、以下の手順を説明します：

1. ディレクトリの初期化
2. フロントエンドコンポーネント定義ファイルの追加
3. 実装クラスの追加
4. フロントエンド画面のプレビュー
5. 拡張機能のデバッグ

### 1. ディレクトリの初期化

新しいカスタムタイプ `クラウドサービス` を追加するには、`api/core/moderation` ディレクトリ内に関連するディレクトリとファイルを新規作成します。

```Plain
.
└── api
    └── core
        └── moderation
            └── cloud_service
                ├── __init__.py
                ├── cloud_service.py
                └── schema.json
```

### 2. フロントエンドコンポーネント規格の追加

* `schema.json` ここでは、フロントエンドコンポーネントの規格を定義しています。詳細は [こちら](./ "mention") を参照してください。

```json
{
    "label": {
        "en-US": "Cloud Service",
        "zh-Hans": "クラウドサービス"
    },
    "form_schema": [
        {
            "type": "select",
            "label": {
                "en-US": "Cloud Provider",
                "zh-Hans": "クラウドプロバイダー"
            },
            "variable": "cloud_provider",
            "required": true,
            "options": [
                {
                    "label": {
                        "en-US": "AWS",
                        "zh-Hans": "AWS"
                    },
                    "value": "AWS"
                },
                {
                    "label": {
                        "en-US": "Google Cloud",
                        "zh-Hans": "GoogleCloud"
                    },
                    "value": "GoogleCloud"
                },
                {
                    "label": {
                        "en-US": "Azure Cloud",
                        "zh-Hans": "Azure"
                    },
                    "value": "Azure"
                }
            ],
            "default": "GoogleCloud",
            "placeholder": ""
        },
        {
            "type": "text-input",
            "label": {
                "en-US": "API Endpoint",
                "zh-Hans": "APIエンドポイント"
            },
            "variable": "api_endpoint",
            "required": true,
            "max_length": 100,
            "default": "",
            "placeholder": "https://api.example.com"
        },
        {
            "type": "paragraph",
            "label": {
                "en-US": "API Key",
                "zh-Hans": "APIキー"
            },
            "variable": "api_keys",
            "required": true,
            "default": "",
            "placeholder": "ここにAPIキーを貼り付けてください"
        }
    ]
}
```

### 3. 実装クラスの追加

`cloud_service.py` のコードテンプレートです。ここで具体的なビジネスロジックを実装できます。

{% hint style="warning" %}
注意：クラス変数 name はカスタムタイプの名称であり、ディレクトリ名およびファイル名と一致し、かつ一意である必要があります。
{% endhint %}

```python
from core.moderation.base import Moderation, ModerationAction, ModerationInputsResult, ModerationOutputsResult

class CloudServiceModeration(Moderation):
    """
    The name of custom type must be unique, keep the same with directory and file name.
    """
    name: str = "cloud_service"

    @classmethod
    def validate_config(cls, tenant_id: str, config: dict) -> None:
        """
        schema.json validation. It will be called when user save the config.

        Example:
            .. code-block:: python
                config = {
                    "cloud_provider": "GoogleCloud",
                    "api_endpoint": "https://api.example.com",
                    "api_keys": "123456",
                    "inputs_config": {
                        "enabled": True,
                        "preset_response": "Your content violates our usage policy. Please revise and try again."
                    },
                    "outputs_config": {
                        "enabled": True,
                        "preset_response": "Your content violates our usage policy. Please revise and try again."
                    }
                }

        :param tenant_id: the id of workspace
        :param config: the variables of form config
        :return:
        """

        cls._validate_inputs_and_outputs_config(config, True)

        if not config.get("cloud_provider"):
            raise ValueError("cloud_provider is required")

        if not config.get("api_endpoint"):
            raise ValueError("api_endpoint is required")

        if not config.get("api_keys"):
            raise ValueError("api_keys is required")

    def moderation_for_inputs(self, inputs: dict, query: str = "") -> ModerationInputsResult:
        """
        Moderation for inputs.

        :param inputs: user inputs
        :param query: the query of chat app, there is empty if is completion app
        :return: the moderation result
        """
        flagged = False
        preset_response = ""

        if self.config['inputs_config']['enabled']:
            preset_response = self.config['inputs_config']['preset_response']

            if query:
                inputs['query__'] = query
            flagged = self._is_violated(inputs)

        # return ModerationInputsResult(flagged=flagged, action=ModerationAction.OVERRIDED, inputs=inputs, query=query)
        return ModerationInputsResult(flagged=flagged, action=ModerationAction.DIRECT_OUTPUT, preset_response=preset_response)

    def moderation_for_outputs(self, text: str) -> ModerationOutputsResult:
        """
        Moderation for outputs.

        :param text: the text of LLM response
        :return: the moderation result
        """
        flagged = False
        preset_response = ""

        if self.config['outputs_config']['enabled']:
            preset_response = self.config['outputs_config']['preset_response']

            flagged = self._is_violated({'text': text})

        # return ModerationOutputsResult(flagged=flagged, action=ModerationAction.OVERRIDED, text=text)
        return ModerationOutputsResult(flagged=flagged, action=ModerationAction.DIRECT_OUTPUT, preset_response=preset_response)

    def _is_violated(self, inputs: dict):
        """
        The main logic of moderation.

        :param inputs:
        :return: the moderation result
        """
        return False
```

### 4. フロントエンド画面のプレビュー

上記の手順を実行し、サービスを起動すると、新しく追加されたカスタムタイプを確認できます。

<figure><img src="https://langgenius.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmRiMDI5YTU3NGY5M2JmNmY0N2Y0NmEwMGIwNGI0N2VfOGRrVE5GV25BT3laRnVxYlhLMUFYRWZWbm9NVkp3YXJfVG9rZW46Q1JPS2I3bFU0b0NQY2l4YVlhUGNVSTNNblJkXzE2OTk0NTExMTc6MTY5OTQ1NDcxN19WNA" alt=""><figcaption><p>クラウドサービス</p></figcaption></figure>

### 5. 拡張機能のデバッグ

ここまでで、Dify のアプリケーションオーケストレーション画面でカスタム `クラウドサービス` 内容審査拡張タイプを選択してデバッグすることができます。 ```python
param {テキスト}: {大規模言語モデル}応答の{テキスト}
        :return: {モデレーション}結果
        """
        {フラグ付き} = False
        {プリセット応答} = ""
        
        # ここに独自の{ロジック}を実装してください

        # return {モデレーション出力結果}({フラグ付き}={フラグ付き}, {アクション}={モデレーションアクション}.{上書き}, {テキスト}={テキスト})
        return {モデレーション出力結果}({フラグ付き}={フラグ付き}, {アクション}={モデレーションアクション}.{直接出力}, {プリセット応答}={プリセット応答})
```

## 実装クラスの詳細説明

### def {設定検証}

`{スキーマ}.json` フォーム検証方法、ユーザーが「公開」をクリックして設定を保存するときに呼び出される

* `{設定}` フォームパラメータ
  * `{変数名}` フォームカスタム変数
  * `inputs_config` 入力検証プリセット応答
    * `enabled` 有効化
    * `preset_response` 入力プリセット応答
  * `outputs_config` 出力検証プリセット応答
    * `enabled` 有効化
    * `preset_response` 出力プリセット応答

### def {入力検証}

入力検証関数

* `inputs` ：エンドユーザーによって渡された変数値
* `query` ：エンドユーザーが現在入力している内容、対話型アプリケーションの固定パラメータ。
* `モデレーション入力結果`
  * `flagged` 検証ルールに違反しているかどうか
  * `action` 実行動作
    * `direct_output` プリセット応答を直接出力
    * `overrided` 渡された変数値を上書き
  * `preset_response` プリセット応答（action=direct_outputの場合のみ返される）
  * `inputs` エンドユーザーによって渡された変数値、キーは変数名、値は変数値（action=overridedの場合のみ返される）
  * `query` 上書きされたエンドユーザーの現在の入力内容、対話型アプリケーションの固定パラメータ（action=overridedの場合のみ返される）

### def {出力検証}

出力検証関数

* `{テキスト}` ：{モデル出力}内容
* `{モデレーション出力}` ：出力検証関数
  * `{テキスト}` ：{大規模言語モデル}回答内容。{大規模言語モデル}の出力が{ストリーミング出力}の場合、これは100文字ごとの分割内容である。
  * `{モデレーション出力結果}`
    * `flagged` 検証ルールに違反しているかどうか
    * `action` 実行動作
      * `direct_output` プリセット応答を直接出力
      * `overrided` 渡された変数値を上書き
    * `preset_response` プリセット応答（action=direct_outputの場合のみ返される）
    * `text` 上書きされた{大規模言語モデル}回答内容（action=overridedの場合のみ返される）
```