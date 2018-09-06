(1)解密链接:<https://jwt.io/> 

(2)参考链接:<https://www.anquanke.com/post/id/145540> 

1.jwt 全称: json web token 



掌握知识点: 

(1)http是无状态的 

(2)session两个弊端: 

​    1.session保存在服务端,占用服务端空间 

    2.服务端为集群时,将session保存到服务器内存中,当用户访问到其他服务器时,会无法访问  

    解决办法:缓存一致性技术 或者 第三方缓存  

(3) jwt 在header的位置 cookie 或者header Authorization: Bearer <token> 

(4) 第二部分base64不能解密 很可能是缺少= 没有填充满 

1.jwt的构成 

    (1)Header   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 

    (2)Payload  eyJhZG1pbiI6InRydWUifQ 

    (3)Signature eyJhZG1pbiI6InRydWUifQ.03jOKnTqOkqP7gzd1rxaIy4wQSntb18yg2noERrVhUU 

  

Example： 

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.03jOKnTqOkqP7gzd1rxaIy4wQSntb18yg2noERrVhUU 



一.header 

    { 

      "alg": "HS256”, 加密签名的散列算法 

      "typ": “JWT” 令牌类型 

    } 

二.payload  

    Data  包含声明的有效负载 声明:关于实体(通常是用户)和其他院数据的声明 

    { 

          "admin": “true”  

     } 

三.Signature 

    HS 256 (HMAC SHA256) 

    HMACSHA256( 

    base64Encode(header)+”.”+base64Encode(payload),secret 

    ) 



二.JWT的攻击手段

    1.算法修改攻击

        原理: 头部的alg可被修改 若公钥泄漏 后台通过alg选择验证算法 可以把rs256(非对称加密)->hs256(对称加密)进行伪装

    2.秘钥可控

      header{

        “type”:”jwt”,

        “alg”:”sha256”,

        “kid”:”8201"

        }

        Kid可以注入 然后获取key进行加密

    3.秘钥爆破问题

    工具:https://[github.com/brendan-rius/c-jwt-cracker](http://github.com/brendan-rius/c-jwt-cracker)

    然后即可进行消息的恶意伪造，篡改

    4.指定算法为none

    原理：有些实现jwt的库直接将使用none算法的token认为是已经过检验的

    利用:”alg”:”none” signature部分为空值即可



三.JWT实战分析
