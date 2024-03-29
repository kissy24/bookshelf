# 【Python】pass と Ellipsis の使い分けについて

## はじめに

Pythonを書く上際に、処理を省略・仮実装・抽象化する場合があると思います。  
その際に、`pass`文を使うべきか`Ellipsis`を使うべきかを考察したいと思います。

※ 本稿は、部内のLTでプレゼンされる内容の忘備録で個人の見解を含んでいます。

## 環境

- Python 3.10.6

## 結論

- `pass`文: 「何も実行しないこと」がその処理として本質的な場合に利用する
- `Ellipsis`: 「未実装な処理」や「コンテナデータ型の拡張表現」として利用する

## pass文

誰もが知っているおなじみの`pass`文です。

> pass はヌル操作 (null operation) です --- pass が実行されても、何も起きません。  
> pass は、構文法的には文が必要だが、コードとしては何も実行したくない場合のプレースホルダとして有用です。[^1]

公式ドキュメント曰く、下記のように未実装関数の仮置きに使えます。

```py
def hoge():
    pass
```

また、処理内で何もしないことを明記したい場合にも活躍します。

```py
try:
    hoge + 2
except TypeError:
    pass
```

```py
if hoge == 2:
    return hoge
elif hoge == 3:
    pass
else:
    return 1
```

## Ellipsis

そもそも`Ellipsis`ってなんですか？みたいな人が多数いると思うのですが、`...`のことです。

```py
print(...)
>>> Ellipsis
```

これを見ても？？？な方がいるかもしれないので公式ドキュメントの説明を翻訳します。

> 省略記号リテラル「...」と同じ。主にユーザー定義のコンテナデータ型の拡張スライス構文と組み合わせて使用される特別な値です。  
> Ellipsisは、types.EllipsisType型の唯一のインスタンスです。[^2]

つまり、省略を表すオブジェクトです。passと異なり、文ではなく単なる**値**です。  
ドキュメントでは、コンテナデータ型の拡張スライス文で使用されると書いてありますが、主に`Numpy`のことです。

```py
import numpy as np

arr = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
print(arr[..., 0])
>>> [[1 3] 
>>> [5 7]]
```

ほかにも、可変長タプルの型アノテーションを書く際に利用します。

```py
def hoge(count) -> tuple[int, ...]:
    return (i for i in range(count))

a, b, c = hoge(3)
print(a, b, c)
>>> 0 1 2
```

## pass と Ellipsis の使い分け

それぞれの公式ドキュメントの見解とよくある使い方を見てきました。  
そこでですが、下記のような表記はどう思いますか？

```py
try:
    hoge + 2
except TypeError:
    ...
```

「違和感を感じる」、「気持ち悪い」、「どっちでも分かる」みたいな否定的な感じでしょうか？  
もちろん、`Ellipsis`自身は何もしないので、`pass`文と同等のヌル操作として捉えることが出来ます。

ここで公式ドキュメントの話を振り返ってみると、それぞれ以下のような意味合いの違いを感じます。  
(一旦公式ドキュメントが推奨している使われ方は無視します)

- `pass`: 操作としてのイメージ
- `Ellipsis`: 何かを省略したいイメージ

そのため、下記ようなコメントアウトがあると**あり**だと感じます。

```py
try:
    hoge + 2
except TypeError:
    # TODO 後で型エラーについての処理を記載します。
    ...
```

この書き方であれば「何か**省略**されているので今後実装される」ように見えます。  

もし、上記の処理が確定した処理の場合、`pass`を使って「何も実行しないことが操作である」ように見せた方が分かりやすいと感じます。

```py
try:
    hoge + 2
except TypeError:
    pass
```

未実装関数・クラスにも同様のことが言えます。  

```py
# 未実装関数
def hoge():
    ...

# 未実装class
def Hoge:
    ...

# おまけ、変数の仮置きにも使えます
hoge: int = ...
```

では、インターフェースを作成する時は、pass, Ellipsisのどちらを使いますか？

```py
from typing import Protocol

class Animal(Protocol)
    def bark(self) -> str:
        ...
```

`Ellipsis`ですね。理由は、将来的に実装クラスで**省略**された処理が実装されるからです。  
こう見ると、`pass`文は本当に何もしないとき以外はあまり使わない方が良いかもしれませんね。

※ 上記の検討は「Python Pass vs. Ellipsis[^3]」という記事でも題材として扱われていました。こちらも一読おすすめです。

## おわりに

結局のところ、こういった考え方があるといっただけで`Python`自体には強制力はありません。  
大事なのは、チームや個人がこのような一貫した考え方を持って開発できるかにあると思います。  
本稿がそういった考え方の一助になると嬉しいです。

## 参考文献

[^1]: https://docs.python.org/ja/3/reference/simple_stmts.html#the-pass-statement  
[^2]: https://docs.python.org/ja/3/library/constants.html#Ellipsis
[^3]: https://tuxtimo.me/posts/2021/11/05/python-pass-vs.-ellipsis/