---
title: "spring boot - properties 분리하기"
categories: 
  - spring boot
tags:
  - java
  - spring boot
last_modified_at: 2019-03-04T13:00:00+09:00
author_profile: true
sidebar:
  - nav: fblogin-nav
---
![1](/assets/img/posts/java/springboot/properties/1.png)

**properties** 파일안에 **db connection** 정보나 **file path** 등등 여러가지 속성들을 기재해서 많이 사용하시죠?

저 또한 예전부터 **properties** 파일에 여러가지를 기재해서 사용을 많이 했습니다.

오늘은 이 **properties** 파일을 외부로 내보내서 여러가지 **properties** 파일을 관리를 하는 작업에 대해 작성을 하려고 합니다.

### properties를 왜 외부로 빼서 관리를 하려는 것인가?
이유는 간단합니다. 어플리케이션을 만든 후 jar나 war로 빌드 후 (jar를 이용합시다!) 서버에 배포 후에 갑작스럽게 테스트를 하거나 다른 작업을 할 때 소스는 변경사항이 없으나 다른 db를 잠시 바라보고 싶을 때 어떻게 하시나요? class path 안에 있는 properties 파일을 변경 후 다시 빌드를 하지 않고 **properties** 파일을 외부로 빼서 경로만 바꿔주고 실행시키면 간단합니다!

Spring doc 에서도 [Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)이라는 페이지가 있는데 한번 참고 해보세요.

## 사용

### spring boot에서 properties 파일을 읽는 방법은 다양합니다.

1. `@Value` 어노테이션 사용하기
2. `Environment` 클래스 사용하기

### [spring doc](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)에서 **properties** 적용 우선순위는 다음과 같습니다.

1. `file:./custom-config/`
2. `classpath:custom-config/`
3. `file:./config/`
4. `file:./`
5. `classpath:/config/`
6. `classpath:/`

###  jar command 에서 spring boot에서 외부의 properties 파일을 불러오는 option은 다음과 같은 방법 들이 있습니다.

1. `--spring.config.name`
        
        java -jar myproject.jar --spring.config.name=myproject

2. `--spring.config.location`

        java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties

### 삽질

spring mvc 프로젝트를 spring boot 프로젝트로 옮기며 rest api로 만드는 작업을 했는데, 기존의 spring mvc 프로젝트를 만든 사람이 `java.util.properties`를 이용해 초기화 블럭을 이용해 사용을 해서 `spring.config.location`이 안먹혔습니다.


    {% highlight java %}
    static {
            loadProperties(); // 초기화 블럭 때문에 --spring.config.location이 안먹힘

        }
        
        synchronized static private void loadProperties() {
            InputStream fis = null;
            try{
                properties = new Properties();
                fis = PropertiesService.class.getClassLoader().getResourceAsStream("application.properties");

                properties.load(fis);
            }catch(FileNotFoundException fne){
                debug(fne);
            }catch(IOException ioe){
                debug(ioe);
            }catch(Exception e){
                debug(e);
            }finally{
                try {
                    if (fis != null) { fis.close(); } 
                    else {
                        fis = PropertiesService.class.getClassLoader().getResourceAsStream("application.properties");
                        properties.load(fis);
                        fis.close();
                    }
                } catch (Exception ex) {
                    //ex.printStackTrace();
                    logger.info("IGNORED: " + ex.getMessage());
                }
            }
        }    
    {% endhighlight %}

이것을 어떻게 해결해야 하나 계속 고민하다가, 결국에는 `System.property`를 이용해서 강제적으로 위치를 바꿔주기로 했습니다. 

    {% highlight java %}
    static {
            loadProperties();
        }
        
        synchronized static private void loadProperties() {
            InputStream fis = null;
            try{
                properties = new Properties();
                            // 여기 고침
                fis = new FileInputStream(System.getProperty("spring_config_location").replaceAll("\\\\","/"));
                properties.load(fis);
            }catch(FileNotFoundException fne){
                debug(fne);
            }catch(IOException ioe){
                debug(ioe);
            }catch(Exception e){
                debug(e);
            }finally{
                try {
                    if (fis != null) { fis.close(); } 
                    else {
                        fis = PropertiesService.class.getClassLoader().getResourceAsStream("application.properties");
                        properties.load(fis);
                        fis.close();
                    }
                } catch (Exception ex) {
                    //ex.printStackTrace();
                    logger.info("IGNORED: " + ex.getMessage());
                }
            }
        }    
    {% endhighlight %}

`--spring.config.location`을 이용했다면 **자동**으로 그 위치를 참조했을 텐데 사용하던 코드 때문에 제가 직접 main이 실행되기 전에 강제적으로 위치를 지정해 줘야 했습니다.    

    {% highlight java %}
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.builder.SpringApplicationBuilder;
    import org.springframework.context.annotation.EnableAspectJAutoProxy;

    @SpringBootApplication
    @EnableAspectJAutoProxy
    public class RestWebApplication {
        
        public static void main(String[] args) {
    //             기존의 spring boot 프로젝트
    //		SpringApplication.run(RestWebApplication.class, args);
            
            String location;                 
            if (System.getProperty("spring_config_location") != null) {
                location = System.getProperty("spring_config_location").replaceAll("\\\\","/");			
            }  else {
                location = "classpath:/application.properties";
            };
            
            new SpringApplicationBuilder(KapRestWebApplication.class)
            .properties(
                    "spring.config.location=" + location +
                    "classpath:/application.properties" +
                    ", file:" + location
                    ).run(args);				
        }
    }    
    {% endhighlight %}

### 강제적으로 이렇게 만들었는데 그럼 어떻게 실행을 해야 하나????

**java VM Arguments** 에서 **custom argumets**를 주는 법을 아시나요? 바로 `-Dcustom_arg`를 이용해 앞에 `-D` 를 붙여주는 겁니다.

그리고 저는 linux에서 실행 시키기 위해 다음과 같은 sh 파일을 만들었습니다.    

    #!/bin/sh
    classpath=$(cd "$(dirname "$0")" && pwd)

    FILEPATH=${classpath}'/RestWeb.jar'

    if [ "$1" = "dev" ] || [ "$1" = "real" ] || [ "$1" = "test" ]; then
        config=''${classpath}'/'$1'/application.properties'
    else
        config='classpath:/application.properties'
    fi

    # echo java -Dspring_config_location=${config} -jar ${FILEPATH} --spring.config.location=${config}
    # java -Dspring_config_location=${config} -jar ${FILEPATH} --spring.config.location=${config}
    nohup java -Dspring_config_location=${config} -jar ${FILEPATH} >> /dev/null 2>&1 &    


---
#### 참고 및 출처


- <https://docs.spring.io/spring-boot/docs/1.3.0.M5/reference/html/howto-properties-and-configuration.html>

- <https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html>
- <https://araikuma.tistory.com/42>
- <https://kingbbode.tistory.com/39>