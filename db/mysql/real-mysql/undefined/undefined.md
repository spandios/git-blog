# 설정 파일

&#x20;

my.cnf라는 이름의 파일로 서버 설정파일을 참조한다.&#x20;

Mysql 서버는 지정된 여러 개의 디렉터리를 순차적으로 탐색하면서 처음 발견된 my.cnf파일을 사용하게 된다.

my.cnf파일의 위치는 아래 명령어를 통해 알아낼 수 있다.

```bash
mysqld --verbose --help | grep -A 1 'Default options'

# 이 순서대로 my.cnf을 탐색한다.
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
```



### 설정 파일 구성

하나의 파일에 여러 개의 설정 그룹을 담을 수 있다. \[mysqld], \[client] 처럼 그룹명을 명시하고 해당 그룹에만 적용하는 설정을 입력한다. Mysql 서버만을 위한 설정 파일이라면 \[mysqld] 그룹에 설정하면 된다.&#x20;

**그룹**

* \[mysqld]: mysql 서버에 대응
* \[client]: client에 공통적으로 대응. mysql client, mysqldump는 모두 클라이언트에 속한다.
* \[mysql]: mysql 클라이언트에 대응
* \[mysqldump]: mysqldump client에 대응

{% hint style="info" %}
mysqlddump

논리적 백업 프로그램으로 스토리지 엔진에 상관없이 백업할 수 있는 도구이다.
{% endhint %}

* \[mysqld\_safe]: mysqld safe에 대응&#x20;

{% hint style="info" %}
mysqld\_safe

mysqld\_safe는 일종의 mysqld를 감시하는 데몬이라고 볼 수 있는데, mysqld 프로세스를 모니터링하고, 어떤 문제로 mysqld가 강제 종료하면 mysqld를 다시 시작하게 합니다.
{% endhint %}



<figure><img src="../../../../.gitbook/assets/스크린샷 2023-02-01 오후 10.47.49.png" alt=""><figcaption><p>docker환경에의 기본 my.cnf</p></figcaption></figure>

****



###







