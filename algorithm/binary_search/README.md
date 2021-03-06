# 概要

二分探索のコードをPythonで書いてみます。あわせて、コードを書き上げるまでにどのようなことを考えていくか追っていきます。

## ゴール

二分探索とは何か・どのように問題を分解し、考えていくことでコードを組み立てられるのか理解することを目指します。

## 目次

[toc]

## 復習-探索処理を考えるときの大まかな流れ

最初に、探索処理をコードで表現するときの基本的な流れを復習しておきます。
やりたいことをざっくり書き出すと、

* リスト、あるいは配列を探索範囲として用意
* 探索対象(数値)・探索開始地点(リスト/配列内の一要素)を決定
* 現在参照している要素が探索対象と一致するか判定
* 一致しない場合は、次の要素を参照
* 探索対象が見つかるか、探索の終了条件を満たすまで探索を続行

といったようになります。
コードで見た方がより明確なイメージが掴めるかと思いますので、線形探索を例に見てみましょう。

```Python
# 線形探索
# search_listが探索範囲・targetが探索対象を表す
def search(search_list: list[int], target: int) -> Union[int, None]:

    # 先頭から順に探索
    # リストの要素を先頭から1つずつ参照
    for index, item in enumerate(search_list):
        # 一致判定
        if item == target:
            return index
        # 一致するものがなかった場合は次の要素(添字を1つ増やしたもの)を参照

    return None
```

引数で探索範囲と対象を決めています。そして、ループ処理の部分が探索の要となる、探索対象が範囲に存在するか探し回っているところです。
二分探索では、ループで書かれている処理のロジックが二分探索を表現したものへと置き換わります。
言い換えれば、二分探索を考えるということは、`どのように次の要素を参照していくのか`・`どんな条件を満たすと探索が終了するのか`を明確にすることだと言えます。

更新処理・終了条件をどのように組み立てていくのか、じっくりと見ていきます。


## 二分探索とは

二分探索とは、探す対象となるリストを半分・更に半分と、2つに分けることを繰り返しながら探索していくアルゴリズムです。
[参考](https://en.wikipedia.org/wiki/Binary_search_algorithm)

### アルゴリズムを分解してみる

線形探索と比べると、一気に難しくなりました。2つに分けるということだけではコードを書くのも難しそうなので、アルゴリズムを小さな要素に分けて考えてみます。もう少し詳しく言語化できれば、アルゴリズムのイメージも固まるはずです。

#### 半分に分ける

半分に分ける、という動作は日常でも馴染み深いものなので、リストを2等分することもなんとなくイメージできるかもしれません。
しかし、コードでこのような操作を表現するには、半分に分ける位置、すなわち`中央`をどのように決めるか明確にしなければなりません。

#### 探す方向

まず、二分探索アルゴリズムは、探索対象となるリストがソート済みであることが前提です。また、話を簡単にするため、今回は固定で昇順であるものを扱うことにします。
ソート済みであれば、探すリストを半分に分けたとき、リストが中央を基準に、探したいものより大きいもの・小さいものへと分かれます。すると、探したいものに応じて、前方(大きい方)・後方(小さい方)と探しに行く方向を一意に定めることができます。

```Python
# 探索対象リスト
# 要素「3」を中央として分割すると、前方には3より大きいもの・後方には3より小さいものだけが残る
sorted_search_list = [1, 2, 3, 4, 5]
# 以下のように探索対象が小さくなることで、問題を小さく扱いやすくすることができる
# 前方
former = [4, 5]
# 後方
latter = [1, 2]
```

双方向をまとめて考えると混乱しそうなので、前方の二分探索・後方の二分探索と分けて見ていくことにします。こうすることで、二分探索の完成形に表れる複雑な分岐も理解できるようになるはずです。


---

さて、二分探索アルゴリズムでやるべきことを少し細かく見てきました。これを振り返って、二分探索をコードで記述する上で考える手順を整理してみましょう。

* 中央とはどこか明確にする
* 前方/後方それぞれの二分探索を別々に考える
* 最後に双方向の探索を組み合わせ、二分探索を完成させる

このように問題を分解してみると、なんとか立ち向かえそうです。以降では小さく分けた問題を1つずつ見ていきます。

## 中央を知る

ここでは、中央がどのようなものか明らかにします。人目では直感的に分かる概念ですが、コンピュータに中央を理解させなければ、二分探索アルゴリズムを実装することはできません。
中央とはなんぞやまで深掘りするのはやめておき、ここではとりあえず、`2つの要素があるとき、要素間の中央はそれぞれの要素との距離が等しい`と捉えることにします。
`[1, 2, 3, 4, 5]`のようなリストを例にしてみます。このようなリストの中央はどこなのでしょうか。

直感的には、2番目の要素である`3`がリストの中央であるように見えます。 リストの要素同士の距離は、添え字の差から求めることができそうです。
例えば、0番目の要素と1番目の要素の距離は、`1 - 0 = 1`・2番目の要素と5番目の要素の距離は、`5 - 2 = 3`といった具合です。
ということは、中央は配列の先頭と末尾との添え字の差、すなわち距離が等しいものを探せば良さそうです。

先ほど、なんとなくで決めた2番目の要素で考えてみると、先頭との距離は、`2 - 0 = 2`・末尾との距離は`4 - 2 = 2`で先頭と末尾からの距離が同じとなりました。
例で見たリストにおける中央は、それぞれとの距離が等しい2番目の要素であることを計算で導き出せるようになりました。

---

これで、中央を算出する方法が見えてきました。今度は、コードでどのように中央を求めるか見てみましょう。

### コードで中央を書いてみる

先ほどまで見てきた例をコードで表してみます。リストとその先頭・末尾の添字を定義し、中央の位置を計算しています。

```Python
import math

# 探索対象リスト
sorted_search_list = [1, 2, 3, 4, 5]
# 先頭・末尾の添字
head = 0
tail = len(sorted_search_list) - 1

# 中央の位置を算出
mid = math.floor((head + tail) / 2)
```

リストを定義し、先頭・末尾の添字をそれぞれ`head・tail`と表現しています。中央は、先頭・末尾の添字をもとに導き出すことができます。コード上では小数部分を切り上げていますが、これは添字としてアクセスできるようにするためなので、切り捨てでも問題はありません。
また、足して2で割る操作がしっくりこない場合は、座標の中点を求める方法を考えてみると理解の助けとなるかもしれません。

---

これで中央を求める、つまり二分探索でリストを半分に分ける操作をコードで表せるようになりました。あとは、リストを前方/後方に進んでいく操作を記述できるようになれば、二分探索ができあがります。
それぞれ順にコードへと落とし込んでいきましょう。

#### 補足: 中央の求め方(別パターン)

二分探索で中央を求める処理は、これまで見てきた式でなく`head + (tail - head) / 2`のように表現しているものを見かけるかもしれません。
一見すると別の求め方をしているように見えますが、左側の`head`を`2 * head / 2`と置き換えれば、同じ形になります。それでは、なぜ別の表現で書くことがあるのでしょうか。

これは、先頭と末尾の添字を足した結果がリストの添字を表現する数値型の最大値を超えないようにするためです。
`head + (tail - head) / 2`は末尾の添字`tail`より大きくなることはありません。しかし、`(head + tail) / 2`は、計算の途中で`tail`よりも大きくなります。例えば添字が4バイトの数値型であれば、`head + tail`が数十億を超えると、問題となってしまいます。

やや細かい話となってしまったので、ひとまずここでは計算過程にて扱う数値が大きくなりすぎないよう、式の形を変えることもあるんだな、ということを覚えておきましょう。

[参考](https://ja.wikipedia.org/wiki/%E4%BA%8C%E5%88%86%E6%8E%A2%E7%B4%A2#%E5%AE%9F%E8%A3%85%E4%B8%8A%E3%81%AE%E9%96%93%E9%81%95%E3%81%84)

## 前方のみ探索

前方は、リストの右側、すなわち添え字がより大きくなる方向を指します。
つまり、線形探索とは進む方向が同じで、進む大きさのみが異なる二分探索を考えてみます。いきなり条件によって前に進んだり後ろに進んだりされると、頭が混乱してしまうので、まずはシンプルに前に進み続けます。

ここでは、前方にだけ進む二分探索が具体的にどのようにリストを走査していくのか・どのような条件を満たすと探索が終わるのか明らかにします。

### 進み方

二分探索がいかに前へと動いていくのか探ってみます。線形探索では、前に進むときに添字を1つずつ増やすように動いていました。
二分探索はどのように添字を変えながら前方へと動いていくのでしょうか。

一言で表すと、リストを中央で半分に分けたときの前方を残しながら進んでいきます。
10個の要素があるリストで実際に進みながら見てみてましょう。

```Python
# 探索範囲 中央は要素「6」
# 要素「6」より前方が次の探索範囲となる
search_list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]

# 探索範囲 中央は要素「9」
# 要素「9」より前方が次の探索範囲となる
former_1st = [7, 8, 9, 10, 11]

# 探索範囲 中央は要素「11」
# 要素「11」より前方は存在しないので打ち切り
former_2nd = [10, 11]
```

リストの中央を決め、中央より前方にあるものを次の探索範囲として置き換えています。
より具体的には、1回目の探索では10個の要素すべてが範囲・2回目では中央の要素「6」より前方の5個の要素が範囲・3回目では中央の要素「9」より大きい2個の要素が範囲となります。
このようにリストを走査していくことで、線形に1つ1つ調べるよりも、効率よく調べることができます。

#### 更新処理

コード上でどのようにリストの前方のみを残すのか、イメージを固めておきます。
中央の位置は計算で求められるようになったので、リストの添え字の形で参照できます。これを1つ進めたものを先頭・元のリストの末尾をそのまま末尾としてリストを作り替えれば、中央よりも先の要素だけを残したものがつくれそうです。
サンプルコードでも見てみましょう。

```Python
search_list = [1, 2, 3, 4, 5]
# 中央の位置
mid_index = 2
# 中央の位置+1番目が先頭となるように分けていけば、中央より前方だけが残る
former_1st = search_list[mid_index+1:]
# [4, 5]
```

このように前方へと進んでいくたびに先頭を動かしていけば、探索範囲のリストをどんどん小さくしていけそうです。

### 終了条件

続いて、いつ探索を終えるのか考えてみます。
線形探索ではすべての要素を走査し終えたとき、という明確な区切りがありました。ですが、二分探索はリストを半分ずつに分けながら進んでいくので、一見しただけではいつ終わるのかが見えづらそうです。
こういうときは小さなリストから順々に追っていくと見えやすくなります。コードで試してみましょう。

```Python
# 要素数1
search_list = [1]
# 中央は0番目
mid_index = math.floor((0 + 0) / 2)
# 中央より前方はこれ以上存在しないので、打ち切り

# 要素数2
search_list = [1, 2]
# 中央は1番目
mid_index = math.floor((0 + 1) / 2)
# 中央より前方はこれ以上存在しないので、打ち切り

# 要素数3
search_list = [1, 2, 3]
# 中央は1番目
mid_index = math.floor((0 + 2) / 2)
# 1番目より前方は2番目の要素しかないので、要素数1のときと同じ状態になる
```

どうやら、中央より前にある要素が無くなったときにこれ以上探索できなくなるようです。中央を求める操作が切り捨てであった場合も、同じ結果となります。
しかし、リストを`中央の位置 + 1番目`の要素が先頭になるように分けていくと、これ以上分けられなくなったときに問題となりそうです。

```Python
search_list = [1]
# 中央は0番目
mid_index = math.floor((0 + 0) / 2)
# 中央より前にあるはずの1番目の要素は存在しないのでサブリストがつくれない
sub_list = search_list[mid_index+1:]
```

そこで、中央より前方だけを残したリストを実際につくるのではなく、先頭と末尾を添字で表現し、添字だけを動かしながら調べることにしてみます。
こうすれば、実際にリストにアクセスしなくとも添字だけで範囲外か判断することができます。さらに、リストを分ける度にメモリを消費することもなくなって性能面でも良さそうです。
コードで試してみましょう。

```Python
# 初期状態
# headが先頭・tailが末尾のリストは、探索範囲のリスト全体を指す
head = 0
tail = 4
search_list = [1, 2, 3, 4, 5]
# 中央の位置
mid_index = 2
# 中央をもとに先頭を更新
# すると、headが先頭・tailを末尾としたリストは、中央より前方のみが残る
head = mid_index + 1
```

探索範囲をリストの`head番目`から`tail番目`とすれば、実体のリストを1つに保ったまま少しずつ前に進みながら探索できそうです。
更に、先頭を動的に変えられるようにしたことで、終了条件も書きやすくなります。要素数が1〜3のリストで確かめてみましょう。

```Python
# 要素数1
head = 0
tail = 0
search_list = [1]
# 中央は0番目
mid_index = math.floor((0 + 0) / 2)
# 先頭が1番目となり、リストの範囲外となる
head = mid_index + 1
# 末尾の添字はつねにリストの終端を指していて固定なので、先頭が末尾より大きくなったらこれ以上探索できなくなる


# 要素数2
head = 0
tail = 1
search_list = [1, 2]
# 中央は1番目
mid_index = math.floor((0 + 1) / 2)
# 先頭が2番目となり、リストの範囲外に到達
head = mid_index + 1


# 要素数3
head = 0
tail = 2
search_list = [1, 2, 3]
# 中央は1番目
mid_index = math.floor((0 + 2) / 2)
# 先頭が2番目となり、要素数1のときと同じ状態
head = mid_index + 1
```

どうやら、先頭の添字が末尾の添字より大きくなった場合、探索を終えるのが良さそうです。言い換えれば、`先頭の添字 <= 末尾の添字`である間は探索を続けられます。

---


### 実装

長くなってしまいましたが、更新処理・終了条件が少しずつ見えてきたはずです。ある程度イメージが固まってきたら、コードへと落とし込んでいきましょう。
理屈で完全にイメージしきれなくても、コードと頭の中を行ったり来たりしていけば、理解も確実に進んでいくはずです。


```Python
# binary_search/binary_search.py

def search_forward(search_list: list[int], target: int) -> Union[int, None]:
    """
    前方のみの二分探索

    :param search_list: 探索対象リスト
    :param target: 探索対象
    :return: 合致するものがリストに存在 -> 探索対象のリスト内インデックス, 存在しない -> None
    """

    # 探索範囲
    # ソート済みであることが前提
    sorted_search_list = sorted(search_list)

    # 先頭・末尾 初期値はリストの先頭・末尾と対応
    head = 0
    tail = len(sorted_search_list) - 1

    # 先頭・末尾の添字がリストをはみ出さない範囲でリストを走査
    while head <= tail:
        # 中央の位置を算出
        mid = math.floor((head + tail) / 2)

        # 中央の要素が現在参照している要素
        # これが探索対象と一致していれば処理を打ち切り
        if sorted_search_list[mid] == target:
            return mid

        # 先頭を中央の1つ先とすることで、先頭から末尾の指すものが、中央より前方のみを残したものとなる
        head = mid + 1

    # 合致なし
    return None
```

もう少し理解を深めるために、具体的なリストで探索するときの様子もあわせて見てみることにします。

```Python
# 探索対象
target = 5
# 探索範囲
sorted_search_list = [1, 2, 3, 4, 5]

# 先頭・末尾の添字の初期状態
head = 0
tail = 4

# 探索1回目
# 中央は2番目
mid = math.floor((0 + 4) / 2)
# 2番目の要素「3」は探索対象「5」と合致しないので続行

# 中央より前方だけを探索範囲とするために、先頭の添字を更新
# 先頭は3番目に
head = mid + 1
tail = 4

# 探索2回目
# 中央は4番目
mid = math.floor((3 + 4) / 2)
# 4番目の要素「5」は探索対象「5」と合致するので処理を打ち切り
```

線形探索では、0番目・1番目・2番目...と順に走査していましたが、前方の二分探索では、3番目・5番目だけが対象となりました。
このように、前方の二分探索は線形探索よりも速く進むことができそうです。

しかし、これだけではリストの一部しか走査していません。例えば探したいものが「4」であったときも4番目の要素にはたどり着くことができないので、探索処理として機能しているとは言えそうにないでしょう。
ですので、前だけでなく後ろにも進む機能を加えていきます。


## 後方のみ探索

続いて、後方の探索を考えてみます。基本的な考え方は前方の二分探索と全く同じです。
違いは、リストを半分に分けたとき、後方を残すようになったことです。さほど大きな差ではないので、いきなりコードから見ても問題なく理解できるはずです。


```Python
def search_backward(search_list: list[int], target: int) -> Union[int, None]:
    """
    後方のみの二分探索

    :param search_list: 探索対象リスト
    :param target: 探索対象
    :return: 合致するものがリストに存在 -> 探索対象のリスト内インデックス, 存在しない -> None
    """

    # 探索範囲
    # ソート済みであることが前提
    sorted_search_list = sorted(search_list)

    # 先頭・末尾 初期値はリストの先頭・末尾と対応
    head = 0
    tail = len(sorted_search_list) - 1

    # 先頭・末尾の添字がリストをはみ出さない範囲でリストを走査
    while head <= tail:
        # 中央の位置を算出
        mid = math.floor((head + tail) / 2)

        # 中央の要素が現在参照している要素
        # これが探索対象と一致していれば処理を打ち切り
        if sorted_search_list[mid] == target:
            return mid

        # 更新条件が前方とは異なる
        # 末尾を中央の1つ前とすることで、先頭から末尾の指すものが、中央より後方のみを残したものとなる
        tail = mid - 1

    # 合致なし
    return None
```

末尾の添字を`中央の添字 - 1`へと更新する操作が、探索対象のリストの後方のみを残す様子を表現しています。
更新条件を変えても問題なく要素を探しに行けるのか、具体的なリストで確かめてみます。中央を算出するとき、小数点を切り上げていることから前方とは少し動きが異なりますが、1つ1つ見ていけば前方と同じ考え方で理解できるはずです。

```Python
# 探索対象
target = 1
# 探索範囲
sorted_search_list = [1, 2, 3, 4, 5]

# 先頭・末尾の添字の初期状態
head = 0
tail = 4

# 探索1回目
# 中央は2番目
mid = math.floor((0 + 4) / 2)
# 2番目の要素「3」は探索対象「1」と合致しないので続行

# 中央より後方だけを探索範囲とするために、末尾の添字を更新
# 末尾は1番目に
tail = mid - 1
head = 0

# 探索2回目
# 中央は1番目
mid = math.floor((0 + 1) / 2)
# 1番目の要素「2」は探索対象「1」と合致しないので続行

# 中央より後方だけを探索範囲とするために、末尾の添字を更新
# 末尾は0番目に
tail = mid - 1
head = 0

# 探索3回目
# 中央は0番目
mid = math.floor((0 + 0) / 2)
# 0番目の要素「1」は探索対象「1」と合致するので処理を打ち切り
```

---

これでリストを前方・後方に探し回れるようになりました。ですが、前/後ろに進み続けるだけでは目的の要素を通り過ぎてしまうこともあります。
ですので、最後の仕上げとして、目的の要素に応じて進む方向を決められるようにしてみます。リストを漏れなく探せるようになれば、二分探索としての機能も完成します。

## 二分探索

二分探索の完成形として、前後に進めるようにすることを考えていきます。
ここでも前方の二分探索をたどったときと同じように、更新処理・終了条件に分けて見ていきます。どのようなことを考慮すればリストを網羅しながら前後に進めるのか、考えていきましょう。

### 更新処理

前方・後方それぞれで進み方は大体見えてきたかと思います。ですので、探したい要素に応じてリストを半分に分けた前方を残すのか・後方に残すのか判断しながら進んでいる様子を簡単なコードで見てみることにします。
動く方向が一度に2つへと増えましたが、やっていることは変わらないので、これまでの知識で立ち向かえるはずです。

```Python
# 探索対象
target = 4
# 探索範囲
sorted_search_list = [1, 2, 3, 4, 5]

# 先頭・末尾の添字の初期状態
head = 0
tail = 4

# 探索1回目
# 中央は2番目
mid = math.floor((0 + 4) / 2)
# 2番目の要素「3」は探索対象「4」と合致しないので続行

# ※ 探したい要素「4」は中央の要素「3」よりも前方にあるので、前に進みたい
# よって、中央より前方だけを探索範囲とするために、先頭の添字を更新
# 先頭は3番目
head = mid + 1
tail = 4

# 探索2回目
# 中央は4番目
mid = math.floor((3 + 4) / 2)
# 4番目の要素「5」は探索対象「4」と合致しないので続行

# ※ 探したい要素「4」は中央の要素「5」よりも後方にあるので、後ろに進みたい
# よって、中央より後方だけを探索範囲とするために、末尾の添字を更新
# 末尾は3番目
head = 3
tail = mid - 1

# 探索3回目
# 中央は3番目
mid = math.floor((3 + 3) / 2)
# 3番目の要素「4」は探索対象「4」と合致したので処理を打ち切り
```

具体例をもとに、`前後に進む`ことをもう少し詳しく表現してみます。
二分探索の探索範囲はソート済みであることが前提です。よって、今回の例(昇順)では、中央より前方は中央より大きい要素・中央より後方は中央より小さい要素しか存在しません。
つまり、`中央 > 探索対象`であれば後方(より小さい方)・`中央 < 探索対象`であれば前方(より大きい方)と進む方向が決まります。

まとめると、中央の要素と探したいものの大小関係を比べることで、次にどちらへ進むべきか一意に定めることができます。

### 終了条件

そして、最後に考えるべきこととして、いつ探索を終えるのか明らかにします。前/後ろだけに進むのであれば、探索範囲の外に到達したときに探索を終わらせてしまえばよさそうです。
ですが、探したいものがリストの真ん中付近にあると、前後に進み続けて終わらないような気がしてきます。頭の中だけで前後に動かしていると混乱してきそうなので、実際に前後に進みそうな例を見てみます。

```Python
# 探索対象
target = 20
# 探索範囲
sorted_search_list = [2, 4, 8, 16, 32]

# 先頭・末尾の添字の初期状態
head = 0
tail = 4

# 探索1回目
# 中央は2番目
mid = math.floor((0 + 4) / 2)
# 2番目の要素「8」は探索対象「20」と合致しないので続行
# 探索対象「20」は中央の要素「8」よりも大きいので、前方を探索
# 先頭が3番目となる
head = mid + 1
tail = 4

# 探索2回目
# 中央は4番目
mid = math.floor((3 + 4) / 2)
# 4番目の要素「32」は探索対象「20」と合致しないので続行
# 探索対象「20」は中央の要素「32」よりも小さいので、後方を探索
# 末尾が3番目となる
tail = mid - 1
head = 3

# 探索3回目
# 中央は3番目
mid = math.floor((3 + 3) / 2)
# 3番目の要素「16」は探索対象「20」と合致しないので続行
# 探索対象「20」は中央の要素「16」よりも大きいので、前方を探索
# 先頭が4番目となる
head = mid + 1
tail = 3

# ※ 先頭が4番目・末尾が3番目となり、矛盾
```

重要なのは、3回目の探索を終えた後の更新処理です。
探索対象はリストに存在しないことから、先頭と末尾が一致した状態でも更に先頭/末尾を更新します。すると、先頭が末尾よりも先を指すようになり、次の探索範囲となるリストが手に入らなくなりました。
つまり、4回目の探索範囲となるリストは、`先頭が4番目・末尾が3番目となるようにリストを分けたもの`となり矛盾が生じます。

※ これは末尾の場合でも探索対象が「12」などであれば、同じように先頭が3番目・末尾が2番目となります。

ここから、前方・後方だけに進んでいたときの二分探索の終了条件を双方向へと拡張してみます。
二分探索は進む方向がどうあれ、中央と探索したい要素からリストを半分・更に半分と分け、分けた後のリストで繰り返し探索しています。
リストをどんどん半分にしていくと、最終的には半分にした結果が空となります。
前方だけ・後方だけ・双方向それぞれ形は異なりますが、中央をもとに更新した先頭・末尾から得られるリストが無くなったときこそが、探索を終えるべきときであることが分かります。

まとめると、リストを半分に・更に半分に分けていき、これ以上分けられなくなったときが終了条件となります。

---

### 実装

さて、少し話が難しくなってしまいました。
言葉だけではイメージしづらくても、前方・後方の二分探索で得た知識を組み合わせていけば、コードから二分探索がどういう条件のもとリストを探し回っているのか、見えてくるはずです。
早速見てみましょう。

```Python
def search(search_list: list[int], target: int) -> Union[int, None]:
    """
    二分探索

    :param search_list: 探索対象リスト
    :param target: 探索対象
    :return: 合致するものがリストに存在 -> 探索対象のリスト内インデックス, 存在しない -> None
    """

    # 探索範囲
    # ソート済みであることが前提
    sorted_search_list = sorted(search_list)

    # 先頭・末尾 初期値はリストの先頭・末尾と対応
    head = 0
    tail = len(sorted_search_list) - 1

    # 先頭・末尾の添字からリストを分割できる範囲で探索
    while head <= tail:
        # 中央の位置を算出
        mid = math.floor((head + tail) / 2)

        # 中央の要素が現在参照している要素
        # これが探索対象と一致していれば処理を打ち切り
        if sorted_search_list[mid] == target:
            return mid

        # 前方
        if sorted_search_list[mid] < target:
            head = mid + 1
            continue

        # 後方
        if sorted_search_list[mid] > target:
            tail = mid - 1
            continue

    # 合致なし
    return None
```

中央を求める処理・終了条件・更新処理と考えることは多いですが、1つ1つを細かく見てきたことで、きっと立ち向かえるようになっているはずです。
コードで何をやっているかが見えてきたら、これまで見てきたことと照らし合わせながら「なぜそう書くのか」まで理解することを目指してみてください。


## まとめ

二分探索のざっくりとした考え方を見てきました。
入門アルゴリズムとして挙げられるものではありますが、なぜそう書くのかまで掘り下げると、中々歯応えのあるものだったと思います。

今回見てきた、複雑な完成形をやりたいことをベースに小さな単位に分解して考えていくことは、色々な問題に応用が利くので、ぜひ身につけてみてください。
