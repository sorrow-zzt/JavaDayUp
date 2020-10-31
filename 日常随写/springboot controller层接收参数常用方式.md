

# springboot controller层接收参数常用方式

#### 1.请求路径参数---GET(@PathVariable/@RequestParam)

获取路径参数。即*url/{id}*这种形式

获取查询参数。即*url?name=*这种形式

```
GET 
http://localhost:8080/demo/123?name=suki_rong 
对应的java代码：
@GetMapping("/demo/{id}")
public void demo(@PathVariable(name = "id") String id, @RequestParam(name = "name") String name) {
    System.out.println("id="+id);
    System.out.println("name="+name);
}
```

获取查询参数。即*url?page=1&limit=1*这种形式

![1572486797505](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1572486797505.png)

![1572486856088](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1572486856088.png)

#### 2.Body参数---POST(@RequestBody/json)

![1564193011140](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564193011140.png)

![1564192828034](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564192828034.png)

1.Post请求接收只能用**实体或map接收**json数据

![1572343578362](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1572343578362.png)

#### 3.表单form传参---GET/POST(Body/无注解)

![1564192958651](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564192958651.png)

#### 4.请求头参数以及Cookie---(@RequestHeader/@CookieValue)

```
@GetMapping("/demo3")
public void demo3(@RequestHeader(name = "myHeader") String myHeader,
        @CookieValue(name = "myCookie") String myCookie) {
    System.out.println("myHeader=" + myHeader);
    System.out.println("myCookie=" + myCookie);
}
---------------------------------------------------------------------------
@GetMapping("/demo3")
public void demo3(HttpServletRequest request) {
    System.out.println(request.getHeader("myHeader"));
    for (Cookie cookie : request.getCookies()) {
        if ("myCookie".equals(cookie.getName())) {
            System.out.println(cookie.getValue());
        }
    }
}
```

