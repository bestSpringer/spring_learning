## 请求参数校验 @Validated
使用 spring validation 包、javax.validation 注解配合异常处理切面实现注解式参数校验，与业务逻辑解耦
* 处理 path variable、request param 需要在类上加 @Validated 注解，校验参数起作用，也就是说，传参为基本类型时，可以在基本类型参数上直接表明参数校验范围，并且在类上表明注解 @Validated
* 传参为java对象时，首先需要在对象的属性上面添加校验规则，其次在传参时，对java对象要添加 @Validated
* 常用的检验注解类型
  * @AssertFalse 校验false
  * @AssertTrue 校验true
  * @DecimalMax(value=,inclusive=) 小于等于value，inclusive=true,是小于等于
  * @DecimalMin(value=,inclusive=) 与上类似
  * @Max(value=) 小于等于value
  * @Min(value=) 大于等于value
  * @NotNull  检查Null
  * @Past  检查日期
  * @Pattern(regex=,flag=)  正则
  * @Size(min=, max=)  字符串，集合，map限制大小
  * @Validate 对po实体类进行校验

```java
@Slf4j
@Validated // 处理 path variable、request param 需要在类上加 @Validated 注解
@RestController
@RequestMapping("/demo")
public class DemoController {
    @GetMapping("/{pathVal}")
    public Resp<String> getDemo(@Max(100) @PathVariable long pathVal, // path variable 校验规则
                                @Max(100) @RequestParam long reqParam) { // request param 校验规则
        String response = String.format("pathVal: %s, reqParam: %s", pathVal, reqParam);
        log.info("GET request, {}", response);
        return new Resp<>(response);
    }

    @Data
    private static class PutReq {
        @Max(100) // request body 校验规则
        @NotNull
        private Long id;
        @NotNull(message = "起个名字吧") // 自定义错误消息
        private String name;
        @Email
        private String email;
    }

    @PutMapping("/{pathVal}")
    public Resp<String> putDemo(@Max(100) @PathVariable long pathVal,
                                @Validated @RequestBody PutReq req) { // 处理 request body 需要在参数上加 @Validated 注解
        String response = String.format("pathVal: %s, req: %s", pathVal, req);
        log.info("PUT request, {}", response);
        return new Resp<>(response);
    }
}
```

