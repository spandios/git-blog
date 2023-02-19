# 사용자 및 권한

## host도 계정의 일부

Mysql은 다른 DMBS와 다르게 사용자의 계정뿐 아니라 사용자의 접속 주소도 계정의 일부가 된다.&#x20;

`'me'@'127.0.1'`이 계정은 로컬 호스트에서 me라는 아이디로 접속할 때만 사용 가능한 계정이다.&#x20;

만약 모든 곳에서 접속을 허용하고 싶다면 호스트 부분에 %를 입력하면 된다.



## 계정 생성

mysql 8 이후부터는 계정 생성은 CREATE USER, 권한 부여는 GRANT 명령로 구분해서 실행해야 한다.&#x20;

아래는 일반적으로 많이 사용되는 CREATE USER 명령이다.&#x20;

```sql
CREATE USER 'user'@'%'
    IDENTIFIED WITH 'mysql_native_password' BY 'password'
    REQUIRE NONE
    PASSWORD EXPIRE INTERVAL 30 DAY
    ACCOUNT UNLOCK
    PASSWORD HISTORY DEFAULT
    
```

각 옵션을 살펴보자.&#x20;

1. IDENTIFIED WITH

사용자 인증 방식과 비밀번호를 설정한다.&#x20;

WITH 다음에는 인증 방식을 명시해야 하며 BY 뒤에는 비밀번호 입력한다.

인증 방식은 다음 2가지가 대표적이다.

* Native Pluggable Authentication: Mysql5.7 까지 기본으로 사용되던 방식으로 SHA-1 알고리즘으로 비밀번호에 대한 해시값으로 비교해 인증하는 방식
* Caching SHA-2 Pluggable Authentication: 8버전 대에서 기본 인증 방식으로 SHA-2 알고리즘을 사용한다. SSL/TS 또는 RSA키페어를 반드시 사용해야 한다.



2. REQUIRED

서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부이다.



3. PASSWORD EXIRED&#x20;

비밀번호 유효 기간을 설정하는 옵션으로 별도로 명시하지 않으면 default\_password\_lifetime 시스템 변수에 저장된 값을 사용한다.



4. PASSWORD HISTORY

한 번 사용했던 비밀번호를 재사용하지 못하게 설정



5. ACCOUNT LOCK / UNLOCK

ALTER USER 명령을 사용해 계정을 사용하지 못하게 잠글지 여부를 결정한다.&#x20;



## 비밀번호 관리

### 고수준 비밀번호

validate\_password 컴퍼넌트를 통해 비밀번호의 보안 수준을 높이도록 할 수 있다.

```sql
# 설치 
INSTALL COMPONENT 'file://component_validate_password'

# 컴퍼넌트 기본 파라미터 확인 , 기본 설치하면 MEDIUM 수준이 기본
SHOW GLOBAL VARIABLES LIKE 'validate_password%';
```

### 이중 비밀번호

Primary와 Secondary 두 가지 비밀번호로 구성해 접속할 수 있다.

```sql
ALTER UER 'root'@'localhost' 
IDENTIFIED BY 'new_password' 
RETAIN CURRENT PASSWORD;
```

위의 명령어를 실행하면 기존 비밀번호가 세컨더리가 되고 'new\_password'가 Primary 비밀번호가 된다.&#x20;

비밀번호를 주기적으로 변경해야하지만 만약 비밀번호를 바꾸면 기존에 운영되고 있던 어플리케이션에 영향이 가기 때문에 생긴 기능이다.&#x20;









####





