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

 ```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<table>
		<tr>
			<th>Name</th>
			<th>Email</th>
		</tr>

		<tr th:each="user : ${users}">
			<td th:text="${user.name}"></td>
			<td th:text="${user.email}"></td>
		</tr>

	</table>
</body>
</html>
 ```

- validation 과정

  - 입력 값 Check 시 client 단에서 주로 server 단에서 Check 한다
  -  @Valid @NotBlank 같은 어노테이션 사용
  - @Model Attribute form 데이터를 객체에 자동으로 넣어주는 @임.
  - @Valid 는 기존의 ModelAttribute + validation

- Controller 에서 에러나면 @notBlank 에서 설정한 메시지가 자동으로 아규먼트에 저장

- 입력이 없으면 계속 페이지에 머뭄. 입력 시 정상 동작

- ppt 91

  ```
  controller
  
  @GetMapping("/signup")
  	public String addUserForm(User user) {
  		return "add-user";
  	}
  	-----------------------------
  	add-user.html
  	
  	<form action="#" th:action="@{/adduser}" th:object="${user}"
  		method="post">
  		<label for="name">Name</label>
  		<input type="text" th:field="*{name}" id="name"> 
  			<span th:if="${#fields.hasErrors('name')}"
  			th:errors="*{name}"></span>
  			 <br />
  		<label for="email">Email</label>
  		<input type="text" th:field="*{email}" id="email"> 
  		<span th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></span> <br />
  		<input type="submit" value="Add User">
  	</form>
  ```
  - add-user.hmtl 에서 user 객체를 사용하니까

  - controller 에서 User타입의 인자로 받아야한다.

    ```
    <span th:if="${#fields.hasErrors('name')}"
    			th:errors="*{name}"></span>
    			
    			th:if 네임 필드에 에러가 있나 확인
                th:errors 네임 필드에 에러메시지를 뿌려준다. 
    ```

    

- @NotBlank(message = "Name is Mandatory") 과 세트로
  - controller에서 @VALID 사용한다 





배포는 클라이언트 + 서버 합쳐서 배포한다.

### CORS 

- @CrossOrigin 어노테이션 사용

- @CrossOrigin(origins="http://localhost:18080")

  ```
  @CrossOrigin(origins = "http://localhost:3000")
  @RestController
  public class RESTUserController {
  	
  	private final UserRepository repository;
  	//autowired 안하고 이렇게 해도됨.
  	public RESTUserController(UserRepository repository) {
  		super();
  		this.repository = repository;
  	}
  ```

  

이런식으로 사용

가로질러서 사용이 가능.

react 에서 같이 사용할려면?

- 일단 리액트 프로젝트 생성.
- package.josn 파일 생성된거 확인 -> 명령어 확인
- npm run build 서버랑 붙일 거 생성중
- build 폴더 생성된거 확인
- build 밑에 파일을 전부 복사해서 resource 의 static 에 붙여넣기
- 하고 서버 스타트 하고 properties 에서 설정한 포트로 접근 및 확인
- 그리고 자르로 하나로 만들어서  ㄱㄱ한다 



- 단점은 수정사항이 생기면 다시 빌드하고 과정이 번거롭게 된다.



## Actuator

- 모니터링 기능 제공



- 의존성 설치

  ```
  <dependency>
   
  <groupId>org.springframework.boot</groupId>
   
  <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  
  ```

  

- http://localhost:8086/actuator 로 접근
- properties 에 옵션 추가 가능
- management.endpoints.web.exposure.include=* 등..
- 시각화 처리
  - https://github.com/codecentric/spring-boot-admin



- 새로운 프로젝트 생성 web 체크 생성

  - ```
    <dependency>
     
    <groupId>de.codecentric</groupId>
     
    <artifactId>spring-boot-admin-starter-server</artifactId>
     
    <version>2.1.4</version>
    </dependency>
    
    
    @SpringBootApplication
    @EnableAdminServer
    public class SpringbootAdminApplication {
    } 
    
    ```

  - 원래 프로젝트에는

  - ```
    <dependency>
     
    <groupId>de.codecentric</groupId>
     
    <artifactId>spring-boot-admin-client</artifactId>
     
    <version>2.1.4</version>
    </dependency>
    
    properties 에 추가
    spring.boot.admin.client.url=http://localhost:8090
    
    ```

  - http://localhost:8090/ 으로 접속

  - 접속후 밑의 8086 으로 끝나는 유알엘 접속 하면 밑에 키가 생김

  - 그 키를 누르면 세부 화면으로 들어감

- devtools 는 개발모드에서만 사용해야 한다. -> spring.devtools.restart.enabled 를 꺼야한다

- porperties 에서 설정 



- critical 한 정보는 뺴주고 싶어 beans 같은거 properties 에서 설정

  - ```
    management.endpoints.web.exposure.include=*
    management.endpoints.web.exposure.exclude=env,beans
    ```





## Security



- security 설정 후 로그인 --> f12 에서 애플리케이션 cookie 확인
- 매번 패스워드로 다시 로그인 할 수는 없으니까 미리 설정된 파일을 만들자.



- 데이터베이스에서 패스워드를 쳐보면 그대로 다나온다. 패스워드를 인코딩해서 넣자.
- 타임리프쓰는 페이지는 tamplate 밑



- ppt 그대로 참고하는게 빠를듯.
- .antMatchers("/mypage/**").authenticated() 이페이지만 인증을 걸고
  .antMatchers("/**").permitAll() 나머지는 전부 허용



- .and() 를 쓰면 다시 http 시큐리티를 반환해준다.
- configure(HttpSecurity http) 



- DI 객체생성하자마자 바로 바로호출하려면 -> @postcontructor 사용







## 추가 학습

- Serializable 에 대해서
- JPA relation 예제확인
- lombok 에 대해서
  - 의존성 추가
  - 해당 자르를 sts가 알고잇어야 정상작동한다
  - 위치로 가서 jar를 등록해서 sts위치를 알려주자. 
  - 디펜던시에서 lombok 위치 확인 탐색기로 그위치까지 간다.
  - cmd 에서 경로복사해서 이동 
  - java -jar lombok-1.18.12.jar
  - 고추화면이 뜬다. 여기서 sts 경로 설정해주기 
  - help 에섯 about spring ~~ 에서 맨밑에 lombok 된 메시지 확인





- 