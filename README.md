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