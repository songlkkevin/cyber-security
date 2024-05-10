# Homework 3
<p align=right><b>宋林恺 PB21051079</b></p>
<p align=right><b>罗胤玻 PB21111627</b></p>

## T1 Same Origin Policy 

----------

standard判断条件是:协议、域名和端口。若仅仅通过域名判断那么有以下例子可以绕过：
用户希望通过转到`https://site:8080/login`登录他们的银行帐户。攻击者建立网站`http://site:8080`，如果仅依据域名判断是否同源，那么攻击者网站和用户希望的网站是同源的。攻击者网站`http://site:8080`将用户重定向到`https://site:8080/login`。此时，用户位于其银行的合法登录站点，就会输入账号和密码进行登录。但是按照修改后的同源策略，`http://site:8080`和`https://site:8080/login`同源，所以攻击者能够直接访问登录的DOM。因此用户登录的信息可以被攻击者获得。


## T2 New Extensions to DNS

----------

## T3 Cross Site Script Inclusion (XSSI) Attacks

----------

## T4 CSRF Defenses

----------

## T5 Content Security Policies

----------

### a

script-src用于描述该页面Javascript的来源，'self'表示只能从同源加载，这样可以防止XSS攻击。如果不设置script-src，那么页面可以加载任意来源的Javascript，这样就会有可能被注入恶意脚本。

#### b
frame-ancestors用于描述该页面可以被嵌入到哪些页面中，'none'表示不允许被嵌入到任何页面中，这样可以防止点击劫持攻击。如果不设置frame-ancestors，那么页面可以被嵌入到任意页面中，这样就会有可能被点击劫持。

#### c

sandbox 'allow-scripts'表示该页面可以执行脚本，但是不能读取cookie。这样可以防止XSS攻击。如果不设置sandbox 'allow-scripts'，那么页面可以读取cookie等数据，这样就会有可能被XSS攻击。

有这个CSP头，沙箱脚本被限制了对非同源cookie的访问，所以无法访问www.xyz.com的cookie。

当一个页面想要在其中嵌入不受信任的内容时。例如，一个页面想要在其中嵌入一个广告，但是不信任广告的来源。这时，可以使用sandbox属性，将广告放在一个iframe中，并且设置sandbox属性，这样广告就无法访问页面的cookie等数据。

## T6 Stealth Port Scanning

----------

#### a
1.从A向主机P发送ICMP ping请求. 2.记来自P的响应包的identification的counter 为x. 3.时间窗口结束后，再从A向P发送一个ICMP ping请求. 4. 记来自P的响应的identification counter 为y. e. 如果 y > x + 1,那么P向其余机器发了packets.

#### b
1.首先从A向P发送一个ICMP ping请求。2.来自P的相应包的identification counter为x。3.伪造一个源地址为P的SYNC包发给V.（V该端口大概则会向P发一个SYN packet，因为P该端口关闭的则发送RST给V，那么P的identification counter 将+1。若V该端口关闭则会向P发一个RST packet,P将不会有任何改变）4.从A向P发送一个ICMP ping请求。5.来自P的相应包的identification counter为y。6.如果y > x + 1,那么V在该端口运行服务。

## T7 Denial of Service attacks

----------

#### a
令TCP等待时间相对可以忽略不记(从发送SYN包到超时)。服务器若$5 \times 30s$没有接到SYN packet就会清楚该连接，那么攻击者可以每隔$150s$向服务器发送一个SYN packet以保证占用端口。256个端口那么rate就是$256 \times \frac{1}{150s} = 1.7$ packets/s。若每个packet的大小为40 bytes，那么带宽就是$1.7 \times 40 \times 8 = 544bps$。

#### b
若包大小为500 bytes，带宽0.5-Mbps，那么频率就是$\frac{0.5 \times 1024^2}{500 \times 8} \approx 131$ packets/s。

若包大小为500 bytes，带宽2-Mbps，那么频率就是$\frac{2 \times 1024^2}{500 \times 8} \approx 524$ packets/s。

若包大小为500 bytes，带宽10-Mbps，那么频率就是$\frac{10 \times 1024^2}{500 \times 8} \approx 2621$ packets/s。

若包大小为60 bytes，带宽0.5-Mbps，那么频率就是$\frac{0.5 \times 1024^2}{60 \times 8} \approx 1092$ packets/s。

若包大小为60 bytes，带宽2-Mbps，那么频率就是$\frac{2 \times 1024^2}{60 \times 8} \approx 4369$ packets/s。

若包大小为60 bytes，带宽10-Mbps，那么频率就是$\frac{10 \times 1024^2}{60 \times 8} \approx 21845$ packets/s。

## Contribution
**宋林恺：** T1, T5, T6, T7
**罗胤玻：** T2, T3, T4