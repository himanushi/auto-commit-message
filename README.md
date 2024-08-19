# auto-commit-message

```
brew install jq
```

```
vi ~/.zshrc
```

```
# OpenAI APIキーを設定
export OPENAI_API_KEY="your_openai_api_key_here"

# GPT-4を使用してコミットメッセージを生成する関数
function generate_commit_message() {
  local diff_output
  diff_output=$(git diff --cached)

  # diff_outputをJSON用にエスケープ（制御文字も削除）
  diff_output=$(echo "$diff_output" | perl -pe 's/\\/\\\\/g; s/"/\\"/g; s/\n/\\n/g; s/\r/\\r/g; s/\t/\\t/g; s/\$\{file\}/\\\\\$\{file\}/g; s/\$\(([^)]*)\)/\\\\\$\($1\)/g; s/[\x00-\x1F]//g')

  # OpenAI APIにリクエストを送信してコミットメッセージを生成
  commit_message=$(curl -s https://api.openai.com/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d "{
      \"model\": \"gpt-4o-mini\",
      \"messages\": [
        {\"role\": \"system\", \"content\": \"日本語で、一行で収まるように commit message を作成しなさい。ダブルクォートなどの記号は不要です。\"},
        {\"role\": \"user\", \"content\": \"${diff_output}\"}
      ]
    }" | jq -r '.choices[0].message.content')

  # 生成されたメッセージを返す
  echo "$commit_message"
}

# tigでコミットメッセージを自動生成してコミットする関数
function commit_with_gpt() {
  # 生成されたコミットメッセージを取得してコミット
  commit_message=$(generate_commit_message)
  echo "$commit_message" | git commit -F -
}

# tigのエイリアスを上書き
alias c="commit_with_gpt"
```

```
source ~/.zshrc
```


## 使い方
```
git add *
c
```
