# 4d-tips-curl-pop3
cURLプラグインを使用してメールを受信するサンプル

**はじめに**

* cURLはさまざまなURLプロトコルをサポートするクライアント・プログラムです。
* POP3はメール受信プロトコルです。[RFC1939](https://www.ietf.org/rfc/rfc1939.txt)
* cURLはURL版のPOP3をサポートしています。[RFC2384](https://www.ietf.org/rfc/rfc2384.txt)

###受信トレイのメッセージ一覧を取得する

```
C_BLOB($in;$out)
C_LONGINT($err)
ARRAY LONGINT($optionNames;0)
ARRAY TEXT($optionValues;0)

APPEND TO ARRAY($optionNames;CURLOPT_USERNAME)
APPEND TO ARRAY($optionValues;"user")
APPEND TO ARRAY($optionNames;CURLOPT_PASSWORD)
APPEND TO ARRAY($optionValues;"pass")

// or
// APPEND TO ARRAY($optionNames;CURLOPT_USERPWD)
// APPEND TO ARRAY($optionValues;"user:pass")

If (False)  //　default
	APPEND TO ARRAY($optionNames;CURLOPT_CUSTOMREQUEST)
	APPEND TO ARRAY($optionValues;"LIST")
End if 

$err:=cURL ("pop3://exchange.4d.com";$optionNames;$optionValues;$in;$out)
```

###ポイント①

ユーザー名とパスワードは，``CURLOPT_USERNAME``と``CURLOPT_PASSWORD``にそれぞれ渡すか，コロン記号で連結した文字列を``CURLOPT_USERPWD``に渡します。

**注記**: POP3のURL仕様は，平文（クリアテキスト）のパスワード記述を明示的に禁止しています。つまり，``pop3://user:pass@exchange.4d.com``のようにパスワードをURLに含めることはできません。とはいえ，cURLは，URL文字列を解析し，ユーザー名とパスワードを取り出すことができるように作られているので，URLにユーザー名とパスワードを含めても，実際にはPOP3の認証ができます。

https://curl.haxx.se/mail/lib-2011-10/0259.html

###ポイント②

POP3のコマンド名は，URLに基づいてcURLが自動的に発行します。URLのパスにコンポーネントが存在しない場合，つまりホスト名・ポート番号・認証メソッドなどが指定されただけの場合，自動的に``LIST``コマンドが発行されます。つまり，メッセージ番号とメッセージサイズの一覧を所得することになります。``CURLOPT_CUSTOMREQUEST``で``LIST``を指定する必要はありません。

メッセージの番号とサイズの一覧は，スペース区切り・``CRLF``改行で返されます。``Match regex``で配列に分解すると便利です。

```
$result:=Convert to text($out;"UTF-8")
$i:=1
ARRAY LONGINT($pos;0)
ARRAY LONGINT($len;0)
ARRAY TEXT($messageIds;0)
ARRAY TEXT($messageNumbers;0)
While (Match regex("(\\d+)\\s+(\\d+)";$result;$i;$pos;$len))
  APPEND TO ARRAY($messageNumbers;Substring($result;$pos{1};$len{1}))
  $i:=$pos{2}+$len{2}
End while 
```
