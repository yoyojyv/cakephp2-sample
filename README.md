# CakePHP 2 Sample

CakePHP 2 를 이용한 샘플 소스 입니다.


## 초기 환경 설정, 확인하기

https://github.com/cakephp/cakephp/tags 에서 원하는 버전을 다운로드 받습니다.

압축을 풀어 web document 경로로 복사합니다.

아파치 vhost, hosts 파일을 수정합니다.
* 본 샘플에서는 host를 ```local.cakephp2.com``` 으로 설정하였습니다.


/app/tmp 경로는 퍼미션을 수정해 줍니다.
sudo chmod -R 777 app/tmp


다음으로 브라우저에서 http://local.cakephp2.com/ 페이지를 열어, 이상이 있는경우 설정 부분들을 수정해줍니다.


## 블로그 만들기 - 브랜치별로 예제 돌려보기

### 01. 초기 환경 설정, 확인하기

본 문서는 http://book.cakephp.org/2.0/en/getting-started.html 를 참조하여 만들어졌습니다.

테스트용 DB 를 만들고 다음의 테이블을 생성합니다.

```
- Creating the Blog Database

/* First, create our posts table: */
CREATE TABLE posts (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(50),
    body TEXT,
    created DATETIME DEFAULT NULL,
    modified DATETIME DEFAULT NULL
);

/* Then insert some posts for testing: */
INSERT INTO posts (title,body,created)
    VALUES ('The title', 'This is the post body.', NOW());
INSERT INTO posts (title,body,created)
    VALUES ('A title once again', 'And the post body follows.', NOW());
INSERT INTO posts (title,body,created)
    VALUES ('Title strikes back', 'This is really exciting! Not.', NOW());
```

- CakePHP Database Configuration

/app/Config/database.php.default 파일을 복사해서 /app/Config/database.php 파일을 만들고 정보를 수정합니다.

```
	public $default = array(
		'datasource' => 'Database/Mysql',
		'persistent' => false,
		'host' => 'localhost',
		'login' => 'root',
		'password' => '1234',
		'database' => 'cakephp_test_db',
		'prefix' => '',
		'encoding' => 'utf8',
	);
```

- Optional Configuration

    - 보안쪽 설정부분 변경하기

/app/Config/core.php ```Security.salt``` 부분을 수정합니다.


- URL Rewriting 부분 수정을 원한다면, 다음의 문서를 참조하여 수정합니다. http://book.cakephp.org/2.0/en/installation/url-rewriting.html












