# RSA

1977年由 Ron Rivest、Adi Shamir 和 Leonard Adleman 一起提出的。而RSA就是他們三人姓氏開頭字母拼在一起所組成。

而RSA的安全性建構在對一個極大的整數很難對其因數分解的情況下。

到目前為止，世界上還沒有任何可靠的攻擊RSA演算法的方式。只要其鑰匙的長度足夠長，用RSA加密的訊息實際上是不能被破解的。

RSA密鑰一般是 1024 bits 或 2048 bits。

---

## 原理

第一步: 隨便找兩個質數a與b

> 質數表 [https://zh.wikipedia.org/wiki/質數列表](https://zh.wikipedia.org/wiki/質數列表)

```
這裡假設a為23，而b為29
```

第二步: 計算φ\(a x b\)

> 歐拉函數φ\(n\)的意思為：小於n並與n互質之數的數量。

```
23x29 = 667  
φ(667)根據歐拉函數得到為 (23-1) * (29-1) = 616
```

第三步: 隨便找一個數字e，使得1&lt; e &lt; φ\(axb\)的數字，且e與φ\(axb\)互質

```
這裡選19
```

第四步: 找到一個數字d，使得 ed ≡ 1 \(mod φ\(n\)\)        意思為尋找 \(19 \* d\) % 616  = 1

> 即為找出 e 對於模數 **φ\(n\) **的模反元素 d

```
//以上面為例子
function te() {
  let X = 0
  while((19 * X) % 616 !== 1) {
     X++;
   }
   return X
 }

 //回傳227
```

第五步: 剛才找出n = 667,  e = 19 , d = 227

得到公鑰為\(667, 19\)  私鑰為\(667, 227\)

> 如果 n 可以輕易被因數分解，d 就可以算出，而私鑰將被破解
>
> 一般來說公私鑰會以ASN.1格式呈現。

#### 加密

假設字串是ABC，轉為**ASCII 為65,66,67 **

```
65,66,67
```

而要加密的明文之輸入數字過大也會產生解密後和輸入數字不同之情況 \( 因涉及Big Integer \)

> 建議將密文以單字為單位轉為ASCII然後個別存入陣列轉換

所以我們把ABC分為三個65,66,67分別加解密再組合起來

1.先算字母A

這時加密的公鑰匙為\(667, 19\) 所以加密方式為

```
(65 的 19 次方) % 667
```

> 這裡因為數字是Big Integer所以算的時候不可直接用...\*\*...% ...，需要用相關Big Integer模組不然會產生計算錯誤
>
> 而Python內建Big Integer但Javascript沒有，所以直接用Javascript計算出來的數字會錯誤。

```
如果已經安裝好Python，打開Terminal輸入Python，進入命令列，然後輸入：

(65 ** 19 ) % 667
```

得到數字451

> 有關大數的次方計算可參考：快速取模運算\(fast-modular-exponentiation\)

#### 解密

私鑰匙是\(667, 227\)

現在要把剛才的數字 451 解密

```
(451 ** 227) % 667
```

這裡牽涉到Big integer時常會出現infinity

> 有關大數的次方計算可參考：快速取模運算\(fast-modular-exponentiation\)

最後得到剛才的數字65即為解密完成。

#### 簡單完整範例：

以下為用JS的big-integer模組，分別計算字母再連接的Function

```
npm install big-integer
```

```js
var bigInt = require("big-integer");
let msg = "ABC";
let final = [];

msg.split('').forEach(char => {     // 每個字母分為Array一個元素
  let ascii = char.charCodeAt(0);  // 轉為ASCII
  let a = bigInt(ascii).pow(19).mod(667);   // 用公鑰加密
  let b = bigInt(a).pow(227).mod(667);    //// 用私鑰解密
  final.push(b.toString());
})

let decrypt = final.map(d => {
  return String.fromCharCode(parseInt(d)); // 轉為string
})

console.log(decrypt);
```

# 實際使用RSA：

> 上面的範例為概念教學，以下我們會來講實際使用已經寫好的模組來應用到專案中。

# OpenSSL之Encrypt與Decrypt

1.產生一個private key

> 預設產生長度為 512 bit  在最後可加數字1024 or 4096 產生不同長度私鑰

```
 openssl genrsa -out private.pem
```

2.產生PEM格式的公鑰

```
openssl rsa -in private.pem -out public.pem -outform PEM -pubout
```

3.產生一個稍後用來加密的檔案

```
echo 'some data' > test.txt

然後檔案內容隨意輸入
```

4.用公鑰加密test.txt檔案，將會產生encrypt.rsa檔案

```
openssl rsautl -encrypt -inkey public.pem -pubin -in test.txt -out encrypt.rsa
```

> 如檔案大小過大會產生錯誤

5.用私鑰解密檔案

```
openssl rsautl -decrypt -inkey private.pem -in encrypt.rsa -out decrypt.txt
```

6.最後查看解密後的檔案內容

```
cat decrypt.txt
```

即可看到成功還原為原本檔案之檔案內容

# Node.js之Encrypt與Decrypt

這邊我們使用第三方模組『 node-rsa 』

[https://github.com/rzcoder/node-rsa](https://github.com/rzcoder/node-rsa)

安裝：

```
npm install node-rsa
```

使用：

```js
var NodeRSA = require('node-rsa');
var key = new NodeRSA({b: 1024}); // 產生1024bits的金鑰

var text = 'Hello!';

// 加密
var encrypted = key.encrypt(text, 'base64');
console.log('encrypted: ', encrypted);

// 解密
var decrypted = key.decrypt(encrypted, 'utf8');
console.log('decrypted: ', decrypted);
```

---

# OpenSSL之Sign與Verify

1.產生PEM格式的私鑰

> 預設產生長度為 512 bit  在最後可加數字1024 、2048或 4096 產生不同長度私鑰

```
 openssl genrsa -out private.pem
```

1.從剛才的私鑰來產生對應的PEM格式之公鑰

```
openssl rsa -in private.pem -out public.pem -outform PEM -pubout
```

2.簽名

```
echo -n "hello" | openssl dgst -RSA-SHA256 -sign private.pem > signed_message
```

3.驗證

```
echo -n "hello" | openssl dgst -RSA-SHA256 -verify public.pem -signature signed_message
```

成功會回傳

> ```
> Verified OK
> ```

#### 使用Node.js驗證

```js
var fs = require('fs');
var crypto = require('crypto');

var verify = crypto.createVerify('RSA-SHA256');
verify.update('hello');
var res = verify.verify(fs.readFileSync('./public.pem'),
                        fs.readFileSync('./signed_message'));
console.log(res);
```

> 上面程式為在OpenSSL用私鑰簽名後給Node.js驗證

---

下面程式為全部使用Node.js簽名與驗證

> 但pem格式的key記得要先用OpenSSL產生

```js
const crypto = require('crypto');
const fs = require('fs');
var private_key = fs.readFileSync('./private.pem')
var public_key = fs.readFileSync('./public.pem')
private_key = private_key.toString();
public_key = public_key.toString();

const sign = crypto.createSign('RSA-SHA256');
sign.update('test');
signature = sign.sign(private_key, 'hex');

const verify = crypto.createVerify('RSA-SHA256');
verify.update('test');
output = verify.verify(public_key, signature, 'hex');
console.log(output)
```

# 相關格式

---

公鑰格式

```
1. pkcs1 
公鑰開頭為 '-----BEGIN RSA PUBLIC KEY-----' 而私鑰開頭為 '-----BEGIN RSA PRIVATE KEY-----' 

2. pkcs8
公鑰開頭為 '-----BEGIN PUBLIC KEY-----' 而私鑰開頭為 '-----BEGIN PRIVATE KEY-----' 

其他的公鑰加密標準可參考：https://en.wikipedia.org/wiki/PKCS
```

編碼格式

```
DER (Distinguished Encoding Rules)：
二進位內容，屬於 ASN.1 制定的編碼之一

PEM (Privacy-enhanced Electronic Mail)：
為 DER 格式經過 BASE64 編碼後，通常會有 ----- BEGIN XXX ----- / ----- END XXX ----- 之類的東西包夾起來。

由於 PEM 格式採用了 BASE64 編碼，文字都被編成常用的英文數字符號，方便於網路上傳送及複製 (如即時通訊、電子郵件等)。
OpenSSL 預設產生的檔案都是 PEM 格式。
```



