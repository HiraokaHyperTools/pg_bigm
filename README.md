# pg_bigm

[pgbigm/pg_bigm: The pg_bigm module provides full text search capability in PostgreSQL. This module allows a user to create 2-gram (bigram) index for faster full text search.](https://github.com/pgbigm/pg_bigm) を、PostgreSQL 9.6 - 17.0 Windows 版で楽しめるように助力いたします。 自炊方法については build.md を参照。

## インストール方法

[Releases](https://github.com/HiraokaHyperTools/pg_bigm/releases/) から `20250925.7z` をダウンロードして展開します。

PostgreSQL 17.x Windows x86-64 の場合:

`pg_bigm-postgresql-17.0-x64` フォルダーの中身を `C:\Program Files\PostgreSQL\17` へコピー

pg_bigm は、データベースごとにインストールしてください。

対象のデータベースで SQL 文を実行:

```sql
CREATE EXTENSION pg_bigm;
```

いまくいけば、これで完了です。

## 使用例

[sql/pg_bigm_ja.sql](sql/pg_bigm_ja.sql) を参照してください。 実例が記載されています。

## インデックス作成方法

```sql
-- tests for creation of full-text search index
CREATE TABLE test_bigm (col1 text, col2 text);
CREATE INDEX test_bigm_idx ON test_bigm USING gin (col1 gin_bigm_ops);
```

## 検索方法 (部分一致)

```sql
SELECT col1 FROM test_bigm WHERE col1 LIKE likequery('東京都');
```

## 高速化の原理

`pg_bigm` は [64.4. GINインデックス](https://www.postgresql.jp/document/17/html/gin.html) を実装します。 (逆にいうと GIN 以外の B-Tree, GiST, BRIN, ハッシュインデックス等は実装しません)

インデックスの対象物は、バイグラムです。

バイグラムとは、 2 文字からなる文字列の事です。例: `ai`, `文字`, `👋😀`

UTF-8 エンコード使用時は、 UTF-32 の文字数と同じです。

参考:

| 文字 | UTF-16 | UTF-32 |
|---|---|---|
| `ai` | `U+0061` `U+0069` | `U+00061` `U+00069` |
| `文字` | `U+6587` `U+5B57` | `U+06587` `U+05B57` |
| `👋😀` | `U+D83D` `U+DC4B` `U+D83D` `U+DE00` | `U+1F44B` `U+1F600` |

これらのバイグラム群を、汎用転置インデックス (Generalized Inverted Index) へ保存する事で、高速に検索できるようにするのが pg_bigm です。

わかりにくいですか。 本の巻末索引を思い出してください。 辞書順 (あ ～ ん) に並べた単語と、ページ番号一覧の組み合わせが付録されていると思います。 これが GIN (汎用転置インデックス) の一例です。

但し、 PostgreSQL の GIN の場合はもう少し複雑です。 `りんご` で検索する場合は [`りん`, `んご`] の両方のバイグラムが含まれるレコードの一覧を取得しなくてはなりません。

[`りん`, `んご`] の両バイグラムが含まれているからといって `りんご` という単語が含まれている可能性は 100% ではありません。 [`りんかい`, `たんご`] のバイグラムにヒットした可能性もあります。 そういう場合は実際にデータを検査して、真偽確認をする必要があります。 つまり、ストレージからレコードを読み取ったうえで `りんご` という単語が本当に含まれているかどうかの最終確認が必要になります。 この処理のことを recheck 処理といいます。

この recheck の有無は、アクセスメソッド実装 (pg_bigm など) が GIN に対して都度指示ができます。 pg_bigm では、検索したいユニークなバイグラム数が 2 以上の場合は true になり、1 の場合は false になります。 ([bigm_gin.c](bigm_gin.c) の `gin_extract_query_bigm` を参照)

この recheck 動作を強制的に Off にできる `pg_bigm.enable_recheck` パラメータはありますが、検索結果が不正確になります。 操作しない方が良いでしょう。

図解すると、このようです:

![](images/DuHJoW9WkAAps2b.jpg)

こういった重そうなインデックス機能を使用したとしても、テーブル全体のシーケンシャルスキャンと比較して、高速 (検索時間の短縮) になるケースが感覚的には多いです。
