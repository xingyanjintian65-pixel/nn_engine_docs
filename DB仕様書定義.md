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
