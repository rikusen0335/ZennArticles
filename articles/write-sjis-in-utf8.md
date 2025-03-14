---
title: "UTF8形式のファイルをSJISでエンコードして、vbsで使えるようにする"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['sjis', 'utf-8', 'typescript', '']
published: false
publication_name: "omeroid"
---

こんにちは、クソ雑魚エンジニアです。

今回は、関わっているプロジェクトにて、手作業でファイルの生成作業を行なっていた箇所について、面倒臭すぎたので自動で生成できるようにするためのスクリプトを書きました。
エンジニアというのは面倒くさがりなので、自動化をしたがる生き物なのです。

またその際、Windows特有の面倒な部分にぶちあたったので、未来への知見を残すために書きます。

# 問題

- ファイルを使用するシステムでは、vbsがSJISでなければならない。
- VSCodeでvbsを開いた際に、UTF-8形式で読み込まれる。
- ただし、日本語箇所は文字化けしている。
- Shift-JISでエンコード付きで再度開く、を行うと、日本語が読み込まれる。

以上のファイルを自動生成したい際に、ファイルの読み込み、書き込みをどうすればいいかがわかっていない、という状態

# TL;DR

要約としては長いですが、だいたいこんな感じです。

```ts
import * as fs from "node:fs/promises";
import Encoding from 'encoding-japanese';

// 1 - ファイルの読み込み
//     読み込み自体はfsモジュールからそのままで大丈夫です
//     ブラウザ環境の場合はnode:fsではなくfsからimportしてください
const targetFile = await fs.readFile('path/to/file_original.vbs');

// 2 - コンテンツの置き換え
//     ファイルに{{fuga}}という置き換え文字列があった場合、それをfugaValueで置き換える
//     この際、JSではUNICODEの書き込みしかできないため、"デコード"を行います。
const contentsReplaced =
      Encoding.codeToString(Encoding.convert(targetFile, 'UNICODE'))
        .replace('{{fuga}}', fugaValue);

// 3 - エンコーディング
//     encoding-japaneseを使用し、エンコードを行います
const convertedContentsReplaced = Encoding.convert(contentsReplaced, {
  from: 'UNICODE',
  to: 'SJIS',
  type: 'arraybuffer',
});

// 4 - 書き込み
//     convertedContentsReplacedがArrayBufferになっているので、Buffer.fromからデータを取ります。
await fs.writeFile(
  "path/to/file_replaced.vbs",
  Buffer.from(convertedContentsReplaced)
);

```

# 詳しい解説

## 2の箇所
どこに書いてあったか忘れたのですが、JSでファイルに書き込む際は、Unicodeでの書き込みしかできないようです。
また、Unicodeに合わせることを明示することによって、fsとかの仕様が変わっても(ほぼstdなのでそんなことはないと思いますが)面倒な変更をしなくてすみます。

## 3の箇所
UNICODEで書き換えたテキストを、SJISに変換します。この際、arraybufferを指定していますが、確かないと動かなかったはずです。
このEncoding.convertという関数が、from、to、typeによって返す引数が違うのですが、SJISはUint8かなんかだった気がします。

## 4の箇所
コメントにも書いてますが、arraybufferを指定したのでconvertedContentsReplacedがArrayBufferで帰ってきているので、Buffer.fromで書き込みできる文字列にしています。

あとは、書き出したファイルをVSCodeで読んで、再度SJISでエンコードして開けば日本語が読めるファイルが作られているはずです。

おわり
