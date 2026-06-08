集中して取り組んだことの日記を付けるリポジトリ。
Issueを作成するとGithub ActionsがMarkdown形式に変換して出力してくれる設定をしています。

## Workflow の仕組み

`.github/workflows/issue-to-md.yml` により、Issueから自動的にMarkdownファイルを生成します。

### トリガー

```yaml
on:
  issues:
    types: [opened]
```

Issueが新規作成されたときに自動実行されます。

### 処理の流れ

#### 1. 日付の抽出

```bash
DATE=$(echo "${{ github.event.issue.title }}" | grep -Eo '[0-9]{4}-[0-9]{2}-[0-9]{2}')
```

Issueタイトルから `YYYY-MM-DD` 形式の日付を正規表現で抽出します。日付が含まれていない場合はエラーで終了します。

#### 2. ディレクトリ構造の作成

```bash
YEAR=$(echo $DATE | cut -d'-' -f1)
MONTH=$(echo $DATE | cut -d'-' -f2)
mkdir -p $YEAR/$MONTH
```

抽出した日付から年と月を取り出し、`YYYY/MM/` の階層構造でディレクトリを作成します。

#### 3. Markdownファイルの生成

```bash
FILE="$YEAR/$MONTH/$DATE.md"
echo "# $DATE" > "$FILE"
echo "" >> "$FILE"
echo "${{ github.event.issue.body }}" >> "$FILE"
```

Issue本文を `YYYY/MM/YYYY-MM-DD.md` ファイルとして保存します。ファイルの先頭には日付をタイトルとして付与します。

#### 4. 自動コミット

```yaml
- name: Commit and push
  uses: stefanzweifel/git-auto-commit-action@v5
  with:
    commit_message: "Add diary: $DATE"
```

生成されたファイルを自動的にコミット・プッシュします。

### 使い方

Issueのタイトルに必ず `YYYY-MM-DD` 形式の日付を含めてください。

例: `2026-06-08 の学習記録` または `2026-06-08`
