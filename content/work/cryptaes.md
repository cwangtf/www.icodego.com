---
title: "PHP与Java通用AES加密解密算法"
date: 2018-06-30T19:52:46+08:00
categories: ["算法"]
tags: ["PHP", "Java", "算法"]
---

最近项目中遇到与第三方对接业务，涉及到签名与密码加密AES算法，而两方用的语言不一样，一方是PHP，另一方是Java，对接过程中遇到了算法不一致的问题

AES指高级加密标准（Advanced Encryption Standard）,是当前最流行的一种密码算法，在web应用开发，特别是对外提供接口时经常会用到

下面是查询资料及对接过程中找到可行的一套PHP与Java通用的AES加密解密算法，Mark一下

* PHP代码：

```
<?php
class CryptAES
{
    protected $cipher = MCRYPT_RIJNDAEL_128;
    protected $mode = MCRYPT_MODE_ECB;
    protected $pad_method = NULL;
    protected $secret_key = '';
    protected $iv = '';
 
    public function set_cipher($cipher)
    {
        $this->cipher = $cipher;
    }
 
    public function set_mode($mode)
    {
        $this->mode = $mode;
    }
 
    public function set_iv($iv)
    {
        $this->iv = $iv;
    }
 
    public function set_key($key)
    {
        $this->secret_key = $key;
    }
 
    public function require_pkcs5()
    {
        $this->pad_method = 'pkcs5';
    }
 
    protected function pad_or_unpad($str, $ext)
    {
        if ( is_null($this->pad_method) )
        {
            return $str;
        }
        else
        {
            $func_name = __CLASS__ . '::' . $this->pad_method . '_' . $ext . 'pad';
            if ( is_callable($func_name) )
            {
                $size = mcrypt_get_block_size($this->cipher, $this->mode);
                return call_user_func($func_name, $str, $size);
            }
        }
        return $str;
    }
 
    protected function pad($str)
    {
        return $this->pad_or_unpad($str, '');
    }
 
    protected function unpad($str)
    {
        return $this->pad_or_unpad($str, 'un');
    }
 
    public function encrypt($str)
    {
        $str = $this->pad($str);
        $td = mcrypt_module_open($this->cipher, '', $this->mode, '');
 
        if ( empty($this->iv) )
        {
            $iv = @mcrypt_create_iv(mcrypt_enc_get_iv_size($td), MCRYPT_RAND);
        }
        else
        {
            $iv = $this->iv;
        }
 
        mcrypt_generic_init($td, $this->secret_key, $iv);
        $cyper_text = mcrypt_generic($td, $str);
        $rt=base64_encode($cyper_text);
        //$rt = bin2hex($cyper_text);
        mcrypt_generic_deinit($td);
        mcrypt_module_close($td);
 
        return $rt;
    }
 
    public function decrypt($str){
        $td = mcrypt_module_open($this->cipher, '', $this->mode, '');
 
        if ( empty($this->iv) )
        {
            $iv = @mcrypt_create_iv(mcrypt_enc_get_iv_size($td), MCRYPT_RAND);
        }
        else
        {
            $iv = $this->iv;
        }
 
        mcrypt_generic_init($td, $this->secret_key, $iv);
        //$decrypted_text = mdecrypt_generic($td, self::hex2bin($str));
        $decrypted_text = mdecrypt_generic($td, base64_decode($str));
        $rt = $decrypted_text;
        mcrypt_generic_deinit($td);
        mcrypt_module_close($td);
 
        return $this->unpad($rt);
    }
 
    public static function hex2bin($hexdata) {
        $bindata = '';
        $length = strlen($hexdata);
        for ($i=0; $i amp;< $length; $i += 2)
        {
            $bindata .= chr(hexdec(substr($hexdata, $i, 2)));
        }
        return $bindata;
    }
 
    public static function pkcs5_pad($text, $blocksize)
    {
        $pad = $blocksize - (strlen($text) % $blocksize);
        return $text . str_repeat(chr($pad), $pad);
    }
 
    public static function pkcs5_unpad($text)
    {
        $pad = ord($text{strlen($text) - 1});
        if ($pad > strlen($text)) return false;
        if (strspn($text, chr($pad), strlen($text) - $pad) != $pad) return false;
        return substr($text, 0, -1 * $pad);
    }
}
 
$keyStr = 'UITN25LMUQC436IM';
$plainText = 'this is a string will be AES_Encrypt';
 
$aes = new CryptAES();
$aes->set_key($keyStr);
$aes->require_pkcs5();
$encText = $aes->encrypt($plainText);
$decString = $aes->decrypt($encText);
 
echo $encText,"n",$decString;
 
?>
```
运行结果：  
fhTD0NNIzv4jUEhJuC1htFFXJ/4S/rL6tDCJPiNvJ8mVLHWOD0HWweuxHynxoZf9  
this is a string will be AES_Encrypt

* Java代码：

```
import java.security.Key; 
import javax.crypto.Cipher; 
import javax.crypto.spec.SecretKeySpec; 
 
import org.apache.commons.codec.binary.Base64;
  
public class CryptAES { 
  
    private static final String AESTYPE ="AES/ECB/PKCS5Padding"; 
 
    public static String AES_Encrypt(String keyStr, String plainText) { 
        byte[] encrypt = null; 
        try{ 
            Key key = generateKey(keyStr); 
            Cipher cipher = Cipher.getInstance(AESTYPE); 
            cipher.init(Cipher.ENCRYPT_MODE, key); 
            encrypt = cipher.doFinal(plainText.getBytes());     
        }catch(Exception e){ 
            e.printStackTrace(); 
        }
        return new String(Base64.encodeBase64(encrypt)); 
    } 
 
    public static String AES_Decrypt(String keyStr, String encryptData) {
        byte[] decrypt = null; 
        try{ 
            Key key = generateKey(keyStr); 
            Cipher cipher = Cipher.getInstance(AESTYPE); 
            cipher.init(Cipher.DECRYPT_MODE, key); 
            decrypt = cipher.doFinal(Base64.decodeBase64(encryptData)); 
        }catch(Exception e){ 
            e.printStackTrace(); 
        } 
        return new String(decrypt).trim(); 
    } 
 
    private static Key generateKey(String key)throws Exception{ 
        try{            
            SecretKeySpec keySpec = new SecretKeySpec(key.getBytes(), "AES"); 
            return keySpec; 
        }catch(Exception e){ 
            e.printStackTrace(); 
            throw e; 
        } 
 
    } 
 
    public static void main(String[] args) { 
         
        String keyStr = "UITN25LMUQC436IM";  
 
        String plainText = "this is a string will be AES_Encrypt";
         
        String encText = AES_Encrypt(keyStr, plainText);
        String decString = AES_Decrypt(keyStr, encText); 
         
        System.out.println(encText); 
        System.out.println(decString); 
 
    } 
}
```
运行结果：  
fhTD0NNIzv4jUEhJuC1htFFXJ/4S/rL6tDCJPiNvJ8mVLHWOD0HWweuxHynxoZf9  
this is a string will be AES_Encrypt