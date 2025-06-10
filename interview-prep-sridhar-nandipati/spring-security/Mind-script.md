### 🎥 Tutorial Summary

* **Title**: Spring Security with Spring Boot 3.0 - Complete Guide
* **Topic**: Spring Boot 3.0 Authentication & Authorization with Spring Security
* **Duration**: \~58 mins
* **Skill Level**: Intermediate

---

### ⏱️ Timeline Flow with Explanations

**00:00–00:35** – *Intro & Spring Security Changes in Spring Boot 3*

* `WebSecurityConfigurerAdapter` is removed in Spring Boot 3.x
* New way to configure security using beans

**00:58–02:04** – *Sample App Overview*

* Spring Boot 3.0.1
* Dependencies: Spring Security, Spring Web, Lombok
* Controller with 3 endpoints: `/welcome`, `/products`, `/products/{id}`

**02:04–05:11** – *Basic In-Memory User Setup*

* Security is auto-enabled due to dependency
* Define `spring.security.user.name` and `spring.security.user.password` in `application.properties`
* Access secured endpoints with these credentials

**05:11–13:03** – *Programmatic Configuration Using Java Class*

* Create `SecurityConfig` class
* `@Configuration`, `@EnableWebSecurity`
* Old `WebSecurityConfigurerAdapter` is deprecated/removed
* Instead, define:

    * Bean of `UserDetailsService`
    * Bean of `PasswordEncoder`
    * Use `InMemoryUserDetailsManager`

**13:03–16:38** – *Define Authorization with SecurityFilterChain*

* Create `SecurityFilterChain` bean
* Use `HttpSecurity` for endpoint-level rules
* `/products/welcome` → permit all
* Other endpoints → authenticated via form login

**16:38–21:27** – *Testing In-Memory Authorization*

* Access endpoints based on role/user
* All works as expected

**21:27–24:10** – *Method-Level Authorization with Roles*

* Use `@PreAuthorize("hasAuthority('ROLE_ADMIN')")` etc.
* Replace `@EnableGlobalMethodSecurity` with `@EnableMethodSecurity` (new in Spring Boot 3)

**24:10–26:32** – *Authorization Testing by Role*

* Only Admin can access `/products/all`
* Only User can access `/products/{id}`
* 403 Forbidden for invalid role access

**26:32–30:21** – *Switch to DB-Driven User Authentication*

* Create `UserInfo` entity
* Add required JPA dependency
* Note Jakarta persistence change in Spring Boot 3
* Create `UserInfoRepository`

**30:21–40:04** – *Custom UserDetailsService Implementation*

* Implement `UserDetailsService`
* Inject `UserInfoRepository`
* Create `UserInfoUserDetails` class to wrap DB entity
* Convert DB user into `UserDetails`

**40:04–43:18** – *Add Users via Endpoint*

* Create `addUser` endpoint in controller
* Save user with encoded password using `PasswordEncoder`

**43:18–45:00** – *Permit Add User Endpoint*

* Permit `/products/new` in `SecurityFilterChain`

**45:00–50:03** – *Configure MySQL & Insert Users from Postman*

* Add MySQL dependency, configure in `application.properties`
* POST requests via Postman to insert users
* Confirm insertion in DB

**50:03–54:05** – *Fix Missing Authentication Provider Error*

* Add `DaoAuthenticationProvider` bean
* Inject `UserDetailsService` and `PasswordEncoder`

**54:05–57:06** – *Final Testing*

* Test `/products/all` with Admin – Allowed
* Test with User – Forbidden
* Test `/products/{id}` with User – Allowed
* Test with Admin – Forbidden

**57:06–58:00** – *Wrap-Up*

* Highlights authentication vs. authorization
* Mentions potential JWT follow-up tutorial

---

### 🧠 Deep Understanding Notes (Detailed)

* **Why WebSecurityConfigurerAdapter was removed**: To enforce a more functional, bean-based configuration aligned with Spring’s declarative style.
* **UserDetailsService**: Core interface to retrieve user info for authentication
* **PasswordEncoder**: Essential to securely store and compare encrypted passwords (BCrypt is the standard)
* **SecurityFilterChain**: Replaces overridden `configure(HttpSecurity)` method
* **@PreAuthorize**: Allows securing methods using SpEL expressions like `hasAuthority()`
* **DaoAuthenticationProvider**: Talks to `UserDetailsService`, wraps logic to generate authentication token
* **In-memory vs. DB users**: Quick testing is fine with in-memory; production systems must rely on DB + encrypted credentials

---

### 📌 Highlights & Key Takeaways (Skimmable)

* 🟢 `@EnableMethodSecurity` replaces `@EnableGlobalMethodSecurity`
* 🔑 Use `SecurityFilterChain`, `UserDetailsService`, and `DaoAuthenticationProvider` in Spring Boot 3
* 💡 Encode passwords using `BCryptPasswordEncoder`
* ⚠️ `authorizedRequests()`, `antMatchers()` are replaced with `authorizeHttpRequests()` and `requestMatchers()`

---

### 🧾 Resume-Ready Points (Directly Usable)

* Implemented custom authentication and role-based authorization using Spring Security 6 in Spring Boot 3
* Migrated legacy security configuration to Spring Boot 3 bean-based setup with `SecurityFilterChain`
* Designed secure REST endpoints using `@PreAuthorize` and DB-driven user roles
* Created and integrated a custom `UserDetailsService` for dynamic user validation from MySQL
* Configured password encryption using `BCryptPasswordEncoder` and validated role-based access control via Postman and browser tests

---

### 🧪 Interview Readiness

1. **Q:** What replaced `WebSecurityConfigurerAdapter` in Spring Boot 3?
   **A:** A combination of `SecurityFilterChain`, `UserDetailsService`, and `AuthenticationProvider` beans.

2. **Q:** How do you secure a method in Spring Security 6?
   **A:** Use `@PreAuthorize("hasAuthority('ROLE_ADMIN')")` and enable it via `@EnableMethodSecurity`.

3. **Q:** Why use `BCryptPasswordEncoder`?
   **A:** It hashes passwords with a salt, making it resilient against rainbow table and brute-force attacks.

4. **Q:** What is `DaoAuthenticationProvider` used for?
   **A:** It links the user details service and password encoder to authenticate username-password logins.

5. **Q:** How does Spring Security 6 handle URL-based authorization?
   **A:** Through the `SecurityFilterChain` bean using methods like `requestMatchers()` and `authorizeHttpRequests()`.

---

Let me know if you want this exported as PDF, Notion doc, Markdown file, or included in your resume workspace. ✅
