# Triple-DES

因為原本的DES安全性不足，而發展出此演算法，其包含 3 個 DES 金鑰，K1，K2和K3，均為 56 位元。

#### 加解密：

加密過程為：使用K1為金鑰進行DES加密，再用K2為金鑰進行DES解密，最後以K3進行DES加密，總共三步驟。

```
密文 = Encrypt-K3(Decrypt-K2(Encrypt-K1(明文)))
```

解密過程為：使用K3解密，再用K2加密，最後以K1解密。

```
明文 = Decrypt-K1(Encrypt-K2(Decrypt-K3(密文)))
```

#### 金鑰選擇：

可以有以下三種

```
1. 三個金鑰是獨立的，每個金鑰不同值。
2. K1 != K2，而 K3 = K1
3. 三個金鑰均相等，K1 = K2 = K3
```

#### 程式範例：

```js
const crypto = require('crypto');

const mode = 'des3'; // 也可用des-ede、des-ede3、des-ecb

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

使用區塊加密

> 每個DES key為8 bytes，所以3-DES 為三把key，需要 24 bytes。

```js
const crypto = require('crypto');

const mode = 'des-ede3-cbc'; // 可替換為des-ede3-cfb, des-ede3-ofb
const iv = Buffer.from(crypto.randomBytes(8))
const key = Buffer.from(crypto.randomBytes(24))
// 加密
const cipher = crypto.createCipheriv(mode, key, iv);
let encrypted = cipher.update('I_am_plaintext', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);

// 解密
const decipher = crypto.createDecipheriv(mode, key, iv);
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
```



