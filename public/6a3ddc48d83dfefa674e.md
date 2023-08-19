---
title: 【ReactNative】チャットバブルの実装方法紹介
tags:
  - chat
  - reactnative
private: false
updated_at: '2021-05-08T01:56:35+09:00'
id: 6a3ddc48d83dfefa674e
organization_url_name: null
slide: false
---
# はじめに

こちらの記事を参考にチャットバブルを実装するサンプルになります。

https://www.freecodecamp.org/news/design-imessage-like-chat-bubble-react-native/

# チャットバブルとは

LINEやiMessageで見かけるようなチャット画面の吹き出しのことです。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/a4ebf3be-e160-975c-9759-4f2a29bd24ab.png" width="300px" />

# HTML/CSS実装

まずはどのように吹き出しを実装しているのか確認します。（分かりやすいように吹き出しの色を黒と緑に変更しています。）

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="takiguchi-yu" data-slug-hash="NWpKKZj" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="ChatBubble">
  <span>See the Pen <a href="https://codepen.io/takiguchi-yu/pen/NWpKKZj">
  ChatBubble</a> by takiguchi-yu (<a href="https://codepen.io/takiguchi-yu">@takiguchi-yu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

# ReactNativeで実装

続いてこれを `ReactNative` で実装してみます。

## 構造化

`<View>`と`<Text>`で文章を配置します。

<details><summary>ソース</summary><div>

```jsx
import React from 'react';
import {
  StyleSheet, Text, View,
} from 'react-native';

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
  },
});

const ChatBubblePage = () => {
  return (
    <View style={styles.container}>
      <View>
        <Text>Hey there! What&apos;s up</Text>
      </View>
      <View>
        <Text>Checking out iOS7 you know..</Text>
      </View>
      <View>
        <Text>Check out this bubble!</Text>
      </View>
      <View>
        <Text>It&apos;s pretty cool!</Text>
      </View>
      <View>
        <Text>Yeah it&apos;s pure CSS &amp; HTML</Text>
      </View>
      <View>
        <Text>Wow that&apos;s impressive. But what&apos;s even more impressive is that this bubble is really high.</Text>
      </View>
    </View>
  );
};

export default ChatBubblePage;
```
</div></details>

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/a64e7eab-6ff7-0256-ed10-1d7d4625f816.png" width="300px" />

## スタイリング

配置したテキストにスタイルを適用します。

<details><summary>ソース</summary><div>

```jsx
import React from 'react';
import {
  StyleSheet, Text, View,
} from 'react-native';

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
  },
  fromMe: {
    backgroundColor: '#0078fe',
    padding: 10,
    marginLeft: '45%',
    marginTop: 5,
    marginRight: '5%',
    maxWidth: '50%',
    alignSelf: 'flex-end',
    borderRadius: 25,
  },
  fromMeText: {
    color: '#fff',
    fontSize: 16,
  },
  fromThem: {
    backgroundColor: '#dedede',
    padding: 10,
    marginLeft: '5%',
    marginTop: 5,
    marginRight: '45%',
    maxWidth: '50%',
    alignSelf: 'flex-start',
    borderRadius: 25,
  },
  fromThemText: {
    color: '#000',
    fontSize: 16,
  },
});

const ChatBubblePage = () => {
  return (
    <View style={styles.container}>
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>Hey there! What&apos;s up</Text>
      </View>
      <View style={styles.fromThem}>
        <Text style={styles.fromThemText}>Checking out iOS7 you know..</Text>
      </View>
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>Check out this bubble!</Text>
      </View>
      <View style={styles.fromThem}>
        <Text style={styles.fromThemText}>It&apos;s pretty cool!</Text>
      </View>
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>Yeah it&apos;s pure CSS &amp; HTML</Text>
      </View>
      <View style={styles.fromThem}>
        <Text style={styles.fromThemText}>Wow that&apos;s impressive. But what&apos;s even more impressive is that this bubble is really high.</Text>
      </View>
    </View>
  );
};

export default ChatBubblePage;
```
</div></details>

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/68bdf0cd-e5da-c807-49a0-5b6ca710bf03.png" width="300px" />

吹き出しっぽく矢印を追加します。width やら height やらがデバイスによって微妙に描画のされ方が異なってくるため、微調整が必要です。

<details><summary>ソース</summary><div>

```jsx
import React from 'react';
import {
  StyleSheet, Text, View,
} from 'react-native';

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
  },
  fromMe: {
    backgroundColor: '#0078fe',
    padding: 10,
    marginLeft: '45%',
    marginTop: 5,
    marginRight: '5%',
    maxWidth: '50%',
    alignSelf: 'flex-end',
    borderRadius: 25,
  },
  fromMeText: {
    color: '#fff',
    fontSize: 16,
  },
  fromThem: {
    backgroundColor: '#dedede',
    padding: 10,
    marginLeft: '5%',
    marginTop: 5,
    marginRight: '45%',
    maxWidth: '50%',
    alignSelf: 'flex-start',
    borderRadius: 25,
  },
  fromThemText: {
    color: '#000',
    fontSize: 16,
  },
  rightArrow: {
    position: 'absolute',
    backgroundColor: '#0078fe',
    width: 20,
    height: 20,
    right: -10,
    bottom: 0,
    borderBottomLeftRadius: 16,
  },
  rightArrowOverlap: {
    position: 'absolute',
    backgroundColor: '#fff',
    width: 26,
    height: 20,
    right: -26,
    bottom: 0,
    borderBottomLeftRadius: 10,
  },
  leftArrow: {
    position: 'absolute',
    backgroundColor: '#dedede',
    width: 20,
    height: 20,
    left: -10,
    bottom: 0,
    borderBottomRightRadius: 16,
  },
  leftArrowOverlap: {
    position: 'absolute',
    backgroundColor: '#fff',
    width: 26,
    height: 20,
    left: -26,
    bottom: 0,
    borderBottomRightRadius: 10,
  },
});

const ChatBubblePage = () => {
  return (
    <View style={styles.container}>
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>Hey there! What&apos;s up</Text>
        <View style={styles.rightArrow} />
        <View style={styles.rightArrowOverlap} />
      </View>
      <View style={styles.fromThem}>
        <Text style={styles.fromThemText}>Checking out iOS7 you know..</Text>
        <View style={styles.leftArrow} />
        <View style={styles.leftArrowOverlap} />
      </View>
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>Check out this bubble!</Text>
        <View style={styles.rightArrow} />
        <View style={styles.rightArrowOverlap} />
      </View>
      <View style={styles.fromThem}>
        <Text style={styles.fromThemText}>It&apos;s pretty cool!</Text>
        <View style={styles.leftArrow} />
        <View style={styles.leftArrowOverlap} />
      </View>
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>Yeah it&apos;s pure CSS &amp; HTML</Text>
        <View style={styles.rightArrow} />
        <View style={styles.rightArrowOverlap} />
      </View>
      <View style={styles.fromThem}>
        <Text style={styles.fromThemText}>Wow that&apos;s impressive. But what&apos;s even more impressive is that this bubble is really high.</Text>
        <View style={styles.leftArrow} />
        <View style={styles.leftArrowOverlap} />
      </View>
    </View>
  );
};

export default ChatBubblePage;
```
</div></details>

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/6161014f-f469-8375-0379-736b5606593d.png" width="300px" />

他のデバイス間でも互換性を持つように表示させるためにMapなどのコンポーネントや関数ではなく [FlatList](https://reactnative.dev/docs/flatlist) に埋め込むことがおすすめです。
以下は、最終的に FlatList を埋め込んだ iPhone(実機)での見た目になります。

<details><summary>ソース</summary><div>

```jsx
import { string } from 'prop-types';
import React from 'react';
import {
  FlatList, StyleSheet, Text, View,
} from 'react-native';

const DATA = [
  {
    id: "1",
    type: "me",
    message: "Hey there! What's up",
  },
  {
    id: "2",
    type: "them",
    message: "Checking out iOS7 you know..",
  },
  {
    id: "3",
    type: "me",
    message: "Check out this bubble!",
  },
  {
    id: "4",
    type: "them",
    message: "It's pretty cool!",
  },
  {
    id: "5",
    type: "me",
    message: "Yeah it's pure CSS & HTML",
  },
  {
    id: "6",
    type: "them",
    message: "Wow that's impressive. But what's even more impressive is that this bubble is really high.",
  },
];

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
  },
  fromMe: {
    backgroundColor: '#0078fe',
    padding: 10,
    marginLeft: '45%',
    marginTop: 5,
    marginRight: '5%',
    maxWidth: '50%',
    alignSelf: 'flex-end',
    borderRadius: 25,
  },
  fromMeText: {
    color: '#fff',
    fontSize: 16,
  },
  fromThem: {
    backgroundColor: '#dedede',
    padding: 10,
    marginLeft: '5%',
    marginTop: 5,
    marginRight: '45%',
    maxWidth: '50%',
    alignSelf: 'flex-start',
    borderRadius: 25,
  },
  fromThemText: {
    color: '#000',
    fontSize: 16,
  },
  rightArrow: {
    position: 'absolute',
    backgroundColor: '#0078fe',
    width: 20,
    height: 20,
    right: -10,
    bottom: 0,
    borderBottomLeftRadius: 16,
  },
  rightArrowOverlap: {
    position: 'absolute',
    backgroundColor: '#fff',
    width: 26,
    height: 20,
    right: -26,
    bottom: 0,
    borderBottomLeftRadius: 10,
  },
  leftArrow: {
    position: 'absolute',
    backgroundColor: '#dedede',
    width: 20,
    height: 20,
    left: -10,
    bottom: 0,
    borderBottomRightRadius: 16,
  },
  leftArrowOverlap: {
    position: 'absolute',
    backgroundColor: '#fff',
    width: 26,
    height: 20,
    left: -26,
    bottom: 0,
    borderBottomRightRadius: 10,
  },
});

const Item = ({ message, type }) => {
  if (type === "me") {
    return (
      <View style={styles.fromMe}>
        <Text style={styles.fromMeText}>{message}</Text>
        <View style={styles.rightArrow} />
        <View style={styles.rightArrowOverlap} />
      </View>
    );
  }

  return (
    <View style={styles.fromThem}>
      <Text style={styles.fromThemText}>{message}</Text>
      <View style={styles.leftArrow} />
      <View style={styles.leftArrowOverlap} />
    </View>
  );
};

Item.propTypes = {
  message: string.isRequired,
  type: string.isRequired,
};

const ChatBubblePage = () => {
  const renderItem = ({ item }) => {
    return <Item message={item.message} type={item.type} />;
  };

  return (
    <FlatList
      style={styles.container}
      data={DATA}
      renderItem={renderItem}
      keyExtractor={(item) => item.id}
    />
  );
};

export default ChatBubblePage;
```
</div></details>

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/d8723cb9-ba52-8f08-d846-f644ae61b82e.jpeg" width="300px" />
