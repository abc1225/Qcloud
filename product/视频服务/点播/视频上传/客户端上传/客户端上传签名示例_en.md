## PHP
```php
<?php
// Determine the cloud API secret key for the App
$secret_id = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX";
$secret_key = "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA";

// Determine the current time and expiration time for the signature
$current = time();
$expired = $current + 86400;  // Validity period: 1 day

// Enter parameters into the parameter list
$arg_list = array(
	"secretId" => $secret_id,
	"currentTimeStamp" => $current,
	"expireTime" => $expired,
	"random" => rand());

// Calculate signature
$orignal = http_build_query($arg_list);
$signature = base64_encode(hash_hmac('SHA1', $orignal, $secret_key, true).$orignal);

echo $signature;
echo "\n";
?>
```

## java
```java
import java.util.Random;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import sun.misc.BASE64Encoder;

public class Signature {
	public String m_strSecId;
	public String m_strSecKey;
	public long m_qwNowTime;
	public int m_iRandom;
	public int m_iSignValidDuration;

	private static final String HMAC_ALGORITHM = "HmacSHA1";
	private static final String CONTENT_CHARSET = "UTF-8";

	public static byte[] byteMerger(byte[] byte_1, byte[] byte_2) {
		byte[] byte_3 = new byte[byte_1.length + byte_2.length];
		System.arraycopy(byte_1, 0, byte_3, 0, byte_1.length);
		System.arraycopy(byte_2, 0, byte_3, byte_1.length, byte_2.length);
		return byte_3;
	}

	String GetUploadSignature() {
		String strSign = "";
		String contextStr = "";
		long endTime = (m_qwNowTime + m_iSignValidDuration);
		try {
			contextStr += "secretId=" + java.net.URLEncoder.encode(this.m_strSecId, "utf8");
			contextStr += "&currentTimeStamp=" + this.m_qwNowTime;
			contextStr += "&expireTime=" + endTime;
			contextStr += "&random=" + this.m_iRandom;

			String s = contextStr;
			String sig = null;
			Mac mac = Mac.getInstance(HMAC_ALGORITHM);
			SecretKeySpec secretKey = new SecretKeySpec(m_strSecKey.getBytes(CONTENT_CHARSET), mac.getAlgorithm());
			mac.init(secretKey);
			byte[] hash = mac.doFinal(contextStr.getBytes(CONTENT_CHARSET));
			byte[] sigBuf = byteMerger(hash, contextStr.getBytes("utf8"));
			strSign = new String(new BASE64Encoder().encode(sigBuf).getBytes());
			strSign = strSign.replace(" ", "").replace("\n", "").replace("\r", "");
		} catch (Exception e) {
			System.out.print(e.toString());
			return "";
		}
		return strSign;
	}
}

class Test {
	public static void main(String[] args) {
		Signature sign = new Signature();
		sign.m_strSecId = "Secret Id in the personal API secret key";
		sign.m_strSecKey = "Secret Key in the personal API secret key";
		sign.m_qwNowTime = System.currentTimeMillis() / 1000;
		sign.m_iRandom = new Random().nextInt(java.lang.Integer.MAX_VALUE);
		sign.m_iSignValidDuration = 3600 * 24 * 2;
		
		System.out.print(sign.GetUploadSignature());
	}
}
```

***Note***

* You need to import third-party packages javax-crpyto.jar and sun.misc.BASE64Encoder.jar.
* If the error "Access restriction" occurred when importing the third-party package sun.misc.BASE64Encoder.jar, you can adjust error level to solve it 
```
Windows -> Preferences -> Java -> Compiler -> Errors/Warnings -> Deprecated and trstricted API -> Forbidden reference (access rules): -> change to warning
```

## Node.js
```javascript
var querystring = require("querystring");
var crypto = require('crypto');

// Determine the cloud API secret key for the App
var secret_id = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX";
var secret_key = "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA";

// Determine the current time and expiration time for the signature
var current = parseInt((new Date()).getTime() / 1000)
var expired = current + 86400;  // Validity period: 1 day

// Enter parameters into the parameter list
var arg_list = {
	secretId : secret_id,
	currentTimeStamp : current,
	expireTime : expired,
	random : Math.round(Math.random() * Math.pow(2, 32))
}

// Calculate signature
var orignal = querystring.stringify(arg_list);
var orignal_buffer = new Buffer(orignal, "utf8");

var hmac = crypto.createHmac("sha1", secret_key);
var hmac_buffer = hmac.update(orignal_buffer).digest();

var signature = Buffer.concat([hmac_buffer, orignal_buffer]).toString("base64");

console.log(signature);
```

##  C#
```csharp
using System;
using System.Security.Cryptography;
using System.Text;
using System.Threading;

class Signature
{
    public string m_strSecId;
    public string m_strSecKey;
    public int m_iRandom;
    public long m_qwNowTime;
    public int m_iSignValidDuration;
    public static long GetIntTimeStamp()
    {
        TimeSpan ts = DateTime.UtcNow - new DateTime(1970, 1, 1);
        return Convert.ToInt64(ts.TotalSeconds);
    }
    private byte[] hash_hmac_byte(string signatureString, string secretKey)
    {
        var enc = Encoding.UTF8; HMACSHA1 hmac = new HMACSHA1(enc.GetBytes(secretKey));
        hmac.Initialize();
        byte[] buffer = enc.GetBytes(signatureString);
        return hmac.ComputeHash(buffer);
    }
    public string GetUploadSignature()
    {
        string strContent = "";
        strContent += ("secretId=" + Uri.EscapeDataString((m_strSecId)));
        strContent += ("&currentTimeStamp=" + m_qwNowTime);
        strContent += ("&expireTime=" + (m_qwNowTime + m_iSignValidDuration));
        strContent += ("&random=" + m_iRandom);

        byte[] bytesSign = hash_hmac_byte(strContent, m_strSecKey);
        byte[] byteContent = System.Text.Encoding.Default.GetBytes(strContent);
        byte[] nCon = new byte[bytesSign.Length + byteContent.Length];
        bytesSign.CopyTo(nCon, 0);
        byteContent.CopyTo(nCon, bytesSign.Length);
        return Convert.ToBase64String(nCon);
    }
}
class Program
{
    static void Main(string[] args)
    {
        Signature sign = new Signature();
        sign.m_strSecId = "Secret Id in the personal API secret key";
        sign.m_strSecKey = "Secret Key in the personal API secret key";
        sign.m_qwNowTime = Signature.GetIntTimeStamp();
        sign.m_iRandom = new Random().Next(0, 1000000);
        sign.m_iSignValidDuration = 3600 * 24 * 2;

        Console.WriteLine(sign.GetUploadSignature());
    }
}

```
