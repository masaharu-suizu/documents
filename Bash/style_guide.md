# はじめに
shellscriptのコードの保守しやすくするために注意していることです。
ちなみにgoogleのスタイルガイドの影響を強く受けています。
もしどなたかのお役に立てば幸いです。

# シェルの種類
bash
googleでもbashが標準らしい

# ファイルの拡張子
- 実行するファイル
 - 拡張子を持たない or .sh
- 実行しないファイル(ライブラリ)
 - .sh


# ファイル名
小文字を使用する。
単語と単語の切れ目はアンダースコアーでつなぐ。
ex) get_access_count

# コメントについて
#### ファイルヘッダー
 - 処理の概要等を書く(コピーライトや著者はお好みで)

```bash
#!/bin/bash
#
# Perform hot backups of Oracle databases.
```

#### 関数
 - 関数の処理内容
 - 引数の説明
 - return値の説明
 - グローバル変数を使用した場合はその旨、それに手を加えた場合はその旨を記載

```bash
#!/bin/bash
#
# Perform hot backups of Oracle databases.

export PATH='/usr/xpg4/bin:/usr/bin:/opt/csw/bin:/opt/goog/bin'

#######################################
# Cleanup files from the backup dir
# Globals:
#   BACKUP_DIR
#   ORACLE_SID
# Arguments:
#   None
# Returns:
#   None
#######################################
cleanup() {
  ...
}
```

#### ToDo
 - 一時的なロジックや改善の余地があるロジックなどに対して書く
 - "TODO"と大文字で明記し、Coderの名前を残しておき、チケットがあればチケット番号を残しておくと良い

```bash
# TODO(mrmonkey): Handle the unlikely edge cases (bug ####)
```

####  その他
 - アルゴリズムが複雑な場合や特殊なことをしている場合は短いコメントを書く


# フォーマットについて
#### インデント
 - 半角スペース2個 (タブは使わない)

#### 1行あたりの文字数
 - 最大80文字 (80文字に収まらなくても大丈夫だが、収まる方法を見つける方が好ましい)

#### パイプ(|)や真偽(|| or &&)を使用する行
 - 1行に収まる場合は1行に収める
 - もし1行に収まらない場合は、改行しコマンドを1行に1つずつ記載する


```bash
# All fits on one line
command1 | command2

# Long commands
command1 \
  | command2 \
  | command3 \
  | command4
```

####  ;do, ;thenを記載する行
 - whileやforやifと同じ行に書く

```bash
for dir in ${dirs_to_cleanup}; do
  if [[ -d "${dir}/${ORACLE_SID}" ]]; then
    log_date "Cleaning up old files in ${dir}/${ORACLE_SID}"
    rm "${dir}/${ORACLE_SID}/"*
    if [[ "$?" -ne 0 ]]; then
      error_message
    fi
  else
    mkdir -p "${dir}/${ORACLE_SID}"
    if [[ "$?" -ne 0 ]]; then
      error_message
    fi
  fi
done
```

# クォート
- 変数
- コマンド
- スペース
- メタ文字

は基本的に引用符で囲む。あと特別な理由がない限り,```"$*"```でなく```"$@"```を使う

# コマンド
```
`command`
```
ではなく

```
$(command)
```
の形式を使う


# ネーミング
#### 変数
 - 機能が分かるようにネーミング。
 - ループ変数は似たような名前にすること。

```bash
for zone in ${zones}; do
  something_with "${zone}"
done
```

#### 関数
 - 小文字を使用する。単語と単語の切れ目はアンダースコアーでつなぐ。
 - 関数名と括弧はスペースを空けない。
 - 関数定義時に"function"キーワードを使うかどうかはお好きにどうぞ。ただし、使う場合は使う、使わない場合は使わないなど統一されていることが大事。

#### 定数
 - 大文字を使用する。単語と単語の切れ目はアンダースコアーでつなぐ。
 - ファイルの上部で宣言する

####  環境変数
 - 大文字を使用する。単語と単語の切れ目はアンダースコアーでつなぐ。

# 宣言に関して
私の好みですが、下記のようにしたほうがわかりやすい気がする。
#### 変数
    declare xxx_xxx='abc'
    declare -i xxx_xxx=1
#### index配列
    declare -a xxx_xxx=(1 2 3)
#### 連想配列
    declare -A xxx_xxx=([Jack]=11 [Queen]=12 [King]=13)
#### 定数
    declare -r XXX_XXX='abc'
#### 環境変数
    declare -rx XXX_XXX='abc'


# その他
####  関数内の変数定義
 - スコープを意識し、"local"をつけて宣言する

####  関数のreturn値を必ずチェックする
 - ```$?``` や ```${PIPESTATUS[0]}``` や ```${PIPESTATUS[1]}``` などでreturn値を必ずチェックする

```bash
if ! mv "${file_list}" "${dest_dir}/" ; then
  echo "Unable to move ${file_list} to ${dest_dir}" >&2
  exit "${E_BAD_MOVE}"
fi

# Or
mv "${file_list}" "${dest_dir}/"
if [[ "$?" -ne 0 ]]; then
  echo "Unable to move ${file_list} to ${dest_dir}" >&2
  exit "${E_BAD_MOVE}"
fi
```

```bash
tar -cf - ./* | ( cd "${dir}" && tar -xf - )
if [[ "${PIPESTATUS[0]}" -ne 0 || "${PIPESTATUS[1]}" -ne 0 ]]; then
  echo "Unable to tar files to ${dir}" >&2
fi
```


# 参考文献
googleのスタイルガイド(shell)
https://google.github.io/styleguide/shell.xml