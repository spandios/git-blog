# 권한

### 객체 권한, 글로벌 권한&#x20;

* 객체 권한 : 데이터베이스나 테이블을 제어하는 데 필요한 권한&#x20;
* 글로벌 권한 : 데이터베이스나 테이블 이외의 객체에 적용하는 권한&#x20;

### 정적권한, 동적권한

* 정적권한 : 소스코드에 고정적으로 명시돼 있는 권한
* 동적권한: 서버가 시작하면서 동적으로 생성하는 권한



{% hint style="info" %}
권한 레퍼런스

[https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html)
{% endhint %}

## 권한 설정&#x20;

```
GRANT (권한 리스트) ON db.table TO '계정'; 
```

### 글로벌 권한 설정

```sql
GRANT SUPER ON *.* TO 'user'@'localhost';
```

글로벌 권한은 특정 DB나 테이블에 부여할 수 없기 때문에 항상 \*.\*이다.&#x20;

### DB 권한 설정

```sql
GRANT EVENT ON *.* TO 'user'@'localhost'; # 전체 DB
GRANT EVENT ON employees.* TO 'user'@'localhost'; # 특정 DB
```

전체 DB와 특정 DB를 설정할 수 있지만 특정 테이블은 불가능하다.

### 테이블 권한 설정&#x20;

```sql
GRANT SELECT ON *.* TO 'user'@'localhost'; # 전체 DB
GRANT SELECT,INSERT ON employees.* TO 'user'@'localhost'; # 특정 DB
GRANT SELECT,INSERT,DELETE ON employees.department TO 'user'@'localhost'; # 특정 Table
```

Table까지 권한 부여 가능하다.







