# Byte Padding

Hash function 與使用 Block cipher mode 的加密算法會需要把原本要加密的 plaintext (明文) 之 byte 補充到一定位數，例如 AES 有固定的 16 bytes data block size，所以會把要加密的明文分為很多個 16 bytes，最後剩下的再補齊 16 bytes。

> 可以參考區塊加密模式：
>
> [https://zh.wikipedia.org/zh-tw/%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F](https://zh.wikipedia.org/zh-tw/%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F)

### 範例

以 AES 使用 PKCS#7 為例，假設明文含有 18 bytes

```
F14ADBDA019D6DB7 EFD91546E3FF8444 9BCB
```

**padding 後會變為**

```
F14ADBDA019D6DB7 EFD91546E3FF8444 9BCB0E0E0E0E0E0E 
0E0E0E0E0E0E0E0E
```

> 因為需要 16bytes 一區塊，9BCB 後還需要 14 bytes (14 的16 進位為 0E)，所以按照  PKCS#7 的規規則為補充 14 個 14 到後面，所以有 14 個 0E，如果是需要 7 個 byte 則為補充七個 0x07 以此類推。

{% embed url="https://www.ibm.com/docs/en/zos/2.2.0?topic=method-pkcs-padding-example-1" %}

{% embed url="https://www.ibm.com/docs/en/zos/2.2.0?topic=rules-pkcs-padding-method" %}

## 其他 Padding 算法

除了 PKCS#5, PKCS#7 以外還包含 ANSI X9.23, ISO 10126, ISO/IEC 7816-4 等等。

{% embed url="https://en.wikipedia.org/wiki/Padding_(cryptography)#Byte_padding" %}
