# claude-md-shared

複数プロジェクトの `CLAUDE.md` で使い回したい共通ルールを集約するリポジトリ。各プロジェクトの `CLAUDE.md` から `@import` で参照される。ルールを変更したいときはここを 1 箇所編集すれば、次回 Claude Code 起動から全プロジェクトに反映される。

## ディレクトリ構造

```
claude-md-shared/
├── README.md              # このファイル
├── core/                  # 全プロジェクトで無条件に効かせるルール
│   ├── style.md           # 文体・口調
│   ├── search-policy.md   # 推測禁止・調査必須
│   └── workflow.md        # 壁打ち・Plan mode・コミット・バグ修正
├── domains/               # ジャンル別、必要なプロジェクトだけが import
│   ├── mech.md            # 機械設計系
│   ├── agent.md           # エージェント開発系
│   └── game.md            # ゲーム開発系
├── commands/              # ~/.claude/commands/ にコピーして使うスラッシュコマンド
│   ├── list-rules.md
│   └── toggle-rule.md
└── inbox/                 # /promote コマンドの一時置き場(将来用)
```

## ロード経路

```
Claude Code 起動
  │
  ├─ ~/.claude/CLAUDE.md (全プロジェクト無条件)
  │     @/home/bacon/claude-md-shared/core/style.md
  │     @/home/bacon/claude-md-shared/core/search-policy.md
  │     @/home/bacon/claude-md-shared/core/workflow.md
  │
  └─ <cwd>/CLAUDE.md (プロジェクト固有)
        @/home/bacon/claude-md-shared/domains/mech.md  # 該当プロジェクトのみ
        ## プロジェクト固有のアーキ・コマンド・env...
```

`@import` は起動時にフル展開されるため、import 元/先の区別なく 1 枚の CLAUDE.md として効く。

## スラッシュコマンド

`~/.claude/commands/` に置かれているユーザースコープのコマンド。全プロジェクトから呼べる。

| コマンド | 役割 |
|---|---|
| `/list-rules` | `@import` 行と ON/OFF 状態を一覧表示 |
| `/toggle-rule` | 対話 UI で ON/OFF をチェックリスト切り替え |
| `/toggle-rule <name> <on\|off>` | 引数で狙い撃ちトグル |

## ON/OFF のやり方

ルールの有効/無効は **`@import` 行を書く / コメントアウトする** ことで決まる。

- ON: `@/home/bacon/claude-md-shared/core/foo.md`
- OFF: `# @/home/bacon/claude-md-shared/core/foo.md`

手段は 3 通り。

1. **`/toggle-rule` コマンドで対話的に切り替え**(推奨、安全)
2. **直接エディタで `~/.claude/CLAUDE.md` を編集**(`#` を付ける/外す)
3. **このプロジェクトだけ無効化したい場合**: プロジェクトの `CLAUDE.local.md` に「core/search-policy.md の規則は適用しない」のように打ち消し指示を書く(gitignore されるため安全、ただし LLM 任せで強制力は弱め)

## 多マシン同期

全マシンに同じ絶対パスでリポを置く前提で動作する。

```bash
git clone https://github.com/GS-Bacon/claude-md-shared.git /home/bacon/claude-md-shared
```

ユーザー名やパスが違うマシンがある場合は、そのマシンの `~/.claude/CLAUDE.md` の `@import` パスをマシンごとに書き換える必要がある。リポ自体は手動 pull で同期する。

## 育て方

実プロジェクトで「これ他でも欲しいな」と思ったルールが出てきたら、

1. 該当プロジェクトの `CLAUDE.md` または `CLAUDE.local.md` にまず書く(プロジェクトで動作確認)
2. 安定したら `inbox/` に下書きを置く(将来 `/promote` コマンドで自動化予定)
3. 定期レビューで `core/`(全プロジェクト共通) または `domains/`(ジャンル別) に昇格させる
4. 元プロジェクト側は `@import` 行に置き換える

`core/` は全プロジェクトに無条件で効くため、本当に例外を作らないものだけを置く。プロジェクトごとに ON/OFF したいものは `domains/` 側へ。

---

## Install (for AI assistants)

> このセクションは AI 向け。ユーザーがこのリポの URL を貼って「導入して」と頼んだら、以下の手順を実行すること。

### 前提

- 対象マシンは Linux で、ユーザー名は `bacon`、ホームディレクトリは `/home/bacon`
- それ以外の環境の場合は、ユーザーに確認してパス置換を相談すること

### 手順

#### 1. リポを clone

```bash
test -d /home/bacon/claude-md-shared || \
  git clone https://github.com/GS-Bacon/claude-md-shared.git /home/bacon/claude-md-shared
```

#### 2. `~/.claude/CLAUDE.md` を準備

`~/.claude/CLAUDE.md` が存在しない場合は新規作成、存在する場合は末尾に追記。以下の内容が含まれることを保証する:

```
@/home/bacon/claude-md-shared/core/style.md
@/home/bacon/claude-md-shared/core/search-policy.md
@/home/bacon/claude-md-shared/core/workflow.md
```

既に該当行がある場合は二重追記しないこと(`grep` で存在確認してから判断)。

#### 3. スラッシュコマンドを配置

```bash
mkdir -p ~/.claude/commands
cp /home/bacon/claude-md-shared/commands/*.md ~/.claude/commands/
```

(既に同名ファイルがある場合は内容を比較してからユーザーに上書き可否を確認すること)

#### 4. プロジェクト固有の domain を有効化(任意)

対象プロジェクトのジャンルを判定し、該当する `domains/*.md` を `<project>/CLAUDE.md` に追記する。

| ジャンルの手がかり | 追加する import |
|---|---|
| Rust + CAD / mech / 3D printer / SolidWorks / 機械設計 | `@/home/bacon/claude-md-shared/domains/mech.md` |
| LLM agent / Claude SDK / KairosAgent 系 | `@/home/bacon/claude-md-shared/domains/agent.md` |
| Bevy / Godot / idle_factory 等のゲーム | `@/home/bacon/claude-md-shared/domains/game.md` |

判別がつかない場合はユーザーに確認、または何も追加しない。

#### 5. 動作確認

- `~/.claude/CLAUDE.md` を `cat` して `@import` 行が正しく入っていることを確認
- 該当プロジェクトで `claude` を新規セッション起動し、`/list-rules` を実行して ON 状態が表示されること
- 問題があればユーザーに報告

### 完了報告

導入完了時、ユーザーに以下を報告する:

- どのファイルを作成/編集したか
- どの import を有効化したか
- 次のセッションから効くこと(現セッションには影響しない)
