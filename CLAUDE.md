- You must read AGENTS.md and strictly follow its instructions.
- コミットメッセージは日本語で書くこと（AGENTS.mdの英語ルールよりこちらを優先）
- コミットはレイヤーごとに分割すること（例: スキーマ変更、バックエンド（Go）、フロントエンドの3つに分ける）
- push前にlintを実行して問題がないことを確認すること（Go: `make lint`、フロントエンド: `cd app/application && npm run lint`）

## ブランチ運用ルール
- プランを承認して実装に入る際、現在のブランチが `develop` や作業内容と無関係なブランチの場合は、そのタスク専用のブランチを `develop` から切ること（例: `feature/xxx`, `fix/xxx`）
- タスク専用ブランチで作業中に、そのブランチの目的と無関係な変更を行おうとした場合は、ユーザーに警告し、別ブランチで作業するよう提案すること