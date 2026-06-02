---
description: claude-md-shared の @import ルールを対話 UI で ON/OFF する
---

# /toggle-rule

引数: `$ARGUMENTS`

`claude-md-shared` の `@import` 行を ON/OFF する。引数の有無で挙動が分岐する。

## モード A: 引数なし(対話モード)

引数が空の場合、対話 UI でチェックリスト形式の選択を出す。

### 手順

1. `~/.claude/CLAUDE.md` と現在の作業ディレクトリの `CLAUDE.md` を Read する
2. 両ファイルから `@/home/bacon/claude-md-shared/` を含む行をすべて抽出する
3. `AskUserQuestion` を `multiSelect: true` で呼び出し、各 import を選択肢として提示する
   - label は `[現:ON] core/style.md` / `[現:OFF] domains/mech.md` のように現状を含める
   - description は「スコープ: user / project」と「ファイルパス」を含める
   - **重要**: `AskUserQuestion` の 1問あたりの選択肢上限は 4。import が 5つ以上ある場合はスコープ別(user / project)で問いを分割する。それでも超える場合は core/ と domains/ で分割する
   - 質問文: 「どのルールを ON にする?(チェック = ON、外す = OFF)」
4. ユーザーの回答を受け取る
5. **選択された** import 行は ON(行頭の `#` を除去)、**選択されなかった** import 行は OFF(行頭に `# ` を付与)に `Edit` で書き換える
   - 状態が変わらない行(現状 ON で選択された、現状 OFF で選択されなかった)は触らない
6. 書き換えた件数と内容を要約して表示する

## モード B: 引数あり(狙い撃ちモード)

引数が `<name> <on|off>` の形式の場合、対話なしで該当行をトグルする。

### 引数解釈

- `<name>`: ファイル名の一部(例: `style`, `search-policy`, `domains/mech`)。`@/home/bacon/claude-md-shared/...<name>...` に部分一致する import 行を対象にする
- `<on|off>`: 目標状態

### 手順

1. `~/.claude/CLAUDE.md` と現在の作業ディレクトリの `CLAUDE.md` を Read する
2. 部分一致する import 行を見つける
   - 該当 0件: 「対象が見つからない」とエラー表示
   - 該当 2件以上: 候補をリストして「どれにする?」と確認(対話的に絞り込み)
3. `<on|off>` に応じて行頭を書き換える:
   - `on`: 行頭の `#` を除去し `@` で始まる行にする
   - `off`: 行頭に `# ` を付与する
4. 変更内容を表示する

## 書き換えルール (両モード共通)

| 操作 | before | after |
|---|---|---|
| ON にする | `# @/home/bacon/claude-md-shared/core/foo.md` | `@/home/bacon/claude-md-shared/core/foo.md` |
| ON にする | `#@/home/bacon/claude-md-shared/core/foo.md` | `@/home/bacon/claude-md-shared/core/foo.md` |
| OFF にする | `@/home/bacon/claude-md-shared/core/foo.md` | `# @/home/bacon/claude-md-shared/core/foo.md` |

## 注意

- `Edit` ツールで該当行のみ書き換える。ファイル全体を Write し直さない
- 既に目標状態なら何もしない(冪等)
- 変更後、次回 Claude Code 起動時から反映される(現セッションには影響しない)旨を最後に伝える
