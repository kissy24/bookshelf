# pytest入門

## はじめに

皆さんはPythonでユニットテストをしますか？なんだか小難しいし、コード自体は動くからやらなくても良いと思っていませんか？  
また、`pytest`でググってもまとまった記事に出会えないから`unittest`を使っているような状況だったりしませんか？  
そんな方々のためにお送りするのが本稿になります。

### 動作環境

OS: Windows10  
Python: 3.10.6  
pytest: 7.1.3  

## 1. Hello pytest

まずは実際に`pytest`でテストを実施してみましょう。下記のように、`pip`で`pytest`をインストールします。

```sh
pip install pytest
```

次に、プロダクションコードとテストコードをそれぞれ任意のスクリプト内に記述します。

```py::hello.py
def hello() -> str:
    return "hello pytest"

def test_hello():
    assert hello() == "hello pytest"
```

最後に、`pytest`をコマンド実行します。

```sh
$ pytest .\hello_pytest.py
============================== test session starts ==============================
platform win32 -- Python 3.10.6, pytest-7.1.3, pluggy-1.0.0
rootdir: C:\Workspace\Scripts
collected 1 item                                                                  

hello_pytest.py .                                                          [100%]

=============================== 1 passed in 0.02s ===============================
```

導入からテストの流れはこのような感じになります。  
次項からそれぞれトピックごとに深堀して説明します。

### 1.1. pytest とは？

[pytest org](https://docs.pytest.org/en/stable/) 曰はく
> pytest is a mature full-featured Python testing tool that helps you write better programs.  
> (pytestは、より良いプログラムを書くための成熟した全機能を備えたPythonのテストツールです。)

という Pythonista ならこれ使えと言わんばかりのテストツールです。以下のような特徴を持っています。

- 失敗したassert文の詳細情報の開示
  - unittestの `self.assertHoge` を覚える必要はありません
- テストモジュールと関数の自動検出
  - test_hoge.py や hoge_test.py, test_hoge関数のように`test`と付くモジュールや関数を自動で検出してくれます。
- ビルトインで多様なfixtureが提供されている
  - 痒い所に手が届くパラメータや前(後)処理がたくさんあると思ってください
- プラグインが豊富(800以上あり、コミュニティにも活気がある)

### 1.2. unittest との違い

Pythonには標準ライブラリに`unittest`というテストライブラリを持っています。  
しかし、このライブラリは上記の`pytest`のような使いやすさは感じられない気がします。  
例えば下記ようなコードと実行結果になるのですが、pytestの方が分かりやすいのではないでしょうか？

```py
def hello() -> str:
    return "hello"


def test_hello_pytest():
    assert hello() == "hello pytest"
```

```sh
=============================== test session starts ===============================
platform win32 -- Python 3.10.6, pytest-7.1.3, pluggy-1.0.0
rootdir: C:\Workspace\Scripts\python
collected 1 item

hello_pytest.py F                                                            [100%]

==================================== FAILURES ===================================== 
________________________________ test_hello_pytest ________________________________ 

    def test_hello_pytest():
>       assert hello() == "hello pytest"
E       AssertionError: assert 'hello' == 'hello pytest'
E         - hello pytest
E         + hello

hello_pytest.py:6: AssertionError
============================= short test summary info ============================= 
FAILED hello_pytest.py::test_hello_pytest - AssertionError: assert 'hello' == 'he...================================ 1 failed in 0.19s ================================ 
```

```py
import unittest

def hello() -> str:
    return "hello"

class Hello_unittest(unittest.TestCase):
    def test_hello_unittest(self):
        self.assertEqual(hello(), "hello pytest")
```

```sh
F
======================================================================
FAIL: test_hello_unittest (hello_pytest.Hello_unittest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "C:\Workspace\Scripts\python\hello_pytest.py", line 14, in test_hello_unittest
    self.assertEqual(hello(), "hello pytest")
AssertionError: 'hello' != 'hello pytest'
- hello
+ hello pytest


----------------------------------------------------------------------
Ran 1 test in 0.001s
-FAILED (failures=1)
```

デフォルトのスタックトレースだけでも`pytest`の方がリッチな気がします。(例が簡単すぎるのでそこまで差異を感じないかもしれませんが...)  
また、テストコードも`pytest`の方が簡潔で読みやすいのではないかなと思います。

ちなみに、`pytest`で`unittest`の機能を併用して使うことが可能です。  
ただし、`unittest`でできることは`pytest`でもできるため、併用はおすすめできないです。

## 2. 実行時設定

### 2.1. 設定ファイル

`pytest`の実行時のオプションや定義することのできる設定ファイルを紹介します。  
これらの設定ファイルは**リポジトリのルート**または**tests フォルダ**上に定義されます。

- `pytest.ini`: 最も優先で読まれるファイルだが個人的には後述する`pyproject.toml`を定義する事が多い。

  ```ini
  # 例
  [pytest]
  minversion = 6.0
  addopts = -ra -q
  testpaths =
      tests
      integration
  ```

- `pyproject.toml`: リンターやフォーマッターの設定ファイルの兼ね合いでこれをよく使います。`tool.pytest.ini_options`が定義されていると読まれます。

  ```toml
  # 例えばblack等を含めた設定ファイルを書くことが出来ます
  [tool.black]
  line-length = 120

  [tool.pytest.ini_options]
  minversion = "6.0"
  addopts = "-ra -q"
  testpaths = [
      "tests",
      "integration",
  ]
  ```

- `tox.ini`: `tox`のようなテストツールにも`pytest`の設定を記述することが可能です。

  ```ini
  # pytest.ini と記法は同じなので移行しやすいですね
  [pytest]
  minversion = 6.0
  addopts = -ra -q
  testpaths =
      tests
      integration
  ```

- `setup.cfg`: `setup.py`実行時のパラメータなどを記述する設定ファイルです。現在は`pyproject.toml`で事足りるのであまり定義する機会はないかな…。

  ```ini
  # pyproject.tomlと似た感じです。flake8とかmetadataなどが記載できます。
  [tool:pytest]
  minversion = 6.0
  addopts = -ra -q
  testpaths =
      tests
      integration
  ```

また、設定ファイルで定義できるオプションをいくつか紹介します。

- `minversion`: `pytest`のバージョンの下限値を指定できます。(そもそも`pipenv`とか`poetry`でバージョン管理したほうが良い)
- `addopts`: 常に実行したい`pytest`のオプション引数を指定できます。
- `testpaths`: テスト対象ディレクトリを指定できます。
- `python_files/python_classes/python_functions`: テスト対象の ファイル/クラス/関数 名を指定できます。(基本いじりません)
- `markers`: カスタムマーカーの登録ができます。(5. Marksで後述します。)

ここまで説明を受けて、「設定ファイル定義するのめんどくさい」と思った方は正直定義不要だと思います。(なくても`pytest`は十分動いてくれます。)  
`pytest`をある程度使ってみて、オプション毎回付けるのが面倒くさいとか、特定のファイルをテストしたいといった思いが出てきてからで十分です。

### 2.2. 実行時オプション

`pytest`を実行する際に、オプション引数を選択することができます。

まず、`pytest --help`(`pytest -h`)でオプションを確認してみましょう。

```sh
usage: pytest [options] [file_or_dir] [file_or_dir] [...]

positional arguments:
  file_or_dir

...
```

実行してみると分かりますが、多様なオプション引数があります。  
今回はその中でもよく使われるオプション引数をいくつか紹介します。

- -k オプション:「`pytest -k "{キーワード}"`」でモジュール名やクラス(関数)名に特定のワードが含まれているものだけをテストする。
  - テストケースが増えてくると活躍します。
  - `"{キーワード}"`は否定や論理和・積にも対応しています。

    ```sh
    # test_hoge 以外(キーワードを含まないもの)が実行されます
    pytest -k "not test_hoge"
    # test_hoge または test_fuga が実行されます(キーワードを含むもの)
    pytest -k "test_hoge or test_fuga"
    # hoge かつ fuga のキーワードを含むものが実行されます
    pytest -k "hoge and fuga"
    ```

- -s オプション:
- -v オプション:

## 3. ファイル構成

## 4. Asserts

## 5. Marks

## 6. Fixtures

## 7. Mocks

## おわりに
