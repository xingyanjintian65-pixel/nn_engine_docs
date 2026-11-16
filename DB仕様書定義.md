# DB仕様書定義
現在のjson構造をテーブル化する定義書を作成
## json構造
```json
{
  "year": 2026,
  "months": [
    {
      "month": 11,
      "dates": [
        {
          "date": 4,
          "goal": "jsonを読み込みxlsxで保存するツールの開発",
          "progress": 1,
          "states": true,
          "limit": "2026-11-05"
        },
        {
          "date": 5,
          "goal": "jsonデータをpostgreSQLに保存するツールの開発",
          "progress": 1,
          "states": false,
          "limit": "2026-11-06"
        },
        {
          "date": 6,
          "goal": "",
          "progress": 0,
          "states": false,
          "limit": ""
        }
      ]
    },
    {
      "month": 12,
      "dates": [
        {
          "date": 1,
          "goal": "独自モデルの開発要件を定義完了",
          "progress": 0,
          "states": false,
          "limit": "2026-11-05"
        },
        {
          "date": 2,
          "goal": "",
          "progress": 0,
          "states": false,
          "limit": ""
        },
        {
          "date": 3,
          "goal": "",
          "progress": 0,
          "states": false,
          "limit": ""
        }
      ]
    }
  ]
}

```
## 定義案
1. オブジェクトごとにテーブルをつける
2. すべての構造を1つのテーブルにまとめる

1を採用する
理由は現在の構造をテーブルで再現するにはRDBを用いたテーブル構造が適している
と考えられるため

## SQL作成フロー
1. yearテーブルを作成
2. monthテーブルを作成
3. dateテーブルを作成
### 詳細フロー
1. yearテーブルのカラムを定義
2. monthと連携させるカラムを定義
3. monthテーブルを作成
4. dateテーブルを作成
5. dateテーブルmonthテーブルを連携させるカラムの定義
6. yearテーブルとmonthテーブルを連携
7.

## yearテーブル要件定義
カラムは3つ
id = primarykey
year = int
month = monthのidを格納する

**id** 
primarykeyで自動生成
ランダムではなく連番で作成

**year**
intで年を格納
not nullを設定

**month**
monthのidを格納
not nullで作成するがmonthテーブルの作成と連携可能になるまでは作成しない
monthテーブルの作成と連携が可能になったらinsertで追加する

---

## 実装での定義（2026-11-16 時点）

上の定義は作成前の案である。実装（`workdir` リポジトリの `src/db.rs`）では、次の形になっている。表は、初めてデータを登録するときに `CREATE TABLE IF NOT EXISTS` で作る。

### year テーブル

| カラム | 型 | 制約 | 内容 |
| --- | --- | --- | --- |
| id | SERIAL | PRIMARY KEY | 連番で自動生成 |
| year | integer | NOT NULL | 年 |

### month テーブル

| カラム | 型 | 制約 | 内容 |
| --- | --- | --- | --- |
| id | SERIAL | PRIMARY KEY | 連番で自動生成 |
| year_id | integer | NOT NULL、`year(id)` を参照、ON DELETE CASCADE | どの年の月か |
| month | int | NOT NULL | 月 |

### date テーブル

| カラム | 型 | 制約 | 内容 | JSON の項目 |
| --- | --- | --- | --- | --- |
| month_id | integer | NOT NULL、`month(id)` を参照、ON DELETE CASCADE | どの月の日か | — |
| date | int | NOT NULL | 日 | `date` |
| goal | text | なし | 目標 | `goal` |
| progress | int | NOT NULL | 進捗 | `progress` |
| states | bool | NOT NULL | 完了したか | `states` |
| limit_time | varchar(50) | なし | 期日（文字列のまま保存） | `limit` |

```sql
CREATE TABLE IF NOT EXISTS year(
    id SERIAL primary key,
    year integer not null
);

CREATE TABLE IF NOT EXISTS month(
    id SERIAL primary key,
    year_id integer not null references year(id) on delete cascade,
    month int not null
);

CREATE TABLE IF NOT EXISTS date(
    month_id integer not null references month(id) on delete cascade,
    date int not null,
    goal text,
    progress int not null,
    states bool not null,
    limit_time varchar(50)
);
```

### 案との違い

| 項目 | 案 | 実装 |
| --- | --- | --- |
| 表のつなぎ方 | year テーブルに `month` カラムを置き、month の id を入れる | 子の側に親の id を置く（month に `year_id`、date に `month_id`）。1つの年に複数の月があるため |
| 作成の順番 | year を作り、あとから `month` カラムを追加する | year → month → date の順に作るだけで、あとからの追加はない |
| date テーブルの主キー | 定義なし | 主キーも一意制約もない |
| 期日のカラム名 | `limit` | `limit_time`（`limit` は SQL の予約語） |

### 残っている課題

- year の `year`、month の `(year_id, month)`、date の `(month_id, date)` に一意制約がない。重複はアプリ側で登録前に調べて防いでいる
- date テーブルに主キーがない
- 期日が文字列のため、日付として比べたり並べ替えたりできない
