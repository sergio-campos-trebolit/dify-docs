# 敏感語審査

私たちがAIアプリケーションと対話する際、コンテンツの安全性、ユーザーエクスペリエンス、法律と規制など多方面で厳しい要件が求められます。このような場合、エンドユーザーにより良いインタラクティブ環境を提供するために「敏感語審査」機能が必要です。プロンプト編成ページで「機能を追加」をクリックし、下部のツールボックス「コンテンツ監査」を見つけます：

<figure><img src="../../../.gitbook/assets/moderation1.png" alt=""><figcaption><p>コンテンツ監査</p></figcaption></figure>

### 機能一：OpenAI モデレーション API の呼び出し

OpenAI やほとんどの大規模言語モデル (LLM) 会社が提供するモデルには、暴力、性、違法行為などの議論を含むコンテンツを出力しないようにするためのコンテンツ審査機能が備わっています。OpenAI はこのコンテンツ審査機能を公開しており、詳細は [platform.openai.com](https://platform.openai.com/docs/guides/moderation/overview) を参照してください。今では Dify でも直接 OpenAI モデレーション API を呼び出すことができます。入力内容や出力内容を監査するには、対応する「プリセット応答」を入力するだけです。

<figure><img src="../../../.gitbook/assets/moderation2.png" alt=""><figcaption><p>OpenAI モデレーション API</p></figcaption></figure>

### 機能二：カスタムキーワード

開発者は監査が必要な敏感語をカスタムキーワードとして設定できます。例えば「kill」をキーワードとして設定し、ユーザーが入力した際に監査動作を行い、プリセット応答内容として「The content is violating usage policies.」と設定します。予測される結果として、ユーザーが「kill」を含むテキストを入力すると、敏感語審査ツールが作動し、プリセット応答内容が返されます。

<figure><img src="../../../.gitbook/assets/moderation3.png" alt=""><figcaption><p>キーワード</p></figcaption></figure>

### 機能三：敏感語審査 モデレーション拡張

企業内部では異なる敏感語審査のメカニズムが存在することが多いです。企業が企業内ナレッジベースチャットボットなどのAIアプリケーションを開発する際、社員が入力したクエリ内容を敏感語審査する必要があります。このため、開発者は自社の敏感語審査メカニズムに基づいて API 拡張を作成することができます。詳細は [moderation.md](../../extension/api\_based\_extension/moderation.md "mention") を参照してください。これにより、Dify 上で呼び出し、高度なカスタマイズとプライバシー保護を実現することができます。

<figure><img src="../../../.gitbook/assets/moderation_settings.png" alt=""><figcaption><p>モデレーション設定</p></figcaption></figure>

例えば、私たちのローカルサービスで、アメリカ大統領の名前に関する質問を禁止する敏感語審査ルールをカスタマイズします。ユーザーが`query`変数に「Trump」と入力すると、対話時に "Your content violates our usage policy." という応答が返されます。テスト結果は以下の通りです：

<figure><img src="../../../.gitbook/assets/moderation_tet.png" alt=""><figcaption><p>モデレーションテスト</p></figcaption></figure>