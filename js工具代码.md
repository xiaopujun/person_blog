# js获取url中的值

```js
/**
     * 获取参数的值
     * 使用方法：getQueryString(参数名）
     * @param name 参数名
     * @returns {string|null}
     */
    function getQueryString(name) {
        var reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)', 'i');
        var r = window.location.search.substr(1).match(reg);
        if (r != null) {
            return unescape(r[2]);
        }
        return null;
    }
```

