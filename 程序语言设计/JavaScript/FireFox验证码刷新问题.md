# FireFox验证码刷新问题


## 问题描述
登录功能中验证码的实现，图片的src是一个请求地址

```html
<img id="yzm" src=""src", "../admin/getValidateCode"/>
```

在请求响应中写入图片验证码

```java
@GetMapping("/getValidateCode")
    public void getImageValidateCode(HttpServletRequest request, HttpServletResponse response) throws Exception {
        String type = request.getParameter("type");
        ValidateCodeProcessor validateCodeProcessor = validateCodeProcessors.get(codeProcessorType);
        if (validateCodeProcessor == null) {
            throw new CommonException(ErrorCode.UNKNOWN_RESOURCE, "processor不存在");
        }
        ServletWebRequest servletWebRequest = new ServletWebRequest(request, response);
        validateCodeProcessor.create(servletWebRequest);
    }
```

在火狐浏览器中，点击图片刷新验证码存在问题，图片不会刷新。

## 解决方式
由于每次请求地址都一样，火狐将请求缓存因此不会刷新。在请求地址加上时间戳，每次请求地址不一样即可

```javascript
$("#yzm").attr("src", "../admin/getValidateCode?type=image&&deviceId=" + localStorage.getItem("deviceId") + "&&t="+ new Date().getTime() ) ;
```

## Firefox中事件不可用问题
```javascript
/**
 * 处理Firefox中window.event不可用问题
 */
if(typeof(window.event) == "undefined")
{
    var $E = function(){
        var c = $E.caller;
        console.log(c.caller);
        while(c.caller) {
            c = c.caller;
        }
        return c.arguments[0]
    };
    window.__defineGetter__("event", $E);
}
```
