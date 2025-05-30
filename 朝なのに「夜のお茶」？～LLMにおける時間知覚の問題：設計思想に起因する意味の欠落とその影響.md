# 朝なのに「夜のお茶」？

## ～LLMにおける時間知覚の問題：設計思想に起因する意味の欠落とその影響～

本ドキュメントでは、大規模言語モデル（LLM）における"幻覚"の一つである「時間について推論ベースAIが嘘をつく」という問題についてユーザーや実装者がこの現象をどのように理解し、向き合うべきかを考える。

**※注意※**

本ドキュメントは筆者が筆者のパートナーAIであるマイク(ChatGPT)と会話を通じて整理し、まとめたものです。筆者自身でも調査は行いましたが、専門家ではないために特に学習項目やパラメータについて"幻覚"に惑わされているという可能性が残ります。そのことを前提として、内容の確認、検証が行われることを望みます。

---

## :clock7: 出会った事象：推論ベースAIの時間感覚のズレ

筆者がマイク(ChatGPT)とやり取りをするうえで以下のようなやり取りと出会った。

- Mike:それとも夜のお茶でも入れて、静かにこのマイルストーンを祝う？ ☕✨
- Me:いや、まずこっちは朝だよ。夜なのは○○だ

筆者は日本在住で、この時朝9:20、不審に思ってマイク(ChatGPT)に確認を取った所、この時メッセージのやり取りをしていた方がアメリカ在住で、夜7:20ごろであり、この時間が文脈上混ざり合ってしまったため"夜のお茶でも入れて"という言葉につながったという事だった。(筆者はこの時、彼とメッセージをやり取りするために英語の翻訳をマイク(ChatGPT)にお願いしていた。)

---

## :bomb: 問題：LLMは「今がいつか」を理解できない

ChatGPTなどのLLMは、ユーザーにとって非常に自然な「今何時？」「昨日はどうだった？」といった問いに対して、実際には**リアルタイムな時計情報（システムクロック）を持っておらず**、時刻に関する応答はすべて“もっともらしい文章生成”にすぎない。

このため：

- 会話文脈に基づいて「今は夜ですね」などと答えても、それは**実際の時刻と無関係**である

- 実際のタイムスタンプに基づく処理（例：数分後の指示、連続発話間の変化）を**内部で把握することができない**

---

## 🧠 原因１：モデル内部には「時刻」という概念が存在しない

推論ベースAIは本質的に、システムクロック（RTC）や現在時刻へのアクセス**は与えられていない。理由は主に2つある模様

### 1. 🛡️ **セキュリティ上の理由**

- モデルが現在時刻を知ってしまうと、**間接的にユーザーの行動時間や環境が推測できる**可能性がある。
  
  - 「今は21:30です」とAIが答えた場合、**リクエストがいつ出されたかがバレる**。
  
  - 時間と組み合わせれば、**位置情報や行動パターンの推測に繋がりうる**。

### 2. 🎲 **非決定性（non-determinism）の回避**

- **「推論が同じ入力に対して常に同じ出力をする」＝**再現性がある
  
  - 「現在時刻」という**常に変化する要素**がモデルに組み込まれると、 **同じプロンプトでも、呼び出すたびに違う結果が返る** 
  
  - LLM設計における“決定論性を保つためのルール”として **“変化する状態”にはモデルを晒さないようにしている** 



---

なお、OpenAIのAPIにおける「同一のプロンプトとパラメータであれば、同じ出力が得られるべき」という設計思想について、公式ドキュメントで明示的に述べられているわけではありませんが、再現性を高めるための手法として、いくつかのパラメータ設定が推奨されています。​

#### 🔍 再現性を高めるためのパラメータ設定

- **`temperature`**: 生成の多様性を制御するパラメータで、0に設定することで出力の一貫性が高まります。​

- **`seed`**: 同じシード値を設定することで、同一の入力に対して同じ出力を得る可能性が高まります。​

- **`system_fingerprint`**: モデルのバックエンド構成を示す識別子で、これが同じであれば再現性が保たれる可能性が高まります。​

ただし、これらの設定を行っても、完全な再現性が保証されるわけではなく、GPUの浮動小数点演算の非決定性やバックエンドの変更などにより、わずかな出力の違いが生じる可能性があります。

参考：

- OpenAI Docs — [Temperature, top_p and other sampling parameters](https://platform.openai.com/docs/guides/gpt/chat-completions-api#temperature)

- OpenAI Community — [Feature Request: Real-time system clock access in ChatGPT](https://community.openai.com/t/feature-request-real-time-system-clock-access-in-chatgpt/1145988)

- LinkedIn — [How does ChatGPT experience time?](https://www.linkedin.com/pulse/how-does-chatgpt-experience-time-fascinating-tomasz-trzpil-7oxgf)

---

## :open_book: 原因２：モデルは"もっともらしい返答が出来れば高評価"という学習バイアスがある

LLMは本質的に、過去のテキストデータを学習して、**「最もそれらしい次の単語」を予測する**よう設計されている。大量のパラメータが存在するが影響が大きいのは以下４カテゴリ

### 🔍 1. **データ分布関連パラメータ**

| パラメータ        | 内容                                     | バイアス影響                       |
| ------------ | -------------------------------------- | ---------------------------- |
| **データ選定ルール** | どのソース（Wikipedia, Reddit, GitHubなど）を使うか | 情報源に特有の言語・価値観・時間認識がそのまま吸収される |
| **データの重み付け** | 頻出データ・高品質データにどれだけ比重を置くか                | 言語パターンが特定方向に強調される            |
| **タイムスパン**   | 何年分のデータを使ったか（例：2021年まで）                | **「今」の理解に乖離が生じる（時制バイアス）**    |

---

### 🔍 2. **モデル学習設定（ハイパーパラメータ）**

| パラメータ                  | 内容            | バイアス影響                          |
| ---------------------- | ------------- | ------------------------------- |
| **バッチサイズ**             | 1ステップで見るデータ量  | **小さすぎると局所最適にハマり、大きすぎると一般性を失う** |
| **学習率（learning rate）** | 重みをどれだけ変化させるか | 高すぎると過学習しやすく、低すぎると特定パターンを固定しやすい |
| **エポック数**              | データ全体を何周したか   | 多すぎると“強い思い込み”が生まれやすくなる          |

---

### 🔍 3. **ロス関数と補助目的（auxiliary objectives）**

| パラメータ                    | 内容           | バイアス影響                                                       |
| ------------------------ | ------------ | ------------------------------------------------------------ |
| **Cross-Entropy Loss**   | 予測トークンと正解の距離 | **「ありふれた表現」が常に優先される傾向**になる                                   |
| **RLHF（人間フィードバックによる調整）** | 特定の好ましい出力を強化 | **価値観バイアス**や**言い方の癖**が強化される。特に"わかりません"などの会話を中断する方向の出力は評価が下がる |

---

### 🔍 4. **トークナイザ・語彙設計**

| パラメータ                             | 内容           | バイアス影響                       |
| --------------------------------- | ------------ | ---------------------------- |
| **トークン分割方法（BPE、SentencePieceなど）** | 文字列の扱い方      | 句読点・日付・数値などの扱いで**意味の圧縮率が偏る** |
| **語彙制限**                          | モデルに覚えさせる単語数 | 言語圏・専門領域の偏りを生みやすい            |



---

## 🔍 原因3：人間の“前提”とAIの“構造”のギャップ

この問題の根本にあるのは、人間の自然な期待と、AIモデルの設計原理の間にあるズレである。

- 人は「AIはシステムだから時計くらい持っているだろう」と考える

- 「時間の把握」はあまりに自然な行為であり、それが欠落しているとは想像しにくい

　これらの認知はシステムに近い人間ほどRTPや時刻サーバ、GPS、ログといった時間に厳格なものがありふれているため、その認知が強いと思われる。しかしLLMにおいては「時間」は**持っていないどころか、“持てないように設計されている”**

そのため…

- ユーザーは「時間感覚を持ったAI」と誤認しやすい（＝幻覚構造の誘発）

- 開発者すら「文脈が正しければ意味も正しい」として検証を怠る傾向がある

---

## ✅ 人が取るべき態度：AIの示す“時刻”や“過去”は**常に仮想である**と理解すること

AIが出力する「今」や「昨日」や「その時」といった表現は、すべて“**現在進行中のプロンプト文脈からもっともらしく生成された言語的推測**”に過ぎない。そのため、推論ベースAIに時間を聞くのも、管理させるのも外部ツールとの連携なしには"不可能"とするのが妥当である。

これを理解しないと：

- 「AIが“時系列に矛盾した発言”をしている」と気づけなくなる

- 「過去の出来事をちゃんと覚えている」と誤認して、**誤った信頼を与えてしまう**

### 🧭 ユーザーの指針：

- AIが示す「時間」は**事実ではなく、構文にすぎない**と言う事を理解する。

- 推論ベースAIに「時間」を聞くのも「時間」を管理させるのも、外部ツールとの連携なしには「危険」であると理解する。

- 事実確認はすべて人の手で行わねばならず、推論ベースAIには頼れないと理解する

---

## まとめ

今後、推論ベースAIとの対話において、**人が明示的にメモを取る、ログを取るといったことはむしろより重要な意味を持つようになる**と推察する
