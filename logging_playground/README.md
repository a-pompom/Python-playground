# 概要

Pythonのログ出力の基本を習得したい。

## ゴール

Pythonでログを出力する方法・各種モジュールの考え方を身につけることを目指す。
[参考](https://docs.python.org/3/howto/logging.html)

## 用語整理

* Logging: プログラムが実行されるときに発生するイベントを追跡するための手段
* Logger: アプリケーションからログを出力するときのインタフェース 実際のログ出力処理はHandlerへ委譲
* LogRecord: 1つのログを表現したオブジェクト Loggerからつくられる
* Handler: ファイルやコンソール・メールなど、ログの出力方法を制御するためのモジュール
* Formatter: LogRecordのフォーマットを指定するためのモジュール
* LogLevel: ログの重要度 DEBUGやINFO, ERRORなどがある LoggerやHandlerにLogLevelを設定しておくことで、必要なログのみ収集できる


## ログ出力の基本

ここでは、複雑な設定なしで利用できるログ出力の機能を試してみる。より具体的には、loggingモジュールが提供している、root Loggerを操作するための関数群を動かしてみる。

### とりあえずログを出力したい

まずは何はともあれログを書き出してみる。

[参考](https://docs.python.org/3/howto/logging.html#a-simple-example)

```Python
import logging

logging.debug('出力されません')
logging.warning('出力されます')
# メッセージ: `WARNING:root:出力されます`
```

ログ出力関数の書式を見ておく。

[参考](https://docs.python.org/3/library/logging.html#logging.debug)

> 書式: `logging.debug(msg, *args, **kwargs)`

単純な用途であれば、引数に出力したいメッセージを指定するだけでよい。
デフォルトではログレベルWARNING以上が出力される。また、内部的にはRootLoggerというクラスのインスタンスを利用してログを書き出している。

そして、デフォルトでは`sys.stderr`へメッセージが出力される。


### ログをファイルへ出力したい

続いて、実際のアプリケーションでよく使われる、ファイルへのログ出力を試してみる。

[参考](https://docs.python.org/3/howto/logging.html#logging-to-a-file)

```Python
import logging

# basicConfig関数はroot Loggerに対する設定を適用
logging.basicConfig(filename='./to_file_log.txt', encoding='utf-8', level=logging.DEBUG)

logging.debug('DEBUGも表示されます')
logging.info('これはINFOです')
```

基本的な処理では、`logging.basicConfig()`を利用してファイル出力先などの設定を記述する。

[参考](https://docs.python.org/3/library/logging.html#logging.basicConfig)

> 書式: `logging.basicConfig(**kwargs)`

※ 色々とオプションの指定はあるが、後述の通りroot Loggerは非推奨なのであまり踏み込まないでおく。

### ログへ出力するメッセージを整形したい

ただメッセージが書かれているだけではイベントを追跡できないので、ログのフォーマットを変えてみる。

[変数出力-参考](https://docs.python.org/3/howto/logging.html#logging-variable-data)
[フォーマット参考](https://docs.python.org/3/howto/logging.html#changing-the-format-of-displayed-messages)

#### 変数埋め込み

よく使う操作として、変数を埋め込む方法を見る。

[参考](https://docs.python.org/3/howto/logging.html#logging-variable-data)
[%フォーマット参考](https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting)

```Python
import logging

logging.warning('log name is %s', 'Warning!!')
```

文字列に変数を埋め込む操作と言えばf-stirngsの方が良さそうだが、ログ出力においては、%フォーマットが推奨されている。
これは、ログレベルに応じて`__str__()`などによる評価を抑止できることに起因しているようだ。

より具体的には、ログメッセージのフォーマットは、%形式であればLogRecordを生成するときに評価される。つまり、ログを出力しない場合には
評価をスキップし、パフォーマンスを下げることなくログ出力処理を記述することができる。

#### フォーマット

loggingモジュールが標準で用意している要素を埋め込むためのフォーマットも見ておく。

[参考](https://docs.python.org/3/library/logging.html#logrecord-attributes)

```Python
import logging

logging.basicConfig(format='%(asctime)s %(message)s')

logging.warning('log name is %s', 'Warning!!')
```

ログのフォーマットは、`%(フォーマット名)s`のように指定することが多い。指定できるものは、LogRecordの属性に従う。
[参考](https://docs.python.org/3/library/logging.html#logrecord-attributes)

よく使うものを挙げておく。

* asctime: ログが出力された日時
* levelname: ログレベル
* message: `debug()`, `info()`などで指定されるメッセージ


#### なぜroot Loggerは非推奨なのか

root Loggerは特別な設定なしにすぐに使うことができるが、あまり推奨されていない。
これは、root Loggerの設定がログ出力全体に影響を与えることによる。

Loggerは用途に応じて使い分けることが多いので、よほどシンプルなアプリケーションでない限りは、root Loggerを直接利用しない方がよい。


## モジュールを利用したロギング

より柔軟にログを出力できるよう、loggingモジュールが提供しているモジュール群を使ってみる。
ここでは、Logger, Handlerモジュールを介してログが出力されるまでの流れを理解する。

### Loggerをつくりたい

ログ出力のインタフェースを提供するLoggerをつくってみる。

[参考](https://docs.python.org/3/howto/logging.html#loggers)

```Python
import logging

# name引数をもとにLoggerクラスのインスタンスを生成
# 生成されたインスタンスはManagerクラスにnameをキーとした辞書で保持されているので、
# 同一のname引数からは同一のLoggerクラスのインスタンスが得られる
print(__name__)
logger = logging.getLogger(__name__)
logger.warning('from logger message')
```

`logging.getLogger()`からLoggerクラスのインスタンスを得る。

[参考](https://docs.python.org/3/library/logging.html#logging.getLogger)

> 書式: `logging.getLogger(name=None)`

name引数を指定しなかった場合はRootLoggerが返却される。

### Logger・Handlerを利用してログを出力したい

実際のログ出力を担うHandlerを介して、ログを出力してみる。

[参考](https://docs.python.org/3/howto/logging.html#handlers)

```Python
import logging

# ログ出力のインタフェースを提供するLoggerを生成
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# 実際のログ出力を制御するHandlerを生成
handler = logging.FileHandler('./using_logger_and_handler_log.txt')
handler.setLevel(logging.INFO)
# LoggerがLogRecordを渡すHandlerを設定
logger.addHandler(handler)

# Loggerを介してログ出力
logger.debug('ファイルに出力されません。')
logger.info('ファイルに出力されます。')
logger.warning('これもファイルに出力されます。')
```

Logger・Handlerが連携してログを出力するときの流れは、公式が提供している図を見ると分かりやすい。

![ログ出力の流れ](https://docs.python.org/3/_images/logging_flow.png)

#### FileHandler

ファイルを対象としたHandler。

[参考](https://docs.python.org/3/library/logging.handlers.html#logging.FileHandler)

> 書式: `class logging.FileHandler(filename, mode='a', encoding=None, delay=False, errors=None)`

filename引数へファイル名を指定すると、指定したファイルへログを出力する。

#### setLevel

Logger・Handlerそれぞれへログレベルを設定する方法を見てみる。

[参考](https://docs.python.org/3/library/logging.html#logging.Logger.setLevel)

> 書式: `setLevel(level)`

level引数には、loggingモジュールが提供している定数を指定。
ログレベルは、`debug < info < warning`などの順に高くなり、指定したログレベルより低いログレベルのログ出力処理が
呼ばれた場合は、メッセージが捨てられる。

```Python
# sample
import logging
logger = logging.getLogger(__name__)
logger.setLevel(logging.WARNING)
# ...Handler設定
# DEBUGはWARNINGよりもログレベルが低いので、ログとして出力されることなく捨てられる
logger.debug('これは出力されません')
```

こうすることで、開発環境・本番環境など、それぞれに必要なメッセージにログを絞り、ストレージなどが圧迫されないようにしている。

#### なぜLogger/Handlerそれぞれにログレベルが存在するのか

Loggerのログレベルは、環境やLoggerによって収集したいログを絞り込むために設定される。
例えば開発環境ではDEBUGレベルに設定しておき、本番環境ではINFOレベルに設定しておくなど。

一方、Handlerのログレベルは、重要度などに応じて出力先を切り替えるために設定される。
例えばERRORレベルのものはメール通知し、INFOレベルはファイルに出力するなど切り替えることができる。これは、実際にはLoggerへログレベルの異なる複数のHandlerを持たせることで実現する。

つまり、Loggerのログレベルは出力の粒度を・Handlerのログレベルは出力対象を制御するために定義されている。

#### addHandler

LoggerがLogRecordを送信するHandlerを追加。
Handlerは複数追加でき、Loggerが所有するそれぞれのHandlerへメッセージ(LogRecord)が渡される。

[参考](https://docs.python.org/3/library/logging.html#logging.Logger.addHandler)

> 書式: `addHandler(hdlr)`


### 設定値からLoggerやHandlerをつくりたい

Logger・Handlerなど、ログ出力の設定は記述量が増えやすい。また、環境によって記述内容を切り替えたいので、手続き的に設定するのではなく、
宣言的に設定を記述できるようにしたい。

このようなとき、公式ではdictConfigと呼ばれる設定方法が推奨されている。
dictConfigは辞書をもとにLogger・Handler・Formatterなど、ログ出力に必要なモジュールを定義するための関数である。


```Python
import logging
import logging.config

config_dict = {
    'version': 1,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'DEBUG',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.FileHandler',
            'level': 'INFO',
            'filename': './dict_config_example_log.txt'
        }
    },
    'loggers': {
        'simpleExample': {
            'level': 'DEBUG',
            'handlers': ['console', 'file'],
            'propagate': 'no'
        }
    }
}

# 内部的には設定値をもとにHandlerクラスのインスタンスをつくったり、Logger.getLogger()を呼び出している
# つまり、Loggerでログを出力できる環境を設定値をもとに構築している
logging.config.dictConfig(config_dict)
# 設定値からLogger名をもとに既にLoggerがつくられているので、ここでは既存のLoggerが得られる
logger = logging.getLogger('simpleExample')
logger.debug('debug log')
logger.info('info log')
```

`dictConfig()`へ設定値を記述した辞書を渡すことで、設定済みのLoggerが得られる。

[参考](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig)

> 書式: `logging.config.dictConfig(config)`

また、辞書のフォーマットは以下のリンクを参照。

[参考](https://docs.python.org/3/library/logging.config.html#configuration-dictionary-schema)


## サンプルアプリケーション

実際のユースケースを想定してログ出力機能を実装してみる。
簡単なFizzBuzzアプリへログ出力機能を記述することで、具体的にどのように処理を書くのか・どのようなログが出力されるのか理解を深めたい。

### ログ出力設定を記述したい

まずはログ出力設定を定義。
今回はアプリケーションの呼び出し元で定義しておく。

```Python
from sample_app.fizz_buzz import get_fizz_buzz_message, InvalidFizzBuzzValue
import logging

# パッケージ階層のrootに相当するパッケージ名でLoggerを作成
# こうすることで、各種モジュールは__name__でLoggerをつくると、階層構造でLogRecordが伝播し、
# 1つのLoggerでパッケージのログを管理することができる
logger = logging.getLogger('sample_app')
logger.setLevel(logging.DEBUG)

formatter = logging.Formatter(fmt='%(asctime)s - %(levelname)s: %(message)s')

# すべてのログはlog.txtへ出力
handler = logging.FileHandler('./log.txt')
handler.setLevel(logging.DEBUG)
handler.setFormatter(formatter)

# エラーログのみerror_log.txtへ出力
error_handler = logging.FileHandler('./error_log.txt')
error_handler.setLevel(logging.ERROR)
error_handler.setFormatter(formatter)

logger.addHandler(handler)
logger.addHandler(error_handler)

if __name__ == '__main__':
    try:
        get_fizz_buzz_message(15)
    except InvalidFizzBuzzValue as e:
        logger.exception('Invalid FizzBuzzValue')
```

#### Formatter

ログ出力メッセージを整形するために、Formatterも設定しておく。
フォーマットはイニシャライザのfmt引数へ記述する。

[参考](https://docs.python.org/3/howto/logging.html#formatters)

> 書式: `logging.Formatter.__init__(fmt=None, datefmt=None, style='%')`


### 各処理でログを出力するモジュールをつくりたい

まずは実装コードから雰囲気を見ておく。

```Python
from typing import Union
import logging

# Logger自体の設定は、mainモジュールでパッケージ名のLoggerを対象に反映済み
# よって、モジュール上では特に設定することなく利用できる
logger = logging.getLogger(__name__)

DIVISOR_FIZZ = 3
DIVISOR_BUZZ = 5
DIVISOR_FIZZBUZZ = 15

FIZZBUZZ_VALUE_MIN = 1
FIZZBUZZ_VALUE_MAX = 100


class InvalidFizzBuzzValue(Exception):
    """ FizzBuzzにおいて無効な値が渡されたことを表現 
        例外が送出されたときのログ出力を検証するために利用
    """


def get_fizz_buzz_message(value: int) -> Union[str, int]:
    """
    入力の数値と対応するFizz Buzzメッセージを取得

    :param value: 入力値
    :return: FizzBuzzメッセージまたは入力値
    """
    logger.info('call FizzBuzz')
    logger.debug('value: %s', value)

    if value < FIZZBUZZ_VALUE_MIN or value > FIZZBUZZ_VALUE_MAX:
        raise InvalidFizzBuzzValue('無効な値が渡されました')

    if value % DIVISOR_FIZZBUZZ == 0:
        logger.debug('return FizzBuzz')
        return 'FizzBuzz!!'

    if value % DIVISOR_FIZZ == 0:
        logger.debug('return Fizz')
        return 'Fizz'

    if value % DIVISOR_BUZZ == 0:
        logger.debug('return Buzz')
        return 'Buzz'

    logger.debug('return value: %s', value)
    return value
```

モジュール側ではLoggerの設定を記述していない。
しかし、Loggerが階層構造を持つことから、呼び出し元で定義したLoggerへLogRecordが伝播される。
これにより、モジュールごとにLoggerの設定を書く必要が無くなる。

```bash
# sample
2022-04-30 14:06:18,464 - INFO: call FizzBuzz
2022-04-30 14:06:18,464 - DEBUG: value: 15
2022-04-30 14:06:18,464 - DEBUG: return FizzBuzz
```

#### logger.exception

例外が発生したときは、`logger.exception()`でログへ出力すると、メッセージだけでなく例外情報も追記される。

[参考](https://docs.python.org/3/library/logging.html#logging.Logger.exception)

> 書式: `exception(msg, *args, **kwargs)`

```bash
# エラーメッセージサンプル

2022-04-30 14:04:40,029 - ERROR: Invalid FizzBuzzValue
Traceback (most recent call last):
  File "/logging_playground/sample_app/main.py", line 25, in <module>
    get_fizz_buzz_message(-1)
  File "/logging_playground/sample_app/fizz_buzz.py", line 31, in get_fizz_buzz_message
    raise InvalidFizzBuzzValue('無効な値が渡されました')
sample_app.fizz_buzz.InvalidFizzBuzzValue: 無効な値が渡されました
```


## ライブラリでロギング

ライブラリとしてアプリケーションをつくり、かつログ出力を組み込みたい場合には、いくつか考慮すべき点があるようだ。
ただ公式ドキュメントをみるだけではイメージしづらかったので、実際にライブラリっぽいものを書いて試してみる。

### NullHandlerを設定したい

ログ出力メッセージはデフォルトでは、ログレベルWARNING以上のものが`sys.stderr`へ出力される。
つまり、ライブラリへログ出力処理を記述しており、さらにアプリケーション側でロギングの設定が無かった場合、ログレベルWARNING以上のメッセージがコンソールへ出力される。

どのメッセージをログとして出力すべきかは利用者が制御できるべきである、という思想から、ライブラリのトップレベルにはNullHandlerというHandlerを設定しておく。
NullHandlerは文字通り何もしないHandlerで、無条件にログ出力メッセージを切り捨てる。

[参考](https://docs.python.org/3/howto/logging.html#configuring-logging-for-a-library)

```Python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# NullHandlerを指定しなかった場合、ライブラリを呼び出すと何も設定していなくてもログが出力される
# ライブラリのログは利用者側で制御できるようになっているべきなので、NullHandlerを指定
# もし実装中にログを検証したい場合は、親階層のパッケージ名でLoggerをつくるのがよい

# handler = logging.FileHandler('./sample_lib_log.txt')
handler = logging.NullHandler()
handler.setLevel(logging.DEBUG)
logger.addHandler(handler)


def decorate_text(text: str) -> str:
    """
    入力テキストを装飾

    :param text: 入力値
    :return: 入力値に装飾を加えたテキスト
    """
    logger.warning('TEST WARNING')
    logger.debug('text: %s', text)

    return f'** {text} **'
```

このようにライブラリのLoggerのHandlerをNullHandlerとしておくことで、ログレベルWARNING以上のメッセージであっても意図せず出力されることはない。

### ライブラリのログを取得したい

とはいえ、ライブラリを開発していたり、何かライブラリで問題を見つけたときはログを取得できるようにしたい。
呼び出し側に設定を加えることで、ライブラリのログを出力できるようにしてみる。

```Python
import logging
from sample_library.lib.sample_lib import decorate_text

# ライブラリの親階層を指定することで、伝播されたライブラリのログが得られる
logger = logging.getLogger('sample_library.lib')
logger.setLevel(logging.DEBUG)

handler = logging.FileHandler('./log.txt')
handler.setLevel(logging.DEBUG)
logger.addHandler(handler)

# ライブラリの処理を呼び出す
if __name__ == '__main__':
    decorate_text('text')
```

Loggerの名前にライブラリの親階層を指定しておくことで、ライブラリから出力されたログを呼び出し元で定義したHandlerで扱えるようになる。
今回の例では、ライブラリ側のNullHandlerとは別にFileHandlerを設定しておくことで、ログ出力メッセージが得られた。
