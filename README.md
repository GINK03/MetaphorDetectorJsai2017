# ドメインにより意味が変化する単語の抽出と、フィルタリングの手法の提示
立命館の学生さんが発表して、炎上した論文を、わたしもJSAI2017に参加していた関係で、公開が停止する前に入手することができました。 
論文中では、幾つかのPixivに公開されているBL小説に対して定性的な分類をして、終わりという、機械学習が入っていないような論文でしたので、わたしなりに機械学習を使ってできることを示したいという思いがあります。（そんなに大変な問題でないように見えて、かつ、問題設定も優れていたのに、なぜ...）

## 一応、炎上に対して思うところ
PixivのBLのコンテンツを参照し、論文にハンドル名を含めて記述してしまっており、作家の方に精神的な不可をかけてしまうという事件がありました。
非常にRTされている代表的なツイートは、以下のようになっています。
```console
何度も言うけど立命館の問題は
・pixivの規約で当該著作者の許可を得ず転載することは禁止行為
・調査対象者の精神に危害を与えた
・調査対象者のプライバシーを軽んじた
・社会に対して研究者への不信感を与え、他の研究の妨げを行った
のが問題で二次創作とかは二の次だから
```
多くの人がいろいろなことを言っており、情報は混沌としています。  
わたしが解決すべき問題と捉えているのは、二点です。
```console
- 引用することで、不利益を被る人の対応と配慮
- 転載と引用の混同を正す
```
解決できるように鋭意、TPOをわきまえて話しましょう。良い未来が開けることを期待しています。

## （アダルト表現などにおける）意味が変化する単語とは
元論文では、バナナなどの意味が変化するとされていますが、もう少し詳細に考えていましょう。  
文脈によって意味が変化するということが成り立つとき、たとは、「熱い」とあっただけであっても、以下のような文脈依存で意味が変化しているようにみえることがあります。  

例1. 
```console
今日は38℃まで上がるらしい。熱い。クーラーをつけようか
```
例2. 
```console
どうやら彼が達したらしい。彼自身から熱いものを感じた。
```
温度が熱いのか、その、熱いといういわゆるその文脈でつかっているのか人間なら簡単に理解できます。

人間だからこの文脈によりなにかしら、意味というか、感じ方が違うものをうまくピックアップすることはできないでしょうか。  
実はskip thought, 文章のベクトル化などの技術を使うとできるということを示します。

## 