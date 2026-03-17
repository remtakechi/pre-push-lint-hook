# PHP Git Hooks / PHP Gitフック

Global Git hook for PHP/Laravel projects — a `pre-push` hook.

---

## 日本語

### フック一覧

| フック | タイミング | 対象ファイル |
|---|---|---|
| `pre-push` | プッシュ直前 | デフォルトは各ツールの設定ファイルに従ってプロジェクト全体 (`LINT_NEW=1` でプッシュ対象コミットのみに絞れる) |

### 実行されるツール

| ツール | pre-push | プロジェクトにインストール必要？ |
|---|---|---|
| Laravel Pint | デフォルトはプロジェクト全体 (`pint.json` の設定に従う。`LINT_NEW=1` でプッシュ対象のみ) | ✅ `vendor/bin/pint` |
| PHP_CodeSniffer | デフォルトはプロジェクト全体 (`phpcs.xml` の設定に従う。`LINT_NEW=1` でプッシュ対象のみ) | ✅ `vendor/bin/phpcs` |
| PHPStan / Larastan | 常にプロジェクト全体 (`phpstan.neon` の設定に従う) | ✅ `vendor/bin/phpstan` |

インストールされていないツールは自動的にスキップされます。

> **Laravel Pint — 除外について**
> デフォルトモードではPintが `pint.json` の設定をそのまま使用するため、除外設定はすべて `pint.json` で管理してください。

### インストール

**Step 1 — フックファイルを移動する**

```sh
mkdir -p "$HOME/.githooks"
mv /path/to/pre-push "$HOME/.githooks/pre-push"
```

**Step 2 — 実行権限を付与する**

```sh
chmod +x "$HOME/.githooks/pre-push"
```

**Step 3 — Gitに場所を教える**

```sh
git config --global core.hooksPath "$HOME/.githooks"
```

この設定でマシン上のすべてのリポジトリで使用できるようになります。

### インストール確認

```sh
# $HOME/.githooks と表示されれば成功
git config --global core.hooksPath

# x (実行可能) パーミッションが付いていることを確認する
ls -la "$HOME/.githooks/"
```

### テストコマンド

```sh
# pre-push のLintチェックだけを実行する — 実際にはプッシュしない
# 現在のブランチで、プッシュ前のコミットでpre-pushのLintチェックをテストできます。ブランチにコミットが1つ以上必要です。
# Runs only the lint checks — no actual push happens
# Simulates the pre-push lint check on the current branch for commits that arent pushed. Requires at least one commit.
BRANCH=$(git branch --show-current)
REMOTE_SHA=$(git rev-parse --verify "origin/$BRANCH" 2>/dev/null || echo 0000000000000000000000000000000000000000)
echo "refs/heads/$BRANCH $(git rev-parse HEAD) refs/heads/$BRANCH $REMOTE_SHA" | "$HOME/.githooks/pre-push" origin x
```

### 出力例

**すべてパス:**
```
[GITHOOK] Running pre-push checks (full project)

→ Laravel Pint                 ✔
→ PHP_CodeSniffer              -  (not installed)
→ PHPStan / Larastan           ✔

✔ All checks passed
```

**失敗あり:**
```
[GITHOOK] Running pre-push checks (full project)

→ Laravel Pint                 ✗
  ... pint errors ...
  ↳ Laravel Pint failed
→ PHP_CodeSniffer              -  (not installed)
→ PHPStan / Larastan           ✔

✖ Pre-push checks failed
```

### 既存のフックがある場合

`core.hooksPath` を設定すると、Git は `.git/hooks/` を完全に無視します。
このフックはプロジェクト固有の `.git/hooks/pre-push` を検出し、チェック完了後に実行します。

### Lintチェックをスキップする

`LINT=0` を使用するとLintチェックをすべてスキップできます。
プロジェクト固有の `.git/hooks/pre-push` は引き続き実行されます。

```sh
LINT=0 git push
```

ターミナルには以下の警告が表示されます:
```
[GITHOOK] ⚠️  Global pre-push lint checks skipped (LINT=0)
```

### デフォルトのLint対象

デフォルトでは各ツールをファイル指定なしで実行します。各ツールが自身の設定ファイル (`pint.json` / `phpcs.xml` / `phpstan.neon`) に従ってスキャン対象・除外対象を決定します。プロジェクトでそのツールを直接実行した場合と同じ結果になります。

### プッシュ対象コミットのみLintする

`LINT_NEW=1` を使用するとプッシュするコミットに含まれるファイルのみを対象にします。
プッシュ範囲を検出できない場合や対象PHPファイルがない場合はデフォルトモード (プロジェクト全体) にフォールバックします。

```sh
LINT_NEW=1 git push
```

### アンインストール

```sh
git config --global --unset core.hooksPath
```

### 動作環境

| 環境 | サポート状況 |
|---|---|
| macOS | ✅ 動作確認済み |
| WSL / WSL2 | 🔵 理論上動作可能 (未検証) |
| ネイティブWindows (Git Bash) | ⚠️ 未検証。CMDおよびPowerShellは非対応 |

> **ネイティブWindowsユーザーへの注意**
> このファイルをWindows上で編集した場合、改行コードがCRLFに変換されスクリプトが動作しなくなります。編集後に `dos2unix "$HOME/.githooks/pre-push"` を実行してください。

---

## English

### Hooks

| Hook | When it runs | Files scanned |
|---|---|---|
| `pre-push` | Before each push | Full project by default, using each tool's own config (`LINT_NEW=1` to scope to pushed commits only) |

### What it runs

| Tool | pre-push | Requires install? |
|---|---|---|
| Laravel Pint | Full project by default, respecting `pint.json` (`LINT_NEW=1` for pushed commits only) | ✅ `vendor/bin/pint` |
| PHP_CodeSniffer | Full project by default, respecting `phpcs.xml` (`LINT_NEW=1` for pushed commits only) | ✅ `vendor/bin/phpcs` |
| PHPStan / Larastan | Full project always, respecting `phpstan.neon` | ✅ `vendor/bin/phpstan` |

Tools that are not installed are automatically skipped — no errors.

### Install

**Step 1 — Move the hook file**

```sh
mkdir -p "$HOME/.githooks"
mv /path/to/pre-push "$HOME/.githooks/pre-push"
```

**Step 2 — Make it executable**

```sh
chmod +x "$HOME/.githooks/pre-push"
```

**Step 3 — Tell Git where to find it**

```sh
git config --global core.hooksPath "$HOME/.githooks"
```

This applies to every repo on your machine — you only need to do this once.

**Step 4 — Native Windows users only (not needed on WSL/WSL2)**

If you edited the file on native Windows, run:

```sh
dos2unix "$HOME/.githooks/pre-push"
```

### Verify install

```sh
# Should print the path to $HOME/.githooks
git config --global core.hooksPath

# Should show the file with x (executable) permissions
ls -la "$HOME/.githooks/"
```

### Test without pushing

```sh
# Runs only the lint checks — no actual push happens
# Simulates the commits that would be pushed. Requires at least one commit.
BRANCH=$(git branch --show-current)
REMOTE_SHA=$(git rev-parse --verify "origin/$BRANCH" 2>/dev/null || echo 0000000000000000000000000000000000000000)
echo "refs/heads/$BRANCH $(git rev-parse HEAD) refs/heads/$BRANCH $REMOTE_SHA" | "$HOME/.githooks/pre-push" origin x
```

### Example output

**All passing:**
```
[GITHOOK] Running pre-push checks (full project)

→ Laravel Pint                 ✔
→ PHP_CodeSniffer              -  (not installed)
→ PHPStan / Larastan           ✔

✔ All checks passed
```

**With failures:**
```
[GITHOOK] Running pre-push checks (full project)

→ Laravel Pint                 ✗
  ... pint errors ...
  ↳ Laravel Pint failed
→ PHP_CodeSniffer              -  (not installed)
→ PHPStan / Larastan           ✔

✖ Pre-push checks failed
```

### If you already have a pre-push hook

Setting `core.hooksPath` causes Git to **stop looking** in `.git/hooks/` entirely.
This hook detects any existing `.git/hooks/pre-push` in the project and runs it after the checks complete.

### Skip lint checks

Use `LINT=0` to skip all lint checks. The project-level `.git/hooks/pre-push` still runs.

```sh
LINT=0 git push
```

The following warning will be printed:
```
[GITHOOK] ⚠️  Global pre-push lint checks skipped (LINT=0)
```

### Default lint scope

By default each tool is run with no file arguments — identical to running it directly in the project. Each tool uses its own config (`pint.json`, `phpcs.xml`, `phpstan.neon`) to determine what to scan and what to exclude.

### Lint only pushed commits

Use `LINT_NEW=1` to scope lint checks to only the files in the commits being pushed.
If the push range cannot be determined, or no PHP files changed, it falls back to default mode (full project).

```sh
LINT_NEW=1 git push
```

### Uninstall

```sh
git config --global --unset core.hooksPath
```

### Supported environments

| Environment | Status |
|---|---|
| macOS | ✅ Verified |
| WSL / WSL2 | 🔵 Should work in theory (runs as a native Linux environment — untested) |
| Native Windows (Git Bash) | ⚠️ Untested. CMD and PowerShell are not supported |

> **Native Windows users**
> Editing this file on Windows may silently convert line endings to CRLF, breaking the script. Run `dos2unix "$HOME/.githooks/pre-push"` after editing.
