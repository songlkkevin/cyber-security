# Homework 1
<p align=right><b>宋林恺 PB21051079</b></p>
<p align=right><b>罗胤玻 PB21111627</b></p>

## T1

##### 风险:
- 1.该算法可能存在后面以及漏洞，美国政府可能可以通过该漏洞encrypt发送的密文
- 2.算法过于单一，若该算法存在vulnerability那么大量用该算法加密的用户将收到威胁
- 3.缺乏可信度，其余国家的政府，机构以及个人可能不回使用该算法
##### 怎样避免：
- 1.使用收到广泛接受的开源算法。
- 2.自己开发一种加密算法，不依赖美国政府的加密算法。
- 3.设计时使用多种加密手段避免单一加密算法导致的问题。
- 4.成立中立机构指定加密标准
----

## T2

##### block cipher的意义
- 1.不可以加密任意长度的数据，然而很多场景下需要加密不定长或变长的数据如流数据的加密。
- 2.不满足IND-security，对于同一个明文加密的密文是相同的容易被攻击。
以下表格为CBC和CTR的对比

|                        | CBC        | CTR      |
|------------------------|------------|----------|
| Sequential or Parallel | Sequential | Parallel |
| Error Propagation      | Yes        | No       |
| Can reuse IV/Nonce     | No         | No       |
| Padding                | Yes        | No       |
----

## T3 Hash Collision

- 哈希碰撞原因：输入空间大于输出空间，即不同的输入可能映射到相同的输出。
- 碰撞不是哈希函数的缺陷：哈希函数通常具有强抗碰撞性，攻击者如果不能在多项式时间内找到碰撞。

----

## T4  TLS Certificat

### ustc.edu.cn
**1. 证书日期及CA**
![ustc](./img/1-1.png)

**2. 证书公钥**
![](./img/1-2.png)

**3. pem文件计算**
![](./img/1-3.png)

### 12306.cn
**1. 证书日期及CA**
![12306](./img/1-4.png)

**2. 证书公钥**
![](./img/1-5.png)

**3. pem文件计算**
![](./img/1-6.png)

### www.bing.com
**1. 证书日期及CA**
![bing](./img/1-7.png)

**2. 证书公钥**
![](./img/1-8.png)

**3. pem文件计算**

![](./img/1-9.png)


---


## T5  Encrypt/Decrypt/Sign through openssl

### (a)-(b)
![](./img/2-1.png)
1. `openssl rand -hex 16` 生成128位密钥与随机向量
2. `openssl enc -e -aes-128-cbc -in message -out enc_message -K <key> -iv <initial vector>` 加密
3. `openssl enc -d -aes-128-cbc -in enc_message -out dec_message -K <key> -iv <initial vector>` 解密
**对比解密文件与原文件，无误。**

### (c)-（e）
![](./img/2-2.png)
1. `openssl genrsa -out privkey.pem` 生成私钥
2. `openssl rsa -in privkey.pem -pubout -out pubkey.pem` 生成公钥
3. `openssl dgst -sha256 message` 计算消息SHA256 hash值
4. `openssl dgst -sha256 -sign privkey.pem -out signature message` 生成签名
5. `openssl base64 -in signature` 使用base64编码输出签名
6. `openssl dgst -sha256 -verify pubkey.pem -signature signature message` 验证签名

**最终结果：Verified OK**

---

## T6

### 选择论文 A public key cryptosystem and a signature scheme based on discrete logarithms

#### 总结

##### Elgamal加密方案

场景:A将m加密传递给B

定义与约束:B的私钥$x_A$,一个大素数$p$以及其原根$\alpha$且$p-1$有一个大的素因子,B的公钥$y_B\equiv \alpha^{X_A} mod P$, 明文$m, m < p-1$。除B的私钥以及明文外均为公开信息。

加密: 

1. 生成随机数$0\leq k \leq p - 1$
2. $K \equiv {y_B}^k mod p, c_1 = \alpha^k mod p, c_2 = Km mod p$
3. 发送c_1,c_2

解密:
1. 计算$K = \alpha^{kx_B} mod P = (\alpha^k)^{x_B}$
2. 还原明文 $m = c_2 \cdot K^{-1}$

注意与性质：
1. 由于k的随机性，相同明文m不同次加密是不同的所以满足 IND-security
2. 对于在该加密系统中不同明文$m_1, m_2$之间的加密没有明显的练习
3. 若elgamal的p和RSA的p的size相同，那么相同的明文elgamal密文长度是RSA密文的两倍
4. 加密与解密的复杂度为$O(log p)$
5. 一个k只能使用一次，两次加密中若知道一次的明文可以简单计算出另一次的明文

##### Elgamal签名方案

场景B对m进行签名

定义与约束:与加密过程定义相同

签名:

1. 生成随机数$0\leq k \leq p - 1$
2. 计算$r \equiv \alpha^k mod P$
3. 求解$\alpha^m \equiv {y_B}^rr^s mod p$
4. 可转换为求解$m \equiv x_Br+ks$,此时用拓展欧几里得算法接触s
5. 发送签名(r,s)

验证:
1.验证$\alpha^m = y^rr^s$是否成立

注意与性质:
1. 签名空间有$p^2$,明文空间为$p$。一个明文可以有多个签名，但一个签名只对应一个明文
2. 签名过程的复杂度通过一系列操作可降为1.875幂
3. ElGamal 签名的长度与 RSA 方案所需的长度相同，并且是 Ong 和 Schnorr 发布的基于二次型的新签名方案的一半大小。
4. 一个k只能使用一次

##### 可能的攻击方法：

- 恢复私钥X

1. 方法一:生成l个明文$\{m_1, m_2, ..., m_l\}$，得到n个签名$\{(r_i, s_i): i = 1,2...l\}$。尝试从l个等式中解出私钥x。然而有l+1个未知的元且p-1有一个大的质因子。所以恢复的复杂度是签名对指数级的。然而若k使用了两次，那么可以轻松求解这说明了为什么签名不能使用重复的k。
2. 方法二：直接求解$\alpha^m \equiv {y_B}^rr^s mod p$，这和求解离散对数问题难度相同。
3. 方法三：在$k_i$之间建立一些线性关系，这也与求解离散对数的难度行同

- 伪造签名

4. 方法四：计算s与计算离散对数等价，假设已经拥有s那么就需求解$r^sy^r\equiv A mod P$，解决该问题复杂度未被证明，但作者认为不能再多项式时间内做出。
5. 方法五：$\alpha^m = y^rr^s$，看上去可以同时解出r和s，但是目前没有找到有效的方法。
6. 方法六：通过已知的明文m和签名(r,s)可以生成另一对合法签名(r',s')但这并不违反该密码系统

#### 评价
Pros:论文清晰展示了一种新的加密系统与算法，结构清晰。新的数字签名方法非常有创新性。另外与RSA公钥相比概率的加密方式实现了IND安全。

Cons:这是一篇密码学论文，对于该如何评判一个密码学论文不是非常了解。从计算机系统研究领域来看我觉得该论文对于在实际workload中该加密系统与之前密码的对比缺失。如在某些场景下因为其加密后的密文长度大于RSA加密的密文长度

----

## Contribution