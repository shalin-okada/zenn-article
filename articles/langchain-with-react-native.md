---
title: "ReactNativeでLangChainを動かす"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["reactnative", "langchain", "expo", "typescript"]
published: true
publication_name: "climt"
---
# はじめに

株式会社CLIMTのshalinです。
社内で「ChatGPTを使ったアプリを作りたいね」という話になり、ReactNativeでLangChain動いたら楽だなーと考えていたのですが、似たようなことをしている方が見当たらなかったので、記事にしようと思います。

# 前提

- node.jsのLTSがインストールされている

# 環境構築

## プロジェクト作成

```bash:Terminal
# 今回はexpoを利用してプロジェクトを作成する
npx create-expo-app <任意の名前> -t expo-template-blank-typescript
```

## 必要なライブラリのインストール

```bash:Terminal
cd <任意の名前>
npx expo install langchain react-native-url-polyfill text-encoding-polyfill
```

今回インストールするライブラリは以下の通りです。
- langchain
- react-native-url-polyfill
- text-encoding-polyfill

LangChainはGPT-4のようなLLMを利用してサービスの開発をしたいときに便利なライブラリです。
他二つについては特定機能をちゃんと動かすためのpolyfillです。
これらを入れてあげないとChatGPTへのリクエストをする際にエラーが出てしまいます。

# 実装

```tsx:App.tsx
import { StyleSheet, Text, TextInput, View } from 'react-native';
import { ChatOpenAI } from 'langchain/chat_models/openai';
import { HumanChatMessage } from 'langchain/schema';
import { useState, useCallback } from 'react';

import 'text-encoding-polyfill'
import 'react-native-url-polyfill/auto'

export default function App() {
  const [text, setText] = useState<string>('')
  const [output, setOutput] = useState<string>('')

  const handleSubmit = useCallback(async () => {
    setOutput('');
    const model = new ChatOpenAI({
      openAIApiKey: '<ChatGPTのAPIキー>',
      modelName: 'gpt-3.5-turbo',
    });
    try {
      const response = await model.call([new HumanChatMessage(text)])
      setOutput(response.text);
    } catch (e) {
      console.error(e)
    }
  }, []);
  
  return (
    <View style={styles.container}>
      <TextInput returnKeyType='send' style={styles.text} value={text} placeholder='promptを入力してください。' onChangeText={setText} onSubmitEditing={handleSubmit} />
      <Text style={styles.text}>{output}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'flex-start',
    paddingTop: 40
  },
  text: {
    borderWidth: 1,
    width: '80%',
    padding: 10,
    marginVertical: 20,
  },
});
```

## 解説

```tsx
import { StyleSheet, Text, TextInput, View } from 'react-native';
import { ChatOpenAI } from 'langchain/chat_models/openai';
import { HumanChatMessage } from 'langchain/schema';
import { useState, useCallback } from 'react';

import 'text-encoding-polyfill'
import 'react-native-url-polyfill/auto'
```

意外と重要なimportのブロックです。
`text-encoding-polyfill` と `react-native-url-polyfill/auto` をimportすることで、ChatGPTへのリクエスト時にエラーが出ないようになります。

```tsx
const [text, setText] = useState<string>('')
const [output, setOutput] = useState<string>('')

const handleSubmit = useCallback(async () => {
  setOutput('');
  const model = new ChatOpenAI({
    openAIApiKey: '<ChatGPTのAPIキー>',
    modelName: 'gpt-3.5-turbo',
  });
  try {
    const response = await model.call([new HumanChatMessage(text)])
    setOutput(response.text);
  } catch (e) {
    console.error(e)
  }
}, []);
```

実際にChatGPTにリクエストしレスポンスを表示するためのコードです。
`<ChatGPTのAPIキー>` には自身のAPIキーを入れてください。
langchainは内部でいい感じのpromptを作ってくれるようで、日本語との相性は悪いです。
そのあたりは他で記事にしている方などもいるので今回は英語だけ使うようにします。

```tsx
<View style={styles.container}>
  <TextInput returnKeyType='send' style={styles.text} value={text} placeholder='promptを入力してください。' onChangeText={setText} onSubmitEditing={handleSubmit} />
  <Text style={styles.text}>{output}</Text>
</View>

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'flex-start',
    paddingTop: 40
  },
  text: {
    borderWidth: 1,
    width: '80%',
    padding: 10,
    marginVertical: 20,
  },
});
```

ここはスタイル定義ですね。 `<TextInput />` の `onSubmitEditing` に `handleSubmit` を渡すことでキーボードの送信ボタンが押下された際にChatGPTへのリクエストが送信されます。

# 動かしてみる

ターミナルで `npm run start` して、適宜アプリを読み込んでください。
本当は動画を載せたかったのですがリサイズやスピードの調整など面倒で仕方ないので諦めます。
英語でpromptを入力すればある程度はうまくいくはずですが、LangChainならではのpromptハックも一定必要になると思います。

# 最後に

無事LangChainをReactNative上で動かすことができました。
実際はAPIキーをクライアントサイドのコードに入れて動かすのどうなんだろうとか、考えることいっぱいあるんですが、社内アプリだったりPoCなどにはちょうどいいかも知れません。

