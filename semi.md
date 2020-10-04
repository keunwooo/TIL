- 도메인을 기준으로 Micro Service 로 쪼갠다.
- jpa 사용시 rest까지 생성가능하기 떄문에 서비스랑 컨트롤러 안만들어도 되는 장점이 있음.
- 자동으로 생성됨.
- 프로젝트 생성시 롬복체크하고 가면 가끔 세터게터가 안만들어지는 경우가 생김
- value object 가 아니라 엔티티로 표현-> 식별자가 존재하면 엔티티 없으면 그냥 밸루오브젝트
  둘이 같이 묶여야 한다면 aggregate 로 묵는다.



- 기능별로 정의한다
- gson 으로 java - json 변환



- //입력값 - Filter,InterceptorFilter



- DTO 는 데이터를 담아서 통신할때 Serializable 을 상속받아 구현하는 객체
- Store(DAO) 에서 결과 데이터를 Service 에 리턴할 때 같은 VM 이 아니고 분산된 VM에 전달되어야 하면 현재 적용하고 있는 Book 이나 Todo를 DTO로 작성
- controller에서 service 호출할 때 전달되거나 리턴받는 데이터가 분산되어 있을 경우 Serializable 상속받는 DTO 작성



- Domain Object
- DB랑 OR 해서 데이터를 담아쓰는 객체. 
- Entity Object : 최소 정보 단위를 담는 데이터 DB랑 연결되기 때문에 Identify 가 존재
- Value Object : 식별자가 없음. 
- Aggregate : 실제 서비스들의 기본 Entity Object가 여러개 있으면 하나의 프로세스를 수행할때(주문) 모든 과정이 하나로 묶이는 트랜잭션에서 하나로 묶이는 데이터.
- 분산해서 데이터를 담는 것을 목적으로 하는 거에 DTO

- 



- @PersistenceContext
- 스프링프레임워크 jpa 를 지원하는 라이브러리에서 디비에 사용하는 디비에 연결하고 해당되는
- statment 객체 얻고 쿼리수행하고 결과잇으면 결과 리턴해주는 것들을 내부에서 구현을 안햇음에도 불구하고 사용이 가능하다.

- 하이버네티스는 엔티티매니저를 자동으로 해준다



- ojdbc6 추가
- Build Path 에서 configure 들어가서 path 설정-> edit -> ojdbc6 경로 설정



- pom.xml 에 추가

- ```
  <repositories>
  		<repository>
  			<id>oracle</id>
  			<url>http://maven.icm.edu.pl/artifactory/repo/</url>
  		</repository>
  	</repositories>
  ```

  

```
<dependency>
			<groupId>com.oracle</groupId>
			<artifactId>ojdbc6</artifactId>
			<version>11.2.0</version>
		</dependency>
```

m.2 /reository/com/oracle/ojdbc6/11.2.0 에 기존의 ojdcb6 복사해서 넣고 이름 

ojdbc6-11.2.0.jar 로 바꾸기

 

API 작성

- api
  - bookapi.js
  - model
    - bookapimodel.js



- https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise
- https://developer.mozilla.org/ko/docs/Learn/JavaScript/Asynchronous/async_await



model.js 생성 -> sts 의 변수랑 같게 설정
BookAPI.js 생성 -> axios 로 받아오는거 설정
Store 에서 BookApi 객체 생성해서 리턴값? 저장 , @action 정의해서 원하는 함수 (BookList)만듬
데이터 조작하는 Container.js 부분에서 componentDidMount() 로
라이프사이클? 에 맞춰서 실행되게 하고.
    const { bookStore } = this.props; 프롭스로 받아옴

bookStore.bookList(); store에 api로 받아온 List 저장해주는 역할



- bookApi.js

```
import axios from "axios";
class BookApi {
  // URL = "http://localhost:9000/api/books";
  URL = "/api/books/";

  bookList() {
    //return 추가 return 해야 response값 넘겨줄수있음
    return axios
      .get(this.URL)
      .then((response) => {
        return response.data;
      })
      .catch((error) => console.log(error));
  }

  //   bookDetail(ISBN) : URL: URL/ISBN method: Get return: Book

  //   bookCreate(BookApiModel) : URL POST

  //   bookModify(BookApiModel) : URL PUT

  //   bookDelete(ISBN) : URL/ISBN DELETE

  //   search(searchType,keyword) : URL/searchType/ketword GET Book[]
}

export default BookApi;
```

- BookStore.js

  ```
  import { observable, computed, action } from "mobx";
  import BookApi from "../api/BookApi";
  
  class BookStore {
  
    constructor() {//api객체 생성
      this.bookApi = new BookApi();
      //현재 BookStore의 bookApi 변수에 담아주기
      //변수 생성,의존성 주입 같은 느낌
    }
  
    @observable books = []; //BookApi - bookList
    @observable book = null; //BookApi - bookDetail(ISBN)
  
    //   constructor(bookApi) {
    //     this.bookApi = bookApi;
    //   }
  
    @computed get _book() {
      return this.book ? { ...this.book } : {};
    }
  
    @computed get _books() {
      console.log(this.books);
      return this.books ? this.books.slice() : [];
    }
  
    @action select = (book) => {
      this.book = book;
    };
  
    @action
    async bookList() {
      //bookApi에서 넘겨준 reponse를 books에 저장
      //
      const data = await this.bookApi.bookList();
      console.log(data);
      this.books = data;
    }
  }
  
  export default new BookStore();
  
  ```

  

- BookListContainer.js

```
import React, { Component } from "react";
import BookListView from "../view/BookListView";
import { observer, inject } from "mobx-react";

@inject("bookStore")
@observer
class BookListContainer extends Component {
  componentDidMount() {
    const { bookStore } = this.props;

    // bookStore.bookStore(); //초기화해서 목록뿌릴 데이터를 가져옴
    // if (bookStore) {
    //   bookStore.bookList();
    // }

    //승훈 수정
    bookStore.bookList(); //store에 api로 받아온 정보 저장
  }

  onSelect = (book) => {
    const { bookStore } = this.props;
    bookStore.select(book);
  };

  render() {
    const { bookStore } = this.props;
    console.log(bookStore._books);
    return <BookListView books={bookStore._books} onSelect={this.onSelect} />;
  }
}

export default BookListContainer;

```

- package.json

  ```
  "proxy": "http://localhost:9000",
  ```

- BookApiModel.js

  ```
  class BookApiModel {
    ISBN = "";
    title = "";
    author = "";
    publisher = "";
    price = 0.0;
    imgUrl = "";
    introduce = "";
  
    constructor(ISBN, title, author, publisher, price, imgUrl, introduce) {
      this.ISBN = ISBN;
      this.title = title;
      this.author = author;
      this.publisher = publisher;
      this.price = price;
      this.imgUrl = imgUrl;
      this.introduce = introduce;
    }
  }
  
  export default BookApiModel;
  
  ```

  