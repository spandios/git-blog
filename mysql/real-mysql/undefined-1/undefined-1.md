# 역할

* 8버전 부터 권한을 묶어 역할을 사용할 수 있게 됐다.&#x20;
* 보통 계정에 부여하여 사용한다.

### 생성&#x20;

```sql
CREATE ROLE role_emp_read, role_emp_write;
```

### 권한 부여하기&#x20;

```sql
GRANT SELECT ON employees.* TO role_emp_read;
GRANT INSERT, ON employees.* TO role_empt_write;
```



### 계정에 역할 부여하기&#x20;

```sql
GRANT role_emp_read, role_emp_write TO '계정'
```



### 역할 활성화 하기&#x20;

부여만 해서는 활성화가 되지 않으므로 역할이 정상적으로 동작하지 않는다.&#x20;

```sql
SET ROLE 'role_emp_write';
```

{% hint style="info" %}
계정이 로그아웃하면 역할 활성화 상태가 초기화 됨

SET GLOBAL activate\_all\_roles\_on\_login=ON;

명령어를 통해 로그인과 동시에 부여된 역할이 자동으로 활성화 하게 끔 하자.
{% endhint %}

