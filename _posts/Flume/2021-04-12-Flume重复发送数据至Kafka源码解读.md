```java

for (; processedEvents < batchSize; processedEvents += 1) {
    try{
          kafkaFutures.add(producer.send(record, new SinkCallback(startTime)));
        } catch (NumberFormatException ex) {
          throw new EventDeliveryException("Non integer partition id specified", ex);
        } catch (Exception ex) {
          // N.B. The producer.send() method throws all sorts of RuntimeExceptions
          // Catching Exception here to wrap them neatly in an EventDeliveryException
          // which is what our consumers will expect
          throw new EventDeliveryException("Could not send event", ex);
        }
      }

      //Prevent linger.ms from holding the batch
      producer.flush();

      // publish batch and commit.
      if (processedEvents > 0) {
        for (Future<RecordMetadata> future : kafkaFutures) {
          future.get();
        }
        long endTime = System.nanoTime();
        counter.addToKafkaEventSendTimer((endTime - batchStartTime) / (1000 * 1000));
        counter.addToEventDrainSuccessCount(Long.valueOf(kafkaFutures.size()));
      }

      transaction.commit();
```






````java
//#### send(ProducerRecord<K, V> record, Callback callback) 实现

try{
						int partition = partition(record, serializedKey, serializedValue, metadata.fetch());
            int serializedSize = Records.LOG_OVERHEAD + Record.recordSize(serializedKey, serializedValue);
            ensureValidRecordSize(serializedSize);
            TopicPartition tp = new TopicPartition(record.topic(), partition);
            log.trace("Sending record {} with callback {} to topic {} partition {}", record, callback, record.topic(), partition);
            RecordAccumulator.RecordAppendResult result = accumulator.append(tp, serializedKey, serializedValue, callback, remainingWaitMs);
            if (result.batchIsFull || result.newBatchCreated) {
                log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
                this.sender.wakeup();
            }
            return result.future;
            // handling exceptions and record the errors;
            // for API exceptions return them in the future,
            // for other exceptions throw directly
        } catch (ApiException e) {
            log.debug("Exception occurred during message send:", e);
            if (callback != null)
                callback.onCompletion(null, e);
            this.errors.record();
            return new FutureFailure(e);
        }
````
```java

    /**
     * Validate that the record size isn't too large
     */
    private void ensureValidRecordSize(int size) {
        if (size > this.maxRequestSize)
            throw new RecordTooLargeException("The message is " + size +
                                              " bytes when serialized which is larger than the maximum request size you have configured with the " +
                                              ProducerConfig.MAX_REQUEST_SIZE_CONFIG +
                                              " configuration.");
        if (size > this.totalMemorySize)
            throw new RecordTooLargeException("The message is " + size +
                                              " bytes when serialized which is larger than the total memory buffer you have configured with the " +
                                              ProducerConfig.BUFFER_MEMORY_CONFIG +
                                              " configuration.");
    }
```



```java

private static class FutureFailure implements Future<RecordMetadata> {

        private final ExecutionException exception;

        public FutureFailure(Exception exception) {
            this.exception = new ExecutionException(exception);
        }

        @Override
        public RecordMetadata get() throws ExecutionException {
            throw this.exception;
        }

    }
```



`producer.send()` 中会判断单条数据的大小，假如单条数据大于1M则会返回 `new FutureFailure(e)` , `producer.flush()` 会将数据正常flush至kafka中，但是当futer.get()时会获取到异常。事务提交失败，就会重新发送数据，会不断的重复发送数据。

