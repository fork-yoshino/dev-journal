# GitHub Copilot / Claude Code のコミットAI署名設定

> VS Code 1.118では、VS Code内蔵のGit機能でコミットする場合に、Copilotのチャット上で作成されたコードやエージェントが書いたコードがコミット対象に含まれていれば、コミットメッセージに`Co-authored-by: Copilot <copilot@github.com>`を追加する設定がデフォルトで有効になった。

参考: [VS Code 1.118 リリースノート（gihyo.jp）](https://gihyo.jp/article/2026/05/vscode-1.118)

## 実際のコミットメッセージの構造

たとえば「ログインフォームを追加した」というコミットなら、このような形になります。

```text
feat: ログインフォームを追加

メールアドレスとパスワードによる認証フォームを実装。
バリデーションとエラーハンドリングも追加。

Co-authored-by: Copilot <copilot@github.com>
```

## 無効化する設定方法

`settings.json`（ユーザー全体）またはプロジェクトの`.vscode/settings.json`に以下を記述することで、この自動付与を無効にできます。

```json
{
  "git.addAICoAuthor": "off"
}
```

## 設定値は3段階

`git.addAICoAuthor`には3つの値があります。

- `chatAndAgent` — Copilot ChatとエージェントモードでのAI生成コードのコミット時にのみ追加（`1.118`以降のデフォルト）
- `all` — インライン補完（Tab補完）を含む**すべてのAI関与コード**のコミット時に追加
- `off` — 機能を完全に無効化。`Co-authored-by`の署名行を追加しない

---

# Claude Codeでのコミットメッセージの署名設定

Claude Codeでも、`git commit`を依頼すると、コミットメッセージの末尾に以下のような署名がデフォルトで付与されます。

```text
feat: ログインフォームを追加

メールアドレスとパスワードによる認証フォームを実装。
バリデーションとエラーハンドリングも追加。

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

付与されるのは、Gitが共著者として認識する`Co-Authored-By: Claude ...`の署名行と、「Claude Codeで生成されました」と示す案内文の2つです。

## 設定方法

`~/.claude/settings.json`（ユーザー全体）またはプロジェクトの`.claude/settings.json`に`attribution`を記述します。`commit`（コミットメッセージ用）と`pr`（プルリクエスト本文用）にそれぞれ好きな文字列を設定でき、空文字列`""`を指定するとその場所への署名付与をオフにできます。

無効化する例:

```json
{
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

カスタム文言にする例:

```json
{
  "attribution": {
    "commit": "🤖 Generated with Claude Code",
    "pr": ""
  }
}
```

> 以前使用されていた`includeCoAuthoredBy`の設定は非推奨となりました。同等の設定は`attribution`で実現できます。

## GitHub Copilotとの違い

| 観点 | GitHub Copilot (VS Code) | Claude Code |
| --- | --- | --- |
| 設定キー | `git.addAICoAuthor` | `attribution`（`includeCoAuthoredBy`は非推奨） |
| 設定ファイル | VS Codeの`settings.json` | `~/.claude/settings.json` |
| 設定値 | 3段階 (`off` / `chatAndAgent` / `all`) | commit / pr 別に任意の文字列（`""`で無効） |
| 署名の文言 | `Copilot <copilot@github.com>` | `Claude <noreply@anthropic.com>` |
| 追加文言 | `Co-authored-by`の署名行のみ | `Co-Authored-By`の署名行 + 「Claude Codeで生成」の案内文 |

参考: [Claude Code 設定リファレンス](https://code.claude.com/docs/ja/settings)