# 爬虫

## XPath

### XPath语法

- 基础
	- `/` 			子节点
	- `//` 			子孙节点
	- `.` 			当前节点
	- `..` 			父节点
	- `@` 			属性
	- `[]` 			约束
	- `|`			或（多个筛选结果的合并）

- 数组选择
	- `index`		第几个元素（从`1`开始）
	- `last()`		最后一个（可以加减）
	- `position<3`	小于3的元素

- 约束
	- `[@name]`						有`name`属性
	- `[@name='user']`				有`name`属性，且值为`user`
	- `[span>100]`					子元素`span`的值大于`100`
	- `[contains(text(), 'abc')]`	该节点的内容包含`abc`

- 计算
	- `<`、`>`、`=`、`<=`、`>=`、`!=`
	- `+`、`-`、`*`、`div`、`mod`
	- `and`、`or`

- 通配符
	- `*`			所有节点
	- `@*`			所有属性
	- `node()`		所有节点

### 提取`点赞数`超过`10000`的段子
```python
# http://neihanshequ.com/

//*[@id="detail-list"]/li/div/div[3]/ul/li[1][span>10000]/../../../div[2]
```

### 利用etree的xpath方法提取

```python
from lxml import etree

text = """
<div> 
    <ul> 
        <li class="item-1"><a href="link1.html"></a></li> 
        <li class="item-1"><a href="link2.html">second item</a></li> 
        <li class="item-inactive"><a href="link3.html">third item</a></li> 
        <li class="item-1"><a href="link4.html">fourth item</a></li> 
        <li class="item-0"><a href="link5.html">fifth item</a> 
    </ul> 
</div>
"""

html = etree.HTML(text)  # `HTML`方法和`tostring`方法互为逆方法
ret = list()

for item in html.xpath('//li/a'):
    url, content = item.xpath('./@href'), item.xpath('./text()')

    item_dict = {}
    item_dict['url'] = url[0] if len(url) else ''
    item_dict['content'] = content[0] if len(content) else ''
    ret.append(item_dict)

print(ret)
```

