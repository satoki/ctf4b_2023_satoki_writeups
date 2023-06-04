# polyglot4b

## 問題文
polyglotってなに？ たぶんpolyglotを作れるエディタを開発したよ！  
`nc polyglot4b.beginners.seccon.games 31416`  
[polyglot4b.tar.gz](files/polyglot4b.tar.gz) 747f374dc20897d039ab329d60d57cfe181d88d3  

## 難易度
**beginner**  

## 作問にあたって
[Ricerca CTF 2023](https://2023.ctf.ricsec.co.jp/)のps converterのpolyglotを試みていた時に思いつきました。  
サンプルファイルなどもあり、beginner向けになっていると思います。  

## 解法
接続すると以下のような、Polyglot Editorなる謎サービスが動いている。  
```bash
$ nc polyglot4b.beginners.seccon.games 31416
 ____       _             _       _     _____    _ _ _
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|
              |___/ |___/
--------------------------------------------------------------------
>> s
s
~~~
s
--------------------------------------------------------------------
| JPG: 🟥 | PNG: 🟥 | GIF: 🟥 | TXT: 🟩 |
FLAG: No! File mashimashi!!
```
入力を複数行受け取り、その種類を返すようだ。  
ソースを見ると主要部分は以下のようであった。  
```python
~~~
file = b""
for _ in range(10):
    text = sys.stdin.buffer.readline()
    if b"QUIT" in text:
        break
    file += text

print(f"{'-' * 68}\033[0m")

if len(file) >= 50000:
    print("ERROR: File size too large. (len < 50000)")
    sys.exit(0)

f_id = uuid.uuid4()
os.makedirs(f"tmp/{f_id}", exist_ok=True)
with open(f"tmp/{f_id}/{f_id}", mode="wb") as f:
    f.write(file)
try:
    f_type = subprocess.run(
        ["file", "-bkr", f"tmp/{f_id}/{f_id}"], capture_output=True
    ).stdout.decode()
except:
    print("ERROR: Failed to execute command.")
finally:
    shutil.rmtree(f"tmp/{f_id}")

types = {"JPG": False, "PNG": False, "GIF": False, "TXT": False}
if "JPEG" in f_type:
    types["JPG"] = True
if "PNG" in f_type:
    types["PNG"] = True
if "GIF" in f_type:
    types["GIF"] = True
if "ASCII" in f_type:
    types["TXT"] = True

for k, v in types.items():
    v = "🟩" if v else "🟥"
    print(f"| {k}: {v} ", end="")
print("|")

if all(types.values()):
    print("FLAG: ctf4b{****REDACTED****}")
else:
    print("FLAG: No! File mashimashi!!")
```
入力を予測不可能な名前のファイルとして保存し、`file -bkr`の結果に特定の文字列が含まれているかでファイルの種類を決定している。  
JPG、PNG、GIF、TXTのすべてである場合にフラグが得られるようだ。  
試しに以下のようにGIFのマジックナンバーを送信してみる。  
```bash
$ nc polyglot4b.beginners.seccon.games 31416
 ____       _             _       _     _____    _ _ _
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|
              |___/ |___/
--------------------------------------------------------------------
>> GIF87a
QUIT
--------------------------------------------------------------------
| JPG: 🟥 | PNG: 🟥 | GIF: 🟩 | TXT: 🟩 |
FLAG: No! File mashimashi!!
```
GIFとTXTのpolyglotであると判定された。  
fileコマンドを確認すると以下のように二種類の出力が得られている。  
```bash
$ echo 'GIF87a' | file -bkr -
GIF image data, version 87a,
- , ASCII text
```
ただし、JPG、PNG、GIF、TXTのすべてとなる出力を得ることは難しい。  
ここでfileコマンドにファイルのどこかの中身を出力させる手法を思いつく。  
例えば以下のようなshebangなどだ。  
```bash
$ echo '#!satoki' | file -bkr -
a satoki script, ASCII text executable
```
このようにファイルの中身に`JPEGPNGGIFASCII`のような文字列を含ませれば`in`による比較なので判定結果はTrueとなる。  
shebangを用いるか、sampleのsushi.jpgのimagedescriptionに`CTF4B`なる文字列があり、fileコマンドで出力されるとわかるので、こちらを編集してもよい。  
以下のように行う(Windowsを使用している場合はプロパティからGUIで変更できる)。  
```bash
$ exiftool -imagedescription=JPEGPNGGIFASCII sushi.jpg
    1 image files updated
$ file sushi.jpg
sushi.jpg: JPEG image data, Exif standard: [TIFF image data, big-endian, direntries=4, description=JPEGPNGGIFASCII], baseline, precision 8, 1404x790, components 3
$ cat sushi.jpg | nc polyglot4b.beginners.seccon.games 31416
 ____       _             _       _     _____    _ _ _
|  _ \ ___ | |_   _  __ _| | ___ | |_  | ____|__| (_) |_ ___  _ __
| |_) / _ \| | | | |/ _` | |/ _ \| __| |  _| / _` | | __/ _ \| '__|
|  __/ (_) | | |_| | (_| | | (_) | |_  | |__| (_| | | || (_) | |
|_|   \___/|_|\__, |\__, |_|\___/ \__| |_____\__,_|_|\__\___/|_|
              |___/ |___/
--------------------------------------------------------------------
>> --------------------------------------------------------------------
| JPG: 🟩 | PNG: 🟩 | GIF: 🟩 | TXT: 🟩 |
FLAG: ctf4b{y0u_h4v3_fully_und3r5700d_7h15_p0ly6l07}
```
flagが得られた。  

## ctf4b{y0u_h4v3_fully_und3r5700d_7h15_p0ly6l07}