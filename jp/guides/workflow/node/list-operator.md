# リスト操作

リスト変数は、文章、画像、音声、映像など、さまざまなファイルを同時にアップロードすることができます。ユーザーがファイルをアップロードすると、すべてのファイルが同じ `Array[File]` 配列変数に保存されますが、**その後の個別ファイルの処理が難しくなります。**

> `Array` データ型は、変数の実際の値が \[1.mp3、2.png、3.doc] である可能性があることを意味します。LLM は、入力変数として画像やテキストなどの単一の値の読み取りのみをサポートしており、配列変数を直接読み取ることはできません。

### ノード機能

リストフィルタは、ファイルの形式のタイプ、名、サイズなどの属性に基づいてフィルターして抽出し、異なる形式のファイルをそれぞれの処理ノードに渡すことで、ファイル処理の流れを正確に制御します。

例えば、あるアプリでは、ユーザーが文章と画像を同時にアップロードすることを許可しています。異なるファイルは**リスト操作ノード**を通じて分類され、異なる処理フローに引き渡されます。

<figure><img src="../../../../zh_CN/.gitbook/assets/image (15).png" alt=""><figcaption><p>異なるファイルタイプのフロー分岐</p></figcaption></figure>

リスト操作ノードは通常、配列変数から情報を抽出するために使用され、条件を設定して、下流ノードが受け入れることのできる変数タイプに変換します。その構造は、入力変数、フィルタ条件、ソート、上位 N 項目、および出力変数に分かれています。

<figure><img src="../../../../zh_CN/.gitbook/assets/image (17).png" alt=""><figcaption><p>リスト操作ノード</p></figcaption></figure>

#### 入力変数

リスト操作ノードは以下のデータ構造変数のみを受け入れます：

- Array\[string]
- Array\[number]
- Array\[file]

#### フィルタ条件

入力変数の配列を処理し、フィルタ条件を追加します。条件に合致するすべての配列変数を選択し、その属性をフィルタリングします。

例：ファイルにはファイル名、ファイルタイプ、ファイルサイズなど、複数の属性が含まれる可能性があります。フィルタ条件では、特定のファイルを配列変数から選択し抽出するための条件を設定します。

以下の変数を抽出できます：

- type：ファイルのタイプ（画像、文章、音声、映像など）
- size：ファイルサイズ
- name：ファイル名
- url：ユーザーがURL経由でアップロードしたファイルを指し、完全なURLを入力してフィルタリングできます。
- extension：ファイルの拡張子
- mime_type：

    [MIMEタイプ](https://datatracker.ietf.org/doc/html/rfc2046)は、ファイルのコンテンツタイプを識別するための標準化された文字列です。例: "text/html" はHTMLドキュメントを示します。

- transfer_method：

    ファイルのアップロード方法（ローカルアップロードまたはURL経由でのアップロード）を示します。

#### ソート

入力変数の配列をファイルの属性に基づいて並べ替える機能を提供します。

- 昇順ソート：

    デフォルトの並べ替えオプションで、値が小さい順に並べ替えます。文字やテキストの場合はアルファベット順（A - Z）に並べ替えます。

- 降順ソート：

    値が大きい順に並べ替えます。文字やテキストの場合は逆アルファベット順（Z - A）に並べ替えます。

このオプションは通常、出力変数の first_record および last_record と組み合わせて使用されます。

#### 上位 N 項目

1から20の値を選択し、配列変数の上位 n 項目を選択します。

#### 出力変数

すべてのフィルタ条件を満たす配列要素。フィルタ条件、ソート、制限は個別に有効にできます。すべての条件を同時に有効にした場合、すべての条件を満たす配列要素が返されます。

- Result：フィルタリング結果で、データ型は配列変数です。配列に1つのファイルしか含まれていない場合、出力変数には1つの要素のみが含まれます。
- first_record：フィルタリングされた配列の最初の要素、すなわち result\[0]；
- last_record：フィルタリングされた配列の最後の要素、すなわち result\[array.length-1]。

***

### 設定例

ファイルのインタラクティブな質疑応答シナリオでは、アプリの使用者が文章や画像ファイルを同時にアップロードする可能性があります。LLMは画像ファイルを認識できる能力しか持たず、文章を読み取ることができません。その場合、[リスト操作](list-operator.md)ノードを使用して、ファイル変数の配列を事前処理し、異なるファイルタイプをそれぞれの処理ノードに送信する必要があります。手順は以下の通りです：

1. [機能](../additional-features.md)を有効にし、ファイルタイプで「画像」および「文章」を選択します。
2. フィルタ条件で、画像変数とドキュメント変数をそれぞれ抽出するための2つのリスト操作ノードを追加します。
3. 文章変数を抽出し、「ドキュメントエクストラクタ」ノードに渡し、画像ファイル変数を抽出してLLMノードに渡します。
4. 最後に「直接返信」ノードを追加し、LLMノードの出力変数を記入します。

<figure><img src="../../../../zh_CN/.gitbook/assets/image (375).png" alt=""><figcaption></figcaption></figure>

アプリの使用者が文章と画像ファイルを同時にアップロードした場合、文章は自動的にドキュメントエクストラクタノードに、画像ファイルは自動的にLLMノードに分流され、混合ファイルの共通処理が実現されます。