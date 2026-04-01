# JWT

## Creating a signed JWT token

Steps to create a signed JWT token using `io.jsonwebtoken` :

1. Create the signing key with `openssl` command (it's recommended to store this key in application properties).
2. Include `io.jsonwebtoken` dependencies (refer to [`pom.xml`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-e65985dc2a105850c4dd749631555ca777f97f1a9a5e8fca45eecdb528e9ff7a) as an example).
3. Define a custom filter (refer to [`JwtAuthenticationFilter.java`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-e65985dc2a105850c4dd749631555ca777f97f1a9a5e8fca45eecdb528e9ff7a) as an example).
4. Set the filter in the filter chain (line 38 in [`SecurityConfig.java:38`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-691d1d44d6149eeb5c17cb1381723f68b54d174a6f926e36b888321cbeea1fc0R38)).
5. Define a function to generate JWT tokens and invoke this function to return this token to users (line 47 in [`OAuth2ServiceImpl.java`](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096#diff-1ba61cf3c87c78f179197d746ca77cb4833ef9dca3a518a8d99d43ac752a682bR47))

All files linked above were extracted from the [c1a1dbc](https://github.com/brunopk/mis-gastos-backend/commit/c1a1dbc619fdfbcbca2ed6d4078e1cd4afb28096) commit in https://github.com/brunopk/mis-gastos-backend.

## Links

- [Spring Security: Upgrading the Deprecated WebSecurityConfigurerAdapter](https://www.baeldung.com/spring-deprecated-websecurityconfigureradapter).
- [Stack overflow: My CustomAuthenticationProvider is not getting called](https://stackoverflow.com/questions/75516863/my-customauthenticationprovider-is-not-getting-called).
- [Stack overflow: Spring Security blocks POST requests despite SecurityConfig](https://stackoverflow.com/questions/51026694/spring-security-blocks-post-requests-despite-securityconfig).
