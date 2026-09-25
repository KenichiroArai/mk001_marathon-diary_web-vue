# AGENTS.md — marathon-diary web-vue

AI コーディングエージェント向けの作業ガイド。
Cursor / Codex / Claude Code など複数ツールで共通利用する。

## プロジェクト概要

- **役割**: マラソン日記の Web フロントエンド（Vue.js）
- **含むもの**: UI、画面遷移、API クライアント、フロント側状態管理
- **含めないもの**: サーバー側ドメインロジック、DB スキーマ、REST API 本体
- **設計方針**: 初めから大規模向け。**デザイン・画面** と **ロジック・処理** を層で分離し、機能単位で拡張できる構成とする

## 技術スタック

- フレームワーク: Vue.js（Composition API / `<script setup>` を基本とする）
- 言語: TypeScript
- ビルド: Vite
- ルーティング: Vue Router
- 状態管理: Pinia
- パッケージ管理: （TODO: npm / pnpm / yarn）
- テスト: Vitest + Vue Test Utils（TODO: 導入後に確定）
- リント / フォーマット: ESLint + Prettier（TODO: 導入後に確定）

## アーキテクチャ方針（必須）

画面（見た目）と処理（ロジック）を混ぜない。依存の向きは次のとおり固定する。

```text
pages / layouts / ui  →  features/*/ui  →  features/*/model  →  shared / infrastructure
     （デザイン・画面）      （画面パーツ）     （ロジック・処理）      （共通・外部I/O）
```

| 層 | 責務 | 置いてよいもの | 置いてはいけないもの |
| --- | --- | --- | --- |
| **presentation**（`pages` / `layouts` / `ui`） | デザイン・画面表示・ユーザー操作の受け口 | テンプレート、スタイル、表示用 props / emit、薄い配線 | API 呼び出し、業務判定、複雑な変換 |
| **feature model**（`features/*/model`） | 機能ごとのロジック・処理 | composable、store、バリデーション、画面用状態の組み立て | HTML / CSS、見た目専用の分岐 |
| **infrastructure**（`infrastructure`） | 外部との I/O | API クライアント、ストレージ、認証トークン扱い | UI コンポーネント |
| **shared** | 横断的な共通部品 | 純関数、共通型、デザインシステムの基礎 UI | 特定機能の業務ロジック |

### 分離ルール

- `.vue`（pages / ui）は **表示とイベントの橋渡しのみ**。分岐・計算・API は `model` 側へ出す
- 機能固有の処理は必ず `features/<feature>/model` に置く（横断ロジックのみ `shared`）
- API レスポンスの解釈・エラー正規化は `infrastructure` または `features/*/model` で行い、テンプレートに生データ処理を書かない
- 機能追加時は既存 `features` を肥大化させず、新 feature ディレクトリを切る

## ディレクトリ構成

```text
public/                          # 静的アセット
src/
  app/                           # アプリ起動・グローバル設定
    App.vue
    main.ts
    providers.ts                 # プラグイン登録など（あれば）

  pages/                         # 【画面】ルート単位ページ（薄い配線のみ）
    diary/
      DiaryListPage.vue
      DiaryDetailPage.vue
    settings/
      SettingsPage.vue

  layouts/                       # 【画面】レイアウト（ヘッダー・ナビ等の枠）
    DefaultLayout.vue
    AuthLayout.vue

  ui/                            # 【デザイン】見た目専用・機能非依存
    components/                  # ボタン、入力、モーダルなどのデザインシステム
      ButtonBase.vue
      TextField.vue
    styles/                      # グローバルスタイル、トークン、テーマ
      tokens.css
      global.css

  features/                      # 機能単位（画面パーツ + ロジックを同居させつつ層で分離）
    diary/
      ui/                        # 【画面】この機能の表示コンポーネント
        DiaryList.vue
        DiaryForm.vue
      model/                     # 【ロジック・処理】
        useDiaryList.ts          # 画面用 composable（状態・操作の組み立て）
        diaryStore.ts            # Pinia（必要な場合）
        diaryValidators.ts       # バリデーション
        diaryMappers.ts          # API DTO ↔ 画面モデル変換
      index.ts                   # 公開 API（外から使うものだけ export）
    auth/
      ui/
      model/
      index.ts

  infrastructure/                # 【処理】外部 I/O
    api/
      httpClient.ts
      diaryApi.ts
      types/                     # API 契約型（OpenAPI 由来など）
    storage/
      localStorage.ts

  shared/                        # 横断共通（特定機能に依存しない）
    lib/                         # 純関数ユーティリティ（日付、単位、フォーマット）
    types/                       # 共有型
    constants/
    composables/                 # 機能非依存の小さな hooks（useMediaQuery など）

  router/
    index.ts
    routes.ts

tests/                           # テスト（model を厚く、ui/pages は表示契約を薄く）
index.html
package.json
vite.config.ts
tsconfig.json
```

### 置き場所の判断フロー

1. 見た目だけか？ → `ui/` または `features/*/ui/`
2. 特定機能の処理・状態か？ → `features/*/model/`
3. HTTP / ストレージなど外部か？ → `infrastructure/`
4. 複数機能で使う純関数・型か？ → `shared/`
5. ルートに紐づくページ枠か？ → `pages/` + `layouts/`（中身の処理は feature を呼ぶだけ）

## ビルド・テスト

```bash
# TODO: 実際のコマンドに置き換える
# 依存インストール:
# 開発サーバー:
# ビルド:
# テスト:
# リント:
# 型チェック:
```

## 作業時の原則

- API 契約は `gr001_marathon-diary_api` / `kb001_marathon-diary_doc` の仕様に従う
- **デザイン・画面**（`pages` / `layouts` / `ui` / `features/*/ui`）と **ロジック・処理**（`features/*/model` / `infrastructure` / `shared/lib`）を混ぜない
- ページコンポーネントは feature の公開 API（`features/*/index.ts`）だけに依存する
- コンポーネントは単一責任で小さく保つ
- ユーザー向け文言・日付・単位表示の一貫性を保つ（表示整形は `shared/lib` または `model` の mapper）
- アクセシビリティとモバイル表示を意識する
- 本ドキュメントのコーディングルール・テストルール・JSDoc ルールに従う
- （追記: 命名規則、スタイリング方針、環境変数の扱い）

## 共通のコーディングルール

### 関数・メソッドの戻り値

- 戻り値は変数 `result` で定義する
- 戻り値の変数は先頭で宣言する
- return 文は `return result` に統一する

### 処理コメント

- 機能ごと、処理のまとまり単位に `/* コメント */` で記載する
- 通常コメントは `//` で記載する

### JSDoc

- エクスポートする関数・型・コンポーザブルは JSDoc を必須とする
- 後述の「JSDoc のフォーマットルール」に従う

### 早期リターンパターン

- 早期リターン（ガード節）を使用し、不要なネストを避ける
- 条件が満たされない場合は早期に `return` する
- if-else の代わりにガード節を使い、インデントの深さを最小限に抑える

```typescript
// 望ましくない形式:
if (condition) {
  // 処理A
  // 処理B
}

// 望ましい形式:
if (!condition) {
  return result
}
// 処理A
// 処理B
```

```typescript
export function someMethod(input: string | null): boolean {
  let result = false // 先頭で戻り値変数を宣言

  // 早期リターン（ガード節）
  if (input === null) {
    return result
  }

  // メインの処理
  result = true

  return result // 統一された形式で return
}
```

### Vue コンポーネント（画面層）

- `<script setup lang="ts">` を基本とする
- Props / Emits は型付きで定義する
- テンプレートに複雑な式を書かず、表示用の値は `model` の composable / `computed` から受け取る
- `pages` / `features/*/ui` では API を直接呼ばない（`model` 経由）
- 副作用（購読・タイマー・リスナー）の寿命管理が必要な処理は `model` の composable に寄せる
- `ui/components` はデザインシステムとして機能名・業務用語に依存させない

### feature の公開境界

- 他 feature / pages から import してよいのは `features/<name>/index.ts` が export したものだけ
- `features/<name>/model` の内部ファイルを横断 import しない
- feature 同士の直接依存は避け、必要なら `shared` または上位の orchestration（page）でつなぐ

## テストのコーディングルール

### テスト単位

- **ロジック・処理**（`features/*/model`、`shared/lib`、`infrastructure`）を厚くテストする
- **デザイン・画面**（`ui` / `pages`）は表示契約（テキスト・属性・表示有無・emit）を薄くテストする
- 公開 API（export / props / emit / 表示結果）を対象とする
- 内部実装の詳細に依存しすぎない

### テストファイル

- `model` / `lib` は対象の近く、または `tests/` 配下に配置する
- ファイル名は `*.spec.ts` または `*.test.ts` とする

### テストメソッド名

- `testXxx_パターンYyy` の形式とする
- 「Xxx」の先頭は大文字で対象関数名・振る舞い名を入れる
- 「パターン」は正常系 `normal`、準正常系 `semi`、異常系 `error` とする
- 「Yyy」の先頭は大文字でテスト項目を入れる
- 例: `testXxx_normalYyy` / `testXxx_semiYyy` / `testXxx_errorYyy`

### テストメソッドの中身

- 対象ごとに行い、1 つのテストに 1 つの検証意図を実装する
- 正常系・準正常系・異常系に分けて実装する
  - 正常系: 正常処理が完了するパターン
  - 準正常系: 入力不正などで期待どおり失敗・空表示などになるパターン
  - 異常系: 正常系・準正常系以外の想定外パターン（API 障害など）

### テストの JSDoc

- フォーマット: `対象名 のテスト - パターン:テスト内容`
- パターンには正常系・準正常系・異常系を入れる

```typescript
/**
 * someMethod のテスト - 正常系:引数が1文字の場合
 */
```

### テストコードの実装順序

1. **期待値の定義** — `/* 期待値の定義 */`、`expected` で始まる変数
2. **準備** — `/* 準備 */`、`test` で始まる変数
3. **テスト対象の実行** — `/* テスト対象の実行 */`、`test` で始まる変数
4. **検証の準備** — `/* 検証の準備 */`、`actual` で始まる変数
5. **検証の実施** — `/* 検証の実施 */`
   - `expect` には `actualXXX` と説明が分かるアサーション文言を意識する
   - 期待値は「期待値の定義」の `expectedXXX` を使う

### 検証方法の指定

- 1 つずつ検証する
- 真偽値は `toBe(true)` / `toBe(false)` を使う
- 値比較は `toBe` / `toEqual` を使い分ける（参照同一性と深い等価）
- null / undefined は `toBeNull` / `toBeUndefined` を使う
- DOM / 画面はユーザーが観測できる結果（テキスト・属性・表示有無）で検証する

```typescript
import { describe, expect, it } from 'vitest'

describe('someMethod', () => {
  /**
   * someMethod のテスト - 正常系:有効な入力の場合
   */
  it('testSomeMethod_normalValidInput', () => {
    /* 期待値の定義 */
    const expectedResult = true

    /* 準備 */
    const testInput = 'a'

    /* テスト対象の実行 */
    const testResult = someMethod(testInput)

    /* 検証の準備 */
    const actualResult = testResult

    /* 検証の実施 */
    expect(actualResult).toBe(expectedResult)
  })
})
```

## JSDoc のフォーマットルール

### 基本形式

```typescript
/**
 * 関数・型・モジュールの説明をここに書きます。
 * 複数行の説明の場合は、このように記述します。
 *
 * @param param1 - 最初のパラメータの説明
 * @param param2 - 2番目のパラメータの説明
 * @returns 戻り値の説明
 * @throws 例外が発生する条件の説明
 * @see 関連する関数や型への参照
 * @deprecated 非推奨となった場合の説明（該当する場合）
 */
export function sampleMethod(param1: string, param2: number): string {
  // 実装
}
```

### 主要なタグ

| タグ | 用途 |
| --- | --- |
| `@param` | パラメータの説明 |
| `@returns` | 戻り値の説明 |
| `@throws` | 発生する可能性のある例外の説明 |
| `@see` | 関連する他の関数・型への参照 |
| `@deprecated` | 非推奨であることを示す |
| `@example` | 使用例 |

### 記述ガイドライン

- 最初の文は要約文として簡潔に書く
- 完全な文章で、技術的に正確に記述する
- 必要な情報を漏れなく記載する

```typescript
/**
 * サンプルコードの使用例:
 *
 * @example
 * const result = sampleMethod('test', 123)
 */
```

### チーム統一フォーマット例

```typescript
/**
 * [関数/型/コンポーザブルの名前]の説明
 *
 * 詳細な説明（必要な場合）
 *
 * UI 上の振る舞い・業務上の意味（必要な場合）
 *
 * @param paramName - [引数の説明]
 * @returns [戻り値の説明]
 * @throws [例外] [例外の発生条件]
 * @see [参照すべき他の関数や型]
 * @deprecated [非推奨となった理由と代替手段]（該当する場合）
 */
```

## 変更時のチェックリスト

- [ ] 変更が正しい層に入っている（画面 ↔ ロジックの混在なし）
- [ ] 新機能は `features/<name>/{ui,model}` で追加されている
- [ ] pages から feature 内部（`model` 配下）を直接 import していない
- [ ] API レスポンス変更との整合
- [ ] 主要画面の表示確認（デスクトップ / モバイル）
- [ ] テストの追加 / 更新（model を優先、命名・実装順序・検証方法を含む）
- [ ] コーディングルール（戻り値 `result`、早期リターン、処理コメント）の順守
- [ ] JSDoc の追加 / 更新
- [ ] リント / 型チェック

## やってはいけないこと

- API 仕様をフロントだけで勝手に変えること
- シークレットや個人情報をフロントにハードコードすること
- `.vue`（pages / ui）に API 呼び出し・業務判定・複雑な変換を書くこと
- `ui/components` に特定機能の業務ロジックを入れること
- feature 内部ファイルを他 feature / pages から横断 import すること
- 深いネストのままガード節を使わずに実装すること
- テストに複数ケースを詰め込むこと

## 関連リポジトリ

- 仕様: `kb001_marathon-diary_doc`
- API: `gr001_marathon-diary_api`

## 参考リンク

- README: `./README.md`

## 順守

以上の内容を順守し、タスクを遂行してください。
