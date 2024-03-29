# bigpipe 学习

## 传统的渲染逻辑

网页全部拼装完，生成 HTML 字符串，传给客户端渲染

commit 925c0de

可以看到大概 3s 后返回网页

## 分段传输

大约 1s 后渲染第一块，2s 后渲染第二块，3s 后渲染第三块，体验比传统的渲染好了很多，用户不用干等到全部内容返回了

commit a3d4104

注意本地如果是 nginx 服务，需要配置如下，不然看不到效果：

```
....
fastcgi_buffer_size 1k; 
fastcgi_buffers 16 1k; 
gzip  off;
....
```

## 解决分段传输顺序的问题

上面的分段传输是基于上快下慢，如果是上慢下快呢？我们把上面 demo 的第一块改为 3s，那么效果就是大约 3s 后出现第一块，再 1s 后出现第二块，再 1s 出现第三块

如何优化呢？如果能把最慢的部分放置于底部传过来就好了，我们可以使用 **js 回填** 的方式，先将第一块最慢的部分架空，然后在底部写上 js 回填

commit 43457dd

效果是 1s 后出现第二块，再 1s 后出现第三块，再 3s 后出现第一块

## 回填思路的扩展与并行化

上面的思路非常简单，但是需要事先知道分块的快慢（比如 demo 事先知道了第一块最慢），而且三块的显示，并不是并行的（理想状态应该是 3s 后全部三块显示完）。但是事实上，当块数很多的时候，你很难知道快慢，**我们可以把所有的块都架空**，**然后并行渲染**，谁快谁就先渲染回填 js，这样就可以达到并行且先到先渲染的目的了

这部分可以参考 <https://segmentfault.com/a/1190000005989601>

## 应用

bigpipe 是 facebook 提出的，广泛应用在 SNS 社交网站上，国内的微博、人人应该也有用到这个技术

