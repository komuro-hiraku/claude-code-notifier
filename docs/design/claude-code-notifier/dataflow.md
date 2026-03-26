# claude-code-notifier データフロー図

**作成日**: 2026-03-26
**関連アーキテクチャ**: [architecture.md](architecture.md)
**関連要件定義**: [requirements.md](../../spec/claude-code-notifier/requirements.md)

**【信頼性レベル凡例】**:
- 🔵 **青信号**: EARS要件定義書・設計文書・ユーザヒアリングを参考にした確実なフロー
- 🟡 **黄信号**: EARS要件定義書・設計文書・ユーザヒアリングから妥当な推測によるフロー
- 🔴 **赤信号**: EARS要件定義書・設計文書・ユーザヒアリングにない推測によるフロー

---

## システム全体のデータフロー 🔵

**信頼性**: 🔵 *要件定義・アーキテクチャ設計より*

```mermaid
flowchart TD
    Start[アプリ起動] --> Init[初期化]
    Init --> LoadSettings[設定読み込み<br/>SettingsManager]
    LoadSettings --> StartMonitor[ファイル監視開始<br/>FileMonitor]
    StartMonitor --> WaitEvent[FSEventsイベント待機]

    WaitEvent --> |ファイル変更検出| ReadLog[ログファイル読み込み]
    ReadLog --> Parse[ログエントリ解析<br/>StateDetector]
    Parse --> |パターンマッチ| Detect{状態検出?}

    Detect --> |回答待ち| NotifyWaiting[通知: 回答待ち<br/>NotificationManager]
    Detect --> |指示待ち| NotifyInstruction[通知: 指示待ち<br/>NotificationManager]
    Detect --> |タスク完了| NotifyComplete[通知: タスク完了<br/>NotificationManager]
    Detect --> |該当なし| WaitEvent

    NotifyWaiting --> Display[macOS通知センター表示]
    NotifyInstruction --> Display
    NotifyComplete --> Display
    Display --> WaitEvent

    MenuClick[メニューバークリック] --> ShowMenu[メニュー表示<br/>MenuBarManager]
    ShowMenu --> |設定| ShowSettings[設定画面表示]
    ShowMenu --> |終了| Quit[アプリ終了]
    ShowSettings --> SaveSettings[設定保存<br/>SettingsManager]
    SaveSettings --> StartMonitor
```

## 主要機能のデータフロー

### 機能1: ログファイル監視とイベント検出 🔵

**信頼性**: 🔵 *REQ-001, REQ-002, REQ-003, REQ-004・アーキテクチャ設計より*

**関連要件**: REQ-001, REQ-002, REQ-003, REQ-004

```mermaid
sequenceDiagram
    participant FM as FileMonitor<br/>(FSEvents)
    participant LF as Claude Code<br/>ログファイル
    participant SD as StateDetector
    participant NM as NotificationManager
    participant NC as macOS通知センター

    FM->>LF: ファイル監視開始
    Note over FM,LF: FSEventsストリーム作成
    LF-->>FM: ファイル変更イベント
    FM->>FM: 変更内容読み込み
    FM->>SD: 新規ログエントリを渡す
    SD->>SD: パターンマッチング実行
    alt 回答待ち状態を検出
        SD->>NM: 回答待ち通知要求
        NM->>NC: 通知送信
        NC-->>User: "Claude Code: 回答待ちです"
    else 指示待ち状態を検出
        SD->>NM: 指示待ち通知要求
        NM->>NC: 通知送信
        NC-->>User: "Claude Code: 次の指示待ちです"
    else タスク完了を検出
        SD->>NM: タスク完了通知要求
        NM->>NC: 通知送信
        NC-->>User: "Claude Code: タスク完了しました"
    end
```

**詳細ステップ**:
1. **FileMonitor**: FSEvents APIでログファイルのディレクトリを監視
2. **変更検出**: ファイルに変更があった場合、イベントが発火
3. **ログ読み込み**: 前回読み込み位置から新規エントリのみ読み込み
4. **StateDetector**: ログエントリを正規表現でパターンマッチング
5. **状態判定**: 回答待ち、指示待ち、タスク完了のいずれかを判定
6. **通知送信**: NotificationManagerが適切な通知を作成・送信
7. **ユーザー通知**: macOS通知センターに通知が表示される

### 機能2: 設定管理と永続化 🔵

**信頼性**: 🔵 *設計ヒアリング Q3・アーキテクチャ設計より*

**関連コンポーネント**: SettingsManager, UserDefaults

```mermaid
sequenceDiagram
    participant User
    participant MBM as MenuBarManager
    participant SV as SettingsView
    participant SM as SettingsManager
    participant UD as UserDefaults

    User->>MBM: メニューバーアイコンクリック
    MBM->>MBM: メニュー表示
    User->>MBM: "設定"を選択
    MBM->>SV: 設定画面を開く
    SV->>SM: 現在の設定を取得
    SM->>UD: UserDefaultsから読み込み
    UD-->>SM: 設定データ
    SM-->>SV: 設定データを返す
    SV-->>User: 設定画面表示

    User->>SV: 設定を変更
    User->>SV: "保存"をクリック
    SV->>SM: 新しい設定を保存
    SM->>UD: UserDefaultsに書き込み
    UD-->>SM: 保存完了
    SM-->>SV: 保存成功
    SV-->>User: "設定を保存しました"
```

### 機能3: 通知の重複抑制 🟡

**信頼性**: 🟡 *UX向上のため推測*

**関連コンポーネント**: NotificationManager

```mermaid
sequenceDiagram
    participant SD as StateDetector
    participant NM as NotificationManager
    participant Cache as 通知キャッシュ
    participant NC as macOS通知センター

    SD->>NM: 回答待ち通知要求（タスクID: A）
    NM->>Cache: 最近の通知を確認
    Cache-->>NM: タスクID: A は未通知
    NM->>Cache: タスクID: A を記録（タイムスタンプ付き）
    NM->>NC: 通知送信
    NC-->>User: 通知表示

    Note over SD,NM: 数秒後、同じタスクで再度検出

    SD->>NM: 回答待ち通知要求（タスクID: A）
    NM->>Cache: 最近の通知を確認
    Cache-->>NM: タスクID: A は5秒前に通知済み
    NM-->>SD: 通知スキップ（重複）
```

**備考**: この機能により、同一タスクの連続通知を防ぎ、ユーザーの邪魔にならないようにします。通知の有効期限は5分程度が妥当と推測。

## データ処理パターン

### 同期処理 🔵

**信頼性**: 🔵 *アーキテクチャ設計より*

以下の処理は同期的に実行されます：
- 設定の読み込み・保存（SettingsManager）
- パターンマッチング（StateDetector）
- メニュー表示（MenuBarManager）

### 非同期処理 🔵

**信頼性**: 🔵 *パフォーマンス要件・アーキテクチャ設計より*

以下の処理は非同期的に実行され、メインスレッドをブロックしません：
- ログファイルの読み込み（FileMonitor）
- FSEventsのイベント処理
- 通知の送信（NotificationManager）

```swift
// 非同期処理の例
DispatchQueue.global(qos: .utility).async {
    // ログファイル読み込み
    let logContent = readLogFile()
    DispatchQueue.main.async {
        // メインスレッドでUI更新
        updateUI(logContent)
    }
}
```

## エラーハンドリングフロー 🟡

**信頼性**: 🟡 *一般的なエラー処理パターンから推測*

```mermaid
flowchart TD
    Start[処理開始] --> Try{処理実行}

    Try -->|成功| Success[正常終了]
    Try -->|ログファイル不存在| Error1[エラー: ファイル未発見]
    Try -->|読み込み権限なし| Error2[エラー: アクセス拒否]
    Try -->|パース失敗| Error3[エラー: フォーマット不正]
    Try -->|通知権限なし| Error4[エラー: 通知権限なし]

    Error1 --> Log1[エラーログ出力]
    Error2 --> Log2[エラーログ出力]
    Error3 --> Log3[エラーログ出力]
    Error4 --> Log4[エラーログ出力]

    Log1 --> Notify1[ユーザーに警告通知]
    Log2 --> Notify2[権限要求ダイアログ]
    Log3 --> Retry[リトライまたはスキップ]
    Log4 --> Notify4[権限要求ダイアログ]

    Notify1 --> End[終了]
    Notify2 --> End
    Retry --> End
    Notify4 --> End
    Success --> End
```

### エラーの種類と対応

| エラー種別 | 原因 | 対応 |
|-----------|------|------|
| ログファイル未発見 | ファイルパスが不正 | ユーザーに警告、設定画面を開く |
| アクセス拒否 | ファイル読み込み権限なし | 権限要求ダイアログを表示 |
| フォーマット不正 | ログ形式が予期しない | エラーログを記録、該当エントリをスキップ |
| 通知権限なし | 通知権限が未許可 | 権限要求ダイアログを表示 |

## 状態管理フロー

### アプリケーション状態 🔵

**信頼性**: 🔵 *アーキテクチャ設計より*

```mermaid
stateDiagram-v2
    [*] --> 初期化中
    初期化中 --> 設定読み込み
    設定読み込み --> 監視準備完了
    監視準備完了 --> 監視中: 監視開始
    監視中 --> 監視中: ログ変更検出
    監視中 --> 通知送信中: 状態検出
    通知送信中 --> 監視中: 通知完了
    監視中 --> 監視停止: ユーザーが終了
    監視停止 --> 設定保存
    設定保存 --> [*]
```

### 通知状態 🟡

**信頼性**: 🟡 *UX向上のため推測*

各タスクの通知状態を管理し、重複通知を防ぎます。

```swift
// 通知状態の管理例
struct NotificationState {
    let taskId: String
    let state: ClaudeCodeState // 回答待ち/指示待ち/完了
    let timestamp: Date
}

class NotificationCache {
    private var recentNotifications: [NotificationState] = []

    func shouldNotify(taskId: String, state: ClaudeCodeState) -> Bool {
        // 5分以内に同じタスク・状態の通知があればスキップ
        let fiveMinutesAgo = Date().addingTimeInterval(-300)
        return !recentNotifications.contains {
            $0.taskId == taskId &&
            $0.state == state &&
            $0.timestamp > fiveMinutesAgo
        }
    }
}
```

## ログパース処理フロー 🟡

**信頼性**: 🟡 *REQ-001から妥当な推測*

**備考**: 実際のログフォーマットは調査が必要（prep.mdのタスク）

```mermaid
flowchart TD
    Start[新規ログエントリ] --> Split[行ごとに分割]
    Split --> Loop{各行を処理}
    Loop --> Match1{回答待ちパターン?}
    Match1 -->|Yes| Extract1[タスク情報抽出]
    Match1 -->|No| Match2{指示待ちパターン?}
    Match2 -->|Yes| Extract2[タスク情報抽出]
    Match2 -->|No| Match3{タスク完了パターン?}
    Match3 -->|Yes| Extract3[タスク情報抽出]
    Match3 -->|No| NextLine[次の行へ]

    Extract1 --> Notify[通知送信]
    Extract2 --> Notify
    Extract3 --> Notify
    Notify --> NextLine
    NextLine --> Loop
    Loop -->|全行完了| End[処理完了]
```

**想定パターン例**:
```swift
// パターンマッチングの例（実際のパターンは調査が必要）
let patterns = [
    "回答待ち": #"waiting for user input"#,
    "指示待ち": #"task completed, awaiting next instruction"#,
    "タスク完了": #"task finished successfully"#
]
```

## 関連文書

- **アーキテクチャ**: [architecture.md](architecture.md)
- **要件定義**: [requirements.md](../../spec/claude-code-notifier/requirements.md)
- **準備タスク**: [prep.md](../../spec/claude-code-notifier/prep.md)

## 信頼性レベルサマリー

- 🔵 青信号: 12件 (75%)
- 🟡 黄信号: 4件 (25%)
- 🔴 赤信号: 0件 (0%)

**品質評価**: ✅ **高品質** - 主要なデータフローは要件定義から確実に設計、推測は非本質的なUX改善のみ
