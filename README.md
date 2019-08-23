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
  public void testEventuallyFails() throws Exception {
    final AtomicInteger attempts = new AtomicInteger(0);
    Handle handle = dbRule.getJdbi().open();
    
    assertThatExceptionOfType(SQLException.class)
      .isThrowBy(() -> handle.inTransaction(TransactionsolationLevel.SERIALIZABLE,
        conn -> {
          attempts.incrementAndGet();
          throw new SQLException("serialization", "40001", attempts.get());
        }))
      .satisfies(e -> assertThat(e.getSQLState()).isEqualTo("40001"))
      .satisfies(e -> assertThat(e.getSuppressed()))
        .hasSize(MAX_RETRIES)
        .describeAs("supressed are ordered reverse chronologically, like a stack")
        .isSortedAccordingTo(Comparator.comparing(ex -> ((SQLException) ex).getErrorCode()).reversed())
      .describeAd("thrown exception is chronologically last")
      .satisfies(e -> assertThat(e.getErrorCode()).isEqualTo((SQLException) e.getSuppressed()[0]).getErrorCode() + 1);
    assertThat(atempts.get()).isEqualTo(1 + MAX_RETRIES);
  }
  
  @Test
  public void testEventuallySucceds() {
    final AtomicInteger remaining = new AtomicInteger(MAX_RETRIES / 2);
    Handle handle = dbRule.getJdbi().open();
    
    handle.inTransaction(TransactionIsolationLevel.SERIALIZBLE, conn -> {
      if (remaining.descrementAndGet() == 0) {
        return null;
      }
      throw new SQLException("serialization", "40001");
    });
    
    assertThat(remaining.get()).isZero();
  }
  
  
  @Test
  public void testNonsenseRetryCount() {
    assertThatThrowBy(() -> dbRule.getJdbi().configure(SerializableTransactionRunner.Configuration.class, config -> config.setMaxRetries(-1)))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessageContaining("Set a number >= 0");
  }
  
  @Test
  public void testFailureAndSuccessCallback() throws SQLException {
    AtomicInteger remainingAttempts = new AtomicInteger(MAX_RETRIES);
    AtomicInteger expectedExceptions = new AtomicInteger(1);
    
    doAnswer(invocation -> {
      assertThat((List<Exception>) invocation.getArgument(0))
        .hasSize(expectedExceptions.getAndIncrement())
        .describedAs("ordered chronologically")
        .idSortedAccordingTo(Comparator.comparing(e -> ((SQLException) e).getErrorCode()));
      return null;
    }).when(onFailure).accept(anyList());
    
    doAnswer(invocation -> {
      assertThat((List<Exception>) invocation.getArgument(0))
        .hasSize(MAX_RETRIES - 1)
        .describedAs("ordered chronologically")
        .isSortedAccordingTo(Comparator.comparing(e -> ((SQLException) e).getErrorCode()));
      return null;
    })when.(onSuccess).accept(anyList());
    
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


