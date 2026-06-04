# git checkout から git switch / git restore への置き換え

> Git 2.23（2019年8月リリース）で`git switch`と`git restore`が追加されました。`git checkout`ひとつに集中していた「ブランチの切り替え」と「ファイルの復元」が、役割ごとに別々のコマンドへ整理されています。

参考: [Git 2.23 リリースノート（GitHub Blog）](https://github.blog/2019-08-16-highlights-from-git-2-23/)

## なぜ分けられたのか

`git checkout`は、ブランチの切り替えからファイルの復元まで、まったく別の作業を1つのコマンドでこなせてしまいます。

```bash
# ブランチの切り替えやコミットへの移動
git checkout main          # ブランチを切り替える
git checkout -b feature    # 新しいブランチを作って切り替える
git checkout abc123        # 特定のコミットに移動する

# ファイルを元に戻す操作
git checkout -- file.txt   # file.txt の編集を取り消す
git checkout main file.txt # file.txt を main の内容に戻す
```

特に危ないのが、`git checkout`は**後ろにブランチ名を書くか、ファイル名を書くかによって、動作が変わる**点です。ブランチ名なら「切り替え」（無害）ですが、ファイル名を書くと「そのファイルの編集を破棄する」という、元に戻せない操作になります。

```bash
git checkout main      # ブランチ名 → main ブランチに切り替わる（無害）
git checkout app.js    # ファイル名 → app.js の編集が破棄される（元に戻せない）
git checkout .         # → 作業中の編集がすべて破棄される（元に戻せない）
```

そのため、「checkout で編集が消えてしまうこと」自体に気づきにくく、思わぬ事故につながりやすいのが難点でした。

そこで、役割ごとにコマンドが分けられました。

- `git switch` … **ブランチの切り替え・作成**
- `git restore` … **ファイルの復元・変更の取り消し**

コマンドを分けることで、ブランチとファイルのどちらを操作しているのかが明確になり、うっかり編集を消すような誤操作を防ぐことができます。

## git switch — ブランチの切り替え・作成

```bash
git switch main                       # 既存のブランチに切り替える
git switch -c feature                 # 新しいブランチを作って切り替える（checkout -b に相当）
git switch -                          # 直前にいたブランチに戻る（cd - と同じ感覚）
git switch -c feature origin/feature  # リモートのブランチをもとに作って切り替える
```

`git switch`が扱うのはブランチの切り替えと作成です。後ろにファイル名を書く使い方が無いため、`git checkout`のようにファイルの編集をうっかり破棄してしまう、という事故は起きません。

## git restore — ファイルの復元・変更の取り消し

ファイルの復元・変更の取り消しなどは、`git restore`に置き換わります。

```bash
git restore file.txt                # 作業中の編集を取り消して元に戻す（checkout -- file.txt に相当）
git restore --source=main file.txt  # main の時点の内容に戻す（checkout main file.txt に相当。-s main と短縮も可）
```

> ⚠️ `git restore .`（カレント以下すべて）のように範囲を広げると、コミットしていない編集がまとめてまとめて消えるので注意。

### 補足: git reset によるステージ解除も行える

`git restore`に`--staged`を付けることで、これまで`git reset HEAD <file>`で行っていたステージング解除（`git add`の取り消し）も行えます。

```bash
git restore --staged file.txt   # git add を取り消す（reset HEAD file.txt に相当）
```

## 対応表（checkout からの置き換え）

| やりたいこと | 従来の git checkout | 新しいコマンド |
| --- | --- | --- |
| ブランチを切り替える | `git checkout main` | `git switch main` |
| ブランチを作って切り替える | `git checkout -b feature` | `git switch -c feature` |
| 直前のブランチに戻る | `git checkout -` | `git switch -` |
| 特定のコミットに移動する | `git checkout abc123` | `git switch -d abc123` |
| 作業中の編集を取り消す | `git checkout -- file.txt` | `git restore file.txt` |
| 別ブランチ・コミットの内容に戻す | `git checkout main file.txt` | `git restore --source=main file.txt` |

## 注意点

- `git checkout`は**非推奨ではなく、廃止の予定もありません**。そのため、`switch` / `restore`が増えた今も従来どおり使えます。
- `git switch` / `git restore`は登場時はドキュメントに「experimental（実験的）」と注記されていましたが、現在その注記は外れ、安定して使えます。

## まとめ

`git checkout`の役割は、ブランチを操作する`git switch`と、ファイルを操作する`git restore`に分かれました。

役割がコマンド名から分かり、`checkout`のように操作を取り違えてファイルを消す事故も起きにくいため、これから覚えて使うなら、`switch` / `restore`がおすすめです。

## 参考

- [git-switch 公式ドキュメント](https://git-scm.com/docs/git-switch) / [git-restore 公式ドキュメント](https://git-scm.com/docs/git-restore)
- [git checkout の代替としてリリースされた git switch と git restore（kakakakakku blog）](https://kakakakakku.hatenablog.com/entry/2020/04/08/151627)
- [git checkout はもう迷わない。git switch / git restore を実例で理解する（Qiita / softbase）](https://qiita.com/softbase/items/da6e96699f64243765d9)
