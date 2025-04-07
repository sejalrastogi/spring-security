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

### 4. How Does Spring Security Handle Authentication?

**Answer:**

The following image visually represents the authentication process in Spring Security:

![Spring Authentication Flow](src/main/resources/static/spring_security.png)

📌 **Step 1: Client Request → Filter Chain**

A user sends a request to access a secured resource.

The request goes through the Filter Chain, where security-related filters intercept and process it.

📌 **Step 2: Authentication Filter → Authentication Manager**

The Authentication Filter extracts user credentials (like username and password) and forwards them to the AuthenticationManager.

📌 **Step 3: Authentication Provider → UserDetailsService**

The AuthenticationManager delegates authentication to an Authentication Provider (e.g., DaoAuthenticationProvider).

The Authentication Provider then calls UserDetailsService to fetch user details from the Database using findByUsername().

📌 **Step 4: Password Validation → Security Context**

The password is verified using PasswordEncoder (e.g., BCrypt).

If authentication is successful, the Security Context stores the authenticated user details.

📌 **Step 5: Access Application Controller → Response to Client**

If authentication is successful, the request is processed by the Application Controllers and the response is sent back to the user.

### 5. Let’s see how we configure security in a Spring Boot application and why it’s important to learn this.

**Answer:**

In Spring Boot applications, configuring security is essential to protect your endpoints and data from unauthorized access. Spring Security offers a highly customizable way to define how authentication and authorization should be handled.

We use a SecurityFilterChain bean to define these configurations. Here's a simple example:

```
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
.httpBasic(withDefaults())
.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
return http.build();
}
```

**This code does three important things:**

- Requires authentication for all incoming requests.

- Uses HTTP Basic authentication, which is simple and useful for testing APIs.

- Configures stateless session management, meaning the server doesn't store user sessions—ideal for REST APIs.

**📌 Why is this important?**

Understanding how to configure security ensures:

- Your application is protected from unauthorized access.

- You can choose the right authentication mechanism for your use case.

- You know how to build scalable and secure REST APIs, which are the backbone of modern web apps.

**Now, based on this configuration, here are some follow-up questions you can dive into to understand each part in more detail:**

#### ❓ What is SecurityFilterChain, and why do we use it in Spring Security?

**Answer:**

**SecurityFilterChain** is a core interface in Spring Security that defines a chain of filters used to secure HTTP requests.

Prior to Spring Security 5.7, we used to extend **WebSecurityConfigurerAdapter** and override its methods to customize security. However, with recent versions (especially from 6.x), Spring promotes a more functional, component-based approach using beans.

With SecurityFilterChain, you define a @Bean that directly configures security using HttpSecurity.

Why it's important:

- It's declarative and modern — aligning with the Spring Boot philosophy of configuration via beans.

- Makes security easier to test, modularize, and customize.

- You can define multiple filter chains for different endpoint patterns (e.g., /admin/**, /api/**).

#### ❓ What does .authorizeHttpRequests() do in the configuration?

**Answer:**

**.authorizeHttpRequests()** is a method on the HttpSecurity object that allows you to define authorization rules — essentially, who can access what in your application.

You can use it to:

- Allow unrestricted access to public resources (permitAll())

- Require authentication for sensitive resources (authenticated())

- Restrict access based on roles (hasRole("ADMIN"), etc.)

**Example:**

```
http.authorizeHttpRequests(auth -> auth
.requestMatchers("/public/**").permitAll()
.requestMatchers("/admin/**").hasRole("ADMIN")
.anyRequest().authenticated());
```

**This tells Spring:**

- Let anyone access /public/**

- Only allow admins to access /admin/**

- All other requests must be authenticated 

#### ❓ Why do we use .httpBasic() and when should we choose .formLogin() instead?

**Answer:**

**.httpBasic()** enables HTTP Basic Authentication, where the client sends a username and password with every request (usually via headers).

It’s simple and best for tools like Postman or for internal APIs.

Example header:
**Authorization: Basic c2VqYWw6c2VqYWxJc1RoZUJlc3Q=**

**.formLogin()** enables a login form-based authentication for browser-based applications. It redirects unauthenticated users to a login page (/login) and stores session data after successful login.

When to use:

- Use .httpBasic() for APIs or development/testing.

- Use .formLogin() for traditional web applications where a login page is needed.

#### ❓ What is the role of SessionCreationPolicy.STATELESS, and why is it used here?

**Answer:**

SessionCreationPolicy tells Spring Security how to manage sessions. The STATELESS policy means do not create or use any HTTP session.

**🧠 What does "stateless" mean?**

In a stateless application, each HTTP request is treated independently — the server doesn't store any information about previous requests (no session, no memory of who you are).

**What happens when you set SessionCreationPolicy.STATELESS?**

- No HTTP session is created by Spring Security.

- No JSESSIONID cookie is sent or expected.

- The application does not remember the user between requests.

- You must send authentication credentials (or token) with every request.

**Example Use Case:**

This is ideal for REST APIs, where each request should be self-contained:

```
http
.sessionManagement(session ->
session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

**This tells Spring:**

“Don't save any login session. Every request must come with its own authentication (like a token or basic auth header).”

**❌ Without STATELESS:**

- Spring would create a session after login and store the user's authentication in it.

- Future requests wouldn't need credentials again, just the session cookie.

**⚠️ Important Note:**

- Stateless apps are more scalable (no session memory on server).

- But you must handle auth differently — usually with JWT tokens or passing credentials every time.

**Summary:**

| With STATELESS |	Without STATELESS|
|----------------| ------------------|
|No session stored|	Session created (JSESSIONID)|
|Send credentials every time|	Login once, then use session |
|Perfect for REST APIs |	Better for web apps|

Other Session Policies:

- **ALWAYS:** Always create an HTTP session

- **NEVER:** Don’t create but use if exists

- **IF_REQUIRED:** Default, creates a session if needed

Using STATELESS makes your app more secure, predictable, and scalable.

### 6. How do we configure in-memory users in Spring Security, and why is it important?
Answer:
In Spring Security, we can configure in-memory users using the UserDetailsService bean with InMemoryUserDetailsManager. This allows us to define a list of users, their passwords, and their roles directly in the code, without connecting to a database.

For example:

```
@Bean
public UserDetailsService userDetailsService() {
    UserDetails user1 = User.withUsername("user1")
                            .password("{noop}password1")
                            .roles("USER")
                            .build();

    UserDetails user2 = User.withUsername("user2")
                            .password("{noop}password2")
                            .roles("ADMIN")
                            .build();

    return new InMemoryUserDetailsManager(user1, user2);
}
```

**This is especially useful for:**

- Quickly setting up authentication during development or testing

- Demonstrating role-based access control without complex setup

**However, it’s important to note:**

- The {noop} prefix disables password encoding (not recommended for production).

- In-memory storage is not persistent — users reset when the app restarts.

- For production, users should be stored securely in a database and passwords should be properly encoded using a PasswordEncoder.

### 7. How can we implement role-based access control at the method level in Spring Security?

**Answer:**

We can enforce role-based access control at the method level using annotations like **@PreAuthorize**, along with **@EnableMethodSecurity** in the configuration class. This is known as **method-level security**, and it allows you to secure individual controller methods based on user roles.

**✅ Step-by-step setup:**

1. Enable Method Security

Add this annotation in your security configuration:
    ```
    @EnableMethodSecurity
    ```

This enables annotations like @PreAuthorize, @PostAuthorize, etc.

2. Add Role-based Restrictions using @PreAuthorize:

    ```
    @PreAuthorize("hasRole('USER')")
    @GetMapping("/user")
    public String userEndpoint() {
        return "<h1>Hello, User!</h1>";
    }
   
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/admin")
    public String adminEndpoint() {
        return "<h1>Hello, Admin!</h1>";
    }
   ```

**How it works:**

- When a user accesses **/user**, Spring checks if the authenticated user has the role **USER**.

- Similarly, for **/admin**, the user must have the role **ADMIN**.

- If the condition fails, Spring returns a 403 Forbidden.

**Note:**

- Make sure your users have roles defined correctly in your UserDetailsService.

**Example user setup:**

```
    UserDetails user = User.withUsername("user1")
    .password("{noop}pass")
    .roles("USER") // automatically becomes ROLE_USER
    .build();
```

## H2 Database

### 8. What is the H2 database, and why do we use it in Spring Boot projects?

**Answer:**

H2 is an in-memory, lightweight, fast relational database that is easy to embed into Java applications.

**We use it during development and testing because:**

- It doesn’t require external setup or installation.

- It resets automatically on each application restart.

- It helps simulate real database behavior without needing a full DB engine like MySQL or PostgreSQL.

#### ❓ How do we enable and secure the H2 console in a Spring Boot application?

**Answer:**

To enable and view the H2 console in a Spring Boot app, especially when security is enabled, you need to follow these key steps:

**1. Add Dependencies**

You added the following in pom.xml:

```
<!-- JPA for database operations -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- H2 in-memory database -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

✅ This sets up Spring Data JPA and the H2 database which runs in-memory.

 **2. Configure application.properties**

```
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:test
```

✅ This enables the H2 web console and configures the database to use an in-memory data source.

**3. Allow Access in Security Config**

You did this in your SecurityFilterChain:

```
http.authorizeHttpRequests(auth ->
auth.requestMatchers("/h2-console/**").permitAll()
.anyRequest().authenticated())
.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
.httpBasic(withDefaults());
```

✅ This makes /h2-console/** accessible without authentication, while securing other endpoints.

**4. Handle CSRF and Frame Options**

`What is CSRF?`

**CSRF** stands for Cross-Site Request Forgery.
It's an attack where a malicious website tricks a user (who is already authenticated) into making unintended requests to your app — with their valid credentials.

Spring Security enables CSRF protection by default for stateful applications, especially those using cookies for session tracking (like form-based login apps).

`Why are we disabling CSRF here?`

- The H2 console is a web-based tool that internally uses HTML forms to send data (like queries or settings).

- CSRF protection blocks these form submissions because they don't include a CSRF token.

- But the H2 console doesn't support CSRF tokens.

✅ So during development, we disable CSRF protection using:

```
http.csrf(csrf -> csrf.disable());
```

`What are frame options?`

Spring Security sets the `X-Frame-Options` HTTP header to DENY by default.
This is a security feature to protect your app from **clickjacking** attacks.

| **Clickjacking** is when a malicious site loads your app in an invisible `<iframe>` and tricks users into clicking buttons that perform unintended actions. |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------|

`What does sameOrigin do?`

The H2 console is rendered in an `<iframe>` — which won’t load if Spring Security blocks it due to frame options.

```
http.headers(header -> header.frameOptions(frameOptions -> frameOptions.sameOrigin()));
```

This tells Spring Security:

     "Allow pages to be embedded in <iframe>s only if the request comes from the same origin (i.e., same domain and port)."

✅ **This is a safe middle ground for development** — H2 console loads fine, and you're still protected from external domains trying to embed your app.

#### ❓ What should be avoided when using the H2 console in production?

**Answer:**

- Do not use `.csrf().disable()` in production.

- Do not expose `/h2-console/**` endpoints in a live environment.

- The H2 database itself is not meant for production use — always replace it with a persistent database.

## JDBC Authentication

### 9. What is JDBC Authentication in Spring Security?
Detailed Explanation:

JDBC Authentication is a mechanism where Spring Security authenticates users by querying a relational database using JDBC (Java Database Connectivity). Instead of using hardcoded users like in-memory authentication, it fetches the username, password, and roles directly from database tables (usually named users and authorities).

✅ Why it’s useful:

- Your application can manage users dynamically.

- Credentials are persisted, not lost on app restart.

- It scales well for real-world applications. 

#### ❓ Why do we prefer JDBC Authentication over In-Memory Authentication?

In-memory authentication stores credentials in code like:

```
User.withUsername("admin").password("{noop}pass").roles("ADMIN").build()
```

**This:**
    
- ✅ Works well for demos or tests

- ❌ Doesn't scale

- ❌ Loses users on restart

- ❌ Isn't secure for production

**JDBC Authentication:**

- ✅ Stores users in a database (e.g., H2, MySQL)

- ✅ Allows changes without modifying code

- ✅ Supports secure hashing, role updates, etc.

#### ❓ How do we enable JDBC Authentication in Spring Boot?

**Detailed Steps:**

1. First, we inject a DataSource bean:

```
@Autowired
DataSource dataSource;
```

2. Then return a JdbcUserDetailsManager in your security config:

```
@Bean
public UserDetailsService userDetailsService() {
    JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);
    manager.createUser(...); // Optional user creation
    return manager;
}
```
Spring will now use this manager to:

- Query the users and authorities tables.

- Authenticate users using credentials stored in DB.

#### ❓ Why did we get a bad SQL grammar error when using createUser()?

When we used manager.createUser(...), Spring tried to execute this SQL behind the scenes:

```
INSERT INTO users (username, password, enabled) VALUES (?, ?, ?)
```

But if the `users` table doesn't exist yet in the database, you'll get:

    PreparedStatementCallback; bad SQL grammar

This is Spring's way of saying:

    “I expected a table named users, but I couldn't find it!”

#### ❓ How did we fix the table not found / bad SQL grammar issue?

We created a file named schema.sql in the src/main/resources folder and added the required table definitions:

```
create table users(
    username varchar_ignorecase(50) not null primary key,
    password varchar_ignorecase(500) not null,
    enabled boolean not null
);

create table authorities (
    username varchar_ignorecase(50) not null,
    authority varchar_ignorecase(50) not null,
    constraint fk_authorities_users foreign key(username) references users(username)
);

create unique index ix_auth_username on authorities (username, authority);

```


Spring Boot automatically detects this file and runs it when the app starts (if auto-initialization is enabled).

#### ❓ Where did we get the schema definition from?

Spring Security provides a default schema for JDBC Authentication. We fetched it directly from their GitHub:

[📎 users.ddl - Spring Security](https://github.com/spring-projects/spring-security/blob/main/core/src/main/resources/org/springframework/security/core/userdetails/jdbc/users.ddl)

It ensures Spring can manage users with the expected structure.

## JWT Authentication

### 10. What is JWT and why do we use it?

**Answer:** 

JWT stands for JSON Web Token. It is a compact, URL-safe means of representing claims between two parties.

**🔑 Why use JWT?**

- It enables stateless authentication.

- The token contains all necessary user data (like username, roles), so the server doesn’t need to store session state.

- Commonly used in REST APIs where traditional session management (like JSESSIONID) doesn't scale.

### 11. How does the JWT Authentication flow work?

**Answer:**

**Here’s the full flow 🔁:**

![JWT Authentication Flow](src/main/resources/static/jwt_authentication.png)

1. User logs in → sends username & password.

2. Server authenticates and generates a JWT signed with a secret key.

3. JWT is returned to the client.

4. Client stores it (usually in localStorage or sessionStorage).

5. For future requests, client sends the JWT in the Authorization header:

   `Authorization: Bearer <token>`

6. Server verifies the JWT, extracts user info, and grants access.

### 12. What are the responsibilities of key components like `JwtUtils`, `AuthEntryPointJwt`, and `AuthTokenFilter`?

**Answer:**

`JwtUtils`

- Responsible for creating, validating, and parsing JWT tokens.

- Uses a secret key to sign the token.

- Extracts username and checks expiration during validation.

`AuthEntryPointJwt`

- Handles unauthorized access attempts.

- Sends a 401 Unauthorized response when token is missing or invalid.

- Acts as a gatekeeper when the user is not authenticated.

`AuthTokenFilter`

- A request filter that runs before each secured request.

- Extracts the token from the Authorization header.

- Validates the token using JwtUtils and sets the authentication context if valid.




