---
title: "Zennの使い方"
emoji: "📚"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["メモ", "自分用"]
published: true
---
## 準備
1. 記事を新規作成する
slugという識別子を設定できる
12～50字で、半角英数字と-、_のみ使用可能
```bash
$ npx zenn new:article --slug ~~~
```
2. プレビューで表示を確認可能
```bash
$ npx zenn preview
```
VSCodeではZenn Editorという拡張機能を入れることで、Editorの右上のZennアイコンを押すだけでリアルタイムプレビューできる
![](/Images/how-to-use-zenn/HowToPreview.png)
画像の埋め込み確認はZennアイコンの右にあるZアイコンを押すことで可能

## メタ情報の設定
- titleを設定する
- emojiを設定する
- typeを設定する
- topicsを設定する
- publishedを設定する

## 記事を書く
マークダウン記法の参考
https://zenn.dev/zenn/articles/markdown-guide

記事の構成は以下をテンプレートとして使うと良い
## 1. はじめに

## 2. 対象読者

## 3. 結論

## 4. 本文

## 5. まとめ

こちらを参考に
https://zenn.dev/make_it/articles/7e1e108de83e84#2

## 公開
1. gitにPushする
```bash
git add .
git commit -m "記事を追加: 記事のタイトル"
git push origin main
```