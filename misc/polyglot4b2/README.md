# polyglot4b2

## 問題文
より美しいpolyglotを作れるエディタを開発したよ！ やっぱりpolyglotは簡単に作れる？  
`nc polyglot4b2.beginners.seccon.games 31417`  
[polyglot4b2.tar.gz](files/polyglot4b2.tar.gz) 3ef2f2521482f5845eb03638fa4cc7e7130ecd41  

## 難易度
**medium**  

## 作問にあたって
polyglot4bをpwnyaaさんと捏ねていたら、難しくなったので出題しました。  
若干のミスリードが入っており、意地悪なmedium問題となっています。  
実はクオリティに満足しておらず、前日まで出すか迷っていました。  

## 解法
[polyglot4b](../polyglot4b)の続き問題のようだ。  
接続すると以下のように中身が少し変わっている。  
```bash
$ nc polyglot4b2.beginners.seccon.games 31417
 ____       _             _       _     _____    _ _ _               ____
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __  |___ \
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|   __) |
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |     / __/
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|    |_____|
              |___/ |___/
----------------------------------------------------------------------------
>> s
ERROR: Hi, Hacker.
$ nc polyglot4b2.beginners.seccon.games 31417
 ____       _             _       _     _____    _ _ _               ____
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __  |___ \
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|   __) |
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |     / __/
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|    |_____|
              |___/ |___/
----------------------------------------------------------------------------
>> S
S
~~
S
----------------------------------------------------------------------------
ERROR: You are not a beginner.
| JPG: 🟥 | PNG: 🟥 | GIF: 🟥 | PDF: 🟥 | ELF: 🟥 | TXT: 🟥 |
FLAG: No! File koime!!
```
アルファベット大文字しか受け付けないようで、ファイルの種類もPDF、ELFが追加されている。  
ソースは主に以下の個所が変更されていた。  
```python
~~~
file = ""
for _ in range(10):
    text = sys.stdin.buffer.readline().decode()
    if not re.fullmatch("[A-Z]+", text.replace("\n", "")):
        print("ERROR: Hi, Hacker.")
        sys.exit(0)
    if "QUIT" in text:
        break
    file += text
~~~
# You are a beginner!!!!
_4bflag = True
if "4b" not in f_type:
    print("ERROR: You are not a beginner.")
    _4bflag = False
~~~
f_type = f_type.split("\n")
try:
    if "JPEG" in f_type[0]:
        types["JPG"] = True
    if "PNG" in f_type[1]:
        types["PNG"] = True
    if "GIF" in f_type[2]:
        types["GIF"] = True
    if "PDF" in f_type[3]:
        types["PDF"] = True
    if "ELF" in f_type[4]:
        types["ELF"] = True
    if "ASCII" in f_type[5]:
        types["TXT"] = True
except:
    pass
~~~
if _4bflag and all(types.values()):
    print("FLAG: ctf4b{****REDACTED****}")
else:
    print("FLAG: No! File koime!!")
```
今回は文字列`4b`がfileコマンドの結果に含まれている必要があり、さらにファイルの種類チェックが一行ずつとなっている。  
polyglot4bの解法は、出力が一行となってしまうので利用できない。  
まず初めに、fileコマンドの出力を複数行にできるものを探す。  
fileコマンドは[ここ](https://github.com/file/file/tree/master/magic/Magdir)でファイルの種類を判定しており、出力も書かれている。  
雑に`%s`とすると改行を含めることができない種類もヒットするため、`'%s'`で以下のように検索する。  
```bash
$ grep -rl "'%s'"
aes
att3b
cisco
database
digital
elf
filesystems
freebsd
gnu
hp
icc
images
linux
maple
msdos
netbsd
olf
os2
pmem
sgi
sinclair
sun
ti-8x
varied.out
windows
zfs
```
この中からマジックナンバーがアルファベット大文字のみであるものを調査すると`pmem`がヒットする(おそらくこれだけ)。  
中身をみると出力の構成が書かれているため、こちらを利用して一行ずつのチェックを突破する。  
`pmem`では文字列`PMEMOBJ`の後に4089文字追加すると、その後の文字がfileコマンドの結果に改行込みで表示される。  
以下のように`S`でパディングを行い、送信する。  
```bash
$ python3 -c 'print("PMEMOBJ"+"S"*4089+"JPEG\nPNG\nGIF\nPDF\nELF\nASCII\nQUIT")' | nc polyglot4b2.beginners.seccon.games 31417
 ____       _             _       _     _____    _ _ _               ____
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __  |___ \
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|   __) |
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |     / __/
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|    |_____|
              |___/ |___/
----------------------------------------------------------------------------
>> ----------------------------------------------------------------------------
ERROR: You are not a beginner.
| JPG: 🟩 | PNG: 🟩 | GIF: 🟩 | PDF: 🟩 | ELF: 🟩 | TXT: 🟩 |
FLAG: No! File koime!!
```
これによりpolyglotの作成に成功するが、`4b`が結果に含まれていない場合に`You are not a beginner.`と怒られる。  
入力にはアルファベット大文字のみが使用できるので、何らかの別の出力を利用する必要がある。  
ここで先ほどのpolyglotのfileコマンドの結果を詳細に見てみると以下のようであった。  
```bash
$ python3 -c 'print("PMEMOBJ"+"S"*4089+"JPEG\nPNG\nGIF\nPDF\nELF\nASCII\nQUIT")' | file -bkr -
Persistent Memory Pool file, type: OBJ, version: 0x53535353, compat: 0x53535353, incompat: 0x53535353, ro_compat: 0x53535353, crtime: *Invalid time*, alignment_desc: 0x5353535353535353, machine_class: unknown (83), data: unknown (83), reserved[0]: 83, reserved[1]: 83, reserved[2]: 83, reserved[3]: 83, machine: unknown (21331), obj.layout: 'JPEG
PNG
GIF
PDF
ELF
ASCII
QUIT
'
- , ASCII text, with very long lines (4100)
```
結果の所々に`0x53535353`なる表示が見えることに気づく。  
これはASCIIで`SSSS`を意味し、4089文字のパディングに使った文字のようだ。  
この個所を`0x4b4b4b4b`にできそうであり、幸いなことにASCIIで`4b`の`K`は大文字アルファベットである。  
パディングを`K`に変更し、送信を行う。  
```bash
$ python3 -c 'print("PMEMOBJ"+"K"*4089+"JPEG\nPNG\nGIF\nPDF\nELF\nASCII\nQUIT")' | nc polyglot4b2.beginners.seccon.games 31417                                                                                      ____       _             _       _     _____    _ _ _               ____
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __  |___ \
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|   __) |
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |     / __/
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|    |_____|
              |___/ |___/
----------------------------------------------------------------------------
>> ----------------------------------------------------------------------------
| JPG: 🟩 | PNG: 🟩 | GIF: 🟩 | PDF: 🟩 | ELF: 🟩 | TXT: 🟩 |
FLAG: ctf4b{y0u_54y_p0ly6l07 h45_n07h1n6_70_d0_w17h_17?}
```
flagが得られた。  

## ctf4b{y0u_54y_p0ly6l07 h45_n07h1n6_70_d0_w17h_17?}