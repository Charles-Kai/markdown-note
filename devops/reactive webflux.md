https://springdoc.cn/spring-webflux-retry/

https://www.yuque.com/leifengyang/springboot3

https://gitee.com/leifengyang/spring-boot-3/tree/master

WireMockServer模拟第一次返回503，第二次200
reactive http请求使用失败retry(2)，如何在最后一次失败调用其他方法?


乐观锁update影响行数为0，通常要抛异常回滚吗？
jackson string to date默认的格式
swagger 根据path parameter关联request body schema

Anonymous
Star on Github






2024/1/27 16:19:45

reactive http请求使用失败retry(2)，如何在最后一次失败调用其他方法?

2024/1/27 16:19:54

在使用 reactive http 请求时，可以通过使用 retryWhen 操作符来控制失败重试的行为。例如，你可以使用以下代码片段来实现最多重试 2 次的请求：

java
httpClient.get()
        .uri("http://example.com")
        .retrieve()
        .bodyToMono(String.class)
        .retryWhen(Retry.backoff(2, Duration.ofSeconds(1)))
        .subscribe(
            result -> { /* 处理成功的结果 */ },
            error -> {
                // 最后一次失败时调用其他方法
                otherMethod();
            }
        );
在上面的代码中，我们使用了 retryWhen 操作符并传入了一个 Retry 对象，它表示最多重试 2 次并且每次重试的时间间隔是 1 秒。如果最后一次重试仍然失败，就会触发 error 分支，并在其中调用名为 otherMethod 的其他方法。

需要注意的是，retryWhen 操作符并不是在每次请求失败后都会立即重试，而是在上一次请求完成后才会根据重试策略决定是否重试。因此，如果你需要在最后一次失败后立即执行其他操作，可以在 error 分支中进行处理。




2024/1/27 16:39:28

reactive官方文档和github示例有哪些

2024/1/27 16:39:33

Reactive 官方文档和 GitHub 示例提供了丰富的资源来学习和理解 Reactive 编程以及使用 Reactor 库进行开发。下面是一些官方文档和示例的资源：

Reactor 官方文档：Reactor 官方文档提供了全面的介绍和参考，包括核心概念、操作符、调度器等。你可以在以下链接中找到官方文档：

Reactor Core Reference Guide
Reactor Extra Reference Guide
Reactor GitHub 示例：Reactor 团队维护了一个 GitHub 仓库，其中包含各种示例代码和应用程序，用于演示 Reactor 的用法和最佳实践。你可以在以下链接中找到这些示例：

Reactor Examples
Reactor GitHub Organization
Spring WebFlux 官方文档和示例：如果你使用 Spring 框架进行开发，可以查看 Spring WebFlux 官方文档和示例，了解在 Web 开发中如何使用 Reactive 编程模型。以下是相关资源链接：

Spring WebFlux Documentation
Spring WebFlux GitHub 示例
以上资源将帮助你深入了解 Reactive 编程和 Reactor 库的使用。你可以根据自己的需求选择适合的资源进行学习和实践。




2024/1/27 22:06:21

如何对webflux的webclient的retry进行单元测试

2024/1/27 22:06:31

要对 WebFlux WebClient 的重试逻辑进行单元测试，你可以使用 StepVerifier 工具来模拟请求和响应，并验证重试的行为和结果。下面是一个示例代码片段：

java
import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.client.ClientResponse;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

public class WebClientRetryTest {

    @Test
    void testWebClientRetry() {
        WebClient webClient = WebClient.builder().build();
        int maxRetries = 2;

        StepVerifier.create(
            webClient.get()
                     .uri("http://example.com")
                     .retrieve()
                     .onStatus(HttpStatus::is5xxServerError, response -> Mono.error(new RuntimeException()))
                     .bodyToMono(String.class)
                     .retry(maxRetries)
        )
        .expectNext("Hello World")
        .verifyComplete();
    }
}
在上面的代码片段中，我们使用 WebClient 发起一个 GET 请求，并且设置了最多重试 2 次的机制。onStatus 方法用来定义当服务器返回 5xx 错误码时触发错误。我们使用 StepVerifier 来验证请求的结果，并且期望返回 "Hello World" 的字符串。

如果你需要测试重试机制的行为，可以模拟服务器返回 5xx 错误码，例如修改 URI 为一个不存在的地址。同时，你可以将 maxRetries 的值设置为 0 或 1，来验证重试行为是否符合预期。

需要注意的是，WebClient 的重试机制是基于 Reactive 库实现的，因此在测试时需要使用 Reactive 测试工具来验证结果。




2024/1/27 22:11:42

如何模拟服务器配合上个问题的单元测试

2024/1/27 22:12:09

要模拟服务器来配合上个问题的单元测试，可以使用 WireMock 库来搭建一个虚拟的 HTTP 服务器，并配置它返回指定的响应。下面是一个示例代码片段，演示如何使用 WireMock 来模拟服务器：

首先，需要添加 WireMock 的依赖到项目中（如果使用 Maven）：

xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.31.0</version>
    <scope>test</scope>
</dependency>
然后，在单元测试类中，可以使用 WireMockServer 类启动和停止 WireMock 服务器，并配置它的行为。以下是一个示例代码片段：

java
import static com.github.tomakehurst.wiremock.client.WireMock.*;
import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.client.ClientResponse;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

import com.github.tomakehurst.wiremock.WireMockServer;
import com.github.tomakehurst.wiremock.client.WireMock;

public class WebClientRetryTest {

    private WireMockServer wireMockServer;
    private WebClient webClient;

    @BeforeEach
    void setup() {
        wireMockServer = new WireMockServer(wireMockConfig().port(8080));
        wireMockServer.start();
        WireMock.configureFor("localhost", wireMockServer.port());
        
        webClient = WebClient.builder().baseUrl("http://localhost:" + wireMockServer.port()).build();
    }

    @AfterEach
    void teardown() {
        wireMockServer.stop();
    }

    @Test
    void testWebClientRetry() {
        int maxRetries = 2;

        // 模拟服务器返回 5xx 错误码
        stubFor(get(urlEqualTo("/"))
            .willReturn(aResponse()
                .withStatus(500)
                .withHeader("Content-Type", MediaType.TEXT_PLAIN_VALUE)
                .withBody("Internal Server Error")));

        StepVerifier.create(
                webClient.get()
                         .uri("/")
                         .retrieve()
                         .onStatus(HttpStatus::is5xxServerError, response -> Mono.error(new RuntimeException()))
                         .bodyToMono(String.class)
                         .retry(maxRetries)
        )
        .expectNext("Hello World")
        .verifyComplete();
    }
}
在上面的代码片段中，我们首先创建了一个 WireMockServer 对象，并在 setup() 方法中启动服务器。然后，使用 WireMock.configureFor() 方法来配置 WebClient 的基本URL为 WireMock 服务器的地址。

接下来，在 testWebClientRetry() 方法中，我们使用 stubFor() 方法来定义当请求匹配 / URI 时，返回一个带有 500 错误码和 "Internal Server Error" 响应。

最后，在测试用例中，我们通过 WebClient 发起请求，模拟服务器返回 5xx 错误码，并验证重试机制是否按预期工作。

需要注意的是，在测试结束后，需要在 teardown() 方法中停止 WireMock 服务器，以释放端口和资源。

通过使用 WireMock 来模拟服务器，可以灵活地配置虚拟的响应，以便测试 WebFlux WebClient 的重试行为。




2024/1/27 22:23:17

异步线程发生的请求，如何测试

2024/1/27 22:23:24

要测试异步线程发生的请求，可以使用异步测试工具来验证请求的行为和结果。在 Java 中，可以使用 JUnit 5 的 CompletableFuture 和 Assertions 来进行异步测试。下面是一个示例代码片段：

java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

import static org.junit.jupiter.api.Assertions.assertEquals;

@ExtendWith(SpringExtension.class)
public class AsyncRequestTest {

    @Test
    void testAsyncRequest() throws InterruptedException, ExecutionException {
        CompletableFuture<String> future = new CompletableFuture<>();

        // 模拟异步请求
        new Thread(() -> {
            // 发起异步请求
            String result = makeAsyncRequest();
            
            // 将结果设置到 CompletableFuture 中
            future.complete(result);
        }).start();

        // 等待异步请求完成
        String response = future.get();

        // 验证结果
        assertEquals("Hello World", response);
    }

    private String makeAsyncRequest() {
        // 这里可以编写异步请求的逻辑，例如使用 WebClient 发起异步请求
        // 返回异步请求的结果
        return "Hello World";
    }
}
在上面的代码片段中，我们首先创建了一个 CompletableFuture 对象 future，用于保存异步请求的结果。然后，我们在一个新的线程中模拟异步请求，并将请求的结果设置到 CompletableFuture 中。

之后，我们使用 future.get() 方法等待异步请求完成，并获取请求的结果。最后，我们使用 assertEquals 来验证请求的结果是否符合预期。

需要注意的是，异步测试可能会导致测试用例的执行时间变长，因为需要等待异步请求完成。为了确保测试的稳定性和可靠性，可以设置适当的超时时间，以避免测试用例无限期地阻塞。

通过使用 CompletableFuture 和 Assertions，可以方便地进行异步请求的测试，并验证请求的行为和结果。




2024/1/27 22:24:09

测试方法是添加异步线程池任务

2024/1/27 22:24:22

要测试添加异步线程池任务的方法，可以使用多线程测试工具和断言来验证方法的行为和结果。以下是一个示例代码片段：

java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

import static org.junit.jupiter.api.Assertions.assertTrue;

@ExtendWith(SpringExtension.class)
public class AsyncThreadPoolTest {

    @Test
    void testAddTaskToThreadPool() throws InterruptedException, ExecutionException {
        AsyncThreadPool asyncThreadPool = new AsyncThreadPool();

        // 创建 CountDownLatch 用于等待所有任务完成
        CountDownLatch countDownLatch = new CountDownLatch(3);

        // 添加三个异步任务
        CompletableFuture<Void> future1 = asyncThreadPool.addTask(() -> task(countDownLatch));
        CompletableFuture<Void> future2 = asyncThreadPool.addTask(() -> task(countDownLatch));
        CompletableFuture<Void> future3 = asyncThreadPool.addTask(() -> task(countDownLatch));

        // 等待所有任务完成或超时
        boolean completed = countDownLatch.await(5, TimeUnit.SECONDS);

        // 验证所有任务都已完成
        assertTrue(completed);

        // 验证所有任务都未发生异常
        assertTrue(future1.isDone() && !future1.isCompletedExceptionally());
        assertTrue(future2.isDone() && !future2.isCompletedExceptionally());
        assertTrue(future3.isDone() && !future3.isCompletedExceptionally());
    }

    private void task(CountDownLatch countDownLatch) {
        // 这里可以编写异步任务的逻辑，例如模拟执行耗时的操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        countDownLatch.countDown();
    }
}
在上面的代码片段中，我们首先创建了一个 AsyncThreadPool 对象 asyncThreadPool，用于添加异步任务。然后，我们创建了一个 CountDownLatch 对象，用于等待所有任务完成。

接下来，我们使用 asyncThreadPool.addTask() 方法添加三个异步任务，并将它们保存到 CompletableFuture 对象中。在每个异步任务中，我们模拟执行耗时的操作，并使用 CountDownLatch 来通知主线程任务已完成。

之后，我们使用 countDownLatch.await() 方法等待所有任务完成或超时。最后，我们使用 assertTrue 断言来验证所有任务都已完成，并且都未发生异常。

需要注意的是，如果异步任务发生了异常，CompletableFuture 对象的状态将变为 completed exceptionally，因此需要使用 isCompletedExceptionally() 方法来检查是否出现了异常。

通过使用多线程测试工具和断言，可以方便地验证添加异步线程池任务的方法的行为和结果。


