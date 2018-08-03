# AES

Advanced Encryption Standard，縮寫：AES，又稱Rijndael加密法，但嚴格來說AES和Rijndael加密法並不完全一樣，因為Rijndael加密法可以支援更大範圍的區塊長度，AES的區塊長度固定為128位元而Rijndael使用的區塊長度可以是128，192或256位元。AES用來替代原先的DES。

AES的區塊長度固定為128位元，金鑰長度則可以是128，192或256位元；而Rijndael使用的金鑰和區塊長度均可以是128，192或256位元。

AES加密過程是在一個4×4的位元組矩陣上運作，其初值即為一個明文區塊，矩陣中一個元素大小就是明文區塊中的一個Byte。

其加密方法主要包含四個步驟

```
1. AddRoundKey: 將每個狀態中的位元組與該回合金鑰做異或（⊕）。
2. SubBytes:    矩陣中各位元組被固定的8位元尋找表中對應的特定位元組所替換。
3. ShiftRows:   矩陣中每一行的各個位元組循環向左方位移。位移量則隨著行數遞增而遞增。
4. MixColumns:  每個直行與 AES 定義的多項式 c(x) 進行多項式乘法。
```

> 以上參考至: [https://en.wikipedia.org/wiki/Advanced\_Encryption\_Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)

#### AES-256 範例

```js
const crypto = require('crypto');

const mode = 'aes256' // 可更換為aes128或aes192等等

// 加密
const cipher = crypto.createCipher(mode, 'a password');
let encrypted = cipher.update('I_am_plaintext', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);

// 解密
const decipher = crypto.createDecipher(mode, 'a password');
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
```

# AES之區塊加密模式

> 在CBC、OFB、CFB、CTR等區塊模式 IV 長度均為 16 bytes，GCM模式的 IV 則沒有一定要 16 bytes

#### AES-256-CBC範例

```js
const crypto = require('crypto');

// 因為aes-256要求之key 長度為256bits 也就是32 bytes = 32個ASCII英文字母
// aes-128 要求之key 長度為128bits 也就是16 bytes = 16個英文字母
let key = Buffer.from("abcbbbbbbbbbbbbbabcbbbbbbbbbbbbb", 'utf-8') //or crypto.randomBytes(32);

const IV_LENGTH = 16; 

function encrypt(text) {
    let iv = crypto.randomBytes(IV_LENGTH);
    // 可直接替換為ofb、cfb、ctr等模式
    let cipher = crypto.createCipheriv('aes-256-cbc', new Buffer(key), iv);
    let encrypted = cipher.update(text);
    encrypted = cipher.final();

    // 將IV附上 在解密時須告知
    return iv.toString('hex') + ':' + encrypted.toString('hex');
}

function decrypt(text) {
    let textParts = text.split(':');
    let iv = new Buffer(textParts.shift(), 'hex');
    let encryptedText = new Buffer(textParts.join(':'), 'hex');
    let decipher = crypto.createDecipheriv('aes-256-cbc', new Buffer(key), iv);
    let decrypted = decipher.update(encryptedText);

    decrypted = decipher.final();

    return decrypted.toString();
}

console.log(encrypt("test"));
console.log(decrypt(encrypt("test")))
```

#### AES-256-GCM範例

> 需要加上cipher.getAuthTag\(\); 與 decipher.setAuthTag\(\);

目前 AuthTag 與 Additional Authenticated Data \( AAD \) 在 Node.js 只有 GCM 模式支援，cipher.setAAD需要在 update\(\) 之前使用，而 cipher.getAuthTag\(\) 必須要在[`cipher.final()`](https://nodejs.org/api/crypto.html#crypto_cipher_final_outputencoding)執行後才能使用。

```js
const crypto = require('crypto');


const mode = 'aes-256-gcm';
// 因為aes-256要求之key 長度為256bits 也就是32 bytes = 32個ASCII英文字母
// aes-128 要求之key 長度為128bits 也就是16 bytes = 16個英文字母
let key = Buffer.from("abcbbbbbbbbbbbbbabcbbbbbbbbbbbbb", 'utf-8') //or crypto.randomBytes(32);

const IV_LENGTH = 12;
let tag;

function encrypt(text) {
    let iv = crypto.randomBytes(IV_LENGTH);

    let cipher = crypto.createCipheriv(mode, new Buffer(key), iv);
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    // 需要加上AuthTag
    tag = cipher.getAuthTag();
    // 將IV附上 在解密時須告知
    return iv.toString('hex') + ':' + encrypted.toString('hex');
}

function decrypt(text) {
    let textParts = text.split(':');
    let iv = new Buffer(textParts.shift(), 'hex');
    let encryptedText = new Buffer(textParts.join(':'), 'hex');
    let decipher = crypto.createDecipheriv(mode, new Buffer(key), iv);
    // 需要加上AuthTag
    decipher.setAuthTag(tag);
    let decrypted = decipher.update(encryptedText, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted.toString();
}

console.log(encrypt("test"));
console.log(decrypt(encrypt("test")))
```

> 其他Node.js的相關範例可參考
>
> [https://github.com/nodejs/node-v0.x-archive/blob/master/test/simple/test-crypto-authenticated.js\#L44-L64](https://github.com/nodejs/node-v0.x-archive/blob/master/test/simple/test-crypto-authenticated.js#L44-L64)

# OpenSSL範例

產生範例檔案

```
echo test > file.txt
```

加密\(執行後會要求輸入密碼\)

```
openssl enc -aes-256-cbc -salt -in file.txt -out file.txt.enc
```

解密\(執行後會要求輸入剛才加密的密碼\)

```
openssl enc -aes-256-cbc -d -in file.txt.enc -out result.txt
```



