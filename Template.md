#### Recoilの公式ドキュメントをgoogle翻訳するとコードまで翻訳されてしまうのが面倒なのでQiitaにまとめてみます。

追々追加していきます。(多分)

[公式ドキュメント]()

## 目次

* **前書き**　[①](https://qiita.com/Daichi44/items/4236857dac4a3365f434)　[②](https://qiita.com/Daichi44/items/f6f8995a73387d104777)　[③](https://qiita.com/Daichi44/items/b46d9659c8fcfad2c227)　[④](https://qiita.com/Daichi44/items/3356aaeb7a387b520621)
* **基本チュートリアル**　[⑤](https://qiita.com/Daichi44/items/ee1ced6040d06e33e165)　[⑥](https://qiita.com/Daichi44/items/d6c2472043b473c9819d)　[⑦](https://qiita.com/Daichi44/items/af3d6363ac3ce3e7af5a)
* **ガイド**　[⑧](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)　[⑨](https://qiita.com/Daichi44/items/ae631b34ac50fb9bbd1d)　[⑩](https://qiita.com/Daichi44/items/45093185eb06fb4e1171)
* **APIリファレンス**
 * **Core**　[⑪](https://qiita.com/Daichi44/items/b440f33d3831b86c62b9)　[⑫](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)　[⑬](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)　[⑭](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)　[⑮](https://qiita.com/Daichi44/items/2ec591ec3e952f1784c3)　[⑯](https://qiita.com/Daichi44/items/f78a2ce0cfc01de354c5)　[⑰](https://qiita.com/Daichi44/items/ab0e98e0a48d0a59bdb4)　[⑱](https://qiita.com/Daichi44/items/a887cfa33260136b1d68)　[⑲](https://qiita.com/Daichi44/items/679030df40e81a608be5)　[⑳](https://qiita.com/Daichi44/items/f682baf85f0e2673abfc)　[㉑](https://qiita.com/Daichi44/items/1c38716dc0738da3bc02)　[㉒](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7)　[㉓](https://qiita.com/Daichi44/items/de7e39f80cd1fdce8b7c)　[㉔](https://qiita.com/Daichi44/items/eda197468b0204349d5f)　[㉕](https://qiita.com/Daichi44/items/0bbe5d6643462b55df11)　[㉖](https://qiita.com/Daichi44/items/0a276eb144f443a72efd)　[㉗](https://qiita.com/Daichi44/items/78292a5c30b1b1e4813f)
 * **実用(utils)**　[㉘](https://qiita.com/Daichi44/items/c6046e45ba7447a64aed)　[㉙](https://qiita.com/Daichi44/items/f552dfda4765596d83d7)　[㉚](https://qiita.com/Daichi44/items/7e568ccfb6d2e504de13)　[㉛](https://qiita.com/Daichi44/items/6973c6d140b697b2c97e)　[㉜](https://qiita.com/Daichi44/items/b929ff5376d0672dafa0)　[㉝](https://qiita.com/Daichi44/items/e9720f6ceb7cea57b2ce)　[㉞](https://qiita.com/Daichi44/items/c9bb722022a003d72aa9)　[㉟](https://qiita.com/Daichi44/items/ca11fcef04d764ef13a8)

※[全目次]()は一番下にあります

***

Coming soon...

***

### 参考サイト
・[公式ドキュメント]()
・[みらい翻訳](https://miraitranslate.com/trial)

***

>## 全目次
* **前書き**
 * [①動機](https://qiita.com/Daichi44/items/4236857dac4a3365f434)
 * [②コアコンセプト](https://qiita.com/Daichi44/items/f6f8995a73387d104777)
 * [③インストール](https://qiita.com/Daichi44/items/b46d9659c8fcfad2c227)
 * [④入門](https://qiita.com/Daichi44/items/3356aaeb7a387b520621)
* **基本チュートリアル**
 * [⑤概要](https://qiita.com/Daichi44/items/ee1ced6040d06e33e165)
 * [⑥Atoms](https://qiita.com/Daichi44/items/d6c2472043b473c9819d)
 * [⑦Selectors](https://qiita.com/Daichi44/items/af3d6363ac3ce3e7af5a)
* **ガイド**
 * [⑧非同期データクエリ](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)
 * [⑨非同期状態・同期状態](https://qiita.com/Daichi44/items/ae631b34ac50fb9bbd1d)
 * [⑩Stateの永続性](https://qiita.com/Daichi44/items/45093185eb06fb4e1171)
* **APIリファレンス**
 * **Core** <br>
   •　[⑪ \<RecoilRoot /\>](https://qiita.com/Daichi44/items/b440f33d3831b86c62b9) <br>
   •　**State** <br>
   　・　[⑫`atom()`](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2) <br>
   　・　[⑬`selector()`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9) <br>
   　・　[⑭Loadable](https://qiita.com/Daichi44/items/812aa5ebf149c849e108) <br>
   　・　[⑮`useRecoilState()`](https://qiita.com/Daichi44/items/2ec591ec3e952f1784c3) <br>
   　・　[⑯`useRecoilValue()`](https://qiita.com/Daichi44/items/f78a2ce0cfc01de354c5) <br>
   　・　[⑰`useSetRecoilState()`](https://qiita.com/Daichi44/items/ab0e98e0a48d0a59bdb4) <br>
   　・　[⑱`useResetRecoilState()`](https://qiita.com/Daichi44/items/a887cfa33260136b1d68) <br>
   　・　[⑲`useRecoilValueLoadable()`](https://qiita.com/Daichi44/items/679030df40e81a608be5) <br>
   　・　[⑳`useRecoilStateLoadable()`](https://qiita.com/Daichi44/items/f682baf85f0e2673abfc) <br>
   　・　[㉑`isRecoilValue()`](https://qiita.com/Daichi44/items/1c38716dc0738da3bc02) <br>
   •　**Snapshots** <br>
   　・　[㉒Snapshot](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7) <br>
   　・　[㉓`useRecoilTransactionObserver()`](https://qiita.com/Daichi44/items/de7e39f80cd1fdce8b7c) <br>
   　・　[㉔`useRecoilSnapshot()`](https://qiita.com/Daichi44/items/eda197468b0204349d5f) <br>
   　・　[㉕`useGotoRecoilSnapshot()`](https://qiita.com/Daichi44/items/0bbe5d6643462b55df11) <br>
   •　[㉖`useRecoilCallback()`](https://qiita.com/Daichi44/items/0a276eb144f443a72efd) <br>
   •　**雑録(Misc)** <br>
   　・　[㉗`useRecoilBridgeAcrossReactRoots()`](https://qiita.com/Daichi44/items/78292a5c30b1b1e4813f) <br>
 * **実用(utils)** <br>
   •　[㉘`atomFamily()`](https://qiita.com/Daichi44/items/c6046e45ba7447a64aed) <br>
   •　[㉙`selectorFamily()`](https://qiita.com/Daichi44/items/f552dfda4765596d83d7) <br>
   •　[㉚`constSelector()`](https://qiita.com/Daichi44/items/7e568ccfb6d2e504de13) <br>
   •　[㉛`errorSelector()`](https://qiita.com/Daichi44/items/6973c6d140b697b2c97e) <br>
   •　[㉜`waitForAll()`](https://qiita.com/Daichi44/items/b929ff5376d0672dafa0) <br>
   •　[㉝`waitForAny()`](https://qiita.com/Daichi44/items/e9720f6ceb7cea57b2ce) <br>
   •　[㉞`waitForNone()`](https://qiita.com/Daichi44/items/c9bb722022a003d72aa9) <br>
   •　[㉟`noWait()`](https://qiita.com/Daichi44/items/ca11fcef04d764ef13a8) <br>

<font color="orange">****</font>