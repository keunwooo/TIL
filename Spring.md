# 스프링



### 	JPA

- JPA 참고 문서

  https://attacomsian.com/blog/spring-data-jpa-many-to-many-mapping#dependencies

- 배포 시 주의사항 -> 배포시 리스타트 기능을 꺼야 한다.

  : 리스타트 하고 싶지 않은 리소스는? spring.devtools.restart.exclude
  : 리스타트 기능 끄려면? spring.devtools.restart.enabled=false



- create 은 테이블을 드랍하고 다시 생성

- update는 유지하면서 변동사항 적용

- validate 는 table 과 entity 가 일치하는지 검사. 

  기존 데이터 상태에서 Account.java 에 필드를 추가하면 에러 발생. 

  반영을 바로하지 않고 검증을 한다. 

  - 해결 방법 업데이트로 바꾸거나 추가한 필드를 지운다

- 개발 모드에선 update 운영모드에선 validate





- optional
- 추상형메소드가 딱 1개 잇는것이 함수형 인터페이스고 이러한걸 람다식으로 표현이 가능
- 아규먼트 타입이 1개일때. 함수형 을 쓸 수 있다. => 함수
- https://jsbin.com/?html,output 자바스크립트 짤수 잇는 함수 

```javascript
//일반적인 함수
function sayHello(msg){
  return 'Hello'+msg;
}

console.log(sayHello(' lambda'));

//애로우 함수
let sayHello2 = msg => 'Hello ' + msg;

console.log(sayHello2('람다'));
```



```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;

public class LambdaTest {
	public static void main(String[] args) {
		List<String> asList = Arrays.asList("자바","람다","부트");
		//1단계 : MyConsumer 클래스를 생성 전달 , 단점 : 계속 class 를 만들어야함
		asList.forEach(new MyConsumer());
		
		//2단계 : Consumber를 anonymous inner class로 생성해서 accept 메서드를 오버라이딩
		//1회용으로 사용.
		asList.forEach(new Consumer<String>() {
			//forEach 가 Consumber 타입이면
			@Override
			public void accept(String t) {
				System.out.println(t);
			}
			
		});
		System.out.println("람다식 Reference");
		//3단계 : Consumer의 accept() 메서드 오버라이딩을 람다식으로 사용 
		//추상메서드가 1개이기 때문에 람다식으로 표현하자.
		asList.forEach(value->System.out.println(value));
		
		//4단계: Consumer의 accept() 메서드 오버라이딩을 Method Reference로 사용하기
		System.out.println("Method Reference");
		asList.forEach(System.out::println);
	}
}

class MyConsumer implements Consumer<String>{

	@Override
	public void accept(String t) {
		System.out.println(t);
	}	
}
```

```java

public class ThreadTest {
	public static void main(String[] args) {
		System.out.println(Thread.currentThread().getName());
		// 1. Thread 생성하는 방법
		// Thread 클래스를 상속받는다.
		Thread t1 = new MyThread();
		t1.setName("둘리의 쓰레드");
		t1.start();

		// 2. Runnable 객체를 생성해서 Thread에 인자로 전달하는 방법
		Thread t2 = new Thread(new MyRunnable());
		t2.setName("호이의 쓰레드");
		t2.start();

		// 3. Lambda 식으로 Runnable 구현 run method 1개이므로 람다식 가능
		Thread t3 = new Thread(() -> {
			for (int i = 0; i < 10; i++) {
				try {
					Thread.sleep(2);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName() + i);
			}
		});
		t3.setName("고길동의 쓰레드");
		t3.start();

	}
}

class MyRunnable implements Runnable {
	@Override
	public void run() {

		System.out.println(Thread.currentThread().getName() + " 시작");
		

		for (int i = 0; i < 10; i++) {
			try {
				Thread.sleep(2);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName() + i);
		}
	}
}

class MyThread extends Thread {
	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + " 시작");
		

		for (int i = 0; i < 10; i++) {
			try {
				Thread.sleep(2);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName() + i);
		}
	}
}
```



- optional 응용 코드

  ```java
  package com.keunwoo.myspringboot.repository;
  
  import java.util.Optional;
  
  import org.junit.Ignore;
  import org.junit.Test;
  import org.junit.runner.RunWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.test.context.junit4.SpringRunner;
  
  import com.keunwoo.myspringboot.entity.Account;
  
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class AccountRepositoryTest {
  	
  	@Autowired
  	private AccountRepository repository;
  	
  	@Test
  	public void finder() {
  		Account account = repository.findByUsername("lambda");
  		System.out.println(account);
  		
  		//있으면 정상적으로 꺼내짐
  		//없으면 JUnit 테스트 에러 get을 무조건 쓰면안되.
  		Account account2;
  		Optional<Account> optional = repository.findById(100L);
  		System.out.println(optional.isPresent());
  		
  		if(optional.isPresent()) {
  			account2 = optional.get();
  			System.out.println(account2);
  		}
  		//요청 ID가 있으면 Account 객체를 반환하고, 없으면 예외를 발생시킨다.
  		Optional<Account> optEmail = repository.findByEmail("keunwo@aa.com");
  		//Supplier 의 함수형 인터페이스의 추상메서드 : T get()
  		Account account3 = optEmail.orElseThrow(()->new RuntimeException("요청한 이메일 주소를 가진 Account가 없습니다"));
  		System.out.println(account3);
  		
  		
  	}
  	@Test @Ignore
  	public void account() throws Exception{
  		System.out.println("==>"+repository.getClass().getCanonicalName());
  		//1.Account 객체 생성 -> 등록
  		Account account = new Account();
  		account.setUsername("lambda");
  		account.setPassword("1234");
  		account.setEmail("keunwoo@aa.com");
  		Account addAccount = repository.save(account);
  		System.out.println(addAccount.getId() + " " + addAccount.getUsername());
  		
  		
  	}
  }
  
  ```

- Optional 은 1개만 저장이 가능하기 때문에 같은 값이 중복된걸 find 하게 되면 에러가 난다.

```
List<Account> accountList = repository.findAll(); //Iterable 타입임
		//accountList.forEach(acct->System.out.println(acct));
		accountList.forEach(System.out::println);
```



- maria DB mysql 데이터 삽입시 주의사항

  poperties 가 update 세팅 상태에서 처음 value를 넣을 때 auto increment 안되는 증상

  hibernate_sequnce 테이블 제거

  auto-increment 가 설정 되야한다.

  

## 부트 MVC

- ResponseEntity https://www.baeldung.com/spring-response-entity

  - 응답 데이터 뿐만 아니라 status 코드 + header + body 를 한번에 담아서 주는게 가능하다.

  ```java
  @DeleteMapping("/users/{id}")
  	public ResponseEntity<?> deleteUser(@PathVariable Long id){
  		//User user = repository.findById(id).orElseThrow(()->new ResourceNotfoundException("user", "Id", id));
  		Optional<User> optional = repository.findById(id);
  		//요청한 id와 매핑하는 User가 없으면
  		if(optional.isEmpty()) {
  			return new ResponseEntity<>("요청한 User가 없습니다.",HttpStatus.NOT_FOUND);
  		}
  		//DB에서 삭제
  		repository.deleteById(id);
  		return new ResponseEntity<>(id+"User 삭제 Ok",HttpStatus.OK);
  		//repository.delete(user);	 
  		//return ResponseEntity.ok(user);
  		//return ResponseEntity.ok().build();
  
  	}
  	
  ```

  

- myBatis 연동하기 mapper.xml 설정
  - https://github.com/vega2k/SpringBoot_Thymeleaf_UserApp/blob/master/src/main/java/mapper/UserMapper.xml
- config 클래스 만들고 설정 아래꺼 참고.
  - https://github.com/vega2k/SpringBoot_Thymeleaf_UserApp/blob/master/src/main/java/me/vega2k/user/restthymeleaf/config/WebConfig.java
- jpa 설정도 백명숙 강사님 깃허브 참고



### xml로 하기

```
@RequestMapping(value="/usersXml",produces = {"application/xml"})
	public List<User> getUsersXML(){
		return repository.findAll();
	}
```

- 이대로 postman 에서 조회하면 에러난다 JOSN 은 알아서 해석해 주지만 XML 은 해석해주는 것이 없다.

  - 의존성 추가

  - ```
    <dependency>
     
    <groupId>com.fasterxml.jackson.dataformat</groupId>
     
    <artifactId>jackson-dataformat-xml</artifactId>
     
    <version>2.9.6</version>
    </dependency>
    
    ```

  - 추가 후 제대로 나옴 다만 출력결과를 맞춰줄려면?

- 1. ppt 80 페이지 참고 각 속성에 @JacksonXmlProperty 추가

- 2. 새로운 entity 객체 생성

  ```
  Controller
  
  @RequestMapping(value="/usersXml",produces = {"application/xml"})
  	public Users getUsersXML(){
  		
  		Users userXml = new Users();
  		userXml.setUsers(repository.findAll());
  		return userXml;
  	}
  ```

  ```
  Users
  
  package com.keunwoo.myspringboot.entity;
  
  import java.io.Serializable;
  import java.util.ArrayList;
  import java.util.List;
  
  import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlElementWrapper;
  import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlProperty;
  import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlRootElement;
  
  @JacksonXmlRootElement
  public class Users implements Serializable {
  	private static final long serialVersionUID = 22L;
  	
  	@JacksonXmlProperty(localName = "User")
  	@JacksonXmlElementWrapper(useWrapping = false)
  	private List<User> users = new ArrayList<>();
  
  	public void setUsers(List<User> users) {
  		this.users = users;
  	}
  
  	public List<User> getUsers() {
  		return users;
  	}
  }
  
  ```

  - @JacksonXmlElementWrapper(useWrapping = false) 를 지우면 기본 true 가 되고

    한번 더 감싸게 된다. 감싸지 말라고 설정한다.

    

  

  

  ###  Spring Boot Web MVC : 정적 리소스 맵핑 커스터 마이징

- src/main/resources 밑에 templates 에 xx.html 을 저장한다. (thymeleaf)

- but!! custom 폴더를 만들고 파일을 추가할려면 새로운 폴더에 대한 정보를 저장해줘야한다.

- config > WebConfig 파일에 설정을 추가한다. ppt 83

  - 1. Navigator 로 경로 확인
    2. mobile 폴더 추가 
    3. html 추가
    4. config 패키지에 configuration 생성
    5. 오버라이드 addresourcehandler 

```
package com.keunwoo.myspringboot.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfiguration implements WebMvcConfigurer{

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		//해당 url 패턴을 매칭
		registry.addResourceHandler("/mobile/**")
		//반드시 디렉토리이름 다음에 /을 주어야 한다.
		.addResourceLocations("classpath:/mobile/")
		.setCachePeriod(20); //20초
	}
	
}

```

- .addResourceLocations("classpath:/mobile/") 리소스 위치
- registry.addResourceHandler("/mobile/**") 매칭된 url 등록



- 파비콘은 static 밑에 favicon.ico 로 생성한다.

  

## ThymeLeaf

JSP 는 잘 안쓰이게됨. War 로 말아야하기 때문에 servlet 엔진 쓰고잇고 뭐 등등

- 서버사이드

- 의존성 주입하기

  ```
  <dependency>
   
  <groupId>org.springframework.boot</groupId>
   
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
  
  ```

- 타임리프 템플릿 위치는 /src/main/resources/templates/
- 참고사이트
  - https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax
  - https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html
- Local 실행은 그대로 나오지만 서버사이드로 실행시키면 제대로 나온다.



- @controller 생성

  ```
  @Controller
  public class UserController {
  
  	@GetMapping("/leaf")
  	public String hello(Model model) {
  		
  		model.addAttribute("myName","keunwoo");
  		
  		return "leaf_first";
  	}
  	
  }
  ```

  

- html 생성

  ```
  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <meta charset="EUC-KR">
  <title>Insert title here</title>
  </head>
  <body>
  	<h2 th:text="${myName}">Hello ThymeLeaf</h2>
  	<h1 th:text="${myName}">Name</h1>
  	<h1>
  		Hello, <span th:text="${myName}"></span>
  	</h1>
  	<h1>Hello, [[${myName}]]</h1>
  
  </body>
  </html>
  ```

- 제대로 되면 myName 값으로 치환되야함.



- UTF-8 로 해야 한글 안깨짐  windows -> preference 에서 설정



### + 디비와 연동하기

- repository 가져오기

  ```
  컨트롤러
  
  	@Autowired
  	UserRepository userRepository;
  	
  	@GetMapping("/index")
  	public ModelAndView getUsers() {
  		List<User> userList = userRepository.findAll();
  		return new ModelAndView("index", "users", userList);
  	}
  ```

  

- 템플릿 밑에 index.html 이 존재해야 한다.



