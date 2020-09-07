## �������У�� @Validated
ʹ�� spring validation ����javax.validation ע������쳣��������ʵ��ע��ʽ����У�飬��ҵ���߼�����
* ���� path variable��request param ��Ҫ�����ϼ� @Validated ע�⣬У����������ã�Ҳ����˵������Ϊ��������ʱ�������ڻ������Ͳ�����ֱ�ӱ�������У�鷶Χ�����������ϱ���ע�� @Validated
* ����Ϊjava����ʱ��������Ҫ�ڶ���������������У���������ڴ���ʱ����java����Ҫ��� @Validated
* ���õļ���ע������
  * @AssertFalse У��false
  * @AssertTrue У��true
  * @DecimalMax(value=,inclusive=) С�ڵ���value��inclusive=true,��С�ڵ���
  * @DecimalMin(value=,inclusive=) ��������
  * @Max(value=) С�ڵ���value
  * @Min(value=) ���ڵ���value
  * @NotNull  ���Null
  * @Past  �������
  * @Pattern(regex=,flag=)  ����
  * @Size(min=, max=)  �ַ��������ϣ�map���ƴ�С
  * @Validate ��poʵ�������У��

```java
@Slf4j
@Validated // ���� path variable��request param ��Ҫ�����ϼ� @Validated ע��
@RestController
@RequestMapping("/demo")
public class DemoController {
    @GetMapping("/{pathVal}")
    public Resp<String> getDemo(@Max(100) @PathVariable long pathVal, // path variable У�����
                                @Max(100) @RequestParam long reqParam) { // request param У�����
        String response = String.format("pathVal: %s, reqParam: %s", pathVal, reqParam);
        log.info("GET request, {}", response);
        return new Resp<>(response);
    }

    @Data
    private static class PutReq {
        @Max(100) // request body У�����
        @NotNull
        private Long id;
        @NotNull(message = "������ְ�") // �Զ��������Ϣ
        private String name;
        @Email
        private String email;
    }

    @PutMapping("/{pathVal}")
    public Resp<String> putDemo(@Max(100) @PathVariable long pathVal,
                                @Validated @RequestBody PutReq req) { // ���� request body ��Ҫ�ڲ����ϼ� @Validated ע��
        String response = String.format("pathVal: %s, req: %s", pathVal, req);
        log.info("PUT request, {}", response);
        return new Resp<>(response);
    }
}
```

