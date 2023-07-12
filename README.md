# 分析过程
抓取访问企查查的流量分析，找到真实的查询接口

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1679821243800-69358422-3276-4802-bb28-8ed1184174da.png#averageHue=%23fbfafa&clientId=u64941258-761a-4&from=paste&height=838&id=uc71dd834&originHeight=838&originWidth=1512&originalType=binary&ratio=1&rotation=0&showTitle=false&size=209366&status=done&style=none&taskId=u88639952-7ff2-4661-8acf-6c36d73cf30&title=&width=1512)

请求如下

```http
POST /api/search/searchMulti HTTP/2
Host: www.qcc.com
Cookie: QCCSESSID=9b221faa2ec489f517d6aa605a; qcc_did=c5d26a9e-92c1-4032-9f93-6988b5f091be; UM_distinctid=18718b8b8726d3-04bae1982d8a078-d545429-1fa400-18718b8b873da6; CNZZDATA1254842228=200268840-1679746321-https%253A%252F%252Fwww.baidu.com%252F%7C1679811161; acw_tc=7d27879f16798125801396702e2a36e6450c52885a31bb328009b08b87
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/111.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json
X-Requested-With: XMLHttpRequest
X-Pid: 729498f1e3bde9e085a0a6d3398d9f6b
52077eb68d4550367680: 810332bf9f03b02e81ef674d2527318e4aa8b19eaade5cbd5e8638d3b85b0d3a75266edb8314a87626cac1d9e6a2d0b0bf999183b597336c6dcdb1a30f2914ef
Content-Length: 47
Origin: https://www.qcc.com
Referer: https://www.qcc.com/web/search?key=360&p=3
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers

{"searchKey":"360","pageIndex":1,"pageSize":20}
```

分析该请求头，含有两个非常见参数X-Pid及一串16进制字符串（52077eb68d4550367680），其中X-Pid可在页面源码中获取，16进制字符串未在源码中发现

```http
X-Pid: 729498f1e3bde9e085a0a6d3398d9f6b
52077eb68d4550367680: 810332bf9f03b02e81ef674d2527318e4aa8b19eaade5cbd5e8638d3b85b0d3a75266edb8314a87626cac1d9e6a2d0b0bf999183b597336c6dcdb1a30f2914ef
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1679821415525-b7289992-b07c-481d-99b8-f8f65854f545.png#averageHue=%23f0f0f0&clientId=u64941258-761a-4&from=paste&height=82&id=ud4f4059c&originHeight=82&originWidth=1358&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8628&status=done&style=none&taskId=ua3d975bd-a366-46ec-8ca4-4298eec5493&title=&width=1358)

测试中发现如下情况
1. 删掉X-Pid参数，在不修改post请求体的条件下，依然能获取到查询数据
2. 保持请求头中16进制字符串不变，修改请求体，后端返回查询错误

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1679821702740-9f2d6407-147d-4499-a060-f41ccea2d798.png#averageHue=%23f9f7f7&clientId=u64941258-761a-4&from=paste&height=599&id=ua4f5cd14&originHeight=599&originWidth=1324&originalType=binary&ratio=1&rotation=0&showTitle=false&size=146340&status=done&style=none&taskId=ub8e31c5d-d1d0-4603-9de1-8e10e709f20&title=&width=1324)

因此，如何准确构造16进制字符串便是关键，
尝试在源代码中搜索encrypt关键字，并设置断点，但是浏览器并未执行该段JS

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681029696214-967772a3-e252-43fe-a4b5-edebb120357e.png#averageHue=%23fefefc&clientId=u568f7325-ad2a-4&from=paste&height=860&id=udb7ec236&originHeight=860&originWidth=765&originalType=binary&ratio=1&rotation=0&showTitle=false&size=89048&status=done&style=none&taskId=u252fe2ad-a935-48f8-95fd-84e2a5a00ae&title=&width=765)

搜索发现相关分析文章，该种场景下可以搜索"headers["，

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681029812662-2e0b2579-5eb4-476e-838a-8a72f5dc7ab0.png#averageHue=%23f2d987&clientId=u568f7325-ad2a-4&from=paste&height=203&id=ue836ffdd&originHeight=203&originWidth=485&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19951&status=done&style=none&taskId=ufa49ca7e-f5aa-46ce-815d-4668af62956&title=&width=485)

注意到5958行x-pid的生成逻辑即是页面源码中的x-pid，将注意力关注于5853行的e.headers[i] = l

```java
(e.headers['x-pid'] = window.pid)
```

由浏览器及yakit数据包对比可以发现，该处i值即为加密后的请求头，l为具体内容

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681030105802-e5af4594-5b43-45a2-be3a-e059531f85bb.png#averageHue=%23f4e9bc&clientId=u568f7325-ad2a-4&from=paste&height=515&id=uf0387278&originHeight=515&originWidth=774&originalType=binary&ratio=1&rotation=0&showTitle=false&size=79239&status=done&style=none&taskId=u9c16f0c8-77b8-4c58-ad11-5d1e8f1f64a&title=&width=774)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681030421411-c29e3503-e5ff-4230-a6dd-209995adee5e.png#averageHue=%23e8eef3&clientId=u568f7325-ad2a-4&from=paste&height=375&id=ue3711d37&originHeight=375&originWidth=758&originalType=binary&ratio=1&rotation=0&showTitle=false&size=59172&status=done&style=none&taskId=uf547d33c-4f1b-468d-ba1f-12592958b97&title=&width=758)
# 跟踪
## i

关于i生成的函数，传递两个参数值，请求的url和post请求体

```javascript
t="/api/search/searchmulti"
e.data="{"searchKey":"淘宝","pageIndex":2,"pageSize":20}"
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681093210224-92e767cc-ed3d-414c-a6ce-5d25a32802a9.png#averageHue=%23fefdfd&clientId=u145fcc00-2d23-4&from=paste&height=497&id=u2c39cb63&originHeight=497&originWidth=708&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61114&status=done&style=none&taskId=ua0b65187-9eff-45f2-8a4d-70a9523c2fa&title=&width=708)

跟进上图中的a.default

```javascript
var s = function () {
  var e = arguments.length > 1 && void 0 !== arguments[1] ? arguments[1] : {
  },
    t = (arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : '/').toLowerCase(),
    n = JSON.stringify(e).toLowerCase();
  return (0, o.default) (t + n, (0, a.default) (t)).toLowerCase().substr(8, 20)
}
```

这6474，6476两行代码三目运算符计算变量

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681093980829-e0a2d5a4-97f4-4e60-bf51-401cb0a9cb02.png#averageHue=%23fefcfc&clientId=u145fcc00-2d23-4&from=paste&height=328&id=u5ade7d66&originHeight=328&originWidth=814&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42076&status=done&style=none&taskId=u2f3cf64d-f333-49e3-a8cc-20eb07f165d&title=&width=814)

查找到关于类似(0,o.default)((0,r.default)(d))格式的解释

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681054263146-9a599238-681a-4779-8082-4465139c448c.png#averageHue=%23fcfbfa&clientId=u2381ffce-0b63-4&from=paste&height=229&id=iV0j6&originHeight=229&originWidth=928&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24545&status=done&style=none&taskId=ua5da2ca7-7ba5-4d6a-9f92-6184a524f47&title=&width=928)

6478行即为

```javascript
return o.default(t + n, a.default(t)).toLowerCase().substr(8, 20)
```

跟进a.default

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681095439534-51e160ea-b3a3-4a80-8580-7ad772650076.png#averageHue=%23fefcfb&clientId=u145fcc00-2d23-4&from=paste&height=343&id=uae81abe0&originHeight=343&originWidth=859&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41491&status=done&style=none&taskId=ua96581eb-7a77-4f27-9874-2bef15c2f4a&title=&width=859)

```javascript
var r = function () {
        for (var e = (arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : '/').toLowerCase(), t = e + e, n = '', i = 0; i < t.length; ++i) {
          var a = t[i].charCodeAt() % o.default.n;
          n += o.default.codes[a]
        }
        return n
      }
```

逻辑大概为

```javascript
将请求的url新赋值为url+url
计算新url中每个字符的ascii相对o.default.n（20）取余
以取余的结果a为下标，将新url中字符赋值为o.default[a]
```

这里的o.default字典为

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681096003260-568d8922-56b5-4a4a-89e7-5b7ec871dd90.png#averageHue=%23fefefe&clientId=u145fcc00-2d23-4&from=paste&height=327&id=u0e4fb18c&originHeight=327&originWidth=787&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12233&status=done&style=none&taskId=u8fd806af-00aa-400b-ab2d-0b143b12cf7&title=&width=787)

再回到6478行，看o.default

```javascript
return o.default(t + n, a.default(t)).toLowerCase().substr(8, 20)
参数为t+n为原始url和post请求体的拼接
a.default(t)为新url取余编码运算后的值
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681097003908-46217b95-4dc4-4a57-b2cc-cc3040ca283b.png#averageHue=%23fefcfb&clientId=u145fcc00-2d23-4&from=paste&height=125&id=ude72ad9c&originHeight=125&originWidth=829&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18864&status=done&style=none&taskId=ue6985647-6ae6-40db-baec-afa2ebfdd8d&title=&width=829)

该层中还有o.default，继续跟进，这里应该为调用内置加密函数进行加密，这里没有继续跟进

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681097140341-01cede2f-1daf-48d2-9e9f-468952769462.png#averageHue=%23fefefd&clientId=u145fcc00-2d23-4&from=paste&height=93&id=u82a822a5&originHeight=93&originWidth=825&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13368&status=done&style=none&taskId=u03b28ba4-9c6b-4ded-8adf-8e0c56e4e70&title=&width=825)

shift+F11后发现，已经成功计算出加密的请求参数

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681097647994-d11371f7-50f1-47af-9bde-085a6eca6051.png#averageHue=%23fefdfd&clientId=u145fcc00-2d23-4&from=paste&height=244&id=udece4d55&originHeight=244&originWidth=855&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34585&status=done&style=none&taskId=uc7f0ed8a-df07-4d6d-8f1b-8431f2a2e7e&title=&width=855)
## l

跟踪加密参数值的逻辑

```javascript
l = (0, r.default) (t, e.data, (0, s.default) ()); 即为
l = r.default (t, e.data, s.default());
```

首先是s.default（），返回获取页面中的window.tid的值

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681099074859-2378d255-9866-4701-8056-b5fc2d8c6a07.png#averageHue=%23fefcfc&clientId=u145fcc00-2d23-4&from=paste&height=459&id=u97dd3ab9&originHeight=459&originWidth=811&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42030&status=done&style=none&taskId=u2ff4d72e-f676-4c26-8807-07424831981&title=&width=811)

然后是r.default，函数逻辑与i中对应函数类似，t参数为window.tid的值

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681106554030-192d0585-41bf-4bd1-a65e-ea2fd65c3ba7.png#averageHue=%23fefcfc&clientId=u145fcc00-2d23-4&from=paste&height=249&id=udc7f5696&originHeight=249&originWidth=816&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32970&status=done&style=none&taskId=u69743939-6945-4609-ae78-4b496441cc0&title=&width=816)

之后的a.default和o.default函数功能与i中一致，返回加密后的请求参数

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681108175765-a71972e2-1bfd-4cfa-8be5-dc6010a514c7.png#averageHue=%23e4c58e&clientId=u145fcc00-2d23-4&from=paste&height=249&id=u4d2daf5f&originHeight=249&originWidth=945&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42840&status=done&style=none&taskId=u1a40b938-07ed-4237-b096-ac89b306305&title=&width=945)

画出生成加密请求头和请求参数值的调用图如下

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28328736/1681107573073-8076de35-02a3-44f6-aa66-23299a232ddd.png#averageHue=%23f9f9f9&clientId=u145fcc00-2d23-4&from=paste&height=1005&id=ue035a5b9&originHeight=1005&originWidth=2285&originalType=binary&ratio=1&rotation=0&showTitle=false&size=171675&status=done&style=none&taskId=ua091617d-b738-4034-bc92-4390dc9bf73&title=&width=2285)

# 代码编写

之后便是找寻真实的请求接口，模拟请求解析数据
插件ID 8e9ce911-b233-4f9e-8839-ffbf182991a4
