# Solr の削除処理の考察
## 参考資料
* [Lucene Index 考察(4)](https://qiita.com/MingchunZhao/items/d75910027574edf41e2c)
* [Apache Solr and Optimizing Your Index](https://lucidworks.com/post/solr-optimize-merge-expungedeletes-tips/)
* [Apache Solr Segment Merging, Deleted Documents, and Why Optimize May Be Bad For You](https://lucidworks.com/post/solr-segment-merge-frees-wasted-space-caused-by-deleted-documents/)

## 事前準備
### Solr コンテナの起動
```sh
docker pull solr
docker run -p 8983:8983 -t solr
```

Solr 8.9.0

### core の準備
```sh
docker exec "$(docker ps | grep solr | awk '{print $1}')" /opt/solr/bin/solr create -c collection1
```

[solr - dockerhub](https://hub.docker.com/_/solr)

## 実験1 - maxDocs 2

1. データ登録
```sh
curl 'http://localhost:8983/solr/collection1/update?commit=true' -H "Content-Type: text/xml" --data-binary @queries/1_add.xml
```

結果を確認

```console
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" ls -l /var/solr/data/collection1/data/index
total 60
-rw-r--r-- 1 solr solr  158 Jul 13 13:05 _0.fdm
-rw-r--r-- 1 solr solr  268 Jul 13 13:05 _0.fdt
-rw-r--r-- 1 solr solr   64 Jul 13 13:05 _0.fdx
-rw-r--r-- 1 solr solr 1614 Jul 13 13:05 _0.fnm
-rw-r--r-- 1 solr solr   61 Jul 13 13:05 _0.nvd
-rw-r--r-- 1 solr solr  247 Jul 13 13:05 _0.nvm
-rw-r--r-- 1 solr solr  530 Jul 13 13:05 _0.si
-rw-r--r-- 1 solr solr  206 Jul 13 13:05 _0_Lucene80_0.dvd
-rw-r--r-- 1 solr solr 1377 Jul 13 13:05 _0_Lucene80_0.dvm
-rw-r--r-- 1 solr solr   92 Jul 13 13:05 _0_Lucene84_0.doc
-rw-r--r-- 1 solr solr  100 Jul 13 13:05 _0_Lucene84_0.pos
-rw-r--r-- 1 solr solr  304 Jul 13 13:05 _0_Lucene84_0.tim
-rw-r--r-- 1 solr solr   80 Jul 13 13:05 _0_Lucene84_0.tip
-rw-r--r-- 1 solr solr  490 Jul 13 13:05 _0_Lucene84_0.tmd
-rw-r--r-- 1 solr solr  220 Jul 13 13:05 segments_2
-rw-r--r-- 1 solr solr    0 Jul 13 13:05 write.lock
```

```console
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" od -c /var/solr/data/collection1/data/index/_0.fdt
0000000   ? 327   l 027 034   L   u   c   e   n   e   8   7   S   t   o
0000020   r   e   d   F   i   e   l   d   s   F   a   s   t   D   a   t
0000040   a  \0  \0  \0 004 002 214   ` 366 213 023 301 373 331  \n   s
0000060 367 213 032 352 247  \0  \0  \n  \0  \t  \a 243   4  \0 020 001
0000100 022 022 022 022 022 022 022 022 022 017  \0 360 001  \0 003   U
0000120   S   D  \b  \n   O   n   e       D   o   l   l   a 360 001   r
0000140 030 017   B   a   n   k       o   f       A   m   e   r   i 360
0000160 001   c   a   ( 003   b   o   a   0  \b   c   u   r   r   e   n
0000200   c 360 001   y   0 003   U   S   D   @ 017   C   o   i   n   s
0000220       a   n 360 001   d       n   o   t   e   s   P 005   1   ,
0000240   U   S   D   ` 001 360 001   T  \0 003   E   U   R  \b  \b   O
0000260   n   e       E   u   r   o 360 001 030 016   E   u   r   o   p
0000300   e   a   n       U   n   i   o   n 360 001   ( 002   e   u   0
0000320  \b   c   u   r   r   e   n   c   y   0 003 360 001   E   U   R
0000340   @ 017   C   o   i   n   s       a   n   d       n 340   o   t
0000360   e   s   P 005   1   ,   E   U   R   ` 001   T 300   ( 223 350
0000400  \0  \0  \0  \0  \0  \0  \0  \0   z 323 355 001
0000414
```

"USD", "ERU" それぞれ存在することを確認

2. USD を削除
```sh
curl 'http://localhost:8983/solr/collection1/update?commit=true' -H "Content-Type: text/xml" --data-binary @queries/2_delete.xml
```

```console
docker exec "$(docker ps | grep solr | awk '{print $1}')" ls -l /var/solr/data/collection1/data/index
total 64
-rw-r--r-- 1 solr solr  158 Jul 13 13:05 _0.fdm
-rw-r--r-- 1 solr solr  268 Jul 13 13:05 _0.fdt
-rw-r--r-- 1 solr solr   64 Jul 13 13:05 _0.fdx
-rw-r--r-- 1 solr solr 1614 Jul 13 13:05 _0.fnm
-rw-r--r-- 1 solr solr   61 Jul 13 13:05 _0.nvd
-rw-r--r-- 1 solr solr  247 Jul 13 13:05 _0.nvm
-rw-r--r-- 1 solr solr  530 Jul 13 13:05 _0.si
-rw-r--r-- 1 solr solr   67 Jul 13 13:11 _0_1.liv
-rw-r--r-- 1 solr solr  206 Jul 13 13:05 _0_Lucene80_0.dvd
-rw-r--r-- 1 solr solr 1377 Jul 13 13:05 _0_Lucene80_0.dvm
-rw-r--r-- 1 solr solr   92 Jul 13 13:05 _0_Lucene84_0.doc
-rw-r--r-- 1 solr solr  100 Jul 13 13:05 _0_Lucene84_0.pos
-rw-r--r-- 1 solr solr  304 Jul 13 13:05 _0_Lucene84_0.tim
-rw-r--r-- 1 solr solr   80 Jul 13 13:05 _0_Lucene84_0.tip
-rw-r--r-- 1 solr solr  490 Jul 13 13:05 _0_Lucene84_0.tmd
-rw-r--r-- 1 solr solr  220 Jul 13 13:11 segments_3
-rw-r--r-- 1 solr solr    0 Jul 13 13:05 write.lock
```

削除前と差分なし

```console
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" od -c /var/solr/data/collection1/data/index/_0.fdt
0000000   ? 327   l 027 034   L   u   c   e   n   e   8   7   S   t   o
0000020   r   e   d   F   i   e   l   d   s   F   a   s   t   D   a   t
0000040   a  \0  \0  \0 004 002 214   ` 366 213 023 301 373 331  \n   s
0000060 367 213 032 352 247  \0  \0  \n  \0  \t  \a 243   4  \0 020 001
0000100 022 022 022 022 022 022 022 022 022 017  \0 360 001  \0 003   U
0000120   S   D  \b  \n   O   n   e       D   o   l   l   a 360 001   r
0000140 030 017   B   a   n   k       o   f       A   m   e   r   i 360
0000160 001   c   a   ( 003   b   o   a   0  \b   c   u   r   r   e   n
0000200   c 360 001   y   0 003   U   S   D   @ 017   C   o   i   n   s
0000220       a   n 360 001   d       n   o   t   e   s   P 005   1   ,
0000240   U   S   D   ` 001 360 001   T  \0 003   E   U   R  \b  \b   O
0000260   n   e       E   u   r   o 360 001 030 016   E   u   r   o   p
0000300   e   a   n       U   n   i   o   n 360 001   ( 002   e   u   0
0000320  \b   c   u   r   r   e   n   c   y   0 003 360 001   E   U   R
0000340   @ 017   C   o   i   n   s       a   n   d       n 340   o   t
0000360   e   s   P 005   1   ,   E   U   R   ` 001   T 300   ( 223 350
0000400  \0  \0  \0  \0  \0  \0  \0  \0   z 323 355 001
0000414
```

削除前と差分なし

```console
$ curl 'http://localhost:8983/solr/collection1/select?q=*:*&rows=0&wt=json&indent=true'
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "q":"*:*",
      "indent":"true",
      "rows":"0",
      "wt":"json"}},
  "response":{"numFound":1,"start":0,"numFoundExact":true,"docs":[]
  }}
```

検索はヒットしない

3. optimize
```sh
curl 'http://localhost:8983/solr/collection1/update?optimize=true'
```

```console
docker exec "$(docker ps | grep solr | awk '{print $1}')" ls -l /var/solr/data/collection1/data/index
total 60
-rw-r--r-- 1 solr solr  158 Jul 13 13:16 _1.fdm
-rw-r--r-- 1 solr solr  175 Jul 13 13:16 _1.fdt
-rw-r--r-- 1 solr solr   64 Jul 13 13:16 _1.fdx
-rw-r--r-- 1 solr solr 1614 Jul 13 13:16 _1.fnm
-rw-r--r-- 1 solr solr   59 Jul 13 13:16 _1.nvd
-rw-r--r-- 1 solr solr  247 Jul 13 13:16 _1.nvm
-rw-r--r-- 1 solr solr  567 Jul 13 13:16 _1.si
-rw-r--r-- 1 solr solr  143 Jul 13 13:16 _1_Lucene80_0.dvd
-rw-r--r-- 1 solr solr 1377 Jul 13 13:16 _1_Lucene80_0.dvm
-rw-r--r-- 1 solr solr   78 Jul 13 13:16 _1_Lucene84_0.doc
-rw-r--r-- 1 solr solr   88 Jul 13 13:16 _1_Lucene84_0.pos
-rw-r--r-- 1 solr solr  231 Jul 13 13:16 _1_Lucene84_0.tim
-rw-r--r-- 1 solr solr   80 Jul 13 13:16 _1_Lucene84_0.tip
-rw-r--r-- 1 solr solr  488 Jul 13 13:16 _1_Lucene84_0.tmd
-rw-r--r-- 1 solr solr  220 Jul 13 13:16 segments_4
-rw-r--r-- 1 solr solr    0 Jul 13 13:05 write.lock
```

`write.lock` 以外はすべて更新された

```console
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" od -c /var/solr/data/collection1/data/index/_1.fdt
0000000   ? 327   l 027 034   L   u   c   e   n   e   8   7   S   t   o
0000020   r   e   d   F   i   e   l   d   s   F   a   s   t   D   a   t
0000040   a  \0  \0  \0 004 002 214   ` 366 213 023 301 373 331  \n   s
0000060 367 213 032 352 255  \0  \0 006  \t   M  \0  \b 001  \t  \t  \t
0000100  \t  \t  \t  \t  \t  \t 006  \0 200  \0 003   E   U   R  \b  \b
0000120   O 200   n   e       E   u   r   o 030 200 016   E   u   r   o
0000140   p   e   a 200   n       U   n   i   o   n   ( 200 002   e   u
0000160   0  \b   c   u   r 200   r   e   n   c   y   0 003   E 200   U
0000200   R   @ 017   C   o   i   n 200   s       a   n   d       n   o
0000220 200   t   e   s   P 005   1   ,   E   P   U   R   ` 001   T 300
0000240   ( 223 350  \0  \0  \0  \0  \0  \0  \0  \0   K   M 363   "
0000257
```

USD が削除された

4. expunge delete で EUR を削除
```sh
curl 'http://localhost:8983/solr/collection1/update?commit=true&expungeDeletes=true' -H "Content-Type: text/xml" --data-binary @queries/3_delete.xml
```

```console
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" ls -l /var/solr/data/collection1/data/index
total 4
-rw-r--r-- 1 solr solr 135 Jul 13 13:21 segments_5
-rw-r--r-- 1 solr solr   0 Jul 13 13:05 write.lock
```

いろいろ削除された

max docs 2 → 1 では expunge delete しても index の内容が変更されなかったので max docs 10 に変更して再実験

## 実験2 maxDocs 10
```sh
curl 'http://localhost:8983/solr/collection1/update?commit=true' -H "Content-Type: text/xml" --data-binary @queries/1_add.xml
```

```xml
<add>
  <doc><field name="id">1</field></doc>
  <doc><field name="id">2</field></doc>
  <doc><field name="id">3</field></doc>
  <doc><field name="id">4</field></doc>
  <doc><field name="id">5</field></doc>
  <doc><field name="id">6</field></doc>
  <doc><field name="id">7</field></doc>
  <doc><field name="id">8</field></doc>
  <doc><field name="id">9</field></doc>
  <doc><field name="id">10</field></doc>
</add>
```

```
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" \ls /var/solr/data/collection1/data/index
_4.fdm
_4.fdt
_4.fdx
_4.fnm
_4.si
_4_Lucene80_0.dvd
_4_Lucene80_0.dvm
_4_Lucene84_0.doc
_4_Lucene84_0.tim
_4_Lucene84_0.tip
_4_Lucene84_0.tmd
segments_a
write.lock
```
```
$ docker exec "$(docker ps | grep solr | awk '{print $1}')" od -c /var/solr/data/collection1/data/index/_4.fdt
0000000   ? 327   l 027 034   L   u   c   e   n   e   8   7   S   t   o
0000020   r   e   d   F   i   e   l   d   s   F   a   s   t   D   a   t
0000040   a  \0  \0  \0 004 002 214   ` 366 213 023 301 373 331  \n   s
0000060 367 213 032 352 275  \0  \0   *  \0 001 003   m 266 333   p  \0
0000100 004 001 005 005 005 005 005 005 005 004  \0   @  \0 001   1  \0
0000120   @ 001   2  \0 001   @   3  \0 001   4   @  \0 001   5  \0   @
0000140 001   6  \0 001   @   7  \0 001   8   @  \0 001   9  \0   0 002
0000160   1   0 300   ( 223 350  \0  \0  \0  \0  \0  \0  \0  \0 224 331
0000200   ?   :
0000202
```

2. delete (optimize のみ)
```sh
curl 'http://localhost:8983/solr/collection1/update?commit=true' -H "Content-Type: text/xml" --data-binary @queries/2_delete.xml
```
```xml
<delete>
  <id>5</id>
</delete>
```

```
☁  ~  docker exec "$(docker ps | grep solr | awk '{print $1}')" ls -l /var/solr/data/collection1/data/index
_4.fdm
_4.fdt
_4.fdx
_4.fnm
_4.si
_4_1.liv
_4_Lucene80_0.dvd
_4_Lucene80_0.dvm
_4_Lucene84_0.doc
_4_Lucene84_0.tim
_4_Lucene84_0.tip
_4_Lucene84_0.tmd
segments_b
write.lock
```

`segments_a` が削除され、 `segments_b` が発生
中身は異なる

それ以外は一緒

3. delete (expunge delete)
```
curl 'http://localhost:8983/solr/collection1/update?commit=true&expungeDeletes=true' -H "Content-Type: text/xml" --data-binary @queries/3_delete.xml
```
```xml
<delete>
  <id>7</id>
</delete>
```

構成ファイルが変わった
