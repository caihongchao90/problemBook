---
description: 2023.5.22
---

# 1. express后端服务实现跨域

### **一、使用nginx代理实现跨域**

* 1、创建前端应用 前端应用跑在5173端口，前端 npm run dev 后还是会报跨域的错，但是没关系，只要打包后，用nginx部署好就不会有相关错误。

```javascript
axios.get('http://localhost:8088/testapi')
    .then(function (response) {
        console.log(response);
        var text = response.data;
        
        document.getElementById("myDiv").textContent = Object.entries(text).map(x=>x.join(":")).join("\n");
    })
    .catch(function (error) {
        console.log(error);
    });
```

2、在nginx中写一个接口 当前端代码跑在nginx端口的时候，使用这个接口就不会出现跨域的情况。

```javascript
location  /testapi {
    add_header Access-Control-Allow-Origin 'http://localhost:3030' always;
    proxy_pass  http://localhost:3030/testmima/; 
}
```

3、后端服务

加这一层是因为使用nginx转发post请求，没有配置成功，**正常的话用nginx就可以直接转发，加这一层也是没有办法的办法！**

```javascript
app.get("/testmima", function (req, res) {
    let appsecret = 'e9c5aceff2c74f58';
    let nonce = encrypt.getPassword(8);
    let ticket = "edfafefeq";
    let timestamp = new Date().getTime();
    let content = "appsecret=" + appsecret + "&nonce=" + nonce + "&ticket=" + ticket + "&timestamp=" + timestamp;
    let sign = encrypt.sha1(content)
    axios.post("https://sgpt.zjwater.com/oapi/water-manage-auth/api/ssologin/validateTicket", {
        sign: sign,
        ticket: ticket,
        timestamp: timestamp,
        nonce: nonce,
    }).then(response => {
        res.json(response.data);
        return;
    })
})
```

大概流程：

> 1. 前端使用nginx接口发起请求->&#x20;
> 2. nginx将请求转到后端服务器（可选的一层）->&#x20;
> 3. 后端服务器接到请求后对指定的网址进行请求(后端请求不会涉及到跨域问题)

<figure><img src="../.gitbook/assets/后端服务转发避免跨域.jpg" alt=""><figcaption></figcaption></figure>

* 之所以需要在nginx服务器内写接口，是因为前端转发服务只能在开发环境下使用，生产环境下会失效，所以需要在**nginx的conf文件里配置代理**
