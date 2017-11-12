# การ generate public key/private key

ถ้ายังไม่เข้าใจเรื่อง PKI (Public Key Infrastructure) แนะนำให้อ่านทฤษฎี PKI ก่อน

> http://na5cent.blogspot.com/2012/04/public-key-infrastructure.html  

ในที่นี้เราจะใช้เครื่องมือที่ชื่อว่า `openssl` ในการช่วย generate key  

### 1. generate private key format `.pem`

```shell
$ openssl genrsa -out private-key.pem 2048
```

### 2. generate public key จาก `private-key.pem`

```shell
$ openssl rsa -in private-key.pem -pubout > public-key.pub
```

### 3. ทำการแปลง private key format `.pem` ให้ไปเป็น format `.der` 

```shell
$ openssl pkcs8 -topk8 -inform PEM -outform DER -in private-key.pem -out private-key.der -nocrypt
```

### 4. generate public key จาก `private-key.pem` format `.der`
```shell
$ openssl rsa -in private-key.pem -pubout -outform DER -out public-key.der
```

ตอนนี้เรามี key file ทั้งหมด 4 file ได้แก่

- private-key.pem
- private-key.der
- public-key.pub
- public-key.der
  
  
`.pem` กับ `.pub` ทำไว้ เผื่อระบบอื่นเอาไปใช้  
ส่วน `.der` เราจะเอามาใช้ใน java ของเรา

# Example Code
RSAPrivateKeyReader.java
```java
/*
 * Copyright 2017 Pamarin.com
 */
package com.pamarin.oauth2.security;

import com.pamarin.oauth2.exception.RSAKeyReaderException;
import java.io.IOException;
import java.io.InputStream;
import java.security.KeyFactory;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPrivateKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import org.apache.commons.io.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * @author jittagornp &lt;http://jittagornp.me&gt; create : 2017/11/11
 */
@Component
class RSAPrivateKeyReaderImpl implements RSAPrivateKeyReader {

    private static final Logger LOG = LoggerFactory.getLogger(RSAPrivateKeyReaderImpl.class);

    @Override
    public RSAPrivateKey readFromDERFile(InputStream inputStream) {
        try {
            if (inputStream == null) {
                throw new RSAKeyReaderException("Required inputStream.");
            }
            byte[] keyBytes = IOUtils.toByteArray(inputStream);
            PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(keyBytes);
            KeyFactory kf = KeyFactory.getInstance("RSA");
            return (RSAPrivateKey) kf.generatePrivate(spec);
        } catch (IOException | NoSuchAlgorithmException | InvalidKeySpecException ex) {
            LOG.warn(null, ex);
            throw new RSAKeyReaderException(ex);
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException ex) {
                    LOG.warn(null, ex);
                }
            }
        }
    }

}
```
RSAPublicKeyReader.java
```java
/*
 * Copyright 2017 Pamarin.com
 */
package com.pamarin.oauth2.security;

import com.pamarin.oauth2.exception.RSAKeyReaderException;
import java.io.IOException;
import java.io.InputStream;
import java.security.KeyFactory;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.X509EncodedKeySpec;
import org.apache.commons.io.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * @author jittagornp &lt;http://jittagornp.me&gt; create : 2017/11/11
 */
@Component
class RSAPublicKeyReaderImpl implements RSAPublicKeyReader {

    private static final Logger LOG = LoggerFactory.getLogger(RSAPublicKeyReaderImpl.class);

    @Override
    public RSAPublicKey readFromDERFile(InputStream inputStream) {
        try {
            if (inputStream == null) {
                throw new RSAKeyReaderException("Required inputStream.");
            }
            byte[] keyBytes = IOUtils.toByteArray(inputStream);
            X509EncodedKeySpec spec = new X509EncodedKeySpec(keyBytes);
            KeyFactory kf = KeyFactory.getInstance("RSA");
            return (RSAPublicKey) kf.generatePublic(spec);
        } catch (IOException | NoSuchAlgorithmException | InvalidKeySpecException ex) {
            LOG.warn(null, ex);
            throw new RSAKeyReaderException(ex);
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException ex) {
                    LOG.warn(null, ex);
                }
            }
        }
    }

}
```
### ตัวอย่างการใช้งาน

https://github.com/jittagornp/oauth2/blob/master/src/test/java/com/pamarin/oauth2/security/JwtSignerTest.java  
