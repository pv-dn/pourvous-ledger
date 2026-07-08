# 入出荷台帳（pourvous-ledger）

| 項目 | 内容 |
|------|------|
| **ID** | `pourvous-ledger` |
| **呼び名** | 入出荷台帳 |
| **フォルダ** | `C:\Users\e--yo\Apps\入出荷台帳` |
| **会社** | 株式会社プゥル・ヴー（ホワイト事業部） |

## 概要

手書きの入出荷管理表をタブレット入力に置き換えるアプリ。客先・品目は **プゥルヴー伝票** の Firestore マスタから**読み取りのみ**。

## 重要：伝票アプリは変更しない

| 方針 | 内容 |
|------|------|
| **プゥルヴー伝票** | **コード・画面は触らない**。今までどおり単独で運用 |
| **連携** | 台帳は伝票の `customers` / `prices` を読むだけ |
| **伝票への渡し方** | 台帳の「伝票入力メモ」で「出」の数量を表示 → 担当者が伝票アプリへ手入力 |
| **Firestore** | 台帳データは `pvdata/ledgers` のみ新規書き込み（伝票の既存キーは変更しない） |

将来、伝票へ自動連携する場合も**伝票側に任意機能を足す**形にし、既存の発行フローは残す。

## 技術スタック

- 単体 HTML（`index.html` が正本）
- Firebase Auth + Firestore（プロジェクト `pourvoussystem`、伝票と同じ）
- ログインセッションはタブ内のみ（`inMemoryPersistence`・伝票と混ざらない）
- **1日1回ログイン**：朝ログイン後はその日は再入力不要（`indexedDBLocalPersistence` + 日付チェック）

## Firestore

| キー | 読/書 | 用途 |
|------|-------|------|
| `pvdata/customers` | 読のみ | 客先一覧（伝票と共有） |
| `pvdata/prices` | 読のみ | 品目一覧（伝票と共有） |
| `ledgerRecords` | 読書 | 完了台帳（1件ずつ保存・複数端末同時利用向け） |

**Firebase ルール:** `firestore.rules` を Firebase コンソールへ反映してください（`ledgerRecords` を追加）。

認証済みユーザーは読み書き可能。

## 運用フロー

1. 台帳アプリで客先を選び、入/出を入力
2. **完了して保存** → 履歴に残る
3. **伝票入力メモ** → 「出」の数量一覧（伝票アプリを別途開いて転記）
4. 伝票アプリでいつも通り納品書発行

## 本番 URL

https://pv-dn.github.io/pourvous-ledger/

## リポジトリ

https://github.com/pv-dn/pourvous-ledger

## 同時利用

- **別の客先**なら複数端末で同時入力OK
- 履歴は **1件ずつ** Firestore に保存（上書きしにくい）
- 履歴タブで **↻ 更新** → 他端末の保存を反映

## Firebase ルール反映（初回のみ）

Firebase コンソール → Firestore → ルール に `firestore.rules` の内容を反映するか:

```powershell
# Firebase CLI がある場合
cd "C:\Users\e--yo\Apps\入出荷台帳"
firebase deploy --only firestore:rules --project pourvoussystem
```

## 他アプリとの関係

| アプリ | 関係 |
|--------|------|
| **プゥルヴー伝票** | マスタ読取・運用は別アプリのまま |
| プゥルヴー在庫 | 無関係 |
| 価格参照 | 無関係 |

## Cursor で開く

`Apps\Cursor用\workspaces\入出荷台帳.code-workspace`

## デプロイ（GitHub Pages）

```powershell
cd "C:\Users\e--yo\Apps\入出荷台帳"
git init
git remote add origin https://github.com/pv-dn/pourvous-ledger.git
git add index.html manifest.webmanifest README.md PROJECT.md .gitignore
git commit -m "入出荷台帳 初回"
git push -u origin main
```

GitHub リポジトリの Settings → Pages → `main` / root で公開。

## 変更履歴メモ

| 日付 | 内容 |
|------|------|
| 2026-07-08 | たたき台作成・テンキー右配置 |
| 2026-07-08 | 伝票アプリ無変更方針・伝票入力メモに切替 |
