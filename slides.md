---
theme: seriph
class: "text-center"
highlighter: shiki
background: ''
colorSchema: 'dark'
lineNumbers: false
info: |
  アプリケーションエンジニアから見たPostgreSQL15 の新機能
   2022-10-01 OSC 2022 オンライン広島
drawings:
  enabled: true
  presenterOnly: true
  persist: false
css: unocss
layout: center

---

# アプリケーションエンジニアから見た<br>PostgreSQL15 の新機能

2022-10-01 OSC 2022 オンライン広島 <br>
日本 PostgreSQL ユーザー会 中国支部長 高橋 一騎

---

# 本セッションについて

- スライドは公開しています。
- 質問は **#osc22hi** に投稿してもらえれば、後ほど拾う事もできるかと思いますので是非活用してください。
- ある程度PostgreSQLを初めて見る人も対象にしてお話する予定です。
  - アプリケーションを開発されてる向けなので、SQLの基礎知識的な部分は省略しました。
- このセッションを通じて、「PostgreSQLを使ってみよう」 「PostgreSQLのバージョンアップをしてみよう」という気持ちになってもらえればと思います。

---

# おしながき

- 自己紹介
- PostgreSQLとは
- PostgreSQL15の新機能

---

# 自己紹介

<div class="grid grid-cols-2 gap-4">
  <div>
    <ul>
      <li>高橋 一騎 (<a href="https://twitter.com/ikkitang">@ikkitang</a>)</li>
      <li>スターフェスティバル株式会社<br>TechPM 兼 アプリケーションエンジニア</li>
      <li>岡山在住</li>
      <li>日本PostgreSQLユーザー会 中国地方支部長</li>
    </ul>
  </div>
  <div>
    <img src="/images/icon.png" alt="icon" class="rounded-1/2"/>
  </div>
</div>

---

# 自己紹介

- 日本PostgreSQLユーザー会中国地方支部長ではありますが、普段は普通にWebアプリケーションのコードを書いてていわゆるDBA的な仕事はしていません。
  - 割とPostgreSQLやMySQLはAWSのAmazon RDSでシュッと導入して使っています。
- 本セッションでは、今回のPostgreSQL15に入る機能をアプリケーションエンジニアとしての立場からピックアップをした上でご紹介をさせていただければと思います。

---
layout: center
class: text-center
---

# 2. PostgreSQLとは

---

# 2. PostgreSQLとは

<div class="grid grid-cols-2 gap-4">
  <div>
    <ul>
      <li>代表的なオープンソースのRDBMSの一つ</li>
      <li>もともと、大学の研究用に開発された研究用のRDBMSの <code>ingress</code> が元となっている。</li>
      <li>PostgreSQL開発コミュニティによって開発が行われていて、約1年弱の開発期間を経た後、毎年9~10月頃にメジャーバージョンがリリースされている。</li>
      <li>今年は<code>PostgreSQL15</code>のリリースに向けて開発が行われており、2022-10-13にGAの予定が発表された</li>
    </ul>
  </div>
  <div>
    <Tweet id="1571857779643777024" />
  </div>
</div>

---

# PostgreSQLの特徴

<div class="grid grid-cols-2 gap-10">
  <div>
    <h2>複数のIndexアルゴリズムを<br>サポート</h2>
    <ul>
      <li>B-Tree Index</li>
      <li>Hash Index</li>
      <li>GiST Index, SP-GiST index, GIN Index</li>
      <li>BRIN Index</li>
    </ul>
  </div>
  
  <div>
    <h2>豊富なデータ型をサポート</h2>
    <ul>
      <li>数値型, 文字型, boolean型, 列挙型</li>
      <li>時刻型（タイムスタンプあり）</li>
      <li>UUID型</li>
      <li>JSON型</li>
      <li>配列型</li>
      <li>範囲型</li>
      <li>
        幾何データ型
        <ul>
          <li>座標点</li>
          <li>直線</li>
          <li>円</li>
        </ul>
      </li>
      <li>IPアドレス型</li>
    </ul>
  </div>
</div>

<!-- 
PostgreSQLの特徴を簡単にご紹介します。
特徴というより、強みと言ってもいいかもしれないです。

まずは、複数種類のIndexをサポートしています。
B-tree index これらはMySQLとかの他のRDB製品でもサポートされていますが、
主キーとかだとこのIndexが採用されてたりというイメージですかね。
ソート順がIndexに入ってきたりしているので、等価比較に加えて
範囲で絞ったり、ORDER BY句などにも使えたりします、

次は Hash index です。
主に等価である事を比較する場合にB-Tree Indexよりも高速だったりインデックスサイズが小さいなどといったメリットがあります。

Gist index や GIN index とかは 全文検索とかと相性の良いIndex
最後に BRIN ブロックレンジインデックス .
例えば、ログのタイムスタンプみたいに テーブルのレコードの作成順序がそのまま時系列として意味を持つようなテーブルでかつ大規模なテーブルを
B-Treeよりもより効率的に処理できるインデックスです.

-->

---

# PostgreSQLのバージョニング

- 10より前は `x.y.z` の `x.y`の部分がメジャーバージョン
  - 8.4 => 9.1 => 9.2 => 9.3 （年単位でバージョンアップ)
- 10移行は `x.y` の `x` の部分がメジャーバージョン
  - 10.0 => 11.0 => 12.0 (年単位でバージョンアップ)
  - EOLはリリースから5年間と定められているので今ではサポートされているのはすべて`x.y`の形式

<!--
10からx.y形式になった経緯を調べていたんですが、x.y形式にする事で毎年xの部分を上げる事が明確になるというメリットのためだそうです。
x.y.zの形式の頃はxの部分を上げる明確な基準がなくて各大きな機能開発のタイミングで上げてたという敬意があって、
毎年「今回はxを上げるか？」という議論がされてた、という話がありました。
-->

---
layout: center
---

# 今日はそんなPostgreSQL15の話

---
layout: center
---

# 3. PostgreSQL15の新機能

---

# 3. PostgreSQL15の新機能（抜粋）

- Merge文のサポート
- ロジカルレプリケーションの機能拡張
- パラレルクエリの強化
- バージョン非互換対応（新機能ではないけど）
  - PublicスキーマのCreate権限がデフォルトからなくなる

---
layout: center
---

# Merge文のサポート

---

# Merge文のサポート

<div class="grid grid-cols-2 gap-4">
  <div>
    <ul>
      <li>INSERT・UPDATE・DELETEを一括で処理できる。</li>
      <li>SQL:2003で標準SQLとして定義されていて、OracleやSQL Serverではすでにサポートされている。</li>
      <li>
        条件に合致したとき（<code>WHEN MATCHED 句</code>）
        <ul>
          <li>UPDATE</li>
          <li>DELETE</li>
          <li>DO NOTHING: 何も処理しない</li>
        </ul>
      </li>
      <li>
        条件に合致しなかったとき<br>（<code>WHEN NOT MATCHED 句</code>）
        <ul>
          <li>INSERT</li>
          <li>DO NOTHING: 何も処理しない</li>
        </ul>
      </li>
    </ul>
  </div>
  <div>

例

```sql
MERGE INTO members

 USING (VALUES (1, 'test@example.com', 'test name')) 
   AS i(member_id, email, user_name)
   ON members.id = i.member_id

 WHEN MATCHED THEN
      UPDATE SET user_name = i.user_name
 WHEN NOT MATCHED THEN
      INSERT (member_id, email, user_name) 
      VALUES (i.member_id, i.email, i.user_name);
```

`WHEN MATCHED` 句では条件を複数記述できる

```sql
WHEN MATCHED AND hoge.flag = 1 THEN 
```

  </div>
</div>

---

# UPSERT文

- INSERT・UPDATEを組み合わせた操作を行う事からUPSERTと呼ばれる。
- PostgreSQLにはすでに`UPSERT`相当の機能がある
- 以下2つは成功時の結果が同じ

<br>

<div class="grid grid-cols-2 gap-4">
  <div>

MERGE文

```sql
MERGE INTO members
 USING (VALUES (1, 'test@example.com', 'test name')) 
   AS i(member_id, email, user_name)
   ON members.id = i.member_id
 WHEN MATCHED THEN
      UPDATE SET user_name = i.user_name
 WHEN NOT MATCHED THEN
      INSERT (member_id, email, user_name) 
      VALUES (i.member_id, i.email, i.user_name);
```

  </div>
  <div>

INSERT ON CONFLICT句

```sql
INSERT INTO members (member_id, email, user_name)
 VALUES (1, 'test@example.com', 'test name')
 ON CONFLICT(member_id)
 DO UPDATE SET user_name = 'test name';
```

  </div>
</div>

---

# UPSERT文の違い

<style>
table td, table th {
  border: 1px solid #FFF;
}
</style>

| 項目       | MERGE                                                                                        | INSERT ON CONFLICT                                           |
|----------|----------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| 処理速度     | 簡易比較ではこちらの方が10~100%程高速                                                                       |                                                              |
| 実装方法     | targetとsourceをJOINし、MATCHED句に応じて処理 <br> JOINの結果で予め処理を決めてから実行するのでJOIN時と実データに差があればエラー出ることもある   | INSERTして、制約に違反したらUPDATEする <br> 処理をした結果に応じてUPDATEできるので並行性能が高い |
| 対応範囲     | DELETEに対応, 条件の比較に等号・不等号を扱える                                                                  | DELETE未対応, 条件の比較は等号により比較                                     |
| 実行の注意事項  |                                                                                              | ON CONFLICT に指定したカラムに必ずユニーク制約が必要                             |

<br>

- 処理速度は https://qiita.com/fujii_masao/items/462bac9f6a107d6134c4 を参考にしました

<!-- 
つまり、両者は完全に置き換え可能という間柄ではない
-->

---
layout: center
---

# ロジカル（論理）レプリケーションの機能拡張

---

# PostgreSQLのレプリケーションについておさらい

## ストリーミングレプリケーション

<br>

<div class="grid grid-cols-2 gap-4">
  <div>
    <ul>
      <li>データベースクラスタ全体を<br>WAL転送によってレプリケーションする</li>
      <li>レプリケーション元をプライマリ</li>
      <li>レプリケーション先をスタンバイ</li>
      <li>スタンバイは参照のみ</li>
      <li>主な利用用途は<br>リードレプリカ, フェイルオーバー</li>
    </ul>
  </div>
  <div>

![img.png](/images/streaming_replication.png)

  </div>
</div>

---

# PostgreSQLのレプリケーションについておさらい

## ロジカルレプリケーション

<br>

<div class="grid grid-cols-2 gap-4 h-4/5">
  <div>
    <ul>
      <li>テーブルやデータベース単位で<br>WALを操作の情報に変換した情報をレプリケーションする</li>
      <li>レプリケーション元をパブリッシャー</li>
      <li>レプリケーション先をサブスクライバーといい、Publicationを購読する仕組み</li>
      <li>サブスクライバーに書き込みをしても良い<br>（完全同期の必要がない）</li>
      <li>OSやメジャーバージョンが異なってもレプリケーションできる</li>
      <li>主な利用用途は<br>分析目的のデータベースを作る, バージョンアップ</li>
    </ul>
  </div>
  <div class="flex flex-col place-content-around ">
    <img class="object-contain" src="/images/logical_replication_how_works.png" alt="__"/>
  </div>
</div>

<!--
ロジカルレプリケーションの利用用途として、サブスクライバーは複数のパブリッシャーを持つ事もできるので、
複数のデータソースから特定のパブリッシャーにデータを集約して、分析用途として役立たせるような事も可能です。

また、メジャーバージョンが異なっていても、ロジカルレプリケーションであればレプリケーションできるので
旧バージョンをパブリッシャー、新しいバージョンのインスタンスをサブスクライバーとして定義しておく事で
安全にバージョンアップをすすめる事ができます
-->

---

# ロジカル（論理）レプリケーションの機能拡張

## 行フィルタ機能

- PUBLICATION定義にWHERE句を指定して条件を満たす行だけのPUBLICATIONが作成できるようになった

```sql
CREATE PUBLICATION chugoku_members
    FOR TABLE app.members WHERE address IN ('広島','岡山','島根','鳥取','山口');
```
<br>

## 列フィルタ機能

- PUBLICATION定義でカラムを絞る事で特定のカラムだけのPUBLICATIONが作成できるようになった

```sql
CREATE PUBLICATION member_emails
    FOR TABLE app.members (email);
```
<br>

## スキーマ対象でテーブルを一括指定

```sql
CREATE PUBLICATION app_schemas
    FOR ALL TABLES IN SCHEMA app;
```

<!--

行フィルタ機能は更新される事で条件を満たさなくなったら、サブスクリプション側では削除されます。
逆に更新された事で条件を満たすようになったら、サブスクリプション側では追加されます。

スキーマ対象のテーブル一括指定は
データベースにある全てのテーブルを対象とかはできたんですが、今回からスキーマに所属するテーブルを一括で指定できるようになりました

-->

---

# ロジカル（論理）レプリケーションの機能拡張列挙

- サブスクリプション側でエラーになった時に自動で購読を止めるオプション追加
- サブスクリプション側でLSN（WALログID）を指定して処理をスキップする事ができる機能追加

これまで、煩雑な処理の後に調整していたり、ずっとエラーログが出続けるような問題について<br>
サポートが入った

---
layout: center
---

# パラレルクエリの強化

---

# パラレルクエリーの性能向上

- `SELECT DISTINCT` 文でパラレルクエリが有効化！
  - 私個人的に好きな機能の一つです

![img.png](/images/parallel_query.png)

---

# その他機能

- 正規表現関数の追加
- ログをJSON形式で出力できるように 
- NOT IN句のアルゴリズムを改善して高速化
- ソートアルゴリズムが改善
- ウィンドウ関数 ( row_number, rank, count )の性能改善
- JSON関数の追加 ... は残念ながら見送られた

<!--
その他、PostgreSQL15に入る機能としては記載の通りです。
特に大きく取り上げて説明する事は省略しようかと思いますが、ざっと説明していきます。

正規表現関数の追加
=> 他のRDB製品で実装されている関数に合わせた仕様と同等の関数が追加されました

ログをJSON形式で出力できるように
=> PostgreSQLの設定ファイルにjsonlogオプションを指定するとログがJSONで出力されます。
自分もPostgreSQLのログパーサーとか作った事あるんですけど、パース処理書くの面倒だったりするので
便利な機能だと思います。

NOT IN句のアルゴリズムを改善して高速化
=> IN句ではすでにサポートされていたみたいですが、順に値を比較する構造から内部的にハッシュテーブルを作って比較する手法によって高速化が実現されてる

-->

---
layout: center
class: "text-center"
---

# バージョン非互換の変更

## データベース作成時のデフォルト権限の変更

---

データベース作成時のデフォルト権限の変更

# スキーマとは何か?

![img.png](/images/database.png)

- スキーマがあると
  - リファクタリング前後のような意味のある単位で分割する事ができる
  - 1つのデータベースを多数のユーザが互いに干渉することなく使用できる

<!--
ここからは、PostgreSQL15バージョンアップ時に発生する注意事項について説明していきたいと思います

PostgreSQL15 から Publicスキーマのデフォルト権限からオブジェクトの作成権限がなくなりました。

この話をする前に、スキーマという概念もおさらいしておきましょう。
PostgreSQLでは 1つ以上のデータベースを管理する事ができます。
データベースというのはテーブルとかビューとかそういったSQLオブジェクトを管理する概念のうちで、一番大きな枠組みとなるものというようなイメージです.

データベースの下のは1つ以上のスキーマが含まれています。スキーマってのは何かというと
SQLオブジェクトをデータベースより小さい範囲でグルーピングするためのものですね。
スキーマが別なら、同じテーブル名のテーブルとかもそれぞれのスキーマに存在させる事とかもできます。

例えば、自分がこれまでスキーマを分割した経緯だと
public と new みたいなスキーマに分割して データベースリファクタリングをした後のスキーマだけを newスキーマにするといった事もやったりした事あります

で、データベースは作成をしたタイミングでは public というスキーマを持っていて
データベースに対してテーブルを作ると デフォルトの場合は基本的に publicに対してテーブルを作るような流れでした。 

-->

---

# データベース作成時のデフォルト権限の変更

## 元々

- `public` スキーマに誰でもテーブルなどのオブジェクトを作成可能だった
- これによって、脆弱性が発生したり、攻撃の対象となってた（ CVE-2018-1058 とか )

<br>

## PostgreSQL15から 
 
- データベース所有者かスーパーユーザ以外がオブジェクトを作成する権限がデフォルトから除外された
  - 上記の脆弱性の危険性と今後も付き合っていく必要があるが、 **デフォルトから除外** なので<br>追加すれば元に戻す事はできる
- `public` スキーマの所有者が pg_database_owner に変更された 



---

データベース作成時のデフォルト権限の変更

# バージョンアップ時に想定される影響

- PostgreSQL14でデータベースの中にあるpublicスキーマを運用している場合に<br>PostgreSQL15にバージョンアップした時にどのような影響が発生するか.
- バージョンアップ方法が2種類あるので、それぞれ比較

<br>

## dump & restore によるバージョンアップ

- `pg_dump`で旧バージョンからダンプして`pg_restore`で新バージョンにリストアを行う方式
- **新しい空のデータベースを作った上でリストアする** ので、publicスキーマの書き込み権限は無く<br>publicスキーマを使用している場合は影響を受ける

<br>

## pg_upgrade

- すべてが以前のバージョンと同じように**コピー**される
- 旧バージョン時点でpublicスキーマへの書き込み権限があれば、そのまま継続利用できる

<!-- 
旧バージョンの互換を切る内容ではありますので
旧バージョンから新バージョンにバージョンアップをした時の影響について調べてみました.

まず、PostgreSQLに依存した形でできるバージョンアップ手法は2種類です。
dump -> restore , pg_upgrade という選択肢になります.

-->

---

データベース作成時のデフォルト権限の変更

# AWSでアップデートをする時の挙動

- RDS for PostgreSQLではバージョンアップをコンソールから実行できる
- そこでは `pg_upgrade` を利用してバージョンアップが行われているので、旧バージョンのまま利用できる 
- ちなみに...
  - GCP (Cloud SQL for PostgreSQL) もコンソールでバージョンアップでき、 `pg_upgrate` が使われてる 
  - Azureは、コンソールでメジャーバージョンアップには対応してないので、`dump & restore` 方式を使う必要がある
- 旧バージョンのまま利用できる とは言いつつも **publicスキーマの書き込み権限が脆弱性の起因となる** 事についてはちゃんと考慮の上、運用していきましょう。

--- 

# 最後に...

- PostgreSQL15では、他RDBで使えてる機能を積極サポートしながらも、PostgreSQL自体の強み機能も改善していくようなバージョンアップと感じています。
  - Cloud移行に伴って商用DBからPostgreSQLに乗り換えて頂く事も多い事から他RDBで使えている機能の積極サポートはありがたそう. 
  - Docker公式イメージでPostgreSQL15 beta版を利用する事もできるので興味があれば試してみてください！
- PostgreSQL15のGAリリースは 2022-10-13です！！ お楽しみに！！

---

# 参考文献

- PostgreSQL15 検証レポート SRAOSS 
  - https://www.sraoss.co.jp/tech-blog/wp-content/uploads/2022/08/pg15_report_20220906.pdf
- ロジカルレプリケーションの紹介 (1) SRA OSS Tech Blog
  - https://www.sraoss.co.jp/tech-blog/pgsql/logical-replication-1/
- Changes to the public schema in PostgreSQL 15 and how to handle upgrades
  - https://andreas.scherbaum.la/blog/archives/1120-Changes-to-the-public-schema-in-PostgreSQL-15-and-how-to-handle-upgrades.html
- PostgreSQL14文書
  - https://www.postgresql.jp/document/14/html/index.html
- PostgreSQL15Document
  - https://www.postgresql.org/docs/15/


---

# 参考文献

- PostgreSQL 15 開発最新情報
  - https://www.slideshare.net/masahikosawada98/postgresql-15
- PostgreSQL 15 最新情報解説
  - https://www.sraoss.co.jp/wp-content/uploads/files/event_seminar/material/2022/PG15_sraoss_techwebinar_20220902.pdf
- PostgreSQL の INSERT ON CONFLICT と MERGE の簡易性能比較
  - https://qiita.com/fujii_masao/items/462bac9f6a107d6134c4
- PostgreSQL 15 新機能検証結果 (Beta 1)
  - https://h50146.www5.hpe.com/products/software/oe/linux/mainstream/support/lcc/pdf/PostgreSQL_15_Beta_1_New_Features_ja_20220524-1.pdf
- Wikipedia MERGE (SQL)
  - https://ja.wikipedia.org/wiki/MERGE_(SQL)

---
layout: center
class: text-center
---

# ご清聴ありがとうございました