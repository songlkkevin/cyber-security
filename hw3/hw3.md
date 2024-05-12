# Homework 3
<p align=right><b>宋林恺 PB21051079</b></p>
<p align=right><b>罗胤玻 PB21111627</b></p>

## T1 Same Origin Policy 

----------

standard判断条件是:协议、域名和端口。若仅仅通过域名判断那么有以下例子可以绕过：
用户希望通过转到`https://site:8080/login`登录他们的银行帐户。攻击者建立网站`http://site:8080`，如果仅依据域名判断是否同源，那么攻击者网站和用户希望的网站是同源的。攻击者网站`http://site:8080`将用户重定向到`https://site:8080/login`。此时，用户位于其银行的合法登录站点，就会输入账号和密码进行登录。但是按照修改后的同源策略，`http://site:8080`和`https://site:8080/login`同源，所以攻击者能够直接访问登录的DOM。因此用户登录的信息可以被攻击者获得。


## T2 New Extensions to DNS

### a
**DNS-over-HTTPS protects**：通过使用HTTPS加密，防止第三方拦截和篡改DNS，也即可以抵御DNS欺骗和中间人攻击。

### b
**DNS-over-HTTPS does not protect**：HTTPS仅保护客户端到DNS解析器的通信，但是DNS解析器到目标服务器的通信并没有加密，所以，DNS-over-HTTPS并不能防御对服务器端的攻击，比如DNS缓存投毒。

### c
DoH和DoT不能抵御DNS被作为DDoS放大器使用的攻击，因为这种攻击是通过向DNS服务器发送大量的请求，使其返回大量的响应，从而使目标服务器受到攻击。而DoH和DoT只是对DNS请求进行加密，但是并没有对DNS请求的数量进行限制，所以无法防御DDoS攻击。

### d
DoH和DoT不能抵御DNS重绑定攻击，因为DNS重绑定攻击是通过将域名解析到不同的IP地址，从而绕过浏览器的同源策略，从而进行攻击。而DoH和DoT只是对DNS请求进行加密，仅保护从客户端到DNS解析器，所以无法防御DNS重绑定攻击。

----------

## T3 Cross Site Script Inclusion (XSSI) Attacks

### a
`evil.com`的页面嵌入一个`<script>`标签，指向`bank.com`的`userdata.js`文件，然后重写`displayData`函数来获取John Doe的信息。当John Doe登录`bank.com`后，访问`evil.com`，`evil.com`便会加载`bank.com/userdata.js`并执行`displayData`，将数据发送给`evil.com`

代码示意：
```javascript
<script>
function displayData(data) {
    fetch('https://bank.com/sendData', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
    })
}
</script>
<script src="//bank.com/userdata.js"></script>

```

### b
使用CORS限制`bank.com`域名才可访问`userdata.js`，此时，服务器会为`userdata.js`返回一个`Access-Control-Allow-Origin`头，指定允许访问的域名。`evil.com`无法访问`userdata.js`，因此无法获取John Doe的信息。
代码修改为：
```javascript
<script src="//bank.com/userdata.js" crossorigin="anonymous"> </script>
```

----------

## T4 CSRF Defenses

### a
CSRF攻击：攻击者诱使在登录状态下的用户访问恶意网站，从而利用用户的身份信息向目标网站发送请求，实现攻击。

### b
`token`通常由某种随机或加密算法生成，用户的`token`被保护在浏览器页面的DOM中，攻击者无法访问，而在服务器端，用户的操作生效前必须验证其`token`，以此来防止CSRF攻击。

### c
这样可以防止CSRF攻击，因为随机生成的字符串是难以预测的，攻击者将不能在极短时间内生成一个有效的`token`，从而无法伪造用户的请求。

### d
这样不能有效防止CSRF攻击，因为此时`token`是固定的，攻击者可以在恶意网站上获取到这段时间的`token`，然后伪造用户的请求，从而实现CSRF攻击。

### e
同源策略可以防止攻击者获取服务器端的`token`，若不使用同源策略，攻击者可以通过恶意网站获取到用户的`token`，从而实现CSRF攻击。

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