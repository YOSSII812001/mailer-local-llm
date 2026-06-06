# Detailed Design: Rabbit Mail

## 計画

- [x] プロジェクト名を `Rabbit Mail` に統一する。
- [x] 旧プロジェクト名の前提を除去する。
- [x] ブランド美学を `High-Brand` スタイルのメーラーとして定義する。
- [x] 白基調テーマ、上質なダークモード、横向きの兎シルエットを視覚核にする。
- [x] アーキテクチャを `Tauri 2 + Rust + Canvas` に移行する。
- [x] 旧Webアプリ構成ではなく、ローカルPCへインストールするネイティブデスクトップアプリとして定義する。
- [x] Sassuru-kokoro、Haikei、Kizukai、Auto-Reply Kanban、Self-Improvement LoopをRust中心の構成へ再配置する。
- [x] Action KanbanをCanvas上の流麗な作業面として再設計する。
- [x] Part 1からPart 4までを、最終プロジェクト identity と技術スタックに合わせて更新する。

## Next Steps

- `src-tauri` 配下のRustモジュール構成を、この設計に合わせて分割する。
- CanvasプロトタイプでAction Kanban、Thread Stream、Context Inspectorを検証する。
- ローカルLLMランタイム、暗号化ストレージ、メール接続の境界を実装計画へ落とす。
- デザイントークン、兎シルエットアイコン、ライト/ダークテーマをUI仕様へ展開する。

# Detailed Design - Part 1: Product Identity & Design Philosophy

## 1. Product Premise

Rabbit Mailは、ローカルPCで完結する高品位なAIメーラーである。
本製品は、メール処理を速くするだけの道具ではない。
本製品は、ユーザーの判断を静かに整える作業環境である。

Rabbit Mailは、ユーザーの代わりに暴走しない。
Rabbit Mailは、ユーザーの横で文脈を読み、必要な材料を整える。
AIは、送信、削除、転送などの重要操作を勝手に決めない。

体験の中心は、安心、品位、透明性である。
ユーザーは、AIを管理するのではなく、良い助手へ作法を教える。
この関係性を、Rabbit Mailは「Master Artisan's Workspace」として表現する。

## 2. Final Project Identity

| Item | Definition |
| --- | --- |
| Product Name | `Rabbit Mail` |
| Category | Local-first AI native desktop mailer |
| Brand Aesthetic | High-Brand |
| Design Language | Simple, Modern, Elegant |
| Core Icon | Side-view silhouette of a rabbit |
| Primary Experience | Canvas-based Master Artisan's Workspace |
| Core Runtime | Tauri 2 + Rust |
| Main Workspace | Canvas |
| Execution Model | Local PC Installation |

Rabbit Mailは、企業メールを扱う。
そのため、装飾よりも信頼を優先する。
一方で、事務的なだけのUIにはしない。
細部の所作、余白、文字、暗色面の質感で、日々使う道具としての品位を作る。

## 3. Brand Aesthetic

Rabbit Mailのブランド美学は `High-Brand mailer` である。
これは派手さを意味しない。
上質な素材、正確な余白、静かな反応、破綻しない情報密度を意味する。

ブランドの判断基準は次のとおりである。

- UIは、メール本文と判断材料を主役にする。
- 画面は、軽く、静かで、精密に動く。
- 色は、情報の意味を支えるために使う。
- 兎のモチーフは、かわいさではなく、静けさ、俊敏さ、繊細さを表す。
- AIの存在感は、控えめでよい。
- 信頼に関わる状態は、常に見える場所に置く。

## 4. Visual Identity

### 4.1 Rabbit Silhouette

中心アイコンは、横向きの兎のシルエットである。
形は最小限にする。
耳、背中、前脚、後脚の輪郭だけで、兎と分かることを目指す。

アイコン設計の方針:

- 輪郭は滑らかで、鋭すぎない。
- 目や表情は描き込まない。
- 正面顔やキャラクター表現は避ける。
- ロゴ、アプリアイコン、起動時のローディング、空状態に使う。
- Canvas上では、処理完了や静かな待機状態を示す小さなサインとして使う。

兎は、Rabbit Mailの「察する」性質を表す。
ただし、製品は動物キャラクターではない。
製品は、品位のある業務道具である。

### 4.2 White-Base Theme

ライトテーマは白を基調にする。
ただし、純白だけで画面を作らない。
紙のような温度、墨のような文字、薄い境界で深度を作る。

| Token | Color | Usage |
| --- | --- | --- |
| `surface.canvas` | `#FAFAF8` | Canvasの基底面 |
| `surface.panel` | `#FFFFFF` | 主要パネル、Inspector |
| `surface.soft` | `#F3F1EC` | 控えめな区切り面 |
| `text.primary` | `#1D1F22` | 本文、見出し |
| `text.secondary` | `#62686F` | メタ情報、補助説明 |
| `line.quiet` | `#E5E1D8` | 区切り線、入力枠 |
| `accent.rabbit` | `#2B4051` | 主要アクション、選択状態 |
| `accent.safe` | `#2F7D68` | 承認済み、ローカル処理 |
| `accent.attention` | `#B88A2A` | 確認、保留 |
| `accent.risk` | `#B85C4B` | 送信停止、高リスク |

白基調は、余白の品質で成立する。
境界線を増やしすぎない。
代わりに、整列、間隔、階層で構造を示す。

### 4.3 Sophisticated Dark Mode

ダークモードは、単なる反転色にしない。
長時間のメール処理に耐える暗色面として設計する。

| Token | Color | Usage |
| --- | --- | --- |
| `dark.canvas` | `#121417` | Canvasの基底面 |
| `dark.panel` | `#1A1D21` | 主要パネル |
| `dark.soft` | `#23272D` | 補助面、選択背景 |
| `dark.text.primary` | `#F2F0EA` | 本文、見出し |
| `dark.text.secondary` | `#A8AFB7` | メタ情報 |
| `dark.line` | `#30363D` | 区切り線 |
| `dark.accent.rabbit` | `#9DB6C8` | 主要アクション |
| `dark.accent.safe` | `#7BC7A4` | 承認済み |
| `dark.accent.attention` | `#D9B96E` | 確認 |
| `dark.accent.risk` | `#D88A7B` | 高リスク |

ダークモードでは、過度な青紫グラデーションを使わない。
Rabbit Mailは、AIらしい派手さではなく、道具としての落ち着きを優先する。

## 5. Typography and Spacing

Typographyは、メール本文の可読性を最優先にする。
日本語、英数字、日付、URL、署名が混ざっても行の流れが乱れないことを重視する。

| Role | Spec |
| --- | --- |
| UI Font | `Inter`, `Noto Sans JP`, system UI |
| Reading Font | `Noto Sans JP`, system UI |
| Base Size | 14pxから16px |
| Compact Metadata | 12pxから13px |
| Section Heading | 18pxから22px |
| Line Height | 1.5以上 |
| Letter Spacing | 0 |

余白は4pxグリッドを基準にする。
主要な間隔は8px、12px、16px、24px、32pxを使う。
Canvas上のカード間隔は広く取り、視線が詰まらないようにする。

## 6. Craftsman-Level Micro-Interactions

Rabbit Mailの動きは、職人の手元のように静かで正確にする。
動きは、驚かせるためではなく、理解を助けるために使う。

基本ルール:

- 状態変化は120msから180msのフェードで示す。
- Canvasカードの移動は、軽い慣性と吸着で整える。
- Kanbanの遷移は、線で流れを示してから列へ収める。
- 高リスク警告は点滅させない。
- 成功表示は短く、作業を邪魔しない。
- ローカル処理中は、処理名を表示する。

表示例:

```text
スレッドの未回答事項を確認中
返信案の前提を検証中
送信ルールと監査ログを照合中
```

## 7. Experience Principles

Rabbit Mailの体験は、次の原則で統一する。

1. ユーザーの主導権を守る。
2. 判断材料を返信案より先に出す。
3. 自動化は条件、根拠、監査IDと一緒に見せる。
4. AIの学習は、見える場所で承認し、いつでも戻せるようにする。
5. ローカル処理を信頼の中心に置く。
6. Canvasは見せ場ではなく、思考と処理を整える作業台にする。

# Detailed Design - Part 2: Native Desktop Architecture

## 1. Architecture Contract

Rabbit Mailは、Tauri 2で構築するネイティブデスクトップアプリである。
ブラウザ向けのWebアプリとして配布しない。
ユーザーはローカルPCへインストールして使う。

中核ロジック、高負荷処理、AI推論の制御、メール連携、監査、暗号化はRustで実装する。
UIはTauri 2のデスクトップシェル上で動く。
主作業面はCanvasで構築する。

この構成により、Rabbit Mailは次の性質を持つ。

- メール本文、下書き、要約、プロンプト、学習ルールはローカルPCに残る。
- AI推論はローカルハードウェアで実行する。
- 画面操作はネイティブアプリとして高速に反応する。
- 外部通信は、メール送受信、認証、明示的な更新確認に限定する。
- サーバー常駐型の業務アプリではなく、端末内で完結する作業環境として動く。

## 2. Layered Architecture

| Layer | Technology | Responsibility |
| --- | --- | --- |
| Desktop Shell | Tauri 2 | ウィンドウ、メニュー、権限、OS統合 |
| UI Renderer | Canvas + TypeScript UI | Master Artisan's Workspace、Inspector、設定 |
| IPC Boundary | Tauri Commands / Events | UIとRust処理の安全な接続 |
| Core Domain | Rust | メール、下書き、Kanban、学習、監査 |
| Orchestration | Rust async runtime | ローカルLLM、ルール判定、ジョブ管理 |
| Local Inference | Local LLM runtime | 要約、返信案、検証、学習候補抽出 |
| Storage | SQLite + field encryption | ローカルDB、監査、設定、検索用インデックス |
| OS Security | Credential Manager / Keychain / Secret Service | 鍵保護、資格情報保護 |

UIは美しく動く。
ただし、UIは意思決定の所有者ではない。
重要な状態遷移はRust側で検証する。

## 3. Rust Module Map

Rust側は、責務ごとに明確に分ける。
Canvasの見た目に引きずられず、ドメインの安全性を優先する。

| Module | Responsibility |
| --- | --- |
| `rabbit_mail` | IMAP、SMTP、Graph、Gmail APIとの接続 |
| `rabbit_core` | スレッド、メッセージ、下書き、状態遷移 |
| `rabbit_ai` | ローカルLLMの起動、推論、JSON検証 |
| `rabbit_rules` | 自動返信ルール、送信条件、停止条件 |
| `rabbit_kanban` | Action Kanbanの状態機械 |
| `rabbit_canvas` | Canvas表示用の投影データ生成 |
| `rabbit_audit` | 不変監査ログ、ハッシュチェーン |
| `rabbit_crypto` | フィールド暗号化、鍵ローテーション |
| `rabbit_storage` | SQLite、マイグレーション、検索インデックス |
| `rabbit_learning` | 学習候補、承認済み作法、適用範囲 |
| `rabbit_settings` | ローカル設定、テーマ、ハードウェア検出 |

モジュール間の依存は一方向に保つ。
UI都合の型を中核ロジックへ入れない。
Canvas用データは、Rust側の読み取り専用投影として作る。

## 4. IPC Design

Tauri Commandsは、UIからRustへ依頼する明示的な入口である。
Commandsは、重要操作ごとに入力検証と監査を持つ。

主要Commands:

| Command | Purpose |
| --- | --- |
| `mail_sync_accounts` | 設定済みアカウントからメールを同期する |
| `thread_open` | スレッド詳細を復号して取得する |
| `ai_analyze_thread` | スレッドを解析し、見立てを作る |
| `ai_create_draft` | 返信案と根拠を生成する |
| `draft_update` | 下書きを更新し、承認を取り消す |
| `draft_approve` | 下書きを承認する |
| `draft_send` | 承認済み下書きを送信する |
| `kanban_transition` | Action Kanbanの状態を遷移する |
| `lesson_candidate_create` | 編集差分から学習候補を作る |
| `lesson_approve` | 学習候補を承認済み作法にする |
| `audit_query` | 監査ログを検索する |

Eventsは、RustからUIへ状態変化を通知する。
本文や下書き全文はイベントに載せない。
UIは、必要なときだけCommandで詳細を取得する。

主要Events:

| Event | Purpose |
| --- | --- |
| `mail.synced` | 同期完了を通知する |
| `thread.updated` | スレッド状態の変化を通知する |
| `ai.progress` | ローカルAI処理の進行を通知する |
| `kanban.transitioned` | Kanban遷移を通知する |
| `draft.updated` | 下書き更新を通知する |
| `audit.appended` | 監査ログ追加を通知する |
| `local_model.status` | ローカルLLMの状態を通知する |

## 5. Local Inference Architecture

Rabbit Mailは、AI推論をローカルPCで実行する。
クラウドAI、共有GPUサーバー、外部推論APIは使わない。

Rustの `rabbit_ai` は、次の責務を持つ。

- 端末のCPU、GPU、メモリ、VRAMを検出する。
- タスクごとにモデルと量子化方式を選ぶ。
- ローカルLLMランタイムを起動、監視、停止する。
- プロンプトへ入れる文脈を最小化する。
- JSON出力をスキーマ検証する。
- 失敗時は安全側へ倒す。

モデル選択の目安:

| Task | Preferred Model Class | Notes |
| --- | --- | --- |
| Triage | small local model | 高速な分類を優先する |
| Summary | small to medium local model | スレッド長で切り替える |
| Reply Draft | medium local model | 文体と前提確認を重視する |
| Risk Validation | small to medium local model | ルール検証と併用する |
| Senior Review | larger local model | 契約、金額、法務、人事で使う |
| Lesson Extraction | medium local model | 編集差分から作法を抽出する |

ローカルAIの出力は、必ずRust側で検証する。
AI出力だけで送信判断を確定しない。

## 6. Storage and Encryption

Rabbit Mailの標準ストレージは、ローカルSQLiteである。
DBファイルはユーザー端末内に保存する。
可能な環境ではSQLCipherを使い、さらに重要フィールドをアプリ層で暗号化する。

暗号化対象:

- メール本文
- 件名
- 差出人、宛先、CC
- 添付メタデータ
- 要約
- 返信案
- プロンプト由来の文脈
- 学習ルール
- 監査対象の重要メタ情報

鍵設計:

- Key Encryption KeyはOSの安全な資格情報ストアで保護する。
- Data Encryption Keyは論理グループごとに作る。
- Data Encryption KeyはKey Encryption KeyでラップしてDBへ保存する。
- AES-256-GCMを基本にする。
- nonceは暗号化ごとにランダム生成する。
- Associated Dataにはテーブル名、レコードID、フィールド名、スキーマ版を入れる。

監査ログは改ざん検知を持つ。
各ログは前回ログのハッシュを含める。
重要操作は、監査ログを書けた場合だけ完了扱いにする。

## 7. Local PC Installation Model

Rabbit Mailは、ユーザーのPCへインストールする。
すべてのAI処理とデータ処理は、ユーザーのローカルハードウェアで動く。

インストールモデル:

| Item | Policy |
| --- | --- |
| Application | Tauri 2 native desktop application |
| Data Location | User profile配下のローカルアプリデータ |
| Model Location | ローカルモデルディレクトリ |
| Settings | ローカル設定ファイルと暗号化DB |
| Updates | ユーザー許可後に更新 |
| Telemetry | 既定で無効 |
| Cloud Dependency | メール送受信と認証を除き不要 |

外部通信の例外:

- IMAP、SMTP、Microsoft Graph、Gmail APIなどのメール通信。
- 企業が許可したOIDC、SSO、OAuth。
- ユーザーが明示的に許可した更新確認。

AI処理、要約、返信案、学習抽出、リスク検証は例外なくローカルで完結する。

## 8. Runtime Safety

Rabbit Mailの失敗原則は「送らない、隠さない、壊さない」である。

| Failure | Rust Behavior | UI Behavior |
| --- | --- | --- |
| DB書き込み失敗 | 状態遷移を中止する | 保存失敗を表示する |
| 監査ログ失敗 | 承認、送信、学習承認を止める | 保護操作を無効化する |
| LLM停止 | AI生成を止める | 手動閲覧と手動返信を維持する |
| JSON検証失敗 | 再試行し、失敗時はAttentionへ送る | 手動確認を促す |
| 送信失敗 | Sentにしない | 再試行と監査IDを表示する |
| 新着割り込み | 予定送信を止める | KanbanをAttentionへ移す |

# Detailed Design - Part 3: Core Feature Mapping

## 1. Feature Remapping Overview

既存の中核機能は、Tauri 2 + Rust + Canvas構成へ再配置する。
機能名は維持する。
実行責務はRustへ移す。
表現はCanvas上の作業面へ統合する。

| Feature | Rust Responsibility | Canvas Expression |
| --- | --- | --- |
| Sassuru-kokoro | 文脈解析、未回答事項、次操作の抽出 | Context Hint、Thread Stream、Intent Preview |
| Haikei | 相手、関係性、過去経緯、前提の整理 | Draft Studioの根拠パネル |
| Kizukai | リスク、添付、宛先、表現の配慮 | Attention marker、送信前ガード |
| Auto-Reply Kanban | ルール照合、状態遷移、監査 | Action Kanban lanes |
| Self-Improvement Loop | 編集差分、学習候補、承認済み作法 | Learning Ledger、Redline Teaching |

## 2. Sassuru-kokoro Engine

Sassuru-kokoro Engineは、ユーザーが次に見るべき情報を整える。
このEngineは、返信案を先に押し出さない。
まず、目的、前提、未回答事項、リスクを提示する。

処理手順:

| Step | Rust Process | Output |
| --- | --- | --- |
| 1 | メールとスレッドを正規化する | `message_context` |
| 2 | 差出人、宛先、時系列を整理する | `relationship_context` |
| 3 | 未回答事項と期限を抽出する | `open_questions` |
| 4 | 返信の必要性を判定する | `intent_preview` |
| 5 | UI用の短いヒントへ変換する | `context_hints` |

Canvasでは、ヒントを本文の上に重ねない。
Thread Cardの横、またはContext Inspectorに静かに表示する。

表示例:

```text
目的: 来週の打ち合わせ日程の確認
必要な判断: 参加可否、候補日時、同席者
注意点: 前回、金曜午後を避けたいと伝えています
```

## 3. Haikei

Haikeiは、返信案の背景を整える機能である。
AIは、文面だけを作らない。
AIは、なぜその文面になるのかを短く示す。

Haikeiが扱う情報:

- 相手との関係性。
- 前回のやり取り。
- 未解決の約束。
- 期限、金額、契約、添付の有無。
- 社内外の距離感。
- ユーザーが過去に選んだ表現。

Draft Studioでは、返信案の横にHaikeiを表示する。
根拠は短い単位に分ける。
根拠を押すと、該当メールや学習ルールへジャンプする。

## 4. Kizukai

Kizukaiは、失敗を防ぐための配慮である。
これは警告を増やす機能ではない。
必要なときだけ、送信前の判断を助ける。

検出対象:

- 添付への言及があるが、添付がない。
- 宛先に社外ドメインが含まれる。
- CCやBCCの構成が過去と大きく違う。
- 金額、契約、法務、人事に関わる表現がある。
- 過去の約束と矛盾する。
- 低信頼度のまま送信しようとしている。

Kizukaiは、送信を責める文言を使わない。
文言は、確認を促す形にする。

例:

```text
添付に触れていますが、ファイルが見つかりません。
送信前に確認しますか？
```

## 5. Local LLM Workflow

Rabbit MailのAI処理は、Rust Orchestratorが管理する。
各処理は監査ログと紐付く。
UIは、進行状況と結果だけを受け取る。

| Step | Name | Main Work |
| --- | --- | --- |
| 1 | Intake | メール、スレッド、添付メタ情報を正規化する |
| 2 | Risk Precheck | ルールベースで高リスク要素を先に検出する |
| 3 | Analysis | 意図、緊急度、未回答事項を抽出する |
| 4 | Model Routing | タスクと端末性能に応じてローカルモデルを選ぶ |
| 5 | Crafting | 要約、返信案、次操作を作る |
| 6 | Validation | JSON、根拠、矛盾、禁止条件を検証する |
| 7 | Confidence | 信頼度とリスク上限を計算する |
| 8 | Projection | Canvas表示用の軽量データへ変換する |

AI処理の結果は、平文のまま長期保存しない。
保存が必要な要約、下書き、ヒント、学習候補は暗号化する。

## 6. Confidence and Risk

信頼度は、AIの賢さを示す点数ではない。
信頼度は、ユーザーが確認量を決めるための作業リスク指標である。

| Band | Score | UI Treatment |
| --- | ---: | --- |
| High | 80-100 | 静かな承認候補として表示する |
| Medium | 50-79 | 確認したい点を添える |
| Low | 0-49 | 自動候補から外し、手動確認へ送る |

Hard Cap:

| Condition | Max Score |
| --- | ---: |
| 添付言及があるが添付がない | 65 |
| 社外宛てかつ機密語を含む | 50 |
| 金額、契約、法務、人事を含む | 60 |
| 必須情報が不足している | 70 |
| JSONスキーマ違反 | 40 |
| ルール検証とAI判定が矛盾する | 55 |

## 7. Action Kanban

Action Kanbanは、Canvas上に配置される流麗な作業面である。
これは標準的なリストUIではない。
メールスレッド、AI提案、承認済みルール、送信予定、注意事項が、流れとして見える。

Kanbanの列:

| Column | Meaning |
| --- | --- |
| Proposed | AIが候補を作った状態 |
| Needs Review | 人間確認が必要な状態 |
| Approved Rule | 事前承認済みルールに一致した状態 |
| Scheduled | 送信予定または実行待ち |
| Sent | 送信済み |
| Attention | 失敗、低信頼度、ルール不一致 |

カードは、メール1件ではなく判断単位を表す。
同じスレッドに属する複数メールは束ねる。
ユーザーは、今見るべき流れだけを追える。

カード内容:

- 相手名とドメイン。
- 件名の短縮表示。
- AIが見立てた目的。
- 次のアクション。
- 信頼度とリスク印。
- 送信予定時刻または期限。
- ローカル処理バッジ。
- 監査ID。

Canvas上の表現:

- カードは滑らかに列間を移動する。
- 遷移時は、短い流線で移動理由を示す。
- Attention列は常に見える場所へ固定できる。
- 低リスクのSentカードは自動で折りたためる。
- 同じ相手や案件は束として扱える。

## 8. Auto-Reply Rules

自動返信は、ユーザーが事前に許可した条件内だけで動く。
AIは、自由な送信判断を持たない。

自動返信の実行条件:

- ルールが有効である。
- 対象アカウント、相手、ドメイン、件名条件が一致する。
- 信頼度が指定値以上である。
- Hard Capが許容範囲内である。
- 添付、金額、契約、法務、人事などの停止条件に該当しない。
- 監査ログを書き込める。
- 送信直前に新着割り込みがない。

実行順:

1. Rustがルール候補を抽出する。
2. Risk Precheckが停止条件を確認する。
3. ローカルAIが返信案を作る。
4. Validationが根拠と禁止条件を検証する。
5. `Approved Rule` に一致する場合だけ予定送信へ進む。
6. 送信直前に監査ログを書く。
7. 送信成功後に `Sent` へ遷移する。
8. 失敗時は `Attention` へ遷移する。

監査ログを書けない場合、送信しない。
これはユーザーの安心を守るための必須条件である。

## 9. Self-Improvement Loop

Self-Improvement Loopは、ユーザーがRabbit Mailへ作法を教える仕組みである。
学習は自動保存しない。
AIは候補を作る。
ユーザーが承認したものだけを保存する。

学習タイミング:

- ユーザーが返信案を大きく編集した後。
- 送信前に文体や順序を直した後。
- AI案を破棄した後。
- Kanbanで保留、修正、差し戻しを選んだ後。

Redline Teaching:

- Draft Studioで編集差分を表示する。
- Rustが差分から学習候補を抽出する。
- 候補は `この相手だけ`、`このドメインだけ`、`この案件だけ`、`全体` の範囲を持つ。
- ユーザーが承認するまで、プロンプトへ適用しない。
- 承認済み作法はLearning Ledgerで停止、編集、削除できる。

Learning Ledgerの項目:

- 学習した作法。
- 適用範囲。
- 元になったメールまたは編集差分。
- 作成日時。
- 最終適用日時。
- 状態。
- 監査ID。

# Detailed Design - Part 4: Master Artisan's Workspace

## 1. Workspace Concept

Master Artisan's Workspaceは、Rabbit Mailの主画面である。
この画面は、標準的なメール一覧ではない。
Canvas上に、スレッド、返信案、Action Kanban、学習、監査を配置する。

ユーザーは、メールを行の集合として見るのではない。
ユーザーは、対応すべき流れとして見る。
Rabbit Mailは、処理の流れを静かに整え、ユーザーが判断しやすい形へ並べる。

## 2. Workspace Layout

基本構成:

```text
┌─────────────────────────────────────────────────────────────┐
│ Top Bar: account / search / local model / sync / theme       │
├──────────────┬──────────────────────────────┬───────────────┤
│ Left Rail    │ Canvas Workspace             │ Inspector     │
│              │                              │               │
│ Inbox        │ Thread Stream                │ Context       │
│ Kanban       │ Action Kanban                │ Haikei        │
│ Lessons      │ Draft Studio                 │ Kizukai       │
│ Audit        │ Learning Ledger              │ Audit Trail   │
│ Settings     │                              │               │
├──────────────┴──────────────────────────────┴───────────────┤
│ Assurance Bar: local-only / encryption / audit / model state  │
└─────────────────────────────────────────────────────────────┘
```

Left Railは狭く保つ。
Canvasは主役として広く取る。
Inspectorは、選択中のカードに応じて内容を切り替える。

## 3. Canvas Components

| Component | Purpose |
| --- | --- |
| Thread Stream | 重要スレッドを時系列と関係性で並べる |
| Action Kanban | 自動処理、承認、予定送信、注意を流れで見せる |
| Draft Studio | 返信案、編集、根拠、送信前確認を扱う |
| Context Hint | Sassuru-kokoroの見立てを短く表示する |
| Haikei Panel | 返信案の背景と根拠を表示する |
| Kizukai Guard | 送信前の配慮と停止条件を表示する |
| Learning Ledger | 承認済み作法と候補を管理する |
| Audit Timeline | 操作とAI処理の監査履歴を表示する |
| Local Model Monitor | ローカルAIの状態と端末負荷を表示する |

カードは、角丸を8px以下にする。
カード内の文字は詰めすぎない。
重要な数値や状態は、アイコン、位置、ラベルを組み合わせて示す。

## 4. Workspace Modes

Rabbit Mailは、作業の種類に合わせてCanvasの見え方を変える。
モードは画面全体を作り替えない。
同じCanvas上で、表示密度と焦点を変える。

| Mode | Purpose |
| --- | --- |
| Focus | 1つのスレッドと返信案に集中する |
| Flow | Action Kanbanで処理全体を見る |
| Review | 低信頼度、高リスク、承認待ちを確認する |
| Teach | 編集差分と学習候補を確認する |

Focus Modeでは、Thread Card、Draft Studio、Inspectorを中心に置く。
Flow Modeでは、Action Kanbanを広く表示する。
Review Modeでは、Attention列とKizukai Guardを優先する。
Teach Modeでは、Redline TeachingとLearning Ledgerを並べる。

## 5. Action Kanban Wireframe

Action Kanbanは、Canvas上の流れとして表現する。
列は固定の表ではなく、必要に応じて幅と密度を調整できる。

```text
Proposed        Needs Review      Approved Rule     Scheduled       Sent
┌─────────┐     ┌─────────┐       ┌─────────┐       ┌─────────┐     ┌──────┐
│ A社日程 │ --> │ 契約確認 │       │ 定例返信 │ -->   │ 10:30送信 │ --> │ 完了 │
└─────────┘     └─────────┘       └─────────┘       └─────────┘     └──────┘

Attention
┌──────────────┐
│ 添付確認あり │
└──────────────┘
```

カード遷移時は、なぜ動いたかを短く表示する。
例: `Rule matched`、`Needs attachment check`、`Audit appended`。

## 6. Draft Studio

Draft Studioは、返信作成の中心である。
単なるテキスト入力欄ではない。
文面、根拠、リスク、学習候補を同じ場所で扱う。

構成:

- 返信本文エディタ。
- AI提案の差分表示。
- Haikeiの根拠パネル。
- Kizukaiの送信前確認。
- 信頼度とHard Capの表示。
- 承認、予定送信、保留、破棄。
- Redline Teachingの候補表示。

送信ボタンは、承認状態と監査状態を確認してから有効にする。
高リスク時は、送信ボタンより確認内容を優先して見せる。

## 7. Context Inspector

Context Inspectorは、選択中のCanvas要素に応じて内容を変える。
ユーザーは、詳細を探しに行かない。
選択したものに必要な詳細が右側へ出る。

Thread Card選択時:

- 3行要約。
- 未回答事項。
- 期限。
- 関係者。
- 過去の約束。

Kanban Card選択時:

- 現在状態。
- 次に可能な操作。
- ルール一致理由。
- 信頼度。
- 監査ID。

Draft Studio選択時:

- 返信案の前提。
- 文体。
- リスク。
- 学習候補。
- 送信条件。

## 8. Assurance Bar

Assurance Barは、Rabbit Mailの信頼状態を常に示す。
これは設定画面の奥に隠さない。
日常画面の下部に、静かに表示する。

表示項目:

- `Local Only`: AI処理がローカルで完結している状態。
- `Encrypted`: ローカルDBと重要フィールドの暗号化状態。
- `Audit Ready`: 監査ログが書き込み可能な状態。
- `Model Ready`: ローカルLLMランタイムの状態。
- `Mail Sync`: メール同期の最終時刻。

異常がある場合は、該当操作だけを止める。
アプリ全体を不用意に止めない。

## 9. Keyboard and Pointer Interaction

Rabbit Mailは、マウスでもキーボードでも快適に使えるようにする。
Canvasであっても、業務アプリとしての効率を落とさない。

主要操作:

| Action | Input |
| --- | --- |
| Canvas pan | Drag |
| Canvas zoom | Wheel / pinch |
| Quick search | `Ctrl+K` |
| Focus selected thread | `Enter` |
| Move to next attention item | `J` |
| Move to previous attention item | `K` |
| Approve draft | `Ctrl+Enter` |
| Hold item | `H` |
| Teach from diff | `T` |
| Open audit | `A` |

ショートカットは、画面内で過度に説明しない。
必要な場所にツールチップとして出す。

## 10. Accessibility

Canvas中心のUIでも、アクセシビリティを犠牲にしない。

方針:

- すべてのカードはキーボードで選択できる。
- Canvas要素には論理順序を持たせる。
- 色だけで状態を示さない。
- コントラスト比を確保する。
- 動きが苦手なユーザー向けにアニメーション低減を用意する。
- 監査、送信、承認などの重要操作は読み上げ名を明確にする。

## 11. Acceptance Criteria

この詳細設計の完了条件は次のとおりである。

- 製品名がRabbit Mailに統一されている。
- High-Brand、Simple、Modern、Elegantの体験方針が定義されている。
- 横向き兎シルエット、白基調、上質なダークモードが定義されている。
- Tauri 2 + Rust + Canvasのネイティブデスクトップ構成が明記されている。
- 中核ロジック、高負荷処理、AI推論制御がRustへ移っている。
- 旧Webアプリスタックを前提にした構成が残っていない。
- ローカルPCインストールとローカルAI推論の境界が明記されている。
- Sassuru-kokoro、Haikei、KizukaiがRust処理とCanvas表現へ再配置されている。
- Action KanbanがCanvas上の流麗な作業面として定義されている。
- Self-Improvement LoopがLearning LedgerとRedline Teachingに接続されている。
- 監査、暗号化、失敗時の安全側停止が維持されている。

## 進捗

- `docs/detailed_design.md` をRabbit Mailの最終identityに合わせて全面更新した。
- 旧プロジェクト名の前提を外し、High-Brandなネイティブメーラーとして再定義した。
- Tauri 2 + Rust + CanvasのローカルPCインストール構成へ移行した。
- 旧API中心の設計を、Tauri Commands / EventsとRustモジュールへ置き換えた。
- Sassuru-kokoro、Haikei、Kizukai、Action Kanban、Self-Improvement Loopを新構成へ再配置した。
- Part 4として、Master Artisan's WorkspaceのCanvas構成、画面要素、操作、アクセシビリティを定義した。

## Next Tasks

- Rustモジュール構成とTauri Commandsの詳細インターフェースを実装計画へ展開する。
- Canvas上のAction KanbanとDraft StudioのUIプロトタイプを作る。
- ローカルLLMランタイムの起動、モデル選択、失敗時動作を検証する。
- 暗号化DB、監査ログ、メール送受信の最小実装を設計する。
- 兎シルエットのロゴ仕様とライト/ダークのデザイントークンを確定する。
