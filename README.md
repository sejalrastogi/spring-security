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
    






