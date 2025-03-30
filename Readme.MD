# Spring Security

### 1. How does Spring Security behave when added to a project without any additional configuration?

**Answer:**

If you add the Spring Security dependency and do nothing else, Spring Boot will automatically apply default security settings, which include:

- ✅ **Basic Authentication:** A login form will be enabled if it's a web application.
- ✅ **Auto-Generated Credentials:** The default username will be user, and a random password will be generated and printed in the logs at startup.
- ✅ **Security Filters Enabled:** Spring Security will automatically protect all endpoints, requiring authentication for any request.
- ✅ **No Custom User Details:** Since no users are configured, it uses the default in-memory user.

**Example:**

If you add this dependency:

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

When you start the application, you'll see a log message like:
```
Using generated security password: a1b2c3d4e5f6
```
And when you try to access any endpoint, it will prompt for credentials.

### 2. What happens if I configure a single user with credentials (e.g., foo as both username and password) in application.properties?

**Answer:**

If you explicitly define a user in **application.properties** like this:

```
spring.security.user.name=foo
spring.security.user.password=foo
```

**Spring Boot will:**

- ✅ Override the default user: The auto-generated user will be replaced with foo.
- ✅ Enable Basic Authentication: All endpoints will still be protected, but now require foo as the username and password.
- ✅ No Password Encoding: If foo is stored as plaintext, Spring Security will allow it but may warn about insecure storage.
- ✅ No Role-Based Access Control (RBAC): The user will have the default ROLE_USER unless explicitly assigned.

**Example Scenario:**

1. Start the application and access an endpoint (e.g., http://localhost:8080).

2. You'll be prompted for credentials.

3. Enter foo as both username and password.

4. You’ll be authenticated and granted access.





