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
