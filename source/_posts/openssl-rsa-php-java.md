title: openssl生成密钥 php和java 相互加密解密
date: 2015-04-14 14:25:02
tags:
- rsa
- 非对称加密
category:
- php
---


参考资料

【1】[http://blog.csdn.net/chaijunkun/article/details/7275632](http://blog.csdn.net/chaijunkun/article/details/7275632) 
【2】[http://blog.csdn.net/centralperk/article/details/8538697](http://blog.csdn.net/centralperk/article/details/8538697)
【3】[http://blog.csdn.net/as3luyuan123/article/details/16105435](http://blog.csdn.net/as3luyuan123/article/details/16105435)
【4】[http://blog.jobbole.com/42699/](http://blog.jobbole.com/42699/)

研究了一个星期 太多各种专业概念 感觉还是云里雾里  哪里理解的不对还望高手指正

openssl 是Linux下的一个工具合集是对各自解密协议的具体实现 可以用它来生成密钥 转换密钥格式等，命令的具体用法可参见[官方文档](https://www.openssl.org/docs/apps/openssl.html)
rsa 是一种算法 
pkcs 是一堆协议的集合 定义了各种规范 如密钥文件的语法格式等 其中pkcs#8 定义了私钥文件的语法格式

```base
openssl genrsa -out private.pem 1024 #调用openssl下的genrsa工具生成一个私钥 这个1024是二进制的长度 换算成字节是128 

openssl rsa -in private.pem -out public.pem -pubout #根据上面生成的私钥生成一个公钥 pubout的作用是告诉程序我们要生成的是公钥 默认这个命令是生成私钥的

openssl pkcs8 -in private.pem -out private_pkcs8.pem -topk8 -nocrypt #将私钥转换成pkcs8格式 方便java调用  具体参数解释见[官方文档](https://www.openssl.org/docs/apps/pkcs8.html)吧 
```

从资料【1】看 默认格式的私钥文件 在java中也是可以使用的 但要做处理 而pkcs#8格式的文件在php 和java中都可直接使用 不需处理 为了方便我还是把密钥转成了pkcs#8

通过上面的命令我们就生成了 一对密钥 private_pkcs8.pem 和 public.pem

下面直接上php和java的代码
```php
$cipher = new Cipher("private_pkcs8.pem","public.pem");
$en_out = $cipher->encrypt_by_public_key(str_repeat('w',118));
$de_out = $cipher->decrypt_by_private_key($en_out);
class Cipher {
    private $private_key;
    private $public_key;
    const ENCRYPT_MAX_LENGTH = 117;/*明文的分块区间*/
    const DECRYPT_MAX_LENGTH = 128;/*密文的分块区间*/

    /**
     * @param string $private_key_path 私钥存放路径
     * @param string $public_key_path 公钥存放路径
     */
    function __construct($private_key_path='',$public_key_path='')
    {
        if($private_key_path && file_exists($private_key_path)){
            $private_key_str = file_get_contents($private_key_path);
            $this->private_key = openssl_get_privatekey($private_key_str);
        }
        if($public_key_path && file_exists($public_key_path)){
            $public_key_str = file_get_contents($public_key_path);
            $this->public_key = openssl_get_publickey($public_key_str);
        }
    }

    function encrypt_by_public_key($plain_txt){
        if(!$this->public_key){
            throw new Exception("public key not found");
            return false;
        }
        $data_length = strlen($plain_txt);
        $off = 0;
        $encrypt_data = '';
        while($data_length - $off >0){
            if($data_length - $off > $this::ENCRYPT_MAX_LENGTH){
                $tmp = substr($plain_txt,$off,$this::ENCRYPT_MAX_LENGTH);
            }else{
                $tmp = substr($plain_txt,$off);
            }
            openssl_public_encrypt($tmp,$encrypt_tmp,$this->public_key);
            $off += $this::ENCRYPT_MAX_LENGTH;
            $encrypt_data .= $encrypt_tmp;
        }
        return chunk_split(base64_encode($encrypt_data));
    }

    function decrypt_by_private_key($cipher_txt){
        if(!$this->private_key){
            throw new Exception("private key not found");
            return false;
        }
        $cipher_txt = base64_decode($cipher_txt);
        $data_length = strlen($cipher_txt);
        $off = 0;
        $decrypt_data = '';
        while($data_length - $off >0){
            if($data_length - $off > $this::DECRYPT_MAX_LENGTH){
                $tmp = substr($cipher_txt,$off,$this::DECRYPT_MAX_LENGTH);
            }else{
                $tmp = substr($cipher_txt,$off);
            }
            openssl_private_decrypt($tmp,$decrypt_tmp,$this->private_key);
            $off += $this::DECRYPT_MAX_LENGTH;
            $decrypt_data .= $decrypt_tmp;
        }
        return $decrypt_data;
    }

}
```

```java
package rsa;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import javax.crypto.Cipher;
import jdk.nashorn.internal.codegen.CompilerConstants;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

/**
 *
 * @author Administrator
 */
public class Rsahelper {

    /** 
     * 私钥 
     */  
    public RSAPrivateKey privateKey;  
  
    /** 
     * 公钥 
     */  
    public RSAPublicKey publicKey;  
    
    /**
     * 最大明文长度
     */
    public  static final int MAX_ENCRYPT_BLOCK  = 117;
    
    /**
     * 最大密文长度
     */
    public  static final int MAX_DECRYPT_BLOCK  = 128;
    
    public static Rsahelper rsahelper = null;
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) throws Exception{

       String plaintxt = "";
       for (int i = 0; i < 118; i++) {//超过117后会被分段
            plaintxt += "w";
       }
       Rsahelper rsahelper = Rsahelper.getInstance("private_pkcs8.pem", "public.pem");
       String ripherdata = rsahelper.encrypt(plaintxt);
       System.out.println(ripherdata);
      
       String pliandata = rsahelper.decrypt(ripherdata);
       System.out.println(pliandata);
    }
    
    private Rsahelper(String private_path,String public_path) throws Exception{
        File privatefile = new File(private_path);
        if(privatefile.exists()){
            FileInputStream private_stream = new FileInputStream(privatefile);
            this.loadPrivateKey(private_stream);
        }
        File publicfile = new File(public_path);
        if(publicfile.exists()){
            FileInputStream public_stream = new FileInputStream(publicfile);
            this.loadPublicKey(public_stream);
        }
    }
    
    public static Rsahelper getInstance(String private_path,String public_path)  throws Exception{
        if(rsahelper == null){
            rsahelper = new Rsahelper(private_path,public_path);
        }
        return rsahelper;
    }
    
    /** 
     * 从文件中输入流中加载公钥 
     * @param in 公钥输入流 
     * @throws Exception 加载公钥时产生的异常 
     */  
    public void loadPublicKey(InputStream in) throws Exception{  
        
        BufferedReader br= new BufferedReader(new InputStreamReader(in));  
        String readLine= null;  
        StringBuilder sb= new StringBuilder();  
        while((readLine= br.readLine())!=null){  
            if(readLine.charAt(0)=='-'){  
                continue;  
            }else{  
                sb.append(readLine);  
                sb.append('\r');  
            }  
        }  
        loadPublicKey(sb.toString());  
       
    }  
  
  
    /** 
     * 从字符串中加载公钥 
     * @param publicKeyStr 公钥数据字符串 
     * @throws Exception 加载公钥时产生的异常 
     */  
    public void loadPublicKey(String publicKeyStr) throws Exception{  
      
            BASE64Decoder base64Decoder= new BASE64Decoder();  
            byte[] buffer= base64Decoder.decodeBuffer(publicKeyStr);  
            KeyFactory keyFactory= KeyFactory.getInstance("RSA");  
            X509EncodedKeySpec keySpec= new X509EncodedKeySpec(buffer);  
            this.publicKey= (RSAPublicKey) keyFactory.generatePublic(keySpec);  
       
    }  
  
    /** 
     * 从文件中加载私钥 
     * @param keyFileName 私钥文件名 
     * @return 是否成功 
     * @throws Exception  
     */  
    public void loadPrivateKey(InputStream in) throws Exception{  
        
        BufferedReader br= new BufferedReader(new InputStreamReader(in));  
        String readLine= null;  
        StringBuilder sb= new StringBuilder();  
        while((readLine= br.readLine())!=null){  
            if(readLine.charAt(0)=='-'){  
                continue;  
            }else{  
                sb.append(readLine);  
                sb.append('\r');  
            }  
        }  
        loadPrivateKey(sb.toString());  
       
    }  
  
    public void loadPrivateKey(String privateKeyStr) throws Exception{  
       
        BASE64Decoder base64Decoder= new BASE64Decoder();  
        byte[] buffer= base64Decoder.decodeBuffer(privateKeyStr);  
        PKCS8EncodedKeySpec keySpec= new PKCS8EncodedKeySpec(buffer);  
        KeyFactory keyFactory= KeyFactory.getInstance("RSA");  
        this.privateKey= (RSAPrivateKey) keyFactory.generatePrivate(keySpec);  
      
    }  
  
    /** 
     * 加密过程 
     * @param publicKey 公钥 
     * @param plainTextData 明文数据 
     * @return 
     * @throws Exception 加密过程中的异常信息 
     */  
    public String encrypt(String plainText) throws Exception{  
        if(this.publicKey== null){  
            throw new Exception("加密公钥为空, 请设置");  
        }  
        byte[] plainTextData = plainText.getBytes();
        Cipher cipher= null;  
        cipher= Cipher.getInstance("RSA");  
        cipher.init(Cipher.ENCRYPT_MODE, this.publicKey);  
        
        int inputlen = plainTextData.length;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int offset = 0;
        byte[] cache,output=null;
        while (inputlen - offset >0 ) {
           if(inputlen - offset > MAX_ENCRYPT_BLOCK ){
               cache= cipher.doFinal(plainTextData,offset,MAX_ENCRYPT_BLOCK); 
           }else{
               cache = cipher.doFinal(plainTextData,offset,inputlen - offset);
           }
           out.write(cache, 0, cache.length);
           offset += MAX_ENCRYPT_BLOCK;
        }
        
        output = out.toByteArray();
        out.close();
        BASE64Encoder base64en = new BASE64Encoder();
        String encrypt_str = new String( base64en.encode(output));
        
        return encrypt_str;  
        
    }  
  
    /** 
     * 解密过程 
     * @param privateKey 私钥 
     * @param cipherData 密文数据 
     * @return 明文 
     * @throws Exception 解密过程中的异常信息 
     */  
    public String decrypt(String cipherText ) throws Exception{  
        if (this.privateKey== null){  
            throw new Exception("解密私钥为空, 请设置");  
        }  
        BASE64Decoder base64Decoder= new BASE64Decoder(); 
        byte[] cipherData = base64Decoder.decodeBuffer(cipherText);
        Cipher cipher= null;  
        cipher= Cipher.getInstance("RSA");  
        cipher.init(Cipher.DECRYPT_MODE, this.privateKey);  
        int inputlen = cipherData.length;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int offset = 0;
        byte[] cache,output=null;
        while (inputlen - offset >0 ) {
           if(inputlen - offset > MAX_DECRYPT_BLOCK ){
               cache= cipher.doFinal(cipherData,offset,MAX_DECRYPT_BLOCK); 
           }else{
               cache = cipher.doFinal(cipherData,offset,inputlen - offset);
           }
           out.write(cache, 0, cache.length);
           offset += MAX_DECRYPT_BLOCK;
        }
        
        output = out.toByteArray();
        out.close();
        
        return new String(output);  
          
    }  
    
}
```
