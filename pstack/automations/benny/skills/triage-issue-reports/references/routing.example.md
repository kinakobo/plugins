# 振り分けマップの例（Routing map example）

このファイルを `.cursor/automations/benny/` の外（たとえば `.cursor/benny/routing.md`）にコピーし、すべてのプレースホルダーを置き換える。`routing.map_path` にはコピーしたファイルを指定する。パックを更新しても、このコピーを上書きしてはならない。

トリアージのスキルは、このファイルをデータとして扱う。振り分けるには、報告か原因の追跡から得た証拠が必要である。キーワードの一致だけでは足りない。

```yaml
routes:
  - name: "billing-example"
    match:
      product_areas:
        - "billing-area-placeholder"
      code_paths:
        - "billing-code-path-placeholder"
      error_signatures:
        - "billing-error-placeholder"
    destination:
      slack_channel: "billing-channel-placeholder"
      tracker_team: "billing-team-placeholder"
    owners:
      - "billing-owner-placeholder"
    allow_feature_owner_ping: false

  - name: "desktop-example"
    match:
      product_areas:
        - "desktop-area-placeholder"
      code_paths:
        - "desktop-code-path-placeholder"
      error_signatures:
        - "desktop-error-placeholder"
    destination:
      slack_channel: "desktop-channel-placeholder"
      tracker_team: "desktop-team-placeholder"
    owners:
      - "desktop-owner-placeholder"
    allow_feature_owner_ping: false

fallback:
  destination: ""
  owners: []
  allow_feature_owner_ping: false

ping_policy:
  default: "off"
  allow:
    - "configured-feature-owner"
    - "confirmed-regression-author"
  deny:
    - "broad-on-call-group"
    - "unverified-owner"
```

## 規則（Rules）

- どこにも一致しない報告をすべて引き受けるチームがない限り、`fallback.destination` は空にしておく。
- 変わりにくい製品の領域、コードパス、エラーのシグネチャを使う。
- 公開するコピーに非公開のデータを含めない。
- 公開する例に、ユーザーやチャンネルの ID をそのまま貼らない。
- 機能の担当者へのメンションは、対象のチームが同意するまで無効にしておく。
- 振り分け直しでは、報告者にどこへ行けばよいかを伝える。Automation は決してクロスポストしない。
