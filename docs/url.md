# URL 对象

浏览器内置的 URL 对象，代表一个网址。通过这个对象，就能生成和操作网址。

## 构造函数

URL 可以当作构造函数使用，生成一个实例对象。

它接受一个网址字符串作为参数。

```javascript
let url = new URL('https://example.com');
```

如果网址字符串无法解析，它会报错，所以它要放在`try...catch`代码块里面。

如果这个参数只是一个网站路径，比如`/foo/index.html`，那么需要提供基准网址，作为第二个参数。

```javascript
const url1 = new URL('page2.html', 'http://example.com/page1.html');
url2.href // "http://example.com/page2.html"

const url2 = new URL('..', 'http://example.com/a/b.html')
url3.href // "http://example.com/"
```

这种写法很方便基于现有网址，构造新的 URL。

`URL()`的参数也可以是另一个 URL 实例。这时，`URL()`会自动读取该实例的href属性，作为实际参数。

## 实例属性

一旦得到了 URL 实例对象，就可以从它的各种属性，方便地获取 URL 的各个组成部分。

- href：完整的网址
- protocol：访问协议，带结尾冒号`:`。
- search：查询字符串，以问号`?`开头。
- hash：哈希字符串，以`#`开头。
- username：需要认证的网址的用户名。
- password：需要认证的网址的密码。
- host：主机名，不带协议，但带有端口。
- hostname：主机名，不带协议和端口。
- port：端口。
- origin：包括协议、域名和端口。
- pathname：服务器路径，以根路径`/`开头，不带有查询字符串。
- searchParams：指向一个 URLSearchParams 实例，方便用来构造和操作查询字符串。

下面是用法示例。

```javascript
const url = new URL('http://user:pass@example.com:8080/resource/path?q=1#hash');

url.href // http://user:pass@example.com:8080/resource/path?q=1#hash
url.protocol // http:
url.username // user
url.password // pass
url.host // example.com:8080
url.hostname // example.com
url.port // 8080
url.pathname // /resource/path
url.search // ?q=1
url.hash // #hash
url.origin // http://example.com:8080 
```

这些属性里面，只有`origin`属性是只读的，其他属性都可写，并且会立即生效。

```javascript
const url = new URL('http://example.com/index.html#part1');

url.pathname = 'index2.html';
url.href // "http://example.com/index2.html#part1"

url.hash = '#part2';
url.href // "http://example.com/index2.html#part2"
```

上面示例中，改变 URL 实例的`pathname`属性和`hash`属性，都会实时反映在 URL 实例当中。

下面是`searchParams`属性的用法示例，它的具体属性和方法介绍参见 《URLSearchParams》一章。

```javascript
const url = new URL('http://example.com/path?a=1&b=2');

url.searchParams.get('a') // 1
url.searchParams.get('b') // 2

for (const [k, v] of url.searchParams) {
  console.log(k, v);
}
// a 1
// b 2
```

## 静态方法

### URL.createObjectURL()

`URL.createObjectURL()`方法用来为文件数据生成一个临时网址（URL 字符串），供那些需要网址作为参数的方法使用。该方法的参数必须是 Blob 类型（即代表文件的二进制数据）。

```javascript
// HTML 代码如下
// <div id="display"/>
// <input
//   type="file"
//   id="fileElem"
//   multiple
//   accept="image/*"
//   onchange="handleFiles(this.files)"
//  >
const div = document.getElementById('display');

function handleFiles(files) {
  for (let i = 0; i < files.length; i++) {
    let img = document.createElement('img');
    img.src = window.URL.createObjectURL(files[i]);
    div.appendChild(img);
  }
}
```

上面示例中，`URL.createObjectURL()`方法用来为上传的文件生成一个临时网址，作为`<img>`元素的图片来源。

该方法生成的 URL 就像下面的样子。

```javascript
blob:http://localhost/c745ef73-ece9-46da-8f66-ebes574789b1
```

注意，每次使用`URL.createObjectURL()`方法，都会在内存里面生成一个 URL 实例。如果不再需要该方法生成的临时网址，为了节省内存，可以使用`URL.revokeObjectURL()`方法释放这个实例。

下面是生成 Worker 进程的一个示例。

```html
<script id='code' type='text/plain'>
  postMessage('foo');
</script>
<script>
  var code = document.getElementById('code').textContent;
  var blob = new Blob([code], { type: 'application/javascript' });
  var url = URL.createObjectURL(blob);
  var worker = new Worker(url);
  URL.revokeObjectURL(url);

  worker.onmessage = function(e) {
    console.log('worker returned: ', e.data);
  };
</script>
```

### URL.revokeObjectURL()

`URL.revokeObjectURL()`方法用来释放`URL.createObjectURL()`生成的临时网址。它的参数就是`URL.createObjectURL()`方法返回的 URL 字符串。

下面为上一小节的示例加上`URL.revokeObjectURL()`。

```javascript
var div = document.getElementById('display');

function handleFiles(files) {
  for (var i = 0; i < files.length; i++) {
    var img = document.createElement('img');
    img.src = window.URL.createObjectURL(files[i]);
    div.appendChild(img);
    img.onload = function() {
      window.URL.revokeObjectURL(this.src);
    }
  }
}
```

上面代码中，一旦图片加载成功以后，为本地文件生成的临时网址就没用了，于是可以在`img.onload`回调函数里面，通过`URL.revokeObjectURL()`方法释放资源。

## 实例方法

### toString()

URL 实例对象的`toString()`返回`URL.href`属性，即整个网址。

