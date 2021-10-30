# nginx-docker-spring
간단한 springboot app을 이용해 간단히 실습합니다~

## 1. 간단 rest controller 정의
```java
@SpringBootApplication
@RestController
public class DockerApplication {

    @RequestMapping("/")
    public String home() {
        return "Hello Docker World";
    }
    public static void main(String[] args) {
        SpringApplication.run(DockerApplication.class, args);
    }
}
```
