# PokeSugar
ポケモンを一行で表現する構文。



## バージョン
v0.1.0



# 目的
様々な場所でポケモンに関する議論が展開されていく中で、ポケモン自体の表し方が多様化してしまっている。  
本仕様ではそれを定義化することで統一を図り、人や機械にとって扱いやすい「ポケモンを表現する構文」を目指す。



## 仕様
構文規則はABNFに基づく。
```
* 構文
element              = *VCHAR
elements             = element *("/" element)

key                  = element
value                = element
map                  = key ":" value
maps                 = map *("," [SP] map)

name-block           = element
form-block           = "-" element
nature-block         = "!" element
ability-block        = "?" element
item-block           = "@" element
move-block           = "(" elements ")"
ev-block             = "{" elements "}"
iv-block             = "[" elements "]"
option-block         = "<" maps ">"

single-element-block = form-block / nature-block / ability-block / item-block
elements-block       = move-block / ev-block / iv-block / option-block

sub-block            = [SP] (single-element-block / elements-block)

poke-sugar           = name-block *sub-block

```

**例**
```
ラティオス!臆病?浮遊@拘り眼鏡(流星群/サイコショック/波乗り/めざパ炎){C252/S252}[H31/A30/B31/C30/D31/S30]
ラティオス!臆病?浮遊@拘り眼鏡(流星群/サイコショック/波乗り/めざパ炎){C252S252}[3VacsU]
ラティオス!臆病@拘り眼鏡(流星群/サイコショック/波乗り/めざパ炎){CSぶっぱ}[めざパ炎理想]

ランドロス-霊獣 !意地っ張り ?威嚇 @スカーフ (地震/蜻蛉返り/はたき落とす/馬鹿力) {ASぶっぱ}
ヒードラン !冷静 @拘り眼鏡 (噴火/オバヒ/ラスターカノン/大地の力) {HCぶっぱ} <レベル:100, 性別:♂>
```


## ブロック
ブロックは2種類に分類される。

- 種族ブロック: 種族
- サブブロック: 単要素ブロック, 要素群ブロック

サブブロックはさらに2種類に分類される。

- 単要素ブロック: フォルム, 性格, 特性, 持ち物
- 要素群ブロック: 技, 努力値, 個体値, オプション


### 種族ブロック (name-block)
種族ブロックは要素のみで構成される。  
ブロックの前への空白挿入は許容されない。
```
name-block = element
```

**例**
```
キングドラ
ニョロボン
ニョロトノ
```

### サブブロック (sub-block)
サブブロックは単要素ブロックまたは要素群ブロックで構成される。  
ブロックの前への空白挿入は1つまで許容される。
```
sub-block = [SP] (single-element-block / elements-block)
```

#### 単要素ブロック (single-element-block)
単要素ブロックは識別子と要素で構成される。  
識別子と要素の間への空白挿入は許容されない。
```
form-block           = "-" element
nature-block         = "!" element
ability-block        = "?" element
item-block           = "@" element

single-element-block = form-block / nature-block / ability-block / item-block
```

識別子には以下の4つが定義されている。
- ``-``: フォルム識別子
- ``!``: 性格識別子
- ``?``: 特性識別子
- ``@``: 持ち物識別子

要素には以下の4つが定義されている。
- フォルム要素
- 性格要素
- 特性要素
- 持ち物要素

**例**
```
-霊獣 !意地っ張り ?威嚇 @スカーフ
```


#### 要素群ブロック (elements-block)
要素群ブロックは括弧と要素群で構成される。  
括弧と要素群の間, また各要素の間への空白挿入は許容されない。
```
move-block     = "(" elements ")"
ev-block       = "{" elements "}"
iv-block       = "[" elements "]"
option-block   = "<" maps ">"

elements-block = move-block / ev-block / iv-block / option-block
```

括弧には以下の4つが定義されている。
- ``()``: 技括弧
- ``{}``: 努力値括弧
- ``[]``: 個体値括弧
- ``<>``: オプション括弧

要素群の各要素はスラッシュ``/``を使って区切る。  
要素数の指定はない。

**例**
```
(流星群/サイコショック/波乗り/めざパ炎){C252/S252}[H31/A30/B31/C30/D31/S30]
(流星群/サイコショック/波乗り/めざパ炎){CSぶっぱ}[めざパ炎理想]
```


## 要素
平仮名, カタカナ, 漢字, 英数字, 記号などの印字可能な文字で記述出来る。  
ただし識別子, 括弧, スラッシュ``/``, コロン``:``, コンマ``,``, シングルクオテーション``'``など、本仕様で定義されている記号を記述する際は``\``記号でエスケープする必要がある。  
要素は可読性を意識して一般的に認識されている単語で書き、全角英数字, 全角記号を含めないことが望ましい。
```
element = *VCHAR
```

**例**
```
バンギラス
りゅうせいぐん
流星群
龍星群
竜星群
10万ボルト
十万ボルト
Aのすがた
\!のすがた
\?のすがた
```

1つで複数の意味を表す単語も使用できる。
```
CSぶっぱ
CSぶっぱ余りH
5VaU
3VacsU
5V1U
3V3U
```

単語の種類や意味の解釈は本仕様では定義せず、利用側次第とする。


## オプションブロック
オプション括弧とオプション要素群で構成される。  
オプション要素自体はオプション名, コロン``:``, オプション値で構成され、各オプション要素はコンマ``,``で区切る。
```
key          = element
value        = element
map          = key ":" value
maps         = map *("," [SP] map)

option-block = "<" maps ">"
```

オプション名, オプション値に仕様出来る文字は[要素]の項と同じである。  
本仕様ではオプション名として予め以下が定義されている。
```
レベル
性別
```

このブロックは全ブロックの中で唯一利用側に拡張が許容されており、自由にオプション名, オプション値を追加出来る。  
以下に拡張例を示す。
```
仲良し度
出身地
出身国
表ID
裏ID
親
ニックネーム
懐き度
経験値
色
ポケルス
リボン
```

**例**
```
<レベル:100, 性別:♂>
<リボン:'ゴージャスリボン/ロイヤルリボン/ゴージャスロイヤルリボン', 色:通常色>
```



# ブロックの順序と省略
種族ブロックは必ず先頭でなければならないが、他のブロックの順序に指定はない。  
本仕様では可読性を重視して``フォルム``, ``性格``, ``特性``, ``持ち物``, ``技``, ``努力値``, ``個体値``, ``オプション``の順で書いてある。  
また、種族ブロックを除く他のブロックは省略可能である。

**例**
```
ラティオス!臆病?浮遊@拘り眼鏡(流星群/サイコショック/波乗り/めざパ炎){C252S252}[3VacsU]
ラティオス@拘り眼鏡?浮遊!臆病{C252S252}[3VacsU](流星群/サイコショック/波乗り/めざパ炎)
ラティオス[3VacsU]
ラティオス
```



# 利用側の対応
本仕様は利用側に各ブロックの対応を強制することは無い。  
利用側は必要に応じて各ブロックに対応すればよい。



# 用語
<dl>
    <dt>利用側</dt>
    <dd>本構文を読み取るプログラムやソフトウェアのこと。</dd>
</dl>



# Author
HN: mizdra  
Twitter: <https://twitter.com/mizdra>  
Blog: <http://mizdra.hatenablog.com>



# License
MIT License