
# Security

Examples and documentation on different security related topics. For a specific topic refer to the corresponding sub-folders and files.

Take into account that to authenticate endpoints in Spring 3, some configurations must be done. These configurations are different to [configurations in previous versions of Spring](https://www.baeldung.com/spring-deprecated-websecurityconfigureradapter) based on extending `WebSecurityConfigurerAdapter` which is deprecated in newer versions. The provided configurations below are suitable for custom security configurations. In general, these configurations are commonly used for basic authorization with a username and password, as well as other standard or common methods of authorization.

## Using a custom implementation of the Authentication class

**This way of configuring authentication is not recommended for production** as it consists of manually modifying request context with a custom extension of `Authentication` interface. The common way to do this is relying on Spring, for example through OAuth, to set important authentication information such as the user (principal).

1. Add required dependencies to pom.xml :

     ```xml
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
     </dependency>
     ```

2. Create a class to implement the `Authentication` interface :

     ```java
     package com.yourpackage;

     import jakarta.servlet.http.HttpServletRequest;
     import java.util.Collection;
     import lombok.extern.slf4j.Slf4j;
     import org.springframework.security.core.Authentication;
     import org.springframework.security.core.GrantedAuthority;
 
     @Slf4j
     public class CustomAuthentication implements Authentication {

       /**
        * Constructor.
        */
       public CustomAuthentication() {
       }

       @Override
       public Collection<? extends GrantedAuthority> getAuthorities() {
         throw new UnsupportedOperationException();
       }

       @Override
       public Object getCredentials() {
         throw new UnsupportedOperationException();
       }

       @Override
       public Object getDetails() {
         throw new UnsupportedOperationException();
       }

       @Override
       public Object getPrincipal() {
         return null;
       }

       @Override
       public boolean isAuthenticated() {
         // Implement custom authenticatin logic
       }

       @Override
       public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
         throw new UnsupportedOperationException();
       }

       @Override
       public String getName() {
         throw new UnsupportedOperationException();
       }
     }
     ```

   For this simple case just implement `isAuthenticated`. Throwing exceptions in `getPrincipal` can generate issues on responses.

3. Create a Spring filter using previously implemented authentication:
  
     ```java
     package com.yourpackage;

     import jakarta.servlet.FilterChain;
     import jakarta.servlet.ServletException;
     import jakarta.servlet.http.HttpServletRequest;
     import jakarta.servlet.http.HttpServletResponse;
     import java.io.IOException;
     import java.util.regex.Matcher;
     import java.util.regex.Pattern;
     import javax.validation.constraints.NotNull;
     import org.springframework.lang.NonNullApi;
     import org.springframework.security.core.Authentication;
     import org.springframework.security.core.context.SecurityContextHolder;
     import org.springframework.web.filter.OncePerRequestFilter;

     /**
      * For each request, creates an instance of {@code CustomAuthentication} which implements {@code Authentication}
      * and makes it available for Spring Security by setting it on the {@code SecurityContext}.
      */
     public class AuthenticationFilter extends OncePerRequestFilter {

       public AuthenticationFilter() {
       }

       @Override
       protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
         throws ServletException, IOException {
         // ...
         Authentication authentication = new CustomAuthentication();
         SecurityContextHolder.getContext().setAuthentication(authentication);
         filterChain.doFilter(request, response);
       }
     }
     ```
  
     Important: don't forget to set the Authentication instance on the security context and invoke `doFilter` at the end in order to allow Spring to follow the correct filter chain.

4. Set the filter through a Spring bean in `@Configuration` annotated class:

     ```java
     @Bean
     @Profile({"!integration_test & !local"})
     @Autowired
     public SecurityFilterChain furySecurityFilterChain(HttpSecurity http, SecurityService securityService) throws Exception {
       return http
         .csrf(CsrfConfigurer::disable)
         .addFilterAfter(new AuthenticationFilter(), LogoutFilter.class)
         .authorizeHttpRequests(request ->
           request.requestMatchers("topics/**").authenticated()
             .anyRequest().permitAll())
         .build();
     }
     ```

     Important:

     - Without `csrf(CsrfConfigurer::disable)` Spring will block all POST endpoints (see [this](https://stackoverflow.com/questions/51026694/spring-security-blocks-post-requests-despite-securityconfig) Stack Overflow thread for more information).
     - Requests patterns used in `requestMatchers` method may contain `**`, **not regular expresions**.
     - The `LogoutFilter.class` argument tells Spring in which order to execute filters, which is mandatory.
