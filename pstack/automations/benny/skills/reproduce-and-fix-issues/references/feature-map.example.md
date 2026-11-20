# 機能マップの例（Feature-map example）

Benny が再現する可能性のある、ユーザーから見えるすべての機能をマップに書く。アプリを操作する前に、関係する節を読む。このマップはユーザーの視点で書く。内部の仕組みや現在のコードの道筋は、ここに固定して書かず、実行時に調べる。

このファイルを `.cursor/automations/benny/` の外、たとえば `.cursor/benny/feature-map.md` にコピーし、`control.feature_map_path` をそのコピーに設定する。パックを更新したときに、コピーが上書きされてはならない。

## 機能ごとのテンプレート（Per-feature template）

### `<feature name>`

`<one-line user-visible purpose>`

#### ユーザーがたどり着く方法（How a user gets there）

- クリックの道筋: `<screen> -> <menu, tab, or panel> -> <control>`
- キーボードショートカット: `<shortcut or none>`

#### 操作アダプターによる操作（How the control adapter drives it）

- `<adapter action>` に `<inputs>` を渡すと、`<visible result>` になるはずである。
- リセット: `<how the adapter returns to a fresh state>`。

#### 安定したセレクター（Stable selectors）

- `<role and accessible name>`
- `<ARIA relationship>`
- `<data-component or purpose-named data attribute>`

生成された CSS や StyleX のクラス、動的なハッシュ、子要素のインデックス、壊れやすい DOM 上の位置は決して使わない。

#### 操作する状態（States to exercise）

- 既定、ホバー、focus-visible、アクティブ、無効
- 読み込み中、空、エラー
- 選択中、開いている、展開
- `<relevant feature-specific variants>`

該当しない状態には、そのことを書いておく。

#### 前提条件と準備（Preconditions and setup）

- 認証: `<account state>`
- データ: `<fixture>`
- 権限: `<role>`
- フラグ: `<flag or none>`
- サービス: `<required availability>`

#### 証拠と照合（Evidence and cross-check）

- スクリーンショット: `<app identity, feature, and discriminating state>`
- 動画: `<entry path, interaction, and final state>`
- 照合: `<read-only state or value that confirms the UI>`

#### 注意点（Gotchas）

- `<known dead end or wrong surface>`
- `<safe environment translation>`

## 架空の例（Fictional example）

これらの機能は、架空のタスク管理アプリのものである。例であって、Benny に必須の機能ではない。

### サインイン（Sign in）

ユーザーがタスク管理アプリに入れるようにする。

#### ユーザーがたどり着く方法（How a user gets there）

- アプリを開き、`Sign in` を選ぶ。ショートカットは無い。

#### 操作アダプターによる操作（How the control adapter drives it）

- `open_app`、`click Sign in`、`fill credentials`、`click Continue` で、アイテムの一覧が開くはずである。
- サインアウトして使い捨てのセッションを消すことで、リセットする。

#### 安定したセレクター（Stable selectors）

- ボタン `Sign in`、テキストボックス `Email` と `Password`、`data-component="sign-in-form"`

#### 操作する状態（States to exercise）

- 既定、focus-visible、送信中、無効、読み込み中、エラー

#### 前提条件と準備（Preconditions and setup）

- 使い捨てのアカウントと、利用できる認証サービス

#### 証拠と照合（Evidence and cross-check）

- ランディングページからアイテムの一覧までを録画する。セッションの状態を読み取り専用で確かめる。

#### 注意点（Gotchas）

- マーケティング用のページは、対象とは違う画面である。認証サービスが無いことは、進められない理由になる。

### アイテムの一覧と詳細（Item list and detail）

ユーザーがアイテムを一覧で見て、1 つを開けるようにする。

#### ユーザーがたどり着く方法（How a user gets there）

- `Items` タブを開き、行を 1 つ選ぶ。

#### 操作アダプターによる操作（How the control adapter drives it）

- `select_tab Items` と `click <fixture item>` で、そのアイテムの詳細が開くはずである。
- 詳細を閉じて選択を解除することで、リセットする。

#### 安定したセレクター（Stable selectors）

- `Items` という名前のタブとリスト、フィクスチャの名前が付いた行、`data-component="item-detail"`

#### 操作する状態（States to exercise）

- 読み込み中、空、エラー、選択中、開いている、展開

#### 前提条件と準備（Preconditions and setup）

- 名前の付いたフィクスチャのアイテム、読み取り権限、利用できるアイテムのサービス

#### 証拠と照合（Evidence and cross-check）

- 選択と、それに合う詳細のタイトルを写す。選択中のアイテムの ID を確かめる。

#### 注意点（Gotchas）

- 検索結果は似て見えることがあるが、別の道筋を通る。

### アイテムのエディター（Item editor）

ユーザーがアイテムを作成・編集できるようにする。

#### ユーザーがたどり着く方法（How a user gets there）

- 詳細から `Edit` を選ぶか、一覧から `New item` を選ぶ。

#### 操作アダプターによる操作（How the control adapter drives it）

- `click Edit`、`fill <field>`、`click Save` で、詳細が更新されるはずである。
- フィクスチャを元に戻すことで、リセットする。

#### 安定したセレクター（Stable selectors）

- ボタン `Edit`、`New item`、`Save`、フォーム `Item editor`、ラベルと結びついたフィールド

#### 操作する状態（States to exercise）

- 既定、focus-visible、未保存の変更あり、検証中、無効、保存中、エラー、成功

#### 前提条件と準備（Preconditions and setup）

- 編集できるフィクスチャ、書き込み権限、利用できる保存のサービス

#### 証拠と照合（Evidence and cross-check）

- フィールドの変更から、更新された詳細までを写す。保存されたアイテムの値を読み取り専用で確かめる。

#### 注意点（Gotchas）

- フォームの状態を注入しない。読み取り専用の詳細のフィールドは、エディターではない。

### 設定（Settings）

ユーザーが個人の設定を変えられるようにする。

#### ユーザーがたどり着く方法（How a user gets there）

- プロフィールのメニューを開き、`Settings` を選ぶ。

#### 操作アダプターによる操作（How the control adapter drives it）

- `open_menu Profile`、`click Settings`、`toggle <preference>` で、コントロールが更新されるはずである。
- 設定を最初の値に戻すことで、リセットする。

#### 安定したセレクター（Stable selectors）

- ボタン `Profile`、メニュー項目 `Settings`、領域 `Settings`、目的を表す名前の設定項目の属性

#### 操作する状態（States to exercise）

- 閉じている、開いている、選択中、focus-visible、無効、読み込み中、エラー

#### 前提条件と準備（Preconditions and setup）

- サインイン済みのテスト用アカウント、既知の設定値、利用できる設定のサービス

#### 証拠と照合（Evidence and cross-check）

- メニューの道筋と、コントロールの最終状態を写す。設定値を読み取り専用で確かめる。

#### 注意点（Gotchas）

- OS の設定は別の画面である。

## 完全性のチェックリスト（Completeness checklist）

- 再現しうる、ユーザーから見えるすべての機能に節がある。
- すべての節が、ユーザーの道筋、アダプターのアクション、リセットを挙げている。
- セレクターは、ロール、名前、ARIA、安定したコンポーネントのマーカー、目的を表す名前の属性のどれかを使っている。
- 生成されたクラスや DOM 上の位置を使うセレクターが無い。
- 関係する操作中の状態、読み込み中、空、エラー、選択中、展開の各状態を扱っている。
- 認証、フィクスチャ、権限、フラグ、サービスが明示されている。
- スクリーンショット、動画、裏づけとなる照合の要件が明示されている。
- 対象と違う画面、行き止まり、安全な環境の読み替えが挙げられている。
- 実装の詳細は、実行時に調べるものとして残している。
