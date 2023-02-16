---
description: UserDetailsService
---

# 사용자 관리

사용자 관리를 위해서는 UserDetailsService 및 UserDetailsManager 인터페이스를 이용하면 된다.

UserDetailsService는 사용자 조회만을 UserDetailsManager는 사용자 추가 , 수정, 삭제 작업 기능도 포함한다.

이렇게 두 인터페이스로 나누어 사용자 인증하는 기능만 필요한 경우는 UserDetailsService만 구현하도록 유연하게 설계 되어 있다. 이 사례는 인터페이스 분리 원칙(ISP)을 잘 지킨 훌륭한 사례이다.

UserDetails는 권한에 대한 정보를 가지고 있으며 GrantedAuthority 인터페이스로 나타낼수 있다.

<img src="../../../../.gitbook/assets/file.excalidraw (1) (1) (1).svg" alt="사용자 관리를 수행하는 구성 요소간 종속성" class="gitbook-drawing">



##
