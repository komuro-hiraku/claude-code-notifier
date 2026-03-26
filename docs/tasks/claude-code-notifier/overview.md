# claude-code-notifier タスク概要

**作成日**: 2026-03-26
**プロジェクト期間**: 2026-03-26 - 2026-04-09（14日）
**推定工数**: 54時間
**総タスク数**: 12件

## 関連文書

- **要件定義書**: [📋 requirements.md](../../spec/claude-code-notifier/requirements.md)
- **設計文書**: [📐 architecture.md](../../design/claude-code-notifier/architecture.md)
- **データフロー図**: [🔄 dataflow.md](../../design/claude-code-notifier/dataflow.md)
- **準備タスク**: [🔧 prep.md](../../spec/claude-code-notifier/prep.md)

## フェーズ構成

| フェーズ | 期間 | 成果物 | タスク数 | 工数 | ファイル |
|---------|------|--------|----------|------|----------|
| Phase 1 | 1日 | ログ調査・プロジェクト作成 | 2 | 8h | [TASK-0001~0002](#phase-1-準備と調査) |
| Phase 2 | 0.5日 | Models・Utils実装 | 2 | 6h | [TASK-0003~0004](#phase-2-基盤実装) |
| Phase 3 | 4日 | コアコンポーネント実装 | 6 | 30h | [TASK-0005~0010](#phase-3-コアコンポーネント実装) |
| Phase 4 | 1.5日 | UI・統合テスト | 2 | 10h | [TASK-0011~0012](#phase-4-ui統合テスト) |

## タスク番号管理

**使用済みタスク番号**: TASK-0001 ~ TASK-0012
**次回開始番号**: TASK-0013

## 全体進捗

- [ ] Phase 1: 準備と調査
- [ ] Phase 2: 基盤実装
- [ ] Phase 3: コアコンポーネント実装
- [ ] Phase 4: UI・統合テスト

## マイルストーン

- **M1: 準備完了** (2026-03-27): ログ調査・Xcodeプロジェクト作成完了
- **M2: 基盤完成** (2026-03-28): Models・Utils実装完了
- **M3: コア完成** (2026-04-02): 6つのコアコンポーネント実装完了
- **M4: リリース準備完了** (2026-04-09): UI・統合テスト完了

---

## Phase 1: 準備と調査

**期間**: 1日（8時間）
**目標**: Claude Codeログ調査とXcodeプロジェクト作成
**成果物**: ログパターン定義、Xcodeプロジェクト

### タスク一覧

- [x] [TASK-0001: Claude Codeログファイル調査](TASK-0001.md) - 4h (DIRECT) 🔵
- [ ] [TASK-0002: Xcodeプロジェクト作成とプロジェクト設定](TASK-0002.md) - 4h (DIRECT) 🔵

### 依存関係

```
TASK-0001 → TASK-0007 (StateDetector)
TASK-0002 → TASK-0003, TASK-0004
```

---

## Phase 2: 基盤実装

**期間**: 0.5日（6時間）
**目標**: Models・Utilsの実装
**成果物**: ClaudeCodeState, NotificationContent, Constants, LogParser

### タスク一覧

- [ ] [TASK-0003: Modelsの実装](TASK-0003.md) - 4h (TDD) 🔵
- [ ] [TASK-0004: Utils/Constantsの実装](TASK-0004.md) - 2h (TDD) 🟡

### 依存関係

```
TASK-0002 → TASK-0003 → TASK-0007
TASK-0002 → TASK-0004 → TASK-0005, TASK-0006
```

---

## Phase 3: コアコンポーネント実装

**期間**: 4日（30時間）
**目標**: 6つのコアコンポーネントの実装
**成果物**: SettingsManager, FileMonitor, StateDetector, NotificationManager, MenuBarManager, AppDelegate

### タスク一覧

- [ ] [TASK-0005: SettingsManagerの実装](TASK-0005.md) - 4h (TDD) 🔵
- [ ] [TASK-0006: FileMonitorの実装](TASK-0006.md) - 6h (TDD) 🔵
- [ ] [TASK-0007: StateDetectorの実装](TASK-0007.md) - 6h (TDD) 🔵/🟡
- [ ] [TASK-0008: NotificationManagerの実装](TASK-0008.md) - 5h (TDD) 🔵
- [ ] [TASK-0009: MenuBarManagerの実装](TASK-0009.md) - 5h (TDD) 🔵
- [ ] [TASK-0010: AppDelegateの実装と統合](TASK-0010.md) - 4h (TDD) 🔵

### 依存関係

```
TASK-0004 → TASK-0005 → TASK-0006, TASK-0009
TASK-0005 → TASK-0006 → TASK-0007
TASK-0001, TASK-0003, TASK-0006 → TASK-0007 → TASK-0008
TASK-0003, TASK-0007 → TASK-0008 → TASK-0010
TASK-0005 → TASK-0009 → TASK-0010
TASK-0005, TASK-0006, TASK-0007, TASK-0008, TASK-0009 → TASK-0010 → TASK-0011
```

---

## Phase 4: UI・統合テスト

**期間**: 1.5日（10時間）
**目標**: UI実装と統合テスト
**成果物**: SettingsView、動作確認完了

### タスク一覧

- [ ] [TASK-0011: SettingsViewの実装](TASK-0011.md) - 5h (TDD) 🔵
- [ ] [TASK-0012: 統合テストと動作確認](TASK-0012.md) - 5h (DIRECT) 🔵

### 依存関係

```
TASK-0005, TASK-0009, TASK-0010 → TASK-0011 → TASK-0012
```

---

## 信頼性レベルサマリー

### 全タスク統計

- **総タスク数**: 12件
- 🔵 **青信号**: 11件 (91.7%)
- 🟡 **黄信号**: 1件 (8.3%)
- 🔴 **赤信号**: 0件 (0%)

### フェーズ別信頼性

| フェーズ | 🔵 青 | 🟡 黄 | 🔴 赤 | 合計 |
|---------|-------|-------|-------|------|
| Phase 1 | 2 | 0 | 0 | 2 |
| Phase 2 | 1 | 1 | 0 | 2 |
| Phase 3 | 6 | 0 | 0 | 6 |
| Phase 4 | 2 | 0 | 0 | 2 |

**品質評価**: ✅ **高品質** - ほぼすべてのタスクが要件定義・設計文書から確実に導出されている

## クリティカルパス

```
TASK-0001 (4h) → TASK-0007 (6h) → TASK-0008 (5h) → TASK-0010 (4h) → TASK-0011 (5h) → TASK-0012 (5h)
```

**クリティカルパス工数**: 29時間
**並行作業可能工数**: 25時間（TASK-0002, 0003, 0004, 0005, 0006, 0009）

## 次のステップ

タスクを実装するには:
- 全タスク順番に実装: `/tsumiki:kairo-implement`
- 特定タスクを実装: `/tsumiki:kairo-implement TASK-0001`
- Kairoループ（自動実装）: `/tsumiki:kairo-loop TASK-0001 TASK-0012`

## 実装手順

### TDDタスク（10件）
1. `/tsumiki:tdd-requirements TASK-XXXX` - 詳細要件定義
2. `/tsumiki:tdd-testcases` - テストケース作成
3. `/tsumiki:tdd-red` - テスト実装（失敗）
4. `/tsumiki:tdd-green` - 最小実装
5. `/tsumiki:tdd-refactor` - リファクタリング
6. `/tsumiki:tdd-verify-complete` - 品質確認

### DIRECTタスク（2件）
1. `/tsumiki:direct-setup` - 直接実装・設定
2. `/tsumiki:direct-verify` - 動作確認・品質確認

## 技術スタック

- **言語**: Swift
- **フレームワーク**: AppKit, UserNotifications, CoreServices (FSEvents)
- **ビルドシステム**: Xcode
- **設定管理**: UserDefaults
- **最小対応バージョン**: macOS 11.0 (Big Sur)

## 主要コンポーネント

1. **AppDelegate**: アプリライフサイクル管理
2. **FileMonitor**: FSEvents APIによるログファイル監視
3. **StateDetector**: パターンマッチングによる状態検出
4. **NotificationManager**: macOS通知センター連携
5. **MenuBarManager**: メニューバーUI管理
6. **SettingsManager**: UserDefaultsによる設定管理

## 注意事項

- **TASK-0001の実施推奨**: Claude Codeログファイルの調査は、TASK-0007（StateDetector）の実装品質に直接影響します。実際のログフォーマットを確認してから実装を進めてください。
- **ボトムアップ実装**: 基盤→コアコンポーネント→UIの順で実装することで、依存関係を明確に保ちます。
- **軽量タスク定義**: このタスク分割は軽量版です。実装時に詳細が不明な場合は、各TDD/DIRECTコマンドで詳細化されます。
