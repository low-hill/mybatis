# mybatis

##  Dynamic SQL
### choose(when, otherwise)
### foreach

```
<where>
  <choose>
    <when test="ids == null">
      <if test="name != null">
        displayNm LIKE CONCAT('%', #{name}, '%')
      </if>
    </when>
    <otherwise>
      AND id IN ( <foreach collection="ids" item="id" separator=",">#{id}</foreach> )
    </otherwise>
  </choose>
</where>
```
### selectKey
* keyProperty: selectKey구문의 결과가 셋팅될 property(resultType[class]의 멤버변수와 일치)
* resultType: 결과의 타입
* order: BEFORE / AFTER
  * BEFORE: selectKey구문의 쿼리를 먼저 실행
  * AFTER: selectKey구문의 쿼리를 마지막에 실행

```
<insert id="addUser" parameterType="User">
  <selectKey keyProperty="userId,userName" resultType="User" order="AFTER">
    SELECT USER_ID AS userId
         , USER_NAME AS userName
    FROM USER
    ORDER BY USER_ID DESC
    LIMIT 1
  </selectKey>
  INSERT INTO USER(col1, ...)
  VALUES (#{...}, ...)
</insert>
```
##### Reference
* [https://mybatis.org/mybatis-3/]
* [https://deeplify.dev/back-end/spring/select-key]


## Mapper XML
### Result Maps
* constructor - 인스턴스화되는 클래스의 생성자에 결과를 삽입하기 위해 사용됨
  * idArg - ID 인자. 전반적으로 성능을 향상
  * arg - 생성자에 삽입되는 일반적인 결과
* id – ID 결과. 전반적으로 성능을 향상
```
<id property="id" column="컬럼명"/>
```
* result – 필드나 자바빈 프로퍼티에 삽입되는 일반적인 결과
```
<result property="필드명" column="컬럼명"/>
```
* association – 복잡한 타입의 연관관계. 많은 결과는 타입으로 나타난다.
* 중첩된 결과 매핑 – resultMap 스스로의 연관관계
* collection – 복잡한 타입의 컬렉션
* 중첩된 결과 매핑 – resultMap 스스로의 연관관계
* discriminator – 사용할 resultMap 을 판단하기 위한 결과값을 사용
  * case – 몇가지 값에 기초한 결과 매핑
  
### association
association 엘리먼트는 “has-one”타입의 관계를 다룬다.

마이바티스는 관계를 정의하는 두가지 방법을 제공한다.
* 내포된(Nested) Select: 복잡한 타입을 리턴하는 다른 매핑된 SQL 구문을 실행하는 방법.
* 내포된(Nested) Results: 조인된 결과물을 반복적으로 사용하여 내포된 결과 매핑을 사용하는 방법.

#### 연관(Association)을 위한 중첩된 Select
```
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```
위 방법은 “N+1 Selects 문제” 으로 알려진 문제점을 가진다. N+1 조회 문제는 처리과정의 특이성으로 인해 야기된다.

* 레코드의 목록을 가져오기 위해 하나의 SQL 구문을 실행한다. (“+1” 에 해당).
* 리턴된 레코드별로 각각의 상세 데이터를 로드하기 위해 select 구문을 실행한다. (“N” 에 해당).
이 문제는 수백 또는 수천의 SQL 구문 실행이라는 결과를 야기할 수 있다.
목록을 로드하고 내포된 데이터에 접근하기 위해 즉시 반복적으로 처리한다면 지연로딩으로 호출하고 게다가 성능은 많이 나빠질 것이다.
그래서 다른 방법이 있다.

#### 관계를 위한 내포된 결과(Nested Results)

개별구문을 실행하는 것 대신에 테이블을 함께 조인
```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
  </association>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    B.author_id     as blog_author_id,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio
  from Blog B left outer join Author A on B.author_id = A.id
  where B.id = #{id}
</select>
```
* id 엘리먼트는 내포된 결과 매핑에서 매우 중요한 역할을 담당한다. 결과 중 유일한 것을 찾아내기 위한 한개 이상의 프로퍼티를 명시해야만 한다. 가능하면 결과 중 유일한 것을 찾아낼 수 있는 프로퍼티들을 선택하라. 기본키가 가장 좋은 선택이 될 수 있다.
#### resultMap 재사용하기
```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author"
    resultMap="authorResult" />
  <association property="coAuthor"
    resultMap="authorResult"
    columnPrefix="co_" />       <!--결과매핑을 재사용하기 위해 columnPrefix를 명시-->
</resultMap>

<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```
### collection
“has many”타입의 관계를 다룬다.
```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <result property="body" column="post_body"/>
  </collection>
</resultMap>
```

##### Reference
* [https://mybatis.org/mybatis-3/ko/sqlmap-xml.html]
