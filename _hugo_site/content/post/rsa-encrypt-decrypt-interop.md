---
aliases:
    - /c#/objective-c/javascript/java/rsa/crypto/encryption/decryption/security/web/2015/03/15/rsa-encrypt-decrypt-interop.html
title:  "RSA Encryption and Decryption on the Web"
date:   2015-03-15
---

If you cannot enable SSL for whatever reason, you might still want to encrypt your sensitive information (such as passwords) over the wire. In this post I tried to put together a solution with working examples from various languages to make the client server communication secure using public cryptography with RSA.

<!--more-->

This post focuses on C# and Javascript interoperability. Below is a C# example using [Bouncycastle project](http://www.bouncycastle.org/csharp/):

{% highlight c# %}
const string PrivateKey = @"-----BEGIN RSA PRIVATE KEY-----
MIICXgIBAAKBgQDOFfwbqHOmQWYc50XxsR+dLyNUSwsaQ3tx225AvYEOs9bSS3VV
....
4/uGrlWiOG8EHeL1RUW/s5LezT1RFlL15RuSq4tHH/GI6w==
-----END RSA PRIVATE KEY-----
";

readonly Lazy<IAsymmetricBlockCipher> _cipher = new Lazy<IAsymmetricBlockCipher>(() =>
{
    var rsa = new Pkcs1Encoding(new RsaEngine());
    var pemReader = new PemReader(new StringReader(PrivateKey));
    var keyPair = (AsymmetricCipherKeyPair)pemReader.ReadObject();
    rsa.Init(false, keyPair.Private);
    return rsa;
});

string Decrypt(string base64Input)
{
    var buf = Convert.FromBase64String(base64Input);
    byte[] block = _cipher.Value.ProcessBlock(buf, 0, buf.Length);

    return Encoding.UTF8.GetString(block);
}
{% endhighlight %}

As for the client side I used the [JSEncrypt project](http://travistidwell.com/jsencrypt/) which has a extremely simple API:

{% highlight javascript %}
 var crypt = new JSEncrypt();
crypt.setPublicKey('-----BEGIN PUBLIC KEY-----....-----END PUBLIC KEY-----');
var enc = crypt.encrypt($scope.secret);
{% endhighlight %}

All the examples keys can be generated using [OpenSSL](https://www.openssl.org). You can also validate encryption and decryption results too.

{% highlight bash %}
# Generate new keys:
> openssl genrsa -out key.pem 1024
> openssl rsa -pubout -in key.pem -out public_key.pem

# Encrypt using public key:
> echo "Text to encript" | openssl rsautl -encrypt -inkey public_key.pem -pubin -out enc.bin
> openssl base64 -e -in out.enc -out enc.txt

# Decrypt using private key:
> openssl base64 -d -in enc.txt -out enc.bin
> openssl rsautl -decrypt -inkey key.pem -in enc.bin
{% endhighlight %}


It sounds like it could be easy to find other examples in various languages too. For Objective-C examples check out [Launckey documentation for encryption](https://launchkey.com/docs/api/encryption/objective-c/commoncrypto).

You can find a simple web application (in ASP.Net WebAPI) [under mtmk GitHub secpassx repository](https://github.com/mtmk/secpassx/tree/master/RsaInteropExample/SecPassXchange).

Enjoy
