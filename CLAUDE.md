# CLAUDE.md

ネクロマンサーカードゲーム（仮）— HTML/JS モック。

このファイルは AI エージェント向けの作業契約書。**変更する前に必ず読むこと。**

---

## 0. Notion ページID

★ **検索で当てにいくことを禁止。必ずこの表から引くこと。**
「DECISION_LOG」「SESSION_STATE」「CURRENT_SPEC」は**他企画（旧企画を含む）にも同名ページが実在する**ため、
検索すると別企画のページを踏む。

### この企画（書き込み先はここだけ）

| ページ | ID | 正本 |
|---|---|---|
| 企画ページ（ネクロマンサーカードゲーム（仮）） | `3f8a8a5d-fd45-81f0-add6-f053a7c2f107` | Notion |
| DECISION_LOG | `3f9a8a5d-fd45-810f-bdd8-cde8a332542e` | **Notion**（repo の `docs/DECISION_LOG.md` は写し） |
| SESSION_STATE（写し・閲覧用） | `3f9a8a5d-fd45-818d-9757-f934f963dbe1` | **repo**（`docs/SESSION_STATE.md`。Notion は生成物） |
| CURRENT_SPEC（写し・閲覧用） | `3f9a8a5d-fd45-8120-903f-d9b820b5dac4` | **repo**（`docs/CURRENT_SPEC.md`。Notion は生成物） |
| _tasks（実装タスク） | `3f9a8a5d-fd45-8151-936f-f3f061b6b7ef` | Notion |

タスク番号の接頭辞は **`NC-`**。`CG-` は旧企画のもの（このリポジトリのコード中の `CG-0xx` は旧企画時代の履歴）。
新しいタスクを作る前に、必ず `_tasks` の進行中一覧を見ること。

### v0.2 までの記録（参照のみ・書き込み禁止）

旧企画「持ち駒カードゲーム（仮）」のページ。v0.2 までの記録はここにあり、**コピーしない**（二重に持つとどちらが正かで迷うため）。
★ **読むだけ。ここへは何も書かない。** 設計判断・仕様・状態はすべて上の表のページへ書く。

| ページ | ID | 読む場面 |
|---|---|---|
| 旧 CURRENT_SPEC | `3b8a8a5d-fd45-81bd-b087-d9e61a87148d` | v0.2 のルール（「以下は v0.2（凍結・参照のみ）」の節） |
| 旧 DECISION_LOG | `3b8a8a5d-fd45-8118-9c7f-fe1776262d0d` | 2026-10-10 より前のエントリ（v0.2 までの設計判断） |
| 却下案索引 | `3b8a8a5d-fd45-8153-86b2-ec2c8cae40c4` | v0.2 の却下案。★ **この企画にも効く。再提案しないこと** |

---

## 1. このリポジトリの位置づけ

★ **正本の向きは文書ごとに違う（NC-002・2026-10-14）。** 向きは「誰が書くか」で決まる。

| 文書 | 正本 | 写し |
|---|---|---|
| DECISION_LOG | **Notion**（設計判断はチャット側が書く） | `docs/DECISION_LOG.md` |
| SESSION_STATE | **repo**（`docs/SESSION_STATE.md`。Claude Code が書く） | Notion の写し（閲覧用・生成物。session-end が更新） |
| CURRENT_SPEC | **repo**（`docs/CURRENT_SPEC.md`。実装を見て Claude Code が書く） | Notion の写し（閲覧用・生成物。session-end が更新） |
| 企画ページ・_tasks | **Notion** | なし |

- 写しの先頭には「写し／生成物であり正本ではない・正本の場所・日時」の宣言を置く。**宣言を消さない。**
- 正本が Notion の文書（DECISION_LOG）が食い違ったら **Notion が正しい**。docs 側を直す。
- 正本が repo の文書（SESSION_STATE・CURRENT_SPEC）は、Notion の写しを直接編集しない。直しても次の更新で消える。
- `docs/DECISION_LOG.md` を勝手に「正史」として書き換えない。ミラーの更新は人間の指示に従う。
- 上の表以外の `docs/` のファイル（`REJECTED.md`・`research/`）は
  **旧企画から引き継いだ参照用の資料**（v0.2 まで・参照のみ・更新しない）。
  この企画のコンセプトは Notion の企画ページにある（`CONCEPT.md` は置かない）。
- セッション終了の手順は `.claude/skills/session-end/SKILL.md` に従う。
  同スキルは Notion のマスターの写しで、**企画固有の差分を持たない**。SESSION_STATE の向きは本節が決める（この企画は repo が正本）。

---

## 2. アーキテクチャの絶対制約

### 2.1 engine.js — 純粋なゲームロジック

- **DOM / window / document / localStorage に一切触らない。**
- **Node 単体で実行できること。** `node js/engine.js` が通らない変更は入れない。
- **`(state, action) => newState` の純粋関数として書く。**
  - 戻り値は**新しい state**。既存 state を破壊的に変更しない（no mutation）。
  - `push` / `splice` / 直接代入による既存オブジェクトの書き換えは禁止。
    新しい配列・オブジェクトを作って返す。
  - 同じ `(state, action)` からは常に同じ `newState` が出ること。

### 2.2 action はシリアライズ可能なオブジェクト

- **すべてのプレイヤー行動は、JSON 化できる action オブジェクトで表現する。**
- 関数・クラスインスタンス・DOM 参照を action に入れない。
- 例:
  ```js
  { type: 'play',  cardId: 'c03', slot: 2 }
  { type: 'draft', pick: 0 }
  { type: 'attack', attackerSlot: 1, targetSlot: 3 }
  { type: 'revive', cardId: 'c07' }
  { type: 'endTurn' }
  ```

### 2.3 視点フィルタ filterStateFor

- **`filterStateFor(state, playerId)` を最初から用意する。**
- state から「そのプレイヤーが見てよい情報だけ」を残した新しい state を返す。
- **相手の手札の中身とデッキの中身は必ず落とす**（枚数だけ残す）。
- 後付けで足すのではなく、engine の初期設計に含める。

### 2.4 乱数はシード管理

- **`Math.random()` の使用は禁止。**
- 乱数は engine 内部の seeded RNG のみを使う。RNG の状態は state の一部として持つ。
- **同一シード + 同一 action 列 → 同一結果**が常に成立すること。
- シャッフル・ドラフト・その他ランダム要素すべてがこの規約に従う。

### 2.5 view.js — 描画のみ

- **engine の state を読んで描画するだけ。**
- view からゲームルールを判定しない。勝敗・合法手・ダメージ計算を view に書かない。
- view が engine に働きかける唯一の手段は **action オブジェクトを engine に渡すこと**。

### 2.6 CPU は engine の外側

- **CPU（`js/ai.js`）は engine に入れない。** engine はルール、AI は考え方。
- **AI が見てよいのは `filterStateFor(state, playerId)` を通した state だけ。**
  相手の手札・両者のデッキを覗く CPU は書かない（視点フィルタが落とす構造にしてある）。
- **`(state, playerId) => action` の純粋関数。** 乱数を使わない。同じ state からは同じ手が出ること。
- 合法性の判定は engine のヘルパ（`legalActions` / `canSummon` 等）に訊く。AI 側でルールを書き写さない。
- 評価の重みは `data/ai.js`。`js/ai.js` に数値をベタ書きしない。
- 理由: 権威サーバ側で同じ CPU をそのまま動かせるようにするため（§3 と同じ）。

### 2.7 データ外部化

- **カードの数値・効果は `data/cards.js` に置く。JS にベタ書きしない。**
- `data/cards.js` と `data/ai.js` の中身は、`var NECRO_CARDS =` 行と末尾の
  `module.exports` 行を除けば**素の JSON**。データはこの1ファイルだけが正本で、
  `.json` との二重管理はしない（同期漏れを構造的に作らないため）。
- バランス調整がこのデータファイルの編集だけで完結する状態を保つ。
- ハードコードされた数値を見つけたらデータファイルに追い出す。

### 2.8 file:// で開けること（CG-008）

★ **作者は Windows から `index.html` をダブルクリックして遊ぶ。**
ローカルサーバを立てる手順を前提にしない。そのため:

- **ES モジュールを使わない。** `import` / `export` / `<script type="module">` は
  file:// では CORS で弾かれる（origin が `null` になるため）。
- **`fetch()` を使わない。** 同じ理由で file:// からは読めない。
  データは `<script>` タグで読み、グローバル（`NECRO_CARDS` / `NECRO_AI`）で受け取る。
- **ビルド工程を作らない。** 置いてあるファイルがそのまま動く状態を保つ。
- 各ファイルは即時関数で包み、
  **ブラウザ＝グローバル（`NECRO_ENGINE` / `NECRO_AI_CPU`）、Node＝`module.exports`** の
  両対応にしてある。この包みを外さないこと。`package.json` に `"type": "module"` を戻さないこと。
- 読み込み順は `index.html` で固定（データ → engine → ai → view）。
- この規約は `node js/engine.js` のセルフテストがソース検査で見張っている。

---

## 3. 上記制約の理由

将来 **オンライン対戦を権威サーバ型（authoritative server）で実装する**ため。

- クライアントは action だけをサーバへ送る。サーバが唯一の state を持ち、
  `filterStateFor` で各プレイヤーに見せてよい state だけを返す。
- そのためには engine が「DOM 非依存・決定的・シリアライズ可能」でなければならない。
- **ロックステップ／ロールバック方式は採用しない。**
  本作は隠し情報（手札・デッキ）を持つため、全クライアントが完全な state を
  保持する前提の方式はチート耐性を満たさない。
  → 詳細は旧企画の DECISION_LOG（§0 の参照のみの表）を参照。

---

## 4. 作業時のルール

- 実装前に `docs/SESSION_STATE.md` を読み、Notion の `_tasks` で自分のタスクのページを読む。
- 設計判断は **Notion の DECISION_LOG（§0）が正本**。Claude Code が判断を求められたら停止して報告し、
  裁定後に Notion へ記録する。`docs/DECISION_LOG.md` は写しなので直接足さない。
  （見出しの形: `## YYYY-MM-DD: 決定`。session-end のマスターに合わせてある。）
- **却下した案は DECISION_LOG の各エントリの「却下した案」に理由込みで書く。** 蒸し返し防止が目的。
  v0.2 までの却下案（`docs/REJECTED.md`＝旧企画の却下案索引の写し）も有効。再提案しない。
  再提案する場合は「なぜ却下理由が変わったか」を先に述べる。
- 調査・先行事例は `docs/research/` に置く。規約は `docs/research/README.md` を参照。
- 未確定事項を勝手に確定させない。`docs/SESSION_STATE.md` の「未確定項目」を尊重する。

### 4.1 報告フォーマット — 遠征ギルドログ（EX）と同じ構造で出す

**報告は遠征ギルドログの EX タスクと同じ構造にする。** この企画だけ別形式にしない。
出典は 02_operations の運用規約と、EX-052 をはじめとする実際の EX タスクページ。

#### 実装を終えたとき

```markdown
# 実装（対象・裁定X があれば併記）
**YYYY-MM-DD HH:MM〜HH:MM JST ／ HEAD `hash`（前は `hash`）／ 実行条件・件数**

## やったこと
- （何をどう変えたか。★ で要点を立てる）
- （裁定どおりか、どこを裁定外にしたか）

## 検証
| 完了条件 | 結果 |
|---|---|
| （条件） | ✅ / ⚠️ 未達（数字つきで） |

## 数字
| | 前 `hash` | 後 `hash` |
|---|---|---|
| （指標） | （値） | （値） |

## commit
- `hash` （メッセージ）

## 記録した一般則
- ★ （次に効く形で1行。なければ節ごと省く）
```

#### 判断が要って止まるとき

```markdown
## ★ 停止：（論点の名前）
**論点：**（何が決まっていないか。1行）

### A（推奨）：（案）
- 利点：
- 欠点：★

### B：（案）
- 利点：
- 欠点：

**トレードオフ**：A＝… ／ B＝…
**却下済み**：（該当する却下案と、その理由がこの場合に及ぶか。なければ「なし」）
```

#### 規則

- **測定値には日付・時刻・HEAD ハッシュを必ず併記する。** 3点そろわない数値は
  「いつのものか不明」として扱い、判断の根拠にする前に測り直す。
- **実施日時は推測しない。** `TZ=Asia/Tokyo date` で取り直す。
  チャット側は現在時刻を持たないため、報告の日時が時系列の基準になる。
- **未検証のものを「完了」と書かない。** 検証欄に未達を数字で書く。
- **指示からはみ出した点・未対応の欄を空にしない。** なければ「なし」と書く。
- **勝手に決めた判断は停止ブロックに出す。** 指示のツリーから外れた追加や、
  指示になかった項目の補完は、黙って進めない。
- **却下済みの案を機械的に適用しない。** 却下の理由がこの場合にも及ぶかを先に確かめる
  （EX-052 の攻撃行の例）。

### 4.2 共有欄 — 最後に必ず1つ出す

報告の末尾に、**どこに書いたかとサマリー**をまとめて出す。セッション中に何度報告しても、
共有欄は**最後の1つだけ**。

報告ブロックが「裁定のための要約」なのに対し、共有欄は**後から追うための手がかり**。
役割が違うので**両方出す**（置き換えではない）。

```markdown
■ 共有欄（対象／YYYY-MM-DD HH:MM JST）

Notion 反映先
・ページ名 ID
  → 書いた節 / 状態
・（以下、書いた先すべて）

サマリー
・何をしたか（数行）
・未反映・未対応（なければ「なし」）
・commit
```

- **Notion に書いていないものを反映先に書かない。** 未反映なら状態欄に明記する。
- **リンクは実 URL かページ ID で書く。** ページ名だけにしない。

---

## 5. ディレクトリ

```
/
├ CLAUDE.md          このファイル
├ index.html         モックのエントリポイント（1台プレイ・file:// で開く）
├ lan.html / server.js / js/lan_client.js   LAN 対戦（権威サーバ型・node server.js）
├ .claude/skills/
│  └ session-end/SKILL.md   セッション終了手順（マスターは Notion 02_operations）
├ docs/              正本の向きは §1 の表のとおり（文書ごとに違う）
│  ├ CURRENT_SPEC.md   正本（repo）。実装済みのルール。Notion に写しを出す
│  ├ SESSION_STATE.md  正本（repo）。進捗・次にやること・未確定項目。Notion に写しを出す
│  ├ DECISION_LOG.md   写し。正本は Notion の DECISION_LOG
│  ├ REJECTED.md       旧企画の却下案索引の写し（v0.2 まで・参照のみ）
│  └ research/         旧企画から引き継いだ調査資料（参照のみ）
├ data/             中身は素の JSON。file:// から <script> で読むため拡張子は .js
│  ├ cards.js       カード定義 + ルールフラグ（数値・効果はすべてここ）
│  └ ai.js          CPU の評価関数の重み（ゲームのルールではないので cards.js とは分ける）
├ js/
│  ├ engine.js       純粋なゲームロジック（DOM 非依存）
│  ├ ai.js           評価関数ベースの CPU（engine の外側。filterStateFor 済みの state だけを見る）
│  └ view.js         描画のみ
├ tools/             計測用ハーネス（engine を使う。view には依存しない）
│  ├ harness.js      自動対戦の中身（AI の実装・1試合の進行・集計）
│  ├ simulate.js     harness の CLI。1回まわして要約を出す
│  ├ report.js       単独の集計を docs/research/ の Markdown に書き出す
│  ├ compare.js      AI を差し替えて並べた比較を docs/research/ に書き出す
│  └ compare_tokens.js  ルールフラグを差し替えた前後比較を docs/research/ に書き出す
└ package.json       ★ `"type"` を書かない（CommonJS）。file:// 対応のため §2.8
```

---

## 6. 検証のしかた

```
node js/engine.js          # engine のセルフテスト（非破壊性・決定性・視点フィルタ・ルール）
node js/ai.js              # CPU のセルフテスト（決定性・隠し情報を見ないこと・手の質）
```

ブラウザは **index.html をダブルクリック**して開く（file://）。サーバは要らない。
致命的エラーは画面上部の赤い枠に出る。真っ黒のままにはならない。

`node js/engine.js` と `node js/ai.js` が通らない変更は入れないこと。
★ **UI に関わる変更は、必ず file:// で開いて決着まで動かしてから完了とすること。**
Node のセルフテストだけでは file:// の読み込み失敗を捕まえきれない。

**未確定ルールはハードコードせず `data/cards.js` の `rules` に足してフラグ化する。**
仮ルール・正本の矛盾に対する解釈は、すべてここで切り替えられる状態を保つ。
