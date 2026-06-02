---
description: claude-md-shared の @import ルールを一覧して ON/OFF 状態を表示する
---

# /list-rules

`claude-md-shared` リポから `@import` されているルールを一覧し、ON/OFF 状態(コメントアウトの有無)を表示する。

## 実行手順

1. `~/.claude/CLAUDE.md` を Read する(無ければユーザースコープ側はスキップ)
2. 現在の作業ディレクトリの `CLAUDE.md` を Read する(無ければプロジェクトスコープ側はスキップ)
3. それぞれのファイルから `@/home/bacon/claude-md-shared/` を含む行を抽出する
4. 行頭が `@` で始まれば ON、行頭が `#` で始まれば OFF と判定する
5. 以下の形式で結果を表示する:

```
## User scope (~/.claude/CLAUDE.md)
| File                  | Status |
|-----------------------|--------|
| core/style.md         | ON     |
| core/search-policy.md | ON     |
| core/workflow.md      | ON     |

## Project scope (<cwd>/CLAUDE.md)
| File              | Status |
|-------------------|--------|
| domains/mech.md   | ON     |
```

## 注意

- 表示は読み取り専用。ファイルは書き換えない
- ファイルが存在しないスコープは「(該当ファイルなし)」と表示する
- `@import` 行が1つも無い場合は「(import なし)」と表示する
