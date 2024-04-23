# SQL注入


&lt;!--more--&gt;



### 什么是SQL注入

SQL注入攻击是通过将恶意的SQL语句插入到应用的输入参数中, 再在后台SQL服务器上解析执行的攻击. 是目前对数据库进行攻击的最常用手段之一. 主要原因是程序对用户输入数据的合法性没有判断和处理. 



### 原理

1. **恶意拼接查询**

   都知道SQL语句是用`;`进行分隔两个语句的:

   ```sql
   SELECT * FROM users WHERE user_id = $user_id;
   ```

   其中, `user_id`是传入参数, 但如果传入的参数变为`1;DELETE FROM users;`, 那么语句最终就会变为:

   ```sql 
   SELECT * FROM users WHERE user_id = 1;;DELETE FROM users;
   ```

   如果执行了上面的语句, 那么`user`表中所有数据都被删除了. 

2. **利用注释执行非法命令**

   SQL语句中可以添加注释:

   ```sql
   SELECT * FROM users WHERE user_gender=&#39;男&#39; AND user_age=$age
   ```

   如果`user_age`中包含了恶意的字符串`20 OR 25 AND SLEEP(500)--`, 那么语句最终会变为:

   ```sql
   SELECT * FROM users WHERE user_gender=&#39;男&#39; AND user_age=20 OR 25 AND SLEEP(500)--
   ```

   上面这条语句只是想耗尽系统资源, `SLEEP(500)`会一直执行, 但是如果其中添加了修改, 删除数据的语句, 将会造成更大的破坏. 

3. **传入非法参数**

   SQL语句中的字符串是用单引号包裹的, 但是如果其本身包含单引号而没有处理, 那么就可能篡改SQL语句的作用:

   ```sql
   SELECT * FROM users WHERE user_name=$user_name
   ```

   如果`user_name`传入参数值`M&#39;ustard`, 那么语句最终会变为:

   ```sql
   SELECT * FROM users WHERE user_name=&#39;M&#39;ustard&#39;
   ```

   一般情况下, 执行上面语句就会报错, 但是这种方式可能会产生恶意的SQL语句. 

4. **添加额外条件**

   在SQL语句中添加一些额外添加, 来改变执行行为. 条件一般为真值表达式:

   ```sql
   UPDATE users SET user_password=$user_password where user_id=$user_id
   ```

   如果`user_id`传入的是`1 OR TRUE`, 那么语句最终会变为:

   ```sql
   UPDATE users SET user_password=&#39;123456&#39; where user_id=1 OR TRUE
   ```

   如果执行了上面的语句, 那么所有用户的密码都被更改了. 



### 防御手段

1. **过滤输入内容, 校验字符串**

   过滤掉用户输入中的不合法字符剔除掉, 可以使用编程语言提供的处理函数或自己封装的函数来过滤, 也可以使用正则表达式来进行匹配. 

   也要验证参数的类型, 比如字符串或者整型. 

2. **参数化查询**

   参数化查询是目前被视作预防SQL注入攻击最有效的方法. 指的设计数据库连接并访问数据时, 在需要填入数据的地方, 使用参数(Parameter)来给值. 

   &gt; MySQL的参数格式是以`?`加上参数名称而成:
   &gt;
   &gt; ```sql
   &gt; UPDATE table_1 SET row_1=?row_1, row_2=row_2 WHERE row_3=?row_3
   &gt; ```

   在使用参数化查询下, 数据库不会将参数的内容视为SQL语句的一部分来处理, 而是在数据库完成SQL语句的编译之后, 才套用参数执行. 因此就算参数中含有破坏性的指令, 也不会被数据库所运行. 

3. **安全测试, 安全审计**

4. **避免使用动态SQL**

5. **不要将敏感数据保留在纯文本中**

6. **限制数据库权限和特权**

7. **避免直接向用户显示数据库错误**


---

> 作者:   
> URL: https://buli-home.cn/sqlinjection/  

