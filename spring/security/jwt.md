# JWT

## Creating and validating a signed JWT token with a custom filter

Steps to create and validate signed JWT tokens with a **custom filter** using `io.jsonwebtoken` :

1. Create the signing key with `openssl` command (it's recommended to store this key in application properties).
2. Include `io.jsonwebtoken` dependencies (refer to [`pom.xml`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-e65985dc2a105850c4dd749631555ca777f97f1a9a5e8fca45eecdb528e9ff7a) as an example).
3. Define a custom filter ([`JwtAuthenticationFilter`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-e65985dc2a105850c4dd749631555ca777f97f1a9a5e8fca45eecdb528e9ff7a)). This filter have two purposes: obtaining the JWT token from the `Authentication` header and (if token is validated correctly) setting an instance of `UsernamePasswordAuthenticationToken` that Spring internal security filters can check to verify that requests are authenticated.
4. Set the filter in the filter chain ([line 38 of `SecurityConfig`](https://github.com/brunopk/mis-gastos-backend/blob/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096/src/main/java/com/bruno/misgastos/config/SecurityConfig.java#L38)).
5. Define a function to generate JWT tokens and invoke this function to return this token to users ([line 47 of `OAuth2ServiceImpl`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-1ba61cf3c87c78f179197d746ca77cb4833ef9dca3a518a8d99d43ac752a682bR47))

All files linked above were extracted from https://github.com/brunopk/mis-gastos-backend.

## Links

- [Spring Security: Upgrading the Deprecated WebSecurityConfigurerAdapter](https://www.baeldung.com/spring-deprecated-websecurityconfigureradapter).
- [Stack overflow: My CustomAuthenticationProvider is not getting called](https://stackoverflow.com/questions/75516863/my-customauthenticationprovider-is-not-getting-called).
- [Stack overflow: Spring Security blocks POST requests despite SecurityConfig](https://stackoverflow.com/questions/51026694/spring-security-blocks-post-requests-despite-securityconfig).

## Creating and validating a signed JWT token with filters provided by Spring

This part principally shows how to create a JWT token and validate it **using filters provided by Spring** instead of creating a custom filter.

It also shows how to validate a token by checking a pre-defined (hardcoded) string in the JWT claims, that may be **useful for learning or test purposes not for production**.

1. Add the corresponding dependencies in `pom.xml` :
  
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
    </dependency>
    ```

2. Set the JWT filter invoking `oauth2ResourceServer` :
  
    ```java
      @Bean
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
      return http.csrf(
              csrf ->
                  csrf.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                      .ignoringRequestMatchers("/oauth2/token"))
          .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.NEVER))
          .authorizeHttpRequests(
              (authorizationManagerRequestMatcherRegistry) ->
                  authorizationManagerRequestMatcherRegistry
                      .requestMatchers("/oauth2/**")
                      .permitAll()
                      .anyRequest()
                      .authenticated())
          .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults())) 
          .build();
    }
    ```

3. Create the `JWTDecoder` bean required for the previously defined filter :

  ```java
  @Bean
  public JwtDecoder jwtDecoder() {
    Base64.Decoder base64Decoder = Base64.getDecoder();
    byte[] secretAsByArray = base64Decoder.decode(JWT_SECRET_KEY);
    SecretKey key = new SecretKeySpec(secretAsByArray, JWT_SECRET_KEY_ALGORITHM);

    NimbusJwtDecoder decoder = NimbusJwtDecoder.withSecretKey(key).build();

    OAuth2TokenValidator<Jwt> customValidator =
        jwt -> {
          String value = jwt.getClaim("myField");

          if (!"EXPECTED_VALUE".equals(value)) {
            return OAuth2TokenValidatorResult.failure(
                new OAuth2Error("invalid_token", "Invalid custom field", null));
          }

          return OAuth2TokenValidatorResult.success();
        };

    decoder.setJwtValidator(customValidator);

    return decoder;
  }
  ```

To generate the JWT tokens define a new endpoint like this :

1. Generate the signing key with `openssl` and add it in application properties :

    ```yaml
    security:
      oauth2:
        resourceserver:
          jwt: replace-with-secret-key
    ```

2. Define the endpoint :  

    ```java
    @GetMapping("/get-token")
    public ResponseEntity<Object> getToken() {
      Base64.Decoder base64Decoder = Base64.getDecoder();
      SecretKey secretKey = Keys.hmacShaKeyFor(base64Decoder.decode(JWT_SECRET_KEY));
      Date now = new Date();
      Date expirationTime = new Date(System.currentTimeMillis() + 60 * 1000);
      String token = Jwts.builder()
        .setIssuedAt(now)
        .setClaims(Map.of("myField", "EXPECTED_VALUE"))
        .setExpiration(expirationTime)
        .signWith(secretKey)
        .compact();
      return ResponseEntity.ok(Map.of("token", token));
    }
    ```

    where `JWT_SECRET_KEY` constant (signing key) is obtained from application properties with the `@Value` annotation :

    ```java
    @Value("${spring.security.oauth2.resourceserver.jwt.secret-key}")
    private String JWT_SECRET_KEY;
    ```
