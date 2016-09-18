#### 创建野狗应用
>填写应用名称和应用ID就可以创建了。应用ID需要全网唯一

![](http://7xo2m9.com1.z0.glb.clouddn.com/image/c/df/929875bce1e5432ee74a0db22a77c.png)
 

#### 管理应用
>创建成功之后就可以在控制面板看到应用了，

![](http://7xo2m9.com1.z0.glb.clouddn.com/image/7/39/c8bca2253916cc1dde2a137d54176.png)

#### 开始

###### 1.引入SDK
```html
<script src = "https://cdn.wilddog.com/js/client/current/wilddog.js" ></script>
```

###### 2.创建引用
```javascript
 var ref = new Wilddog("https://<appId>.wilddogio.com/")
 将<appId>替换成申请的应用ID
 var ref = new Wilddog("https://fullstack-top-demo.wilddogio.com/")
```
 因为wilddog是以key-value的形式存储数据，创建引用会定位到根节点。若要定位到子节点，只需在url后追加路径即可，例如：
```javascript
var user_ref = new Wilddong('https://fullstack-top-demo.wilddogio.com/user/')
```
野狗也提供了`child()`方法来获取子节点的引用。
```javascript
var ref = new Wilddog("https://fullstack-top-demo.wilddogio.com/")

var user_ref = ref.child('user')
```
这两种方法是一样的效果

##### 数据操作

1.写入数据。
>创建 Wilddog 引用之后，就可以通过set() 往节点中写入任何合法的JSON数据

```javascript
user_ref.set({
	name : 'lixiaohao',
	age  : 18,
	blogurl : 'ghost.fullstack.top'
})
```
![](http://7xo2m9.com1.z0.glb.clouddn.com/image/4/6b/d979ccb1e9bcf821850a252fb76e3.png)

2.读取数据
>读取数据是通过绑定回调函数来实现的。假设我们按照上面的代码写入了数据，那么就可以使用on()函数来读取user对象的值。
```javascript
user_ref.on('value', function(datasnapshot) {
  console.dir(datasnapshot.val());   // 结果会在 console 中打印出刚刚set的对象
})
```
![](http://7xo2m9.com1.z0.glb.clouddn.com/image/3/0d/19302ae009a72fbaf61b562ee25a7.png)

回调函数的参数是一个DataSnapshot对象类型，调用它的val()函数得到数据对象。上边这个例子中，value这个事件会在初次读取到数据的时候被触发一次，此后每当数据发生改变，都会被触发。

若要只读取一次，不在之后每次数据发生变化的时候触发回掉函数，可以使用once()函数替代on()函数。


3.用户认证
>绝大多数应用都需要一套终端用户账号体系。对终端用户进行唯一标识之后，才能对用户进行个性化的用户体验，控制用户对数据的访问权限。提供终端用户唯一标识的过程被称为终端用户认证。WildDog为开发者提供了多种用户认证方式。
野狗提供了多种用户登录方式，具体可查看[官方文档](https://z.wilddog.com/web/auth)

**这里要注意的一点就是，第三方登录一定要设置OAuth跳转域名白名单**
![](http://7xo2m9.com1.z0.glb.clouddn.com/image/5/0e/5b4ebf1d990be625ba319194d782d.png)
当时因为这个没有配置这个白名单折腾了一下午。不过在本地环境下用`localhost` 或 `127.0.0.1` 访问的话不会有影响。


**好了，了解这3点就可以开始做排行榜了。**

---
#### 游戏排行榜
我们可以去网上找一个html5的小游戏，稍微研究下代码应该就可以找到游戏成绩的结算方法，在游戏结束时给我们的ref`set()`一个值就可以啦。

这里以我写过的一个demo为例

###### 授权登录
```javascript
//创建根节点的引用
var wilddog = new Wilddog(""https://<appId>.wilddogio.com/");

var wilddogAuthData; //野狗用户登录信息

//监听登录状态变化
wilddog.onAuth(function(data) {
	//如果已登录则将用户数据存储到全局变量方便调用
	wilddogAuthData = data;
	if (wilddogAuthData) {
		console.log(wilddogAuthData);
	} else {
		//未登录则调用野狗登录方法，这里只是简单的使用微博授权登录，其他登录方法查看官方文档。
		// 弹出新浪微博OAuth认证
		wilddog.authWithOAuthRedirect("weibo", authHandler);
	}
});

// 创建一个回调来处理终端用户认证的结果，微博登录成功后的回调方法
function authHandler(error, data) {
	if (error) {
		console.log("Login Failed!", error);
	} else {
		console.log("Authenticated successfully with payload:", data);
	}
}
```
授权登录成功后可获得用户信息
![](http://7xo2m9.com1.z0.glb.clouddn.com/image/f/83/a9f53a4c456cafd38e1bdd32cd25f.png)

###### 获取游戏结果
在游戏结束方法里加入
```
//打破自己的记录才上传，一般html5游戏会将最佳成绩存在localstorage中，根据实际情况做修改即可
if(score > bestScore){
	if(!wilddogAuthData){alert('你没有使用微博账号登陆,无法计入成绩!');return false;}
	var ts = new Date().getTime();
	wilddogRef.child('rank').child(wilddogAuthData.auth.uid).set({
		//这里的字段根据自己需求定义
		uid:    wilddogAuthData.auth.uid
		//为了尽量避免伪造数据这里将score做加密处理并放在伪造的token字段里混淆视听，取出成绩时再解密比较token与score字段即可,并不能从根本上防止作弊。
		token:  sjcl.encrypt(ts+"",score+""),
		score:  score,
		ts:     ts,
		rank:   t+'_'+(3000000000000-ts),
		UA:     navigator.userAgent
	});
}
```
>rank字段用于orderByChild()方法，该方法对字符串按照字典顺序来拍的。这里的`t`是在score前面补0到6位数方便排序 ，`score=100` 则 `t=000100`，这样组合之后可以确定高分在前，分数相同则先达到该分数的用户在前

###### 获取排行榜
```
//获取数据，并按照对象中的 rank 字段排序返回结果集中的后10位
wilddogRef.child('rank').orderByChild('rank').limitToLast(10).on("value", function(users) {
    var html = [];
    users.forEach(function (user) {
        var item = user.val();
        //比较score与加密的'score'，不匹配则忽略
        if(sjcl.decrypt(item.ts+"",item.token) == item.score) {
        // .orderByChild()方法是升序，所以这里使用的是'unshift'方法
            html.unshift('<li><span class="ranknum"></span><i class="ran-1"></i><img src="'+ item.avatar +'" alt="'+ item.name +'" class="avatar"><div class="info"><span class="name">'+ item.name +'</span></div><span class="score">'+ item.score +' 分</span></li>');
        }
    });
    document.getElementById('rank-list').innerHTML = html.join('');
});
```

完成啦
![](http://7xo2m9.com1.z0.glb.clouddn.com/image/1/f6/1b04f77d44cafcf34cf5b34b13736.png)

有兴趣的同学可以玩一下，完全实时的哦。简单demo没有做过多优化，打开页面后会直接弹出微博授权页。
[传送门](http://h5.wan2sha.com/80000320160503/)
