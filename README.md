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


### 02. 기본 Model, Controller, View 만들기

Create a Post Model

- Model 은 /app/Model 에 위치합니다.
- 밑의 내용처럼 /app/Model/Post.php 파일을 만들어줍니다.

```
<?php

class Post extends AppModel {
}
```


Create a Posts Controller

- /app/Controller directory 에 PostsController.php 파일을 생성합니다.

```
<?php

class PostsController extends AppController {
    public $helpers = array('Html', 'Form');

    public function index() {
        $this->set('posts', $this->Post->find('all'));
    }
}
```


Creating Post Views

- /app/View/Posts directory 에 index.ctp 파일을 생성합니다.


```
<h1>Blog posts</h1>
<table>
    <tr>
        <th>Id</th>
        <th>Title</th>
        <th>Created</th>
    </tr>

    <!-- Here is where we loop through our $posts array, printing out post info -->

    <?php foreach ($posts as $post): ?>
    <tr>
        <td><?php echo $post['Post']['id']; ?></td>
        <td>
            <?php echo $this->Html->link($post['Post']['title'],
array('controller' => 'posts', 'action' => 'view', $post['Post']['id'])); ?>
        </td>
        <td><?php echo $post['Post']['created']; ?></td>
    </tr>
    <?php endforeach; ?>
    <?php unset($post); ?>
</table>
```

브라우저에서 http://local.cakephp2.com/posts 페이지에 접근하여 페이지가 잘 뜨는지 확인합니다.

view 에서 $this->Html 부분은 CakePHP 의 HtmlHelper class 의 인스턴스를 사용하는 부분임. [Helpers](http://book.cakephp.org/2.0/en/views/helpers.html)


상세보기 페이지 만들기
- PostsController 에 다음의 view 메소드를 추가함.

```
public function view($id = null) {
    if (!$id) {
        throw new NotFoundException(__('Invalid post'));
    }

    $post = $this->Post->findById($id);
    if (!$post) {
        throw new NotFoundException(__('Invalid post'));
    }
    $this->set('post', $post);
}
```


- /app/View/Posts/view.ctp 파일 추가

```
<h1><?php echo h($post['Post']['title']); ?></h1>

<p><small>Created: <?php echo $post['Post']['created']; ?></small></p>

<p><?php echo h($post['Post']['body']); ?></p>
```


브라우저의 http://local.cakephp2.com/posts 페이지에서 게시글의 링크를 클릭하여 view 페이지로 이동하는 것을 확인해봅니다.


### 03. Post 추가, 수정, 삭제 기능 만들기

- PostsController 에 글 추가 action인 ```add``` 추가함.

```
    public function add() {
        if ($this->request->is('post')) {
            $this->Post->create();
            if ($this->Post->save($this->request->data)) {
                $this->Session->setFlash(__('Your post has been saved.'));
                return $this->redirect(array('action' => 'index'));
            }
            $this->Session->setFlash(__('Unable to add your post.'));
        }
    }
```

* ```$this->request->is('post')``` 부분은 post 요청인지 확인 하는 부분임
* $this->request->data


- view 파일인 /app/View/Posts/add.ctp 파일 추가하기

```
<h1>Add Post</h1>
<?php
echo $this->Form->create('Post');
echo $this->Form->input('title');
echo $this->Form->input('body', array('rows' => '3'));
echo $this->Form->end('Save Post');
?>
```


- ```$this->Form->create()``` 부분은 다음의 form 태그를 생성함.

```
<form id="PostAddForm" method="post" action="/posts/add">
```

- ```$this->Form->input()``` 부분은 input 태그를 생성함.

- ```$this->Form->input('body', array('rows' => '3'));``` 부분은 textarea 를 생성함.


add 페이지로 이동하기 위한 버튼을 index.ctp 파일세 추가해주기

```
<?php echo $this->Html->link(
    'Add Post',
    array('controller' => 'posts', 'action' => 'add')
); ?>
```


Validation rules 추가
- Post model 을 다음과 같이 작성

```
class Post extends AppModel {
    public $validate = array(
            'title' => array(
            'rule' => 'notEmpty'
        ),
        'body' => array(
            'rule' => 'notEmpty'
        )
    );
}
```

- http://local.cakephp2.com/posts/add 페이지로 이동하여 validation 체크부분을 확인 함.


수정기능 만들기

- Controller 에 edit() action 추가

```
public function edit($id = null) {
    if (!$id) {
        throw new NotFoundException(__('Invalid post'));
    }

    $post = $this->Post->findById($id);
    if (!$post) {
        throw new NotFoundException(__('Invalid post'));
    }

    if ($this->request->is(array('post', 'put'))) {
        $this->Post->id = $id;
        if ($this->Post->save($this->request->data)) {
            $this->Session->setFlash(__('Your post has been updated.'));
            return $this->redirect(array('action' => 'index'));
        }
        $this->Session->setFlash(__('Unable to update your post.'));
    }

    if (!$this->request->data) {
        $this->request->data = $post;
    }
}
```


- edit view 추가 (edit.ctp)

```
<h1>Edit Post</h1>
<?php
echo $this->Form->create('Post');
echo $this->Form->input('title');
echo $this->Form->input('body', array('rows' => '3'));
echo $this->Form->input('id', array('type' => 'hidden'));
echo $this->Form->end('Save Post');
?>
```


- 리스트 페이지에 edit 페이지 링크 추가

```
<h1>Blog posts</h1>
<p><?php echo $this->Html->link("Add Post", array('action' => 'add')); ?></p>
<table>
    <tr>
        <th>Id</th>
        <th>Title</th>
        <th>Action</th>
        <th>Created</th>
    </tr>

<!-- Here's where we loop through our $posts array, printing out post info -->

<?php foreach ($posts as $post): ?>
    <tr>
        <td><?php echo $post['Post']['id']; ?></td>
        <td>
            <?php
                echo $this->Html->link(
                    $post['Post']['title'],
                    array('action' => 'view', $post['Post']['id'])
                );
            ?>
        </td>
        <td>
            <?php
                echo $this->Html->link(
                    'Edit',
                    array('action' => 'edit', $post['Post']['id'])
                );
            ?>
        </td>
        <td>
            <?php echo $post['Post']['created']; ?>
        </td>
    </tr>
<?php endforeach; ?>

</table>
```



Deleting Posts

- PostsController 에 delete() action 추가

```
public function delete($id) {
    if ($this->request->is('get')) {
        throw new MethodNotAllowedException();
    }

    if ($this->Post->delete($id)) {
        $this->Session->setFlash(
            __('The post with id: %s has been deleted.', h($id))
        );
        return $this->redirect(array('action' => 'index'));
    }
}

```


- index.ctp 파일을 다음과 같이 수정

```
<!-- File: /app/View/Posts/index.ctp -->

<h1>Blog posts</h1>
<p><?php echo $this->Html->link('Add Post', array('action' => 'add')); ?></p>
<table>
    <tr>
        <th>Id</th>
        <th>Title</th>
        <th>Actions</th>
        <th>Created</th>
    </tr>

<!-- Here's where we loop through our $posts array, printing out post info -->

    <?php foreach ($posts as $post): ?>
    <tr>
        <td><?php echo $post['Post']['id']; ?></td>
        <td>
            <?php
                echo $this->Html->link(
                    $post['Post']['title'],
                    array('action' => 'view', $post['Post']['id'])
                );
            ?>
        </td>
        <td>
            <?php
                echo $this->Form->postLink(
                    'Delete',
                    array('action' => 'delete', $post['Post']['id']),
                    array('confirm' => 'Are you sure?')
                );
            ?>
            <?php
                echo $this->Html->link(
                    'Edit', array('action' => 'edit', $post['Post']['id'])
                );
            ?>
        </td>
        <td>
            <?php echo $post['Post']['created']; ?>
        </td>
    </tr>
    <?php endforeach; ?>

</table>
```