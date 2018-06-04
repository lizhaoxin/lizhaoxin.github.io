---
layout: post
title: "Java8重试的优雅实现"
date: 2018-06-04
---

### 重试接口的定义

在Java8中，对于只有一个方法声明的接口，可被认为是函数式接口。函数式接口可以使用lambda表达式赋值。利用这个特性，我们定义重试任务接口。

  interface RetryHandler<T> {
        T tryExecute() throws Exception;
  }
  
  
### 异常处理接口定义

异常发生时，通过异常接口的实现，完成用户自定义的处理。

    interface RetryExceptionHandler {
        void handleException(Throwable cause);
    }


### 定义辅助类，包装重试框架的逻辑

辅助类里面，实现2个方法。设计上，前一个方法提供默认的异常处理实现，在等待指定的延时后，重新尝试任务。后一个方法则提供自定义异常处理实现。

  class RetryHelper {
    static <T> T retry(final int maxRetries, final long delayMillis, RetryHandler<T> runnable) throws Throwable {
        for (int retries = 1;; retries++) {
            try {
                return runnable.tryExecute();
            } catch (Throwable t) {
                if (retries <= maxRetries) {
                    System.out.println("Retry Failed: Total " + retries + " attempts made at interval " + delayMillis + "ms");
                    try {
                        Thread.sleep(delayMillis);
                    } catch (InterruptedException ignored) {
                        // ignored.printStackTrace();
                    }
                    continue;
                } else {
                    throw t;
                }
            }
        }
    }

    static <T> T retry(final int maxRetries,
                       RetryHandler<T> runnable,
                       RetryExceptionHandler exceptionHandler) throws Throwable {
        for (int retries = 1; retries <= maxRetries; retries++) {
            try {
                return runnable.tryExecute();
            } catch (Throwable t) {
                exceptionHandler.handleException(t);
            }
        }
        return null;
    }
  }

### 使用用例

  public class RetryHelperTest {
    public static void main(String[] args) throws Throwable {

          // case 1
          System.out.println(
                  "3 / 9 = " + RetryHelper.retry(3, 5000, () -> 3.0 / 9));

          // case 2
          System.out.println(
                  "3 / 1 = " + RetryHelper.retry(3, () -> 3 / 1, (Throwable cause) -> {

                      System.out.println(cause.getMessage());

                      try {
                          Thread.sleep(2000);
                      } catch (InterruptedException ignored) {
                          // ignored.printStackTrace();
                      }
                  }));

          // case 3
          System.out.println(
                  "3 / 0 = " + RetryHelper.retry(3, 1000, () -> 3 / 0));

      }
  }

---
### 扩展链接
- [how-do-you-implement-a-re-try-catch](https://stackoverflow.com/questions/13239972/how-do-you-implement-a-re-try-catch?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
- [how-to-retry-operation-n-number-of-times-in-java](https://crunchify.com/how-to-retry-operation-n-number-of-times-in-java/)



    }链接
    }
