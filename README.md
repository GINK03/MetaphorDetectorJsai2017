# ドメインにより意味が変化する単語の抽出
立命館の学生さんが発表して、炎上した論文を、わたしもJSAI2017に参加していた関係で、公開が停止する前に入手することができました  
論文中では、幾つかのPixivに公開されているBL小説に対して定性的な分類をして、終わりという、機械学習が入っていないような論文でしたので、わたしなりに機械学習を使ってできることを示したいという思いがあります。（そんなに大変な問題でないように見えて、かつ、問題設定も優れていたのに、なぜ...）

## 一応、炎上に対して思うところ
PixivのBLのコンテンツを参照し、論文にハンドル名を含めて記述してしまっており、作家の方に精神的な不可をかけてしまうという事件がありました。
非常にRTされている代表的なツイートは、以下のようになっています。
<p align="center">
  <img width="450px" src="https://cloud.githubusercontent.com/assets/4949982/26529935/b0091662-4405-11e7-8071-e8ed0995473d.png">
</p>
多くの人がいろいろなことを言っており、情報は混沌としています。  
わたしが解決すべき問題と捉えているのは、二点です。
```console
- 引用することで、不利益を被る人の対応と配慮
- 転載と引用の混同を正す
```
解決できるように鋭意、TPOをわきまえて話しましょう。良い未来が開けることを期待しています。

## （アダルト表現などにおける）意味が変化する単語とは
元論文では、バナナなどの意味が変化するとされていますが、もう少し詳細に考えていましょう  
文脈によって意味が変化するということが成り立つとき、たとは、「熱い」とあっただけであっても、以下のような文脈依存で意味が変化しているようにみえることがあります。  

例1. 
```console
アイスコーヒーを頼んだ。冷たい。こんなに夏の熱い日に、カフェで飲むコーヒーは最高だ。
```
例2. 
```console
事務的な口調で支持される。冷たい。彼はこんなに人間性に乏しかっただろうか。
```
温度が冷たいのか、人の人格が冷たいのか、文脈で人間なら簡単に理解できます。

人間だからこの文脈によりなにかしら、意味というか、感じ方が違うものをうまくピックアップすることはできないでしょうか  
実はskip thoughtなどに代表される文章のベクトル化などの技術を使うとできるということを示します。

## ドメインや文脈により意味が変化する語を抽出する
ドメインという粒度だと、多分ニュースサイトと、小説サイトでは、同じ単語を持っていても微妙にニュアンスが異なるかと思います。  
skip gram的な考え方だと、周辺にある単語の分布を見ることで意味が決定すると言う仮説があります。その応用例がword2vecで馴染みぶかい物かもしれません。  

<p align="center">
  <img width="400px" src="https://cloud.githubusercontent.com/assets/4949982/26526886/265f0210-43c3-11e7-972b-8fe7d30fea3f.png">
</p>
<div align="center">図1. skip gram </div>

Word2Vecではエンピリカルな視点から、意味がベクトル化されているということがわかっています  
今回の提案では、これをより拡張して、**単語の意味が周辺の文脈から何らかの影響を受けている**と仮定して、モデルを作ります。  

<p align="center">
  <img width="400px" src="https://cloud.githubusercontent.com/assets/4949982/26526969/ce9e4502-43c4-11e7-860d-19fbcdec0fd2.png">
</p>
<div align="center">図2. 文脈により相対位置を決定</div>

## 文脈を定義する
単語より大きな粒度である、文脈というものを明確に定義する必要があります  
文脈の粒度を、文としました。文をベクトル化するにに、skip thought vector[1]や、doc2vec[2], fastText[3]などが利用できます  
このベクトルに意味らしきものが含まれるかという議論ですが、ICML2016のtext2imageによると、文章ベクトルから画像の生成に成功していることから鑑みて、なんらかの特徴が内包されているかとわかります[4]
<p align="center">
  <img width="500px" src="https://cloud.githubusercontent.com/assets/4949982/26527081/39127af0-43c7-11e7-879f-11ceb472dbfb.png">
</p>
<div align="center">図3. Sentence Vectorizer</div>

## 文章をベクトル化して、そのベクトルからの相対位置を決めるのは計算量的に面倒なので、量子化する
前後の文脈から、単語がどのようになるかは、求めることができそうのはわかりましたが、ベクトルの位置から計算していくというのは少々厄介なので、ベクトルを教師なしクラスタリング（kmean）で量子化します。  
クラスタリングすると、文章がどのクラスに属するのか唯一に決まり、計算が楽です。  
<p align="center">
  <img width="600px" src="https://cloud.githubusercontent.com/assets/4949982/26527122/0989259e-43c8-11e7-8870-7fb2df0a6562.png">
</p>
<div align="center">図4. 量子化</div>

## 幅を決めて前後の文脈から単語を表現していく
単語をベクトル化するにはWindwos幅を決めて、前後を単語を見ていきますが、これにならない、前後の文章をみて単語に対してどのような文脈が偏在しているか、求めます。  
今回は、前後の文章を三つ見ていきます。  
<p align="center">
  <img width="400px" src="https://cloud.githubusercontent.com/assets/4949982/26527175/df922604-43c8-11e7-9fd1-df00e356e3de.png">
</p>
<div align="center">図5. プログラム内部でのデータ構造のイメージ</div>

<p align="center">
  <img width="600px" src="https://cloud.githubusercontent.com/assets/4949982/26527191/3e0608cc-43c9-11e7-871f-60b0f5453178.png">
</p>
<div align="center">図6. 文脈によりエンベッティングされた単語</div>

## 実験
長くなりましたが、仮説とそれに伴う理論の構築がすみましたので、実験です。
仮説に寄ると、ドメインが異なると、単語の文脈が変化し、通常とことなる分布が期待できます。  
二つのコーパスを利用しました。
1. 2GByteのYahoo News + 小説家になろう
2. 1.8GByteのノクターンノベルズのHTML[5]

この二つのコーパスから、文脈を考慮した単語ベクトルを作成し、同じ単語のcosine similarityが近い、遠いを計算し、直接的でないアダルトドメインのみ固有で意味が変化するものが定量的に検知できます。  

## 結果
結果は、概ね良好かなという印象です。  
ただ、もっとデータがあるべきだなぁと思いました  
やはり計算には膨大な時間が必要で、全データセットを計算して結果を得るのに24時間ぐらいの時間を要したので、それなりにパワーのあるマシンで計算することをおすすめします  
```console
CPU: Ryzen1700x
Memoery: 48GByte
```
意味が変化する（consine similarityが遠い）単語トップ10です。

| 単語     | consine similarity |
|----------|--------------------|
| 肉       | 0.00152            |
| 増し     | 0.00130            |
| 指       | 0.00130            |
| 発射     | 0.00122            |
| 喉       | 0.00116            |
| 繋がっ   | 0.00103            |
| 熱い     | 0.00101            |
| 当たる   | 0.00095            |
| たっぷり | 0.00093            |
| 回し     | 0.00093            |
| 握っ     | 0.00050            |

<div align="center"> 表1. 文脈が変化している単語 top 10 </div>
また、意味が逆にそんなに遠くないものです。

| 単語   | cosine similarity |
|--------|-------------------|
| 思考   | 0.03889           |
| 原因   | 0.03745           |
| かつて | 0.03451           |
| 脳     | 0.03083           |
| 足首   | 0.03005           |
| 怪我   | 0.02739           |
| 無事   | 0.02721           |
| その間 | 0.02608           |
| 部下   | 0.02531           |
| 維持   | 0.02514           |
<div align="center"> 表2. 文脈が変化していない単語 top 10 </div>

## 考察
仮説を一個立てる必要がありましたが、既存のword2vec等が動作している仮説を担保としているため、間違いが無いように思います  
このように、ドメインによって意味やニュアンスが異なる単語は、定量的なアプローチで分析することができるので、小説を定性的な視点から分析する価値をあまり感じなかったのですが、人によりアプローチは色々あると思います  

今書いているコンテンツに応じて簡単にフィルタリングするべき単語などの抽出ができることが、分かりました  
具体的な方法としては、基準となるドキュメントで生成した単語の文脈の分散表現と、ドメイン依存した暗喩や比喩などの分散表現は分布が異なるので、cosine similarityが遠くなることが期待できます  

## ソースコード
```console
https://github.com/GINK03/DomainDependencyMemeJsai2017
```

## 参考文献
[1] [Sent2Vec](https://github.com/ryankiros/skip-thoughts)  
[2] [Doc2Vec](https://radimrehurek.com/gensim/models/doc2vec.html)  
[3] [fastText](https://github.com/facebookresearch/fastText)  
[4] [ICML2016, text2image](https://github.com/reedscot/icml2016)  
[5] [ノクターンノベルズ(R18です、注意してください)](http://noc.syosetu.com/top/top/) 
