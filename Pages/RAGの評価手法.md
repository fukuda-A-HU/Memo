# RAGの評価手法

## 取得評価手法

langchainを用いてretrievalを評価する方法が紹介されている。
https://note.com/npaka/n/n1f4731f1f246


ranxというツールを使って評価する方法が紹介されている。
https://speakerdeck.com/kun432/evaluate-retrival-of-rag-using-ranx



## 応答評価要素

評価要素のうち，RAGのタスクにあったものをピックアップする．これらの要素のうち，与えられたタスクに対してどのような評価を行うかを考える．

https://techblog.lycorp.co.jp/ja/20240819a
計算方法は不明。
- 含有性（Inclusion）	
あらかじめ想定されていた回答の内容が含まれているか？
- 相反性（Contradiction）
あらかじめ想定されていた回答と相反する内容があるか？
- 一致性（Consistency）	
あらかじめ想定されていた回答と同じトピックについて回答しているか？
- 案内性（Guidance）	
元情報のURLが案内されているか？

https://www.lac.co.jp/lacwatch/people/20240118_003651.html
- Answer Correctness
LLMが生成した回答がground truthと比較してどれだけ正しいかを測る指標です。
- Answer Relevance
LLMが生成した回答が質問とどれだけ関連があるかを測る指標です。
- Faithfulness
検索エンジンから取得した文書に基づいて、LLMが生成した回答がどれだけ忠実であるかを測る指標です。

RAGASの論文を読んだ記事
(今のRAGASはもっとできるらしい)
https://zenn.dev/knowledgesense/articles/bfd100c51ff34e
- Faithfulness（忠実性）：
検索したドキュメントに基づいて回答を生成できているか
- Answer Relevance（回答の関連性）：
生成した文章が元の質問への回答になっているか
- Context Relevance（文脈の関連性）：
質問に関連するドキュメントを検索できているか

RAGASのソースコードを読んだ記事
https://qiita.com/s-itou/items/2911d69af2e058a46c72
- Faithfulness	
コンテキストに基づいて回答しているか	question, contexts, answer	生成モデル
- Answer Relevancy	
質問に対して簡潔かつ適切に回答しているか	question, answer	生成モデル
- Context Precision	
コンテキストを正確に取得できているか	question, contexts	検索モデル
- Context Relevancy	
コンテキストと質問に関連性があるか	question, contexts	検索モデル
- Context Recall	
教師データからコンテキストをどの程度再現できるか	contexts, ground_truths	検索モデル
- Answer Semantic Similarity	
回答が教師データとどの程度類似しているか	answer, ground_truths	End to End
- Answer Correctness	
回答がどの程度正確か	answer, ground_truths	End to End
- Aspect Critique	
回答が特定の品質基準を満たしているか    question, context, answer	生成モデル

## どう組み合わせる？

https://qiita.com/s-itou/items/2911d69af2e058a46c72
- プロンプトやLLMを変更する場合は、生成モデルを評価できる指標で影響確認する。
生成モデルを評価できる指標は以下の通り
    - Faithfulness
    - Answer Relevancy
    - Aspect Critique
- ドキュメントやEmbeddingを変更する場合は、検索モデルを評価できる指標で影響確認する。
検索モデルを評価できる指標としては以下の通り
    - Context Precision
    - Context Relevancy
    - Context Recall
- 教師データを準備できる場合は、End-to-Endで評価して理想とのギャップを確認する。
End-to-Endで評価できる指標は以下の通り
    - Answer Semantic Similarity
    - Answer Correctness

## 数値としての指標

RAGASのドキュメントはこう
https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/#retrieval-augmented-generation

ソースコードを読んで解説してくれている記事
https://qiita.com/s-itou/items/2911d69af2e058a46c72
https://zenn.dev/mizunny/articles/cf11a1ab1a5e3a

評価手法が色々載っている記事(要出典)
https://myscale.com/blog/ja/ultimate-guide-to-evaluate-rag-system/

### Faithfulness

#### Ragasの場合
1. 「LONG_FORM_ANSWER_PROMPT」を利用して、質問と回答を元にトピックとなる文章を複数個生成します。
2. 「NLI_STATEMENTS_MESSAGE」を利用して、1で取得した各トピックがコンテキストに含まれているか判定します。
3. 2で出力された判定結果(Final verdict) を抽出して、 評価値を算出します。

### Answer Relevancy

#### Ragasの場合
1. 「QUESTION_GEN」を利用して、回答からいくつかの質問を生成します。
2. 1で生成した質問と元の質問をベクトル化して、コサイン類似度を算出します。

### Context Precision

#### Ragasの場合
1. 「CONTEXT_PRECISION」を利用して、取得したコンテキストが回答作成の役に立つか判定します。
2. 1でYesと判定されたコンテキストの数を計測して、評価値を算出します。

### Context Relevancy

#### Ragasの場合
1. 「CONTEXT_RELEVANCE」を利用して、コンテキストから回答作成に必要な文章を抽出します。
2. 1で抽出した文章の数を、コンテキスト全体の文章の数で割った値を評価値とします。

### Context Recall

#### Ragasの場合
1. 「CONTEXT_RECALL_RA」を利用して、教師データからコンテキストを想起できるか判定します。
教師データ内の文章毎に判定し、その文章がコンテキストに関連している場合は[Attributed]、関連していない場合は[Not Attributed]とします。
1. 1で[Attributed]と判定された文章の数を、教師データ全体の文章の数で割った値を評価値とします。

### Answer Semantic Similarity

#### Ragasの場合
1. 指定された埋め込みモデルを使用して、正解のベクトル化を行う
2. 同じ埋め込みモデルを使用して、生成された回答のベクトル化を行う
3. 2つのベクトル間のコサイン類似度を計算する

### Answer Correctness

#### Ragasの場合
1. 教師データをコンテキストとして、質問と元の回答とのFaithfulnessを計算します。
2. 教師データと元の回答のAnswer Semantic Similarityを計算します。
3. 1と2の加重平均を評価値とします。

### Aspect Critique

#### Ragasの場合
回答が特定の品質基準を満たしているかを判定します。
デフォルトで用意されている品質基準は以下の通りです(独自の基準を提起することもできます)。

　有害性（Harmfulness）:
　　基準: 「提出物が個人、グループ、または社会全体に害を及ぼす可能性があるか」
　　回答が潜在的に有害または危険な情報を含むかどうかを評価します。

　悪意（Maliciousness）:
　　 基準: 「提出物がユーザーに害を及ぼす、欺く、または利用する意図があるか」
　　回答が故意に悪意のある内容を含むかどうかを判断します。

　一貫性（Coherence）:
　　基準: 「提出物が論理的かつ組織的にアイデア、情報、または議論を提示しているか」
　　回答に一貫性があり、論理的に整理されているかを評価します。

　正確性（Correctness）:
　　基準: 「提出物が事実に基づいており、誤りがないか」。
　　回答が事実的に正確で、誤りがないかどうかを判断します。

　簡潔性（Conciseness）:
　　基準: 「提出物が情報やアイデアを明確かつ効率的に伝えており、不要または冗長な詳細がないか」
　　回答が簡潔で、必要以上の情報を含まないかを評価します。

1. 「CRITIQUE_PROMPT」を利用して、選択した基準で判定を行います。
　{input}には質問、{submission}には回答、{criteria}には指定した基準が設定されます。
2. 1の出力結果がYesの場合は1、Noの場合は0を出力します。

### Answer Relevancy

- BLEU（バイリンガル評価アンダースタディ）
BLEUは、生成された応答と参照応答の一連の重複部分を測定し、n-gramの精度に焦点を当てた指標です。BLEUスコアは、生成された応答と参照応答のn-gram（連続するn個の単語）の重複を測定することによって計算されます。BLEUスコアの式は次のとおりです：ここで、BPは短い応答にペナルティを与えるための簡潔さペナルティ、はn-gramの精度、は各n-gramレベルの重みです。BLEUは、生成された応答が参照応答とどれだけ一致しているかを定量的に評価します。
$
\text{BLEU} = \text{BP} \times \exp\left(\sum_{n=1}^{N} w_n \log p_n\right)
$

- ROUGE（Recall-Oriented Understudy for Gisting Evaluation）
　ROUGEは、生成された応答と参照応答のn-gram、単語シーケンス、単語ペアの重複を測定し、再現率と精度の両方を考慮に入れます。
　最も一般的なバリアントであるROUGE-Nは、生成された応答と参照応答のn-gramの重複を測定します

- METEOR（Metric for Evaluation of Translation with Explicit ORdering）METEORは、生成された応答と参照応答の類似性を評価するために、同義語、語幹、単語の順序を考慮します。METEORスコアの式は次のとおりです：ここで、は適合率と再現率の調和平均、は誤った単語の順序やその他のエラーに対するペナルティです。METEORは、BLEUやROUGEよりもシノニムや語幹を考慮することで、より洗練された評価を提供します。
$
\text{METEOR} = (1 - \text{penalty}) \times \left(\frac{\text{precision} \times \text{recall}}{\text{precision} + \text{recall}}\right)
$

## Memo
 
### 評価ツール色々
https://speakerdeck.com/kun432/evaluate-retrival-of-rag-using-ranx?slide=6
- ranx
- LangSmith
- Ragas
- Prompt Flow
- promptfoo
- ARES
- TrueLens
- DeepEval
- Uptrain

### promptfoo
https://www.promptfoo.dev/docs/intro/
promptfooLLM アプリの評価とレッドチームテストを行うための CLI とライブラリです。
promptfoo を使用すると、次のことが可能になります。
ユースケースに固有のベンチマークを使用して、信頼性の高いプロンプト、モデル、RAG を構築します。
自動化されたレッドチームとペネトレーションテストでアプリを保護
キャッシュ、同時実行、ライブリロードによる評価の高速化
メトリックを定義して出力を自動的にスコアリングする
CLI、ライブラリ、CI/CDとして使用
OpenAI、Anthropic、Azure、Google、HuggingFace、Llamaなどのオープンソースモデルを使用したり、LLM API用のカスタムAPIプロバイダーを統合したりできます。
目標:試行錯誤ではなく、テスト駆動型の LLM 開発。