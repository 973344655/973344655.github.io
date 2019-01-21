---
title: hashutil
date: 2019-01-15 16:22:19
tags: wheels
---
```
package util;

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Random;

/**
 * hash(散列),把一些不同长度的输入，转换成相同长度的输出。不可逆，且高度离散性（改变一点点，结果相差很大），导致不可预测.
 * 生成的hash值称为 摘要 ， 摘要 不等于 加密。 加密是通过加密算法将明文转成密文，密文可以通过解密算法还原成明文。
 */
public class HashUtil {

    /**
     * 生成六位随机数
     * @return
     */
    public String creatRandom(){
        Random random = new Random();
        String result="";
        for (int i=0;i<6;i++)
        {
            result+=random.nextInt(10);
        }
        return result;
    }

    //通过getInstance()传入MD5,SHA-1,SHA-256,SHA-512等来选择不同算法
    public static  String getMd5(String str, String... type){
        MessageDigest messageDigest = null;
        try {
            //返回指定算法的MessageDigest对象 抛出NoSuchAlgorithmException
            //messageDigest = MessageDigest.getInstance("SHA-512");
            //messageDigest = MessageDigest.getInstance("MD5");
            messageDigest = (type.length == 0) ? MessageDigest.getInstance("MD5") : MessageDigest.getInstance(type[0]);
            //重置摘要
            messageDigest.reset();
            //使用指定的字节数组或buffer更新摘要,抛出UnsupportEncodingException
            messageDigest.update(str.getBytes("UTF-8"));
        }catch (NoSuchAlgorithmException e1){
            e1.printStackTrace();
        }catch (UnsupportedEncodingException e2){
            e2.printStackTrace();
        }
        //进行hash计算,byte为8位二进制补码表示的，有符号整数(127到-128)
        byte[] md5byte = messageDigest.digest();

        StringBuffer md5Str = new StringBuffer();
        for(int i=0; i<md5byte.length; i++){
            //int类型为32位，byte为8位（byte转int时高24位为1），用0xFF（高24位全补0）进行与运算，保持其补码不变
            if(Integer.toHexString(0xFF & md5byte[i]).length() == 1){
                //位运算后为1位十六进制情况,前面补0
                md5Str.append("0").append(Integer.toHexString(0xFF & md5byte[i]));
            }else {
                //位运算后为2位十六进制情况，直接append
                md5Str.append(Integer.toHexString(0xFF & md5byte[i]));
            }
        }
        return md5Str.toString();
    }

    public static  void main(String[] args){
        System.out.println(getMd5("adminbonc123"));
    }
}

```
