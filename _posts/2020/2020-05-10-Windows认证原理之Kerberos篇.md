## Active Directory(活动目录)

* AD存储关于网络对象的相关信息，使管理员和用户可以轻松地查找并使用这些信息。其使用分成组织的逻辑进行结构化的数据存储。
* 网络对象分为：用户、用户组、计算机、域、组织单位以及安全策略等

## 域认证协议 Kerberos

![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2020/media/2020-05-10-01.png)

**古希腊-地狱恶犬**



**Kerberos 是一种网络认证协议**，其设计目标是通过密钥系统为客户机/ 服务器应用程序提供强大的认证服务。该认证过程的实现不依赖于主机操作系统的认证，无需基于主机地址的信任，不要求网络上所有主机的物理安全，**并假定网络上传送的数据包可以被任意地读取、修改和插入数据。** 在以上情况下，Kerberos 作为一种可信任的第三方认证服务，是通过传统的密码技术（如：共享 密钥）执行认证服务的。

### 参与角色

* 客户端
* 服务器
* KDC（密钥分发中心）= DC

![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2020/media/2020-05-10-02.png)

### KDC的组成

* AD（Account Database）：存储所有客户端的白名单，只有在白名单中的客户端才可以申请TGT
* AS（Authetication Service）：为客户端生成TGT的服务
* TGS（Ticket Granting Service）:为客户端生成某个服务的ticket

**从物理层面看，AD与KDC都是域控（Domain Controller）**



## 域认证流程-粗略流程

1. 客户端向Kerberos服务请求，希望获得访问某个服务/服务器的权限。客户端首先向Kerberos请求身份认证，Kerberos得到消息后，先判断客户端是否可信的既白名单中，在AD查询用户之后，返回到AS中，AS分发TGT给客户端，至此 AS 的工作就算是完成了。
2. 客户端得到TGT后，继续向Kerberos请求，希望获得某个服务/服务器的权限。Kerberos得到消息后，通过客户端得到消息中的TGT，判断客户端是否拥有这个权限，然后给予客户端访问服务器的权限Ticket
3. 客户端得到访问服务器的权限Ticket后，就可以向服务器发起请求，这个Ticket仅针对该服务器，其他Server需要重新向TGS申请。

**类似于动车站买票**


## 域认证流程-具体流程

### 用户登录客户端
* 输入用户ID和密码到客户端
* 客户端利用质询的NTLM协议将密码转换成密钥，形成了客户端的“用户密钥”**（user's secret key）**

### 第一步-客户端认证

**Session Key与Ticket Granting Ticket**

#### 1. 客户端
 1. 向Kerberos发送客户端信息信息和相应的请求服务，例如“用户Tom想要请求服务”（不需要发送密钥或者密码）

#### 2. Kerberos
AS检查该用户ID是否存在于本地数据库中，验证完成后返回2条信息：
1. Client/TGS会话密钥（Client/TGS Session key），这个密钥用来在客户端和TGS之间进行通信，**使用该用户的NTLM Hash进行加密**
2. 票据授权票据（Ticket Granting Ticket）,包含信息有：消息1中的会话密钥，用户ID，用户网址，消息2的有效期。**通过TGS的密钥进行加密**

#### 3. 客户端
客户端收到消息后，首先用自己的用户NTLM Hash解密消息1，获得其中的TGS会话密钥
注意：客户端不需要解密消息2，只需要消息1中的TGS会话密钥就可以向TGS发起请求
![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2020/media/2020-05-10-03.png)
### 第二步-服务授权

#### 客户端

当客户端想要申请指定的服务的时候，向TGS发送两条消息：
![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2020/media/2020-05-10-04.png)
3. 消息2的信息、想要获取的服务的服务ID（不是用户ID）
4. 认证符（Authenticator），其中包含用户ID和时间戳，使用消息1中解密出来的TGS会话密钥进行加密

#### Kerberos

* 在收到客户端发起的两条请求后，TGS先去KDC数据库中查找客户端发来的消息3中的服务ID是否存在，然后用自己的**TGS密钥**解密消息3中的消息2，得到**TGS会话密钥**
* 使用TGS会话密钥解密消息4，得到用户ID信息和时间戳，核对完成之后向客户端发送两条信息

5. 客户端-服务器票据（Client-to-Server Ticket），其中包含**Client/SS会话密钥（Client/Server Session Key）**，用户ID，用户网址和票据有效期。使用服务器密钥（Server's secret key）进行加密。
6. Client/SS会话密钥（Client/Server Session Key），使用**Clent/TGS会话密钥进行加密。**
![](https://raw.githubusercontent.com/is0late/is0late.github.io/master/_posts/2020/media/2020-05-10-05.png)
#### 客户端

* 客户端收到消息后，用消息1中的Clent/TGS解密消息6，得到其中的Client/SS会话密钥。
* 而消息5客户端是无法解密的，它用服务器密码进行加密。


### 第三步-服务请求

#### 客户端
当客户端拿到消息6中的Client/SS会话密钥之后，就可以向服务器请求服务了，它会向服务器发送2条消息

7. 消息5，用服务器密钥加密的 客户端-服务器票据
8. 新的Authentication（包含用户ID和时间戳），**使用Client/SS会话密钥进行加密。**

#### 服务器

* 当服务器收到消息后，会用自己的服务器密钥解密消息7得到客户端-服务器票据，既得到了Client/SS密钥。
* 再使用得到的**Client/SS密钥**解密消息8，得到Authentication，获取其中的用户ID和时间戳。

这一步流程完成后，客户端的认证就可到确认，服务器就会给客户端提供服务，向客户端发送1条消息。

9. 新的时间戳，使用**Client/SS会话密钥进行加密。**

### 第四步-快乐沟通

#### 客户端

客户端拿到消息9之后，用Client/SS会话密钥解密，验证时间戳，确认服务器身份，然后就开始向服务器发送请求

#### 服务器

服务器向客户端提供相应的服务


## Kerberos认证的缺点

* 需要第三方机构认证，如果Kerberos服务器挂了，那就没法请求服务了，可以通过复合Kerberos服务器和缺陷认证来进行弥补
* 时钟上的统一，因为票据具有一定的有效性，机器间的时间一般不可以超过10分钟
* DC被控，全家遭殃
* 客户端防御差，用户密码/哈希也会被拿走








