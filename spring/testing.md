# Mockito

```java
package com.mycompany.component_x.integration;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.mycompany.chronos_commons.dtos.time_bucket_operations.multi_update.TimeBucketMultiUpdateRequestDTO;
import com.mycompany.component_x.services.apiservices.ChronosAdminApiRestClient;
import com.mycompany.component_x.utils.ChronosDispatcherTestUtil;
import com.mycompany.component_x.utils.ContextJobInfrastructureTestInitializer;
import java.lang.instrument.Instrumentation;
import java.util.List;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.mock.mockito.SpyBean;
import org.springframework.context.ApplicationContext;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.web.servlet.MvcResult;

/**
 * DispatcherController tests.
 */
@ContextConfiguration(initializers = ContextJobInfrastructureTestInitializer.class)
public class DispatchControllerTest extends ControllerTest {

  private static final String BASE_URI = "/jobs-dispatch?";

  private static final String PARTITION_PARAM = "partition=";

  private static final String CONCURRENCY_READ_PARAM = "concurrency-read=";

  private static final String CONCURRENCY_SEND_PARAM = "concurrency-send=";

  private static ObjectMapper mapper;

  @SpyBean
  private static ChronosAdminApiRestClient chronosAdminApiRestClient;


  @BeforeAll
  public static void setUp(@Autowired ApplicationContext context) throws Exception {
    mapper = new ObjectMapper();
  }

  @Test
  void dispatchWithoutErrorsTest() throws Exception {
    int partition = 1;
    int concurrency = 2;
    ArgumentCaptor<TimeBucketMultiUpdateRequestDTO> captor = ArgumentCaptor.forClass(TimeBucketMultiUpdateRequestDTO.class);

    MvcResult result = mockMvc.perform(post(
        BASE_URI + PARTITION_PARAM + partition + "&" + CONCURRENCY_READ_PARAM + concurrency + "&" + CONCURRENCY_SEND_PARAM + concurrency)
        .contentType(MediaType.APPLICATION_JSON))
        .andExpect(status().isOk())
        .andReturn();

    Mockito.verify(chronosAdminApiRestClient, Mockito.times(2)).updateBuckets(captor.capture());
    TimeBucketMultiUpdateRequestDTO timeBucketMultiUpdateRequestDTO = captor.getValue();

    assertEquals("", result.getResponse().getContentAsString());
  }

  @Test
  void dispatchWithoutErrorsWithoutSettingConcurrencyTest() throws Exception {
    int partition = 1;
    MvcResult result = mockMvc.perform(post(BASE_URI + PARTITION_PARAM + partition)
        .contentType(MediaType.APPLICATION_JSON))
        .andExpect(status().isOk())
        .andReturn();

    assertEquals("", result.getResponse().getContentAsString());
  }

  @Test
  void dispatchWithAnErrorAtConcurrencyReadTest() throws Exception {
    int partition = 1;
    int concurrency = -1;

    validationMin(BASE_URI + PARTITION_PARAM + partition + "&" + CONCURRENCY_READ_PARAM + concurrency, HttpMethod.POST, "concurrencyRead",
        null, 1L);
  }

  @Test
  void dispatchWithAnErrorAtConcurrencySendTest() throws Exception {
    int partition = 1;
    int concurrency = -1;

    validationMin(BASE_URI + PARTITION_PARAM + partition + "&" + CONCURRENCY_SEND_PARAM + concurrency, HttpMethod.POST, "concurrencySend",
        null, 1L);
  }


  private void validationMin(String uri, HttpMethod method, String fieldName, Object objectDTO, Long min)
      throws Exception {
    String errorMessage = "For field: " + fieldName + " must be greater than or equal to " + min;

    String payLoadJson = null;
    if (objectDTO != null) {
      payLoadJson = mapper.writeValueAsString(objectDTO);
    }
    ChronosDispatcherTestUtil.assertInvalidDtoByValidation(uri, method, mockMvc, List.of(errorMessage), payLoadJson);
  }

}
```
