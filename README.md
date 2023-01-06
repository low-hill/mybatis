# mybatis

##  Dynamic SQL
### if
### choose(when, otherwise)
### foreach

```
<where>
  <choose>
    <when test="ids == null">
      ...
    </when>
    <otherwise>
      id IN ( <foreach collection="ids" item="id" separator=",">#{id}</foreach> )
    </otherwise>
  </choose>
</where>
```
