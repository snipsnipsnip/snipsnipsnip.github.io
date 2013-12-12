---
layout: page
title: SourceTree for Windowsの翻訳に参加した
---

[Transifex](https://www.transifex.com/projects/p/sourcetree-for-windows/)にSourceTree for Windowsの翻訳プロジェクトが今月初め頃に立ち上がったのを見つけて、アナウンスもされない内に勝手に参加して作業した。Mac版からリソースを引き継ぐ用意もあっただろうに悪いことをしたと思う。

[![Screen Shot](https://bytebucket.org/snipsnipsnip/sourcetree.resources.ja/wiki/working.png)](https://bytebucket.org/snipsnipsnip/sourcetree.resources.ja/wiki/working.png)

今のところ作業で勝手に判断したもの。

* キャッシュとインデックスとステージ

Gitは「キャッシュ」と「インデックス」と「ステージングエリア」というほぼ同じ意味の用語がある。だいぶ前に内部用語としてのキャッシュからインデックスと名前が変わった。オプションでもcacheという単語の代わりにindexと指定して使えることが多い。データをインデックスに追加するとことをステージングというので、初心者に向けて解説する時にはインデックスのことをステージングエリアと説明するのが最近の流れというのが僕の印象だが、とりあえずステージとキャッシュを使わず「インデックスに追加」と「インデックスから削除」のように統一した。

* スタッシュとシェルブとシェルフ

スタッシュとシェルブ。前者はmercurialの機能で後者はgitの機能。スタッフのsstreetingさんという人がshelveを一時退避と訳していたのでそれに従ったが、一応（stash）と（shelve）と括弧をつけた。

シェルブとシェルフは英語版でも微妙で、shelveは棚上げするという動詞で、単純に類推すると棚上げした変更内容はshelfに収まるのだろう。実際GUIを見ると棚上げした内容の一覧は"shelves"という見出しがついている。シェルフはある程度カタカナ語として通用すると思うが、シェルブでも充分類推は出来るだろう。どう訳したもんか。

* メニュー項目のショートカットキー
 
windowsでは&hogeと書いたらhがショートカットとなるはずだが、なんかプロジェクトでは_hogeというふうになっている。で、sstreetingさんは_を無視して普通に単語として訳しているのでそれに従った。いいのだろうか。通常の流儀ならほげ (&h) のように訳すもんだけど。

* tracking/matching/current/nothing

Gitのpush.defaultの設定項目の用語。キーワードによってgit pushとだけ打ち込んだ時のコマンドの動作が変わるという仕組み。それぞれ定訳を見たことがないので、日本語に単に訳すと検索するとき苦労するだろうと思ってmatching（対応するブランチ）のように括弧で注釈をつける形にした。

* コマンドの日本語名。

Mercurial用語とかGit用語が困る。hg updateはgit resetと似たようなコマンドらしい。updateというと普通更新と訳してしまう。見分けづらい。リセットはリセットのままカタカナ語共通でいいのだが。hg graftはgit cherry-pickと同様。これはスタッフのsstreetingさんが接ぎ木と訳していたのでそれに従う。

* gitとmercurialでの用語の違い。

gitのfetch＝hgのpull、gitのpull＝hgのrebaseという妙な関係にある。hgのfetchはgitのfetchとmergeを続けて実行する感じ(fast-forwardしない)。hgのtransplantはcherry-pickにあたるのかな。gitのrebaseはhgのgraft。これは翻訳というより僕の知識の問題だが。

* working copyの訳語。

作業コピーと作業ツリーでsstreetingさんの訳がゆれている。作業コピーのほうが馴染みがあるのでそっちにした。

## 日本語化してSourcetree for Windowsが使えるよ

幸いリソースDLLをビルドするキットを作っている人([@koty氏](https://bitbucket.org/koty/sourcetree.resources.ja))がいたので、すぐに試すことができた。日本語化を試したい人は[こちら記事](https://bitbucket.org/snipsnipsnip/sourcetree.resources.ja)を参照。

