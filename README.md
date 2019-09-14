# 踩坑记录🤢
## 格式
+ 标题   
+ 详情
+ 技术标签

### vue router高亮
vue router是根据URL路径和嵌套匹配的.

```
{
	path: '/aaa',	
	children: [
		{
			path: '/aaa/bbb', //success
		},
		{
			path: '/ccc/aaa', //fail
		},
	]
},
```
<span style="display: inline-block; background: #eff3f6; padding: 0 4px ">vue</span>
<span style="display: inline-block; background: #eff3f6; padding: 0 4px ">vue router</span>

## mv隐藏文件
但遇到把origin文件夹所以文件移到target文件夹中，执行

```
mv origin/* target
```

但发现origin里面的 ```.```开头隐藏文件没有迁移。后来查询发现。   
需要运行


```
shopt -s dotglob
```
bash在文件名扩展的结果中包括以点（.）开头的文件名

## iphone内容超入滚动区域显示空白
但块状区域内容过多时，在iphone浏览器会显示空白，但区域是能交互的，例如button可以点击。

```
document.querySelector('element').parentElement.style.display = 'none';
document.querySelector('element').parentElement.style.display = 'block';
```

在stackoverflow解决方案   
<https://stackoverflow.com/questions/17582439/webkit-overflow-scrolling-touch-large-content-gets-cut-off-when-specifying-a>
