# 爬虫

## Xpath


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

html = etree.HTML(text)
ret = list()

for item in html.xpath('//li/a'):
    url, content = item.xpath('./@href'), item.xpath('./text()')
    item_dict = {}
    item_dict['url'] = url[0] if len(url) else ''
    item_dict['content'] = content[0] if len(content) else ''
    ret.append(item_dict)

print(ret)
```

