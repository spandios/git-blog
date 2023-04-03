---
description: 들어가기
---

# Real Mysql

## Mysql 소개

msyql은 1979년 스웨덴의 TcX라는 회사에서 시작된다고 한다. 이후 썬마이크로시스템즈에 인수되었다가 지금의 오라클에 인수되었다. 오라클은 리팩토링을 통해 5.5에서 5.7버전까지는 안정성과 성능 개선에 집중하다가 8.0부터는 본격적으로 상용 DBMS가 가지고 있는 기능을 추가하였다.

오라클은 라이선스 정책을 Mysql 커뮤니티 에디션과 엔터프라이즈 에디션으로 나눴는데, 중요한 건 오라클이 인수 후에도 오픈소스 정책을 유지해 커뮤니티 에디션으로 개인과 기업이 무료로 이용할 수 있다는 것이며 기본 기술과 성능은 엔터프라이즈 에디션과 차이가 없다고 한다. 엔터프라이즈 에디션에는 추가적으로 기술 지원과 부가기능이 있는데, 부가기능은 다른 기술로 보완이 된다고 하니 기술지원이 꼭 필요하지 않은 이상 선택하지는 않을 것 같다.

## 왜 Mysql?

개인적으로 나는 Mysql를 주로 사용하고 PostgresSQL 맛보기 정도 밖에 경험이 없어 익숙한 Mysql을 사용하는 게 가장 큰 이유다.

아래 표는 DB-Engines.com가 발표한 DMBS 랭킹이다. 기준은 주요 인력이나, 얼마나 많이 언급되고 커뮤니티의 활성화 같은 영향력이다. Mysql은 Oracle과 더불어 세계적으로 가장 많이 사용되며 그러기에 활발한 커뮤니티와 많은 레퍼런스를 가지고 있다.

Mysql@5 버전 대에서 기능과 성능이 PostgresSQL보다 비교적 좋지 않아 PostgresSQL를 사용하는 사례를 몇번 본적이 있다. 하지만 8버전대에서는 많은 성능개선과 기능 추가되었으니 문제 없다고 본다.

개인적으로 모두 RDMBS의 본질은 같기 때문에 DB 기술 선택에 있어 가장 중요한 것은 개발 환경에 가장 알맞는지 또는 자신이 가장 잘 활용할 수 있는 부분이라고 생각한다.



<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-02 오전 12.30.47.png" alt=""><figcaption><p><a href="https://db-engines.com/en/ranking_trend/relational+dbms">https://db-engines.com/en/ranking_trend/relational+dbms</a></p></figcaption></figure>

![](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1169fe06-a95b-413d-82bf-b54138abba95/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2023-01-26\_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB\_12.55.20.png)