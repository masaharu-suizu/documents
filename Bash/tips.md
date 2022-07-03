# はじめに
ちょっとしたことをするのにshellは非常に便利です。
ただし、私は普段他の言語で開発をしているので細かいことをよく忘れてしまいます。
そこで、忘れても振り返ることができるようにshellscriptを読んだり書いたりする際に頭に入れておくと便利なことをまとめていこうと思います。

# [bash] 変数展開
|                  |  説明 |
|:-----------------|:-----|
| VAR="word"       | 変数VARにwordという文字を代入する|
| ${VAR}           | 変数VARに代入されている値を展開する|
| ${VAR:=word}     | 変数VARに値が入っていない場合はwordという文字を代入する|
| ${VAR:5}         | 5文字目以降を切り出す|
| ${VAR:2:5}       | 2文字目から5つ切り出す(2文字目-6文字目) |
| ${#VAR}          | 変数VARに代入されている値の文字数を出力する |
| ${VAR#abcd}      | 変数VARに代入されている値の文字列がabcdで始まる場合、先頭のabcdをcutした文字列を出力する |
| ${VAR/1234/abcd} | 変数VARに代入されている値の文字列の1234という部分をabcdに置換する |
| ${VAR/1234//abcd} | 変数VARに代入されている値の文字列の1234という部分全てををabcdで置換する  |
| ${#array[@]}     | 配列の要素数 |
他にもありますが多用すると可読性が落ちるのでほどほどに。

# 特殊なshell変数
| shell変数| 説明 |
|:---------|:--|
| $0       | 実行されているshellscript名 |
| $1       | 第一引数 |
| $2       | 第二引数 |
| $#       | 引数の個数 |
| \$*      | 引数全部 |
| \$@      | 引数全部 |
| $?       | 直前のコマンドの終了ステータスコード |
| $$       | カレントシェルのプロセスID |
| $!       | 直前のバックグラウンドジョブのプロセスID |

# $@ と "$\*" と "$@" の違い
`sh sample.sh "This is a pen" "My name is Ken"
`
と引数が渡された場合\$@ と "\$\*" と "\$@"では引数を下記のように解釈しますので注意しましょう。

```bash:sample.sh
echo "Case1"
for word in $@; do
  echo " [${word}]"
done

echo "Case2"
for word in "$*"; do
  echo " [${word}]"
done

echo "Case3"
for word in "$@"; do
  echo " [${word}]"
done
```

出力結果

```
Case1
 [This]
 [is]
 [a]
 [pen]
 [My]
 [name]
 [is]
 [Ken]
Case2
 [This is a pen My name is Ken]
Case3
 [This is a pen]
 [My name is Ken]

```

# 組み込み変数
組み込み変数は\$HOMEなど色々ありますが、\$IFSを覚えておくとshellscriptを綺麗に書けると思います。

|          | 説明 |
|:---------|:-----|
| $IFS     | 区切り文字の設定。デフォルトはスペースとタブと改行文字。 |

```text:sample1.txt
a:b:c:d:e
```
上記のファイルを下記の2つのshellscriptで処理すると

```bash:sample1.sh
for word in `head -1 ${HOME}/sample.txt`; do
  echo ${word} 
done
```
出力結果
`a:b:c:d:e
`

```bash:sample2.sh
original="$IFS"
IFS=':'

for word in `head -1 ${HOME}/sample.txt`; do
  echo ${word} 
done

IFS="$original"
```

出力結果

```
a
b
c
d
e
```
になります。

# [bash] declareを使用し変数に制約を設ける
|          | 説明 |
|:---------|:--|
| declare -i xxx_xxx=1       | 数値を格納する変数 |
| declare -a xxx_xxx=(1 2 3) | index配列 |
| declare -A xxx_xxx=([Jack]=11 [Queen]=12 [King]=13) | 連想配列 |
| declare -r XXX_XXX='abc' | 定数 |
| declare -rx XXX_XXX='abc'| 環境変数 |

# [bash] localを使いスコープを局所化
なにも制約を設けなければ変数のスコープはグローバルになるのでlocalで局所化しましょう

```
function func_sample() {
  local num=$1
  ...
}
```

# コマンドセパレータ
|          | 説明 |
|:---------|:--|
| command1 && command2 | command1が成功した場合command2を実行する |
| command1 &#124;&#124; command2 | command1が失敗した場合command2を実行する |

if文と組み合わせて使うこともありますよね。

```bash

if command1 && command2 > /dev/null 2>&1
then
  # 全て成功した場合
fi

if command1 || command2 > /dev/null 2>&1
then
  # いずれかが実行された場合
fi

```

# グルーピング
|          | 説明 |
|:---------|:--|
| (command1; command2;) | サブシェルでcommand1,command2を実行 |
| { command1; command2; } | カレントシェルでcommand1,command2を実行。"{"の後と"}"の前にはスペースが必要。 |

# ファイルの内容を行単位に読み込む

```bash
while read LINE
do
    # ループ内の処理
done < /tmp/test.txt
```

# case文

```bash
case $1 in
  a ) # 処理
      ;;
  * ) # 処理
      ;;
esac
```

| 条件指定の形式            | 説明 |
|:-------------------------|:--|
| string )                 | 値がstringという文字列の場合 |
| string1 &#124; string2 ) | 値がstring1 もしくは string2 という文字列の場合 |
| [yY] )                   | 値がy もしくは Y という文字の場合 |
| [!yY] )                  | 値がy でも Y でもない文字の場合 |
| string* )                | stringという文字列が前方一致する場合 |
| *string )                | stringという文字列が後方一致する場合 |
| *string* )               | stringという文字列が部分一致する場合 |
| [a-z]* )                 | 文字の先頭がアルファベット小文字の場合 |
| [!a-z]* )                | 文字の先頭がアルファベット小文字でない場合 |
| ??? )                    | 文字列が3文字の場合 |


# 外部ファイルの読み込み

```bash
. ${HOME}/sample.fnc
```

# setコマンド
| Option   | 説明 |
|:---------|:--|
| -e       | コマンドの終了ステータスコードが0以外だったら、その場でshellを終了する |
| -u       | 未設定の変数を使おうとしたときにエラーにする |
他にもたくさんありますので一度確認してみるとよいと思います。
[Bash Reference Manual: Bash Conditional Expressions](http://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)

# 起動オプション
下記の2つのオプションを覚えておくと効率よくデバッグできると思います。

| Option            | 説明 |
|:------------------|:--|
| bash -n sample.sh | 実行はされずshellscriptの構文チェックが行われる |
| bash -x sample.sh | shellscript内の処理がターミナルにtraceされる  |

# おわりに
簡単にですが重要だと思うことをまとめてみました。
shellは陳腐化しない知識だと思いますので覚えておいて損はないと思います。
どなたかのお役に立てば幸いです。
