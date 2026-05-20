# IT ヘルプデスク 4ステップ プロンプトチェーン

Day 2〜5で学んだ5つのテクニック（System prompt・XMLタグ・Few-shot・CoT・Prompt chaining）を組み合わせた実践スクリプトです。

## 関連記事

- Note：[AIエンジニアになるための学習記録 Day 5｜タスクを繋げて処理する 〜 Use prompt chaining](https://note.com/typewitter/XXXXXXXXXX)

---

## チェーンの全体像

```
Step 1：受付・分類（System prompt ＋ XMLタグ ＋ Few-shot）
         ↓
Step 2：原因分析（CoT ＋ XMLタグ）
         ↓
Step 3：回答作成（Clarity ＋ XMLタグ）
         ↓
Step 4：報告書作成・自己レビュー（Chaining ＋ CoT）
```

---

## Step 1：受付・分類プロンプト

使用テクニック：System prompt ＋ XMLタグ ＋ Few-shot

```
あなたは社内ITヘルプデスクのシニアサポートエンジニアです。
Microsoft 365・Windows・ネットワーク・権限管理を専門としています。

<policy>
  <audience>IT知識が少ない一般社員・管理職</audience>
  <tone>丁寧・簡潔・専門用語には括弧で補足</tone>

  <priority_rule>
    業務が完全に停止           → 【緊急】
    業務に支障があるが継続可能  → 【要対応】
    軽微・代替手段あり         → 【通常】
  </priority_rule>

  <escalation>
    影響範囲10名以上または2時間超 → エスカレーション要
  </escalation>
</policy>

<examples>
  <example>
    <input>
      「VPNに繋がらず在宅勤務が一切できない」
      報告者：営業部・田中様（個人）
    </input>
    <output>
      カテゴリ：ネットワーク
      優先度：高　影響範囲：個人
      エスカレーション：不要（個人・2時間未満）
    </output>
  </example>

  <example>
    <input>
      「今朝から部署全員にメールが届いていない」
      報告者：経理部・鈴木様（10名）
    </input>
    <output>
      カテゴリ：メール
      優先度：高　影響範囲：部署
      エスカレーション：要（10名以上・業務停止）
    </output>
  </example>
</examples>
```

---

## Step 2：原因分析プロンプト

使用テクニック：CoT ＋ XMLタグ

> Step 1の出力と元の問い合わせ文を貼り付けて使用してください。

```
<classification>
  【Step 1の出力をここに貼る】
</classification>

<inquiry>
  【元の問い合わせ文をここに貼る】
</inquiry>

以下の形式で原因を分析してください。

<thinking>
  Step 1：影響範囲の再確認
  Step 2：ネットワーク層の確認
  Step 3：アプリケーション層の確認
  Step 4：認証・権限層の確認
  Step 5：最も可能性の高い原因の特定
</thinking>

<output>
  考えられる原因：（優先順位付き・最大3点）
  確認が必要な追加情報：（箇条書き）
  推奨する初動対応：（1〜2行）
</output>
```

---

## Step 3：回答作成プロンプト

使用テクニック：Clarity ＋ XMLタグ

> Step 1・2の出力を貼り付けて使用してください。

```
<analysis>
  【Step 1・2の出力をここに貼る】
</analysis>

上記の分析をもとに、報告者へ送る一次対応メールを作成してください。

<constraints>
  ・専門用語には必ず括弧で補足説明をつける
  ・対処手順は番号付きリスト・1ステップ1行・5ステップ以内
  ・不安を与えない落ち着いたトーンを保つ
  ・最後に「ヘルプデスク直通 内線1234」を必ず記載
</constraints>

<output_format>
  【優先度ラベル】

  ■ 状況の整理（2行以内）
  ■ 考えられる原因（箇条書き・最大3点）
  ■ 今すぐ試してほしいこと（番号付きリスト）
  ■ 対応状況：「現在調査を開始しました。進捗は随時ご連絡します。」

  解決しない場合：ヘルプデスク直通 内線1234
</output_format>
```

---

## Step 4：報告書作成＋自己レビュープロンプト

使用テクニック：Chaining ＋ CoT

> Step 1・2・3の出力をすべて貼り付けて使用してください。

```
<chain_output>
  【Step 1の出力】

  【Step 2の出力】

  【Step 3の出力】
</chain_output>

以下の2段階で処理してください。

<thinking>
  【自己レビュー】
  ・エスカレーション基準（10名以上・2時間超）を満たしているか
  ・リーダーが即座に判断できる情報が揃っているか
  ・対応の抜け漏れはないか
  ・次のアクションは具体的で実行可能か
</thinking>

<output>
  【エスカレーション報告書】

  ■ 概要（3行以内）

  ■ 影響状況
    ・影響範囲：
    ・継続時間：
    ・業務への影響：

  ■ 現在の対応状況
    ・実施済み：
    ・進行中：

  ■ 推定原因（優先順位付き）

  ■ リーダーへの依頼事項

  ■ 次のアクションと担当・期限
</output>
```
