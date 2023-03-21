# Overview
Framework provides authentication, authorization and  protection against common attracks, support for securing both imperative and reactive applications.
**SpringSecurity source code**: github.com/spring-projects/spring-security/
**Spring Security is Open Source software** released under the Apache 2.0 license.
**Prerequisites**: >= Java8

# Latest version Spring Security 6.0
Requires JDK-17
https://docs.spring.io/spring-security/reference/whats-new.html

### Migration to 6.0
- Upfate to Spring Security 6.0
- Upfate `javax` to `jakarta`
- ....

Care about it after

# Getting Spring Securiy (Quited easy)
### Maven
#### Spring Boot with Maven
```xml
<dependencies>
	<!-- ... other dependency elements ... -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-security</artifactId>
	</dependency>
</dependencies>
```
If would like to use `LDAP`, `OAuth 2` and others, need to include more modules and dependencies.
#### Maven without Spring Boot
```xml
<dependencyManagement>
	<dependencies>
		<!-- ... other dependency elements ... -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-bom</artifactId>
			<version>{spring-security-version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
or
```xml
<dependencies>
	<!-- ... other dependency elements ... -->
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-config</artifactId>
	</dependency>
</dependencies>
```
#### Maven Repositories
All GA release are deployed to Maven Central, so no need more the config in maven repository.

If you use SNAPSHOT version
```xml
<repositories>
	<!-- ... possibly other repository elements ... -->
	<repository>
		<id>spring-snapshot</id>
		<name>Spring Snapshot Repository</name>
		<url>https://repo.spring.io/snapshot</url>
	</repository>
</repositories>
```
If you use milestone or release candidate version
```xml
<repositories>
	<!-- ... possibly other repository elements ... -->
	<repository>
		<id>spring-milestone</id>
		<name>Spring Milestone Repository</name>
		<url>https://repo.spring.io/milestone</url>
	</repository>
</repositories>
```

# Features
### Authentication
comprehensive support for authentication

Authentication is how how we verify the indentity of a user.
Common way is user to enter username and password.

**Password Storage**: 
`PasswordEncoder` interface is used to perform a one way transformation of a password to let the psasword be stored securely, used for storing a password that needs to be compared to a user provided password at the time of authentication.

**Password Storage History**:

1Store by paintext -> data can be dump by SQL injection -> lost the password 

2Store by SHA-256 (Can't be dump data) -> Hacker have `Rainbow Tables` to less the export to get the password then use the hash password in Rainbow Tables to request to our login -> 3Store by a salt + password then generate SHA-256 -> Now computer can generate SHA-256 a bilion per second so they can test with a bilion times in our application via login API. 

=> Less the performance of computer by “work factor” for their own system, A time to authentication take a second at least. So the hacker can't execute many times in a second. Just a time in a second for a thread -> reduce the risk.

**DelegatingPasswordEncoder**

To deal with three real problems.
- Many application use the old password encoding that cann't easily migrate
- The best practice for password storage will change again
- As a framework, Spring Security cann't make breaking changes frequently

By
- Ensuring that passwords are encoded by using the current password storage recommendations.
- Allowing for validating passwords in modern and legacy formats
- Allowing for upgrading the encoding in future.

```java
PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
```
**Password Storage Format**
`{id}encodedPasword` : id is an identifier that is used to look up PasswordEncoder should be used, encodedPassword is encoded password
Example: 
```
{noop}password 
{sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0
```
**Encode with Spring Boot CLI**
```
spring encodepassword password
{bcrypt}$2a$10$X5wFBtLrL/kHcmrOGGTrGufsBX8CJ0WpQpF3pgeuxBB/H73BK1DW6
```

Can use `bcrypt, Argon2, PBKDF2, scrypt` algorithm by respective implementation class

**Change Pasword Configuration**
```java
http.passwordManagement(Customizer.withDefaults());
```
When a password manager navigates to `/.well-known/change-password` then Spring Secrurity will redirect your endpoint `/change-password`
or
```java
http.passwordManagement((management) -> management
        .changePasswordPage("/update-password"));
```
navigate to `/update-password`

## Protection Against Exploits
Protection against common exploits. Whenever possible, the protection is enabled by default.

### Cross Site Request Forgery (CSRF)
Situation: You login to your bank web and without logout  -> go to a evil page with same browser -> evil page can you the cookie and do some request to your bank page -> can be lost the money
**Protecting Against CSRF Attacks**
- The **Synchronizer Token Pattern**
- Specifying the **SameSite Attribute** on your seesion cookie.

Both protections require `Safe methods be Idempotent`

**Safe Methods Must be Idempotent**
That mean HTTP methods `GET`, `HEAD`, `OPTIONS`, and `TRACE` shoudn't change the state of application.

**1. Synchronizer Token Pattern**
It is a predominant and comprehensive way to protect against CSRF, ensure each HTTP request requires session cookie and secure random generated value called CSRF token.

HTTP request submmited => server look up the expected CSRF token and compare it against actual CSRF token in HTTP request => false => reject

Key is actual CSRF token is a part of HTTP request, not automatically included by the browser.

Example : The request can add a `_csrf` parameter with secure random value.
```
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876&_csrf=4bfd1575-3ad1-4d21-96c7-4ef2d9f86721
```
Server recivied the request and match bw actual csrf and expected csrf token.

**2. SameSite Attribute**
Server set a SameSite attribute in cookie, external sites shouldn't be sent.

Spring Security doesn't directly control the creation of seesion cookie. Spring Session provides support for SameSite atrribute in servlet-based applications.

Example:
```
Set-Cookie: JSESSIONID=randomid; Domain=bank.example.com; Secure; HttpOnly; SameSite=Lax
```
Values :
- `Strict`: any request form same-site includes the cookie:
- `Lax`: Cookie are sent when coming from same-side or comes from top-level navigations (when go to a web site by click from a mail for example) and method is imempotent

**When to use CSRF protection**
Any request that could be processed by browser, disable if that is used only by non-browser clients.

**CSRF protection and JSON** ???
Should we protect JSON request made by JavaScript ? It depends.

**CSRF and StateLess browser Application**
Still can be attacks for example application uses a custom cookie contains all the state in it for authentication (Not JSESSIONID). When the CSRF attrack is made the custom cookie is sent with the request in same manner.

#### CSRF Coniderations
**Logging In**
Steps for attacks.
1. Malicious user perform a CSRF login with malicious user's credentials., the victim is now authenticated as the malicious user.
2. The malicious user then tricks the victim into visiting a webside and entering sensitive information.
3. Then the malicious user can see the sensitive information.

Should have the timeout.
**Logging out**
Seesion timeout

#### CSRF  and Session Timeout




