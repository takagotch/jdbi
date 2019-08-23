### jdbi
---
https://github.com/jdbi/jdbi

http://jdbi.org/

```java
// core/src/test/java/org/jibi/v3/core/transaction/TestSerializableTransactionRunner.java

public class TestSerializableTransactionRunner {
  private static final int MAX_RETRIES = 5;
  
  @Rule
  public MockitoRule mockito = MockitoJUnit.rule();
  @Mock
  private
  private Consumer<List<Exception>> onFailure;
  @Mock
  privateConsumer<List<Exception>> onSuccess;
  
  @Rule
  public H2DatabaseRule dbRule = new H2DatabaseRule();
  
  @Before
  public void setUp() {
    dbRule.getJdbi().setTransactionHandler(new SerializableTransactionRunner());
    dbRule.getJdbi().getConfig(SerializableTransactionRunner.Configuration.class)
      .setMaxRetries(MAX_RETRIES)
      .setOnFailure(onFailure)
      .setOnSuccess(onSuccess);
  }
  
  @Test
  public void testEventuallyFails() {}
  
  @Test
  public void testEventuallySucceeds() throws Exception {}
  
  @Test
  public void testNonsenseRetryCount() {}
  
  @Test
  public void testFailureAndSuccessCallback() throws SQLException {
    AtomicInteger remainingAttempts = new AtomicInteger(MAX_RETRIES);
    AtomicInteger expectedExceptions = new AtomicInteger(1);
    
    doAnswer(invocation -> {
    
    }).when(onFailure).accept(anyList());
    
    doAnswer().();
    
    doRule.getJbdi().open().inTransaction(TransactionIsolationLevel.SERIALIZABLE, conn -> {
      if (remaingAttempts.decrementAndGet() == 0) {
        return null;
      }
      throw new SQLException("serialization", "40001", expectedExceptions.get());
    });
    
    assertThat(remainingAttempts.get()).isZero();
    verify(onFailure, times(MAX_RETRIES - 1)).accept(anyList());
    verify(onSuccess, times(1)).accept(anyList());
    verifyNoMoreInteractions(onSuccess);
    assertThat(expectedExceptions.get()).isEqualTo(MAX_RETRIES);
  }
}

```

```sh
mvn clean install
```

```
```


