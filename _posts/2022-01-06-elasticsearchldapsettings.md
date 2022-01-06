---
title: "Elasticsearch Ldap Settings"
excerpt_separator: "<!--more-->"
categories:
  - Elasticsearch
tags:
  - Elasticsearch
  - LDAP
sidebar:
  nav: "docs"
---
예전에 Elastic Stack 을 이용해서 SIEM 을 구축했을 땐 Elasticsearch 의 보안 인증 설정을 LDAP 으로 했었다(~~이후에 SAML 로 바꾸려고 삽질도 했다가 어려워서 포기했던 기억... 담에 기회되면 꼭 도전해보고 싶음~~). 다만 내 손으로 처음부터 구축한 것도 아니었고, 시간이 많이 지나 가물가물해져서 기억 환기용으로 정리해보고자 한다.

## LDAP 과 Active Directory

- **LDAP** 이란, 네트워크 상에서 미리 구성한 디렉토리 정보를 검색하여 사용할 수 있게 한 소프트웨어 프로토콜이다.
- 풀어써 보자면, 다양한 응용 프로그램에서 어떤 서버 내의 (레코드 세트로 구성된)디렉토리에 보관되어 있는 특정 정보를 조회하는 데 사용할 수 있는 응용 프로그램 프로토콜이라고 할 수 있다.
- LDAP 서버는 **쉽게 업데이트되지 않는 정보를 보관하여 빠르게 조회**해야 할 때 유용하게 사용할 수 있다고 한다.
- 이 LDAP 을 이용해서 사용할 수 있게 구현한 서비스를 **디렉토리 서비스**라고 한다.
- 대표적으로 사용되는 디렉토리 서비스로는 **AD(Active Directory)** 가 있는데, Windows Server 2000 이상에서 지원되는 서비스로 LDAP 을 이용한 그룹/사용자/네트워크 정보를 통합 관리할 수 있는 대표적인 디렉토리 서비스이다(LDAP 버전 2, 3 지원).
- LDAP 과 AD 의 관계성을 어떤 이는 http 와 Apache 에 비교했는데, 개인적으로 이해하기 좋은 비유인 듯.
- AD 나 OpenLDAP 과 같은 디렉토리 서비스에서 사용하는 프로토콜이 LDAP 이고, 특히 AD 같은 경우에는 LDAP 뿐만 아니라 Kerberos 나 DNS 를 기반으로 한 인증도 지원해 준다.
    - Kerberos : 인증과 SSO 를 위한 프로토콜
- AD 가 Microsoft 독점 제품인만큼 주로 Windows 에서 쓰인다고 해서 LDAP 도 OS 를 타는가 하면 절대 그렇지 않다. **LDAP 은 거의 모든 OS 상관 없이 사용할 수 있는 프로토콜**이다(Linux 에서 디렉토리 서비스를 사용하려면 OpenLDAP 을 사용하면 될 듯?).

## Elasticsearch ldap settings

Elasticsearch 를 LDAP 서버와 연동해서, ES 에 로그인할 때 LDAP 서버의 정보를 읽어와 로그인하는 보안 설정을 추가할 수 있다.

- elasticsearch.yml

설정 파일에 ldap realm(영역)을 추가한 후 사용자 정보를 읽어 올 LDAP 서버 및 검색에서 사용할 정보를 설정한다.

```yaml
// 사용하는 namespace 는 xpack.security.authc.realms.ldap 이다.
xpack:
  security:
    authc:
      realms:
        ldap:
          // ldap realm 의 이름을 ldap1 로 지정한 것. 복수의 realms 설정할 때 이름이 상이해야 한다
          ldap1:
						// realm 적용 순서. 수치가 작을수록 우선순위가 높다
            order: 0
						// LDAP 서버의 URL 주소로, 다른 건 몰라도 이 주소만은 필수이다
            url: "ldaps://ldap.example.com:636"
						// 사용자 DN 정보. 사용자 검색 모드에서만 사용 가능하다
            bind_dn: "cn=ldapuser, ou=users, o=services, dc=example, dc=com"
            user_search:
							// 사용자를 검색할 컨테이너 DN 정보
              base_dn: "dc=example,dc=com"
							// 사용자 검색할 때 디렉토리를 검색하는 데 사용하는 필터 정보
              filter: "(cn={0})"
						// 사용자가 소속된 그룹을 검색할 컨테이너 DN 정보
            group_search:
              base_dn: "dc=example,dc=com"
						// Role 매핑 파일
            files:
              role_mapping: "ES_PATH_CONF/role_mapping.yml"
						// 이 설정이 true 인 경우, 매핑 파일에 등록되지 않는 그룹은 LDAP 서버의 그룹 이름으로 신규 role 이 등록된다. 기본값 false
            unmapped_groups_as_roles: false
```

참고) bind_dn 사용자에 패스워드를 설정하기 위해, 아래 명령어를 사용해서 elasticsearch 키스토어에  secure_bind_password 설정을 추가해 줘야 한다.

```bash
<path>/bin/elasticsearch-keystore add \
xpack.security.authc.realms.ldap.ldap1.secure_bind_password
```

- role_mapping.yml

```yaml
// 매핑할 role 이름이 monitoring
monitoring: 
	// admins 라는 이름을 가진 LDAP DN(고유 이름. Distinguished Name)
  - "cn=admins,dc=example,dc=com" 
// 매핑할 role 이름이 user
user:
	// users 라는 이름을 가진 LDAP DN
  - "cn=users,dc=example,dc=com" 
  - "cn=admins,dc=example,dc=com"
```

Role Mapping 은 API 로도 설정할 수 있다.

```json
// 위의 admins 라는 유저에 monitoring 과 user 라는 role 과 LDAP 설정을 API 로 put 해준다
PUT _xpack/security/role_mapping/admins
{
  "roles" : [ "monitoring", "user" ],
  "rules" : { "field" : { "groups" : "cn=admins,dc=example,dc=com" } },
  "enabled": true
}
```

- roles.yml

사실 roles.yml 에다가 role 설정을 적어도 되지만, 웬만하면 Kibana 에서 UI 로 만들어주자. 왜? roles.yml 에 이렇게 적혀 있었기 때문...

<div class="notice--info" markdown="1">
📢 The default roles file is empty as the preferred method of defining roles is through the API/UI. File based roles are useful in error scenarios when the API based roles may not be available.
</div>

### 참고

#### LDAP 배경 지식

- [https://ko.strephonsays.com/ldap-and-vs-ad-6331](https://ko.strephonsays.com/ldap-and-vs-ad-6331)
- [https://paulsmooth.tistory.com/166](https://paulsmooth.tistory.com/166)
- [https://yongho1037.tistory.com/796](https://yongho1037.tistory.com/796)

#### Elasticsearch LDAP Settings

- [https://www.elastic.co/kr/blog/demystifying-authentication-and-authorization-in-elasticsearch](https://www.elastic.co/kr/blog/demystifying-authentication-and-authorization-in-elasticsearch)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/ldap-realm.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/ldap-realm.html)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-ldap-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-ldap-settings)
- [https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role-mapping.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role-mapping.html)
- [https://www.popit.kr/엘라스틱서치에서-x-pack을-이용하여-ldap-연동하기/](https://www.popit.kr/%EC%97%98%EB%9D%BC%EC%8A%A4%ED%8B%B1%EC%84%9C%EC%B9%98%EC%97%90%EC%84%9C-x-pack%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-ldap-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0/)
