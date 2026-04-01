# Logging

## Logging requests with the CommonsRequestLoggingFilter class

1. Set the `CommonsRequestLoggingFilter` filter with a Spring bean in a configuration file (`@Configuration` annotated class):
  
    ```java
    @Bean
    public FilterRegistrationBean<CommonsRequestLoggingFilter> loggingFilterRegistration() {
      CommonsRequestLoggingFilter filter = new CommonsRequestLoggingFilter();
      filter.setIncludeQueryString(true);
      filter.setIncludePayload(true);
      filter.setBeforeMessagePrefix("PAYLOAD: ");

      FilterRegistrationBean<CommonsRequestLoggingFilter> registrationBean = new FilterRegistrationBean<>();
      registrationBean.setName("CommonsRequestLoggingFilter");
      registrationBean.setFilter(filter);
      registrationBean.setOrder(2);
      registrationBean.addUrlPatterns("*/bigqueue/consumer");
      return registrationBean;
    }
    ```

2. Set `DEBUG` log level for `CommonsRequestLoggingFilter` in your properties:

    ```yaml
    spring:
      logging:
        level:
          org:
            springframework:
              web:
                filter:
                  CommonsRequestLoggingFilter: 'INFO'
    ```

3. Configure Spring to log all set with `DEBUG`, for example if you have a `logback-spring.xml`, log level is set with something like this :

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration scan="false">
      <!-- use Spring default values -->
      <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

      <springProfile name="local | integration_test">
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
            <!-- @formatter:off -->
            <pattern>
                %clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(--){faint} %clr([%1.15t]){faint} %clr(%-1.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}
            </pattern>
            <!-- @formatter:on -->
            <charset>utf8</charset>
          </encoder>
        </appender>
        <root level="DEBUG">
          <appender-ref ref="STDOUT"/>
        </root>
      </springProfile>
      ...
    ```

## Logging request with a custom filter

1. Create the `CachedHttpServletRequest` class :

    ```java
    package yourpackage;

    import java.io.BufferedReader;
    import java.io.ByteArrayInputStream;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
    import javax.servlet.ServletInputStream;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletRequestWrapper;
    import org.springframework.util.StreamUtils;

    public class CachedHttpServletRequest extends HttpServletRequestWrapper {

      private byte[] cachedPayload;

      public CachedHttpServletRequest(HttpServletRequest request) throws IOException {
        super(request);
        InputStream requestInputStream = request.getInputStream();
        this.cachedPayload = StreamUtils.copyToByteArray(requestInputStream);
      }

      @Override
      public ServletInputStream getInputStream() {
        return new CachedServletInputStream(this.cachedPayload);
      }

      @Override
      public BufferedReader getReader() {
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(this.cachedPayload);
        return new BufferedReader(new InputStreamReader(byteArrayInputStream));
      }

    }
    ```

2. Create the `CachedServletInputStream` class :

    ```java
    package yourpackage;

    import java.io.ByteArrayInputStream;
    import java.io.IOException;
    import java.io.InputStream;
    import javax.servlet.ReadListener;
    import javax.servlet.ServletInputStream;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    public class CachedServletInputStream extends ServletInputStream {

      private final static Logger LOGGER = LoggerFactory.getLogger(CachedServletInputStream.class);

      private InputStream cachedInputStream;

      public CachedServletInputStream(byte[] cachedBody) {
        this.cachedInputStream = new ByteArrayInputStream(cachedBody);
      }

      @Override
      public boolean isFinished() {
        try {
          return cachedInputStream.available() == 0;
        } catch (IOException exp) {
          LOGGER.error(exp.getMessage());
        }
        return false;
      }

      @Override
      public boolean isReady() {
        return true;
      }

      @Override
      public void setReadListener(ReadListener readListener) {
        throw new UnsupportedOperationException();
      }

      @Override
      public int read() throws IOException {
        return cachedInputStream.read();
      }
    }
    ```

3. Create the `RequestCachingFilter` class:

    ```java
    package com.mycompany.chronos_consistency_checker;

    import java.io.IOException;
    import java.nio.charset.StandardCharsets;
    import javax.servlet.FilterChain;
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import org.apache.commons.io.IOUtils;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.web.filter.OncePerRequestFilter;

    public class RequestCachingFilter extends OncePerRequestFilter {

    private final static Logger LOGGER = LoggerFactory.getLogger(RequestCachingFilter.class);

        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
          throws ServletException, IOException {
            CachedHttpServletRequest cachedHttpServletRequest = new CachedHttpServletRequest(request);
            LOGGER.info("REQUEST DATA: " + IOUtils.toString(cachedHttpServletRequest.getInputStream(), StandardCharsets.UTF_8));
            filterChain.doFilter(cachedHttpServletRequest, response);
        }
    }
    ```

4. Set the custom filter with a Spring bean in a configuration file (`@Configuration` annotated class): 

    ```java
      @Bean
      public FilterRegistrationBean<RequestCachingFilter> loggingFilterRegistration() {
        RequestCachingFilter requestCachingFilter = new RequestCachingFilter();

        FilterRegistrationBean<RequestCachingFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setName("CommonsRequestLoggingFilter");
        registrationBean.setFilter(requestCachingFilter);
        registrationBean.setOrder(2);
        registrationBean.addUrlPatterns("*/bigqueue/consumer");
        return registrationBean;
      }
    ```

## Links

- [Spring – Log Incoming](https://www.baeldung.com/spring-http-logging)
