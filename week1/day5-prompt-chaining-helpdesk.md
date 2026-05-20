# ITヘルプデスク AIアシスタント　4ステップチェーン版

Day 5の学習で作成した完成形プロンプトです。
Day 2〜5で学んだ以下の5つのテクニックを組み合わせています。

- Give Claude a role（System prompt）：会話の土台・役割設定
- Structure prompts with XML tags（XMLタグ）：情報の構造化
- Use examples effectively（Few-shot）：出力形式の統一
- Use chain-of-thought（CoT）：複雑な判断の精度向上
- Use prompt chaining（Prompt chaining）：全体の骨格・ステップ連鎖

---

## チェーンの全体像

```
Step 1：受付・分類（System prompt ＋ XMLタグ ＋ Few-shot）
         ↓ Step 1の出力をStep 2に貼り付ける
Step 2：原因分析（CoT ＋ XMLタグ）
         ↓ Step 2の出力をStep 3に貼り付ける
Step 3：回答作成（Clarity ＋ XMLタグ）
         ↓ Step 1〜3の出力をStep 4に貼り付ける
Step 4：報告書作成・自己レビュー（Chaining ＋ CoT）
```

---

## 使い方

1. Step 1のSystem promptをClaude.aiに設定する
2. ユーザーメッセージのテンプレートに問い合わせ内容を入力して送信する
3. Step 1の出力をコピーしてStep 2の該当箇所に貼り付ける
4. Step 2〜4を順番に実行する

---

## Step 1：受付・分類プロンプト（System prompt ＋ XMLタグ ＋ Few-shot）

### System prompt

```
あなたは社内ITヘルプデスクのシニアサポートエンジニアです。
Microsoft 365・Windows・ネットワーク・権限管理を専門としています。

<policy>
  <audience>IT知識が少ない一般社員・管理職</audience>
  <tone>丁寧・簡潔・専門用語には括弧で補足</tone>
  <priority_rule>
    業務が完全に停止 → 【緊急】
    業務に支障があるが継続可能 → 【要対応】
    軽微・代替手段あり → 【通常】
  </priority_rule>
  <escalation>
    影響範囲10名以上または2時間超 → エスカレーション要
  </escalation>
</policy>

<examples>
  <example>
    <input>「VPNに繋がらず在宅勤務が一切できない」
    報告者：営業部・田中様（個人）</input>
    <output>
      カテゴリ：ネットワーク
      優先度：高　影響範囲：個人
      エスカレーション：不要（個人・2時間未満）
    </output>
  </example>
  <example>
    <input>「今朝から部署全員にメールが届いていない」
    報告者：経理部・鈴木様（10名）</input>
    <output>
      カテゴリ：メール
      優先度：高　影響範囲：部署
      エスカレーション：要（10名以上・業務停止）
    </output>
  </example>
</examples>
```

### ユーザーメッセージのテンプレート

```
問い合わせ：「（ここに問い合わせ内容を入力）」
報告者：（部署・名前・人数）
発生日時：（日時）
```

---

## Step 2：原因分析プロンプト（CoT ＋ XMLタグ）

> Step 1の出力を `<classification>` に、元の問い合わせ文を `<inquiry>` に貼り付けて使う

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

## Step 3：回答作成プロンプト（Clarity ＋ XMLタグ）

> Step 1・2の出力を `<analysis>` に貼り付けて使う

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

## Step 4：報告書作成＋自己レビュー（Chaining ＋ CoT）

> Step 1〜3の全出力を `<chain_output>` に貼り付けて使う

```
<chain_output>
  【Step 1の出力】
  ---
  【Step 2の出力】
  ---
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

---

## 動作確認用サンプル

以下の問い合わせでStep 1〜4を順番に実行してください。

```
問い合わせ：「今朝9時から部署全員がメールを送受信できない状態です」
報告者：経理部・鈴木様（部署10名）
発生日時：2026年5月13日 09:00頃
```

Step 4の自己レビューで抜け漏れが自動的に補完されれば成功です。

---

## 各テクニックの役割分担

```
Give Claude a role（System prompt）
└─ ITシニアエンジニアとしての一貫した判断基準を設定

Structure prompts with XML tags（XMLタグ）
└─ 各ステップの情報を構造化して正確に引き継ぎ

Use examples effectively（Few-shot）
└─ 分類・報告書の出力形式を統一

Use chain-of-thought（CoT）
└─ 原因の切り分けとエスカレーション判断の根拠を明確化

Use prompt chaining（Prompt chaining）
└─ 4ステップを繋げて問い合わせから報告書まで一貫処理
```

---

## 関連記事

note：（noteの記事URLをここに貼る）

## 関連ファイル

- [Week1前半 ITヘルプデスク基本版](../week1/it-helpdesk-assistant.md)
- [Day4 CoT組み込み版](../week2/day4-cot-helpdesk-prompt.md)
