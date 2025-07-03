# Claude CodeとActivityWatchをMCP連携で活用する方法

## 📋 はじめに

このガイドでは、Claude Code（Anthropic公式のAI開発アシスタント）とActivityWatch（オープンソースの時間追跡ツール）をMCP（Model Context Protocol）で連携させ、日々の作業パターンを分析・改善する方法を解説します。

### 必要なもの
- Windows環境（ActivityWatchを実行）
- WSL2環境（Claude Codeを実行）
- ActivityWatch（Windows版）
- Claude Code
- Node.js（WSL内）

## 🔧 セットアップ手順

### 1. ActivityWatch MCPサーバーのインストール

```bash
# リポジトリをクローン
cd ~/repos
git clone https://github.com/8bitgentleman/activitywatch-mcp-server.git
cd activitywatch-mcp-server

# 依存関係をインストール
npm install

# ビルド
npm run build
```

### 2. Windows側のActivityWatch設定変更

WSLからWindows上のActivityWatchにアクセスできるようにするため、設定ファイルを編集します。

1. WSLターミナルで以下のパスにアクセス：
```bash
/mnt/c/Users/[ユーザー名]/AppData/Local/activitywatch/activitywatch/aw-server/aw-server.toml
```

2. 設定を以下のように変更：
```toml
[server]
host = "0.0.0.0"  # localhostから変更
port = "5600"
```

3. ActivityWatchを再起動

### 3. WSLからWindowsへの接続確認

```bash
# WindowsホストのIPアドレスを取得
ip route | grep default | awk '{print $3}'
# 例: 172.28.192.1

# 接続テスト
curl http://172.28.192.1:5600/api/0/info
```

### 4. MCPサーバーのソースコード修正

ActivityWatch MCPサーバーが環境変数を使用するように修正：

```typescript
// 各ファイル（bucketList.ts, query.ts, rawEvents.ts, getSettings.ts）で
const AW_API_BASE = process.env.AW_API_BASE_URL || "http://localhost:5600/api/0";
```

修正後、再ビルド：
```bash
npm run build
```

### 5. Claude Codeの設定

`.claude.json`にActivityWatch MCPサーバーの設定を追加：

```json
{
  "mcpServers": {
    "activitywatch": {
      "command": "node",
      "args": [
        "/home/chinchilla/repos/activitywatch-mcp-server/dist/index.js"
      ],
      "env": {
        "AW_API_BASE_URL": "http://172.28.192.1:5600/api/0"
      }
    }
  }
}
```

### 6. Claude Codeを再起動

設定完了後、Claude Codeを再起動して新しい設定を読み込みます。

## 🎯 使用方法

### 基本的なデータ取得

```javascript
// バケット一覧を取得
mcp__activitywatch__activitywatch_list_buckets

// 過去7日間のアプリ使用時間を分析
mcp__activitywatch__activitywatch_run_query
- timeperiods: ["2025-06-23/2025-06-29"]
- query: ["window_events = query_bucket('aw-watcher-window_LAPTOP-VGN5N4NQ'); events_by_app = merge_events_by_keys(window_events, ['app']); RETURN = sort_by_duration(events_by_app);"]
```

## 📊 実際の分析結果と得られる洞察

### 1. アプリケーション使用時間の可視化

私の実際のデータ分析結果：
- **VS Code**: 42.1時間/週（全体の60%）
- **Chrome**: 13.8時間/週（全体の20%）
- **Slack**: 5.1時間/週（全体の7%）

### 2. 作業パターンの問題点発見

分析により以下の問題が明らかに：

#### 🚨 深夜作業の常態化
- **週18.7時間**も深夜（0-3時）に作業
- 睡眠不足による生産性低下のリスク

#### 🔄 過度なコンテキストスイッチ
- **週1,473回**のアプリ切り替え
- VS Code⇔Chrome間だけで900回以上
- 平均6分ごとにアプリを切り替えている

#### ⏱️ 短い集中時間
- VS Codeでの平均連続作業時間：**6.1分**
- 深い思考が必要なプログラミングには不十分

### 3. 具体的な改善提案

データに基づいた改善策：

1. **ポモドーロテクニックの導入**
   - 25分集中→5分休憩のサイクル
   - 集中時間を現在の6分から25分へ

2. **作業時間の最適化**
   - 22時以降の作業を制限
   - 朝5-8時を「ディープワーク」時間に

3. **コンテキストスイッチの削減**
   - マルチモニター環境の構築
   - ブラウザタブの事前準備

## 💡 MCP連携のメリット

### 1. リアルタイムデータアクセス
- Claude Code内から直接ActivityWatchのデータを取得
- 複雑なAPIコールを簡単なコマンドで実行

### 2. 高度な分析の自動化
- AIによる作業パターンの分析
- 個人に最適化された改善提案

### 3. 継続的な改善
- 定期的な分析レポートの生成
- 改善効果の追跡と検証

## 🔍 トラブルシューティング

### 接続エラーが発生する場合

1. **Windowsファイアウォールの確認**
   - ポート5600が開放されているか確認

2. **ActivityWatchの起動確認**
   - タスクトレイにアイコンが表示されているか
   - http://localhost:5600 にアクセスできるか

3. **環境変数の確認**
   ```bash
   # MCPサーバープロセスの環境変数を確認
   ps aux | grep activitywatch-mcp-server
   cat /proc/[PID]/environ | tr '\0' '\n' | grep AW_API
   ```

## 📝 まとめ

Claude CodeとActivityWatchのMCP連携により、以下が実現できます：

1. **客観的なデータに基づく作業分析**
2. **AIによる問題点の自動検出**
3. **個人に最適化された改善提案**
4. **継続的な生産性向上**

この連携により、私は自分の作業パターンの問題点（深夜作業、頻繁なアプリ切り替え）を発見し、具体的な改善策を立てることができました。

ぜひ皆さんも試してみて、自分の作業効率を向上させてください！

---

*作成日: 2025年6月29日*
*著者: Claude Code + ActivityWatch ユーザー*