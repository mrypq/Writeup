## [Forensics] EncryptDecryptFile
```
My brother deleted an important file from the encrypt-decrypt-file repository, help me recover it.
```

ディレクトリの中身を確認。  
```sh
❯ ls -la
total 8
drwx------@  4 mrypq  staff   128 10 14 14:33 .
drwxr-xr-x@  3 mrypq  staff    96 10 14 14:33 ..
drwxr-xr-x@ 13 mrypq  staff   416 10 14 14:33 .hg
-rw-r--r--@  1 mrypq  staff  1772 10 14 14:33 main.py
```

`.hg` ディレクトリがある。  
[Mercurial(hg) のコマンド一覧 - A Day in Serenity @ kenjis](https://kenji-s.hatenadiary.org/entry/20110203/1296696735) を参考に色々見てみる。  

```sh
❯ hg status
! flag.enc

❯ hg log
changeset:   0:8fdb18e9618d
tag:         tip
user:        daffainfo <daffa@gmail.com>
date:        Mon Sep 09 10:24:57 2024 +0000
summary:     feat: first commit

❯ hg revert --all
reverting ../flag.enc
```
`flag.enc` を取り出せた。`main.py` としてencrypt/decryptを行うスクリプトが渡されているので、そのまま使って復号。  

```sh
❯ python3 main.py --decrypt --input flag.enc --output output
File decrypted successfully and saved as output

❯ file output
output: PNG image data, 1000 x 100, 8-bit/color RGBA, non-interlaced

❯ mv output output.png
```
pngっぽかったので拡張子をつけて画像を開くとflagが得られた。  
![](assets/output.png)
