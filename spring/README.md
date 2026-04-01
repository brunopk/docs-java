# Spring

Examples and documentation on different Spring topics, for a specific topic refer to the corresponding sub-folders and files.

## Responder con archivos

Endpoint que retorna un archivo (`byte[]`) a partir de una ruta absoluta (`/Users/brpiaggio/Downloads/latest.tar.gz`) :

```java
@GetMapping(value = "/latest.tar.gz", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
public @ResponseBody byte[] getFile() throws IOException {
  InputStream inputStream = Files.newInputStream(Paths.get("/Users/brpiaggio/Downloads/latest.tar.gz"));
  return inputStream.readAllBytes();
}
```

### Scheduled tasks

```java
package com.yourpackage;

import org.apache.commons.lang3.StringUtils;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class TestTasks {

  @Async
  @Scheduled(cron = "*/10 * * * * *")
  public void testTask() {
    String currentThread = Thread.currentThread().getName();
    System.out.println(String.format("Starting testTask on thread %s...", currentThread));

    long timeElapsed = 0;
    long currentTimeMillis = System.currentTimeMillis();
    long seconds = 0;
    long secondsAux = 0;

    while (timeElapsed < 8 * 1000) {
      if (secondsAux > seconds) {
        seconds = secondsAux;
        System.out.println(String.format("Seconds elapsed (thread %s): %s", currentThread, secondsAux));
      }
      timeElapsed = System.currentTimeMillis() - currentTimeMillis;
      secondsAux = timeElapsed / 1000;
    }

    while (8 * 1000 <= timeElapsed && timeElapsed < (8 * 1000) + 1000 * 1000) {
      if (secondsAux > seconds) {
        seconds = secondsAux;
        System.out.println(String.format("Seconds elapsed (thread %s): %s", currentThread, secondsAux));
      }
      timeElapsed = System.currentTimeMillis() - currentTimeMillis;
      secondsAux = timeElapsed / 1000;
    }

    System.out.println("Finalizing testTask...");
  }
}

```
