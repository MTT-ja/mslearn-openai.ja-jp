---
lab:
    title: 'Azure OpenAI をアプリに統合する'
---

# Azure OpenAI をアプリに統合する

Azure OpenAI サービスを使用すると、開発者は自然な人間の言語を理解するのに優れたチャットボット、言語モデル、その他のアプリケーションを作成できます。Azure OpenAI は、事前にトレーニングされた AI モデルへのアクセスを提供するとともに、これらのモデルをアプリケーションの特定の要件に合わせてカスタマイズし、微調整するための API やツールのスイートを提供します。この演習では、Azure OpenAI でモデルをデプロイし、テキストを要約するために自分のアプリケーションで使用する方法を学びます。

この演習には約 **30** 分かかります。

## Azure OpenAI リソースのプロビジョニング

まだ持っていない場合は、Azure サブスクリプションに Azure OpenAI リソースをプロビジョニングします。

1. `https://portal.azure.com` で **Azure ポータル** にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します：
    - **サブスクリプション**: *Azure OpenAI サービスへのアクセスが承認されている Azure サブスクリプションを選択します*
    - **リソース グループ**: *リソース グループを選択または作成します*
    - **リージョン**: *利用可能なリージョンから **ランダム** に選択します*\*
    - **名前**: *あなたの選択による一意の名前*
    - **価格レベル**: Standard S0

    > \* Azure OpenAI リソースはリージョナルなクォータによって制限されています。ランダムにリージョンを選択することで、他のユーザーとサブスクリプションを共有するシナリオで単一のリージョンがクォータ制限に達するリスクを減らします。演習の後半でクォータ制限に達した場合、別のリージョンで別のリソースを作成する必要があるかもしれません。

3. デプロイが完了するのを待ちます。その後、Azure ポータルでデプロイされた Azure OpenAI リソースに移動します。

## モデルのデプロイ

Azure OpenAI は、**Azure OpenAI Studio** という Web ベースのポータルを提供しており、これを使用してモデルのデプロイ、管理、探索を行うことができます。Azure OpenAI の探索を始めるために、Azure OpenAI Studio を使用してモデルをデプロイします。

1. Azure OpenAI リソースの **概要** ページで、**Azure OpenAI Studio に移動** ボタンを使用して、新しいブラウザ タブで Azure OpenAI Studio を開きます。
2. Azure OpenAI Studio の **デプロイ** ページで、既存のモデル デプロイを表示します。まだ持っていない場合は、次の設定で **gpt-3.5-turbo-16k** モデルの新しいデプロイを作成します：
    - **モデル**: gpt-3.5-turbo-16k *(16k モデルが利用できない場合は、gpt-3.5-turbo を選択します)*
    - **モデル バージョン**: デフォルトに自動更新
    - **デプロイ名**: *あなたの選択による一意の名前。この名前は後でラボで使用します。*
    - **詳細オプション**
        - **コンテンツ フィルター**: デフォルト
        - **分あたりのトークン数のレート制限**: 5K\*
        - **動的クォータの有効化**: 有効

    > \* 分あたり 5,000 トークンのレート制限は、この演習を完了するのに十分であり、同じサブスクリプションを使用する他の人のための容量も残しておきます。

## Visual Studio Codeでアプリの開発準備をする

Azure OpenAIアプリはVisual Studio Codeを使用して開発します。アプリのコードファイルはGitHubリポジトリに提供されています。

> **ヒント**: すでに**mslearn-openai**リポジトリをクローンしている場合は、Visual Studio Codeで開いてください。そうでない場合は、以下の手順に従って開発環境にクローンしてください。

1. Visual Studio Codeを起動します。
2. パレットを開く（SHIFT+CTRL+P）し、**Git: Clone**コマンドを実行して`https://github.com/MicrosoftLearning/mslearn-openai`リポジトリをローカルフォルダにクローンします（どのフォルダでも構いません）。
3. リポジトリがクローンされたら、Visual Studio Codeでそのフォルダを開きます。
4. リポジトリ内のC#コードプロジェクトをサポートする追加ファイルがインストールされるのを待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今はしない**を選択してください。

## アプリケーションを設定する

C#とPythonの両方のアプリケーションが提供されており、要約をテストするために使用するサンプルテキストファイルもあります。どちらのアプリも同じ機能を備えています。まず、Azure OpenAIリソースを使用するためにアプリケーションのいくつかの重要な部分を完成させます。

1. Visual Studio Codeで、**エクスプローラー**ペインを開き、**Labfiles/02-nlp-azure-openai**フォルダに移動して、言語の好みに応じて**CSharp**または**Python**フォルダを展開します。各フォルダには、Azure OpenAI機能を統合するアプリの言語固有のファイルが含まれています。
2. コードファイルが含まれている**CSharp**または**Python**フォルダを右クリックし、統合ターミナルを開きます。次に、言語の好みに応じた適切なコマンドを実行して、Azure OpenAI SDKパッケージをインストールします：

    **C#**:

    ```csharp
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.9
    ```

    **Python**:

    ```python
    pip install openai==1.2.0
    ```

3. **エクスプローラー**ペインの**CSharp**または**Python**フォルダで、好みの言語の設定ファイルを開きます。

    - **C#**: appsettings.json
    - **Python**: .env
    
4. 次の設定値を更新します：
    - AzureポータルのAzure OpenAIリソースの**キーとエンドポイント**ページで利用可能な**エンドポイント**と**キー**
    - Azure OpenAI Studioの**デプロイ**ページで指定した**モデル名**
5. 設定ファイルを保存します。

## Azure OpenAI サービスを使用するためのコードを追加

これで、デプロイされたモデルを利用するために Azure OpenAI SDK を使用する準備が整いました。

1. **エクスプローラー** ペインで、**CSharp** または **Python** フォルダを開き、お好みの言語のコードファイルを開いて、コメント ***Add Azure OpenAI package*** を Azure OpenAI SDK ライブラリを追加するコードに置き換えます：

    **C#**: Program.cs

    ```csharp
   // Add Azure OpenAI package
   using Azure.AI.OpenAI;
    ```

    **Python**: test-openai-model.py

    ```python
   # Add Azure OpenAI package
   from openai import AzureOpenAI
    ```

2. お使いの言語のアプリケーションコードで、コメント ***Add code to build request...*** をリクエストを構築するための必要なコードに置き換えます。`prompt` や `temperature` など、モデルのさまざまなパラメータを指定します。

    **C#**: Program.cs

    ```csharp
   // Initialize the Azure OpenAI client
   OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));

   // Build completion options object
   ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
   {
        Messages =
        {
            new ChatMessage(ChatRole.System, "You are a helpful assistant."),
            new ChatMessage(ChatRole.User, "Summarize the following text in 20 words or less:\n" + text),
        },
        MaxTokens = 120,
        Temperature = 0.7f,
        DeploymentName = oaiModelName
   };

   // Send request to Azure OpenAI model
   ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);
   string completion = response.Choices[0].Message.Content;

   Console.WriteLine("Summary: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
   # Initialize the Azure OpenAI client
   client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )

   # Send request to Azure OpenAI model
   response = client.chat.completions.create(
       model=azure_oai_model,
       temperature=0.7,
       max_tokens=120,
       messages=[
           {"role": "system", "content": "You are a helpful assistant."},
           {"role": "user", "content": "Summarize the following text in 20 words or less:\n" + text}
       ]
    )
    
    print("Summary: " + response.choices[0].message.content + "\n")
    ```

3. コードファイルに変更を保存します。

## アプリケーションをテストする

アプリが設定されたので、実行してリクエストをモデルに送信し、応答を観察しましょう。

1. **エクスプローラー**ペインで、**Labfiles/02-nlp-azure-openai/text-files**フォルダを展開し、**sample-text.txt**ファイルを開きます。このテキストファイルには、モデルに要約してもらうテキストが含まれています。

2. インタラクティブなターミナルペインで、好みの言語のフォルダコンテキストを確認してください。次に、アプリケーションを実行するための以下のコマンドを入力します。

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

    > **Tip**: ターミナルツールバーの**パネルサイズを最大化する** (**^**) アイコンを使用して、コンソールテキストをより多く表示できます。

3. サンプルテキストファイルの要約を観察します。
4. 好みの言語のコードファイルで、リクエスト内の*temperature*パラメータの値を**1.0**に変更してファイルを保存します。
5. アプリケーションを再度実行し、出力を観察します。
6. アプリを数回再実行し、毎回の出力をメモします - 出力は変わる可能性があります。

温度を上げると、同じテキストを提供しても、ランダム性が増すため、要約が変わることがよくあります。出力がどのように変わるかを確認するために、何度か実行してみてください。同じ入力に対して異なる温度の値を試してみてください。

## クリーンアップ

Azure OpenAIリソースの使用が終わったら、**Azureポータル**の `https://portal.azure.com` でデプロイメントを削除するか、リソース全体を削除することを忘れないでください。
