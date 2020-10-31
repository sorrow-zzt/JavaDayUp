## SpringBoot2.X -- Valid & Validated(参数校验)

### 1.简单的参数校验

```java
package com.github.validated.pojo;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.github.validated.annotation.Built;
import lombok.Data;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.Range;
import org.hibernate.validator.constraints.URL;

import javax.validation.constraints.AssertTrue;
import javax.validation.constraints.DecimalMin;
import javax.validation.constraints.Email;
import javax.validation.constraints.Future;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Past;
import javax.validation.constraints.Size;
import java.util.Date;
import java.util.List;

@Data
public class UserComplexDTO {


    /**
     * 字符串，集合，map限制大小
     */
    @NotEmpty(message = "名字不能为空")
    @Size(min = 2, max = 6, message = "名字长度在2-6位")
    @Length(min = 2, max = 6, message = "名字长度在2-6位")
    private String name;

    @Length(min = 3, max = 3, message = "pass 长度不为3")
    private String pass;

    /**
     * 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
     */
    @DecimalMin(value = "10", inclusive = true, message = "salary 低于10")
    private Integer salary;

    @Range(min = 5, max = 10, message = "range 不在范围内")
    private Integer range;

    @NotNull(message = "年龄不能为空")
    @Min(value = 18, message = "年龄不能小于18")
    @Max(value = 70, message = "年龄不能大于70")
    private Integer age;

    @Email
    private String email;

    @AssertTrue
    private Boolean flag;

    @Past
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    private Date birthday;

    @Future
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    private Date expire;

    @URL(message = "url 格式不对")
    private String url;

    @Built(value = {"1", "2", "3"})
    private String built;
    //@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式

    /**
     * 字符串，集合，map限制大小
     */
    @Size(min = 2, max = 6, message = "长度在2-6位")
    private List<Integer> list;
}

```

![1564644040406](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564644040406.png)

![1564644090650](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564644090650.png)

只有参数全部通过校验时才会正常返回数据,否则会抛出异常!

### 2.自定义参数校验返回结果

```java
package com.github.validated.advice;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.github.validated.pojo.Result;
import com.github.validated.pojo.ResultUtil;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import javax.validation.ConstraintViolationException;
import java.util.stream.Collectors;

/**
 * <p>
 * 创建时间为 上午11:03-2019/1/24
 * 项目名称 SpringBootValidated
 * </p>
 *
 * @author shao
 * @version 0.0.1
 * @since 0.0.1
 */

@RestControllerAdvice
public class CheckAdvice {

    //private final static Logger logger = LoggerFactory.getLogger(SysMenuServiceImpl.class);
    private static final ObjectMapper MAPPER = new ObjectMapper();

    /**
     * 请求的 JSON 参数在请求体内的参数校验
     *
     * @param e 异常信息
     * @return 返回数据
     * @throws JsonProcessingException jackson 的异常
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<String> handleBindException1(MethodArgumentNotValidException e) throws JsonProcessingException {
        e.getBindingResult().getAllErrors().forEach(System.err::println);
        //return new ResponseEntity<>("cuowu:" + MAPPER.writeValueAsString(e.getBindingResult().getAllErrors()), HttpStatus.BAD_REQUEST);
        return new ResultUtil<String>().setErrorMsg(HttpStatus.BAD_REQUEST.value(), e.getBindingResult().getAllErrors().stream().map(a->a.getDefaultMessage()).collect(Collectors.toList()).toString());
    }

    /**
     * 请求的 URL 参数检验
     *
     * @param e 异常信息
     * @return 返回提示信息
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(ConstraintViolationException.class)
    public Result<String> handleBindException2(ConstraintViolationException e) {
        e.getConstraintViolations().forEach(System.err::println);
        //return "ConstraintViolationException";
        return new ResultUtil<String>().setErrorMsg(HttpStatus.BAD_REQUEST.value(),e.getMessage());
    }

}
```

![1564647009828](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564647009828.png)

### 3.分组校验

![1564647418668](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564647418668.png)

![1564647444023](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564647444023.png)

![1564647535312](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564647535312.png)

### 4.验证@PathVariable和@RequestParam

![1564648082587](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564648082587.png)

![1564648096982](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564648096982.png)

![1564648146067](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564648146067.png)

自定义参数校验和动态校验:

...

参考:<https://www.jianshu.com/p/764eaf6c0afe>