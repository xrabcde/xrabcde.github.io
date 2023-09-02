---
layout: post
title: Spring/동시성 문제 해결을 위한 Redis 분산락 구현하기
date: 2023-09-02
excerpt: "Java/Kotlin + Spring 환경에서 동시성 이슈 테스트 방법과 Redis 분산락을 통한 해결과정에 대해 알아보자."
tags: [java, kotlin, spring, redis]
comments: true
---

## 동시성 문제란?
여러 스레드가 자원을 공유하고 있는 상황에서 동시에 여러 스레드가 특정 자원에 접근해 데이터 정합성이 깨지는 상황을 **동시성 문제**가 발생했다고 한다. 동시성 이슈는 특정 시간에 트래픽이 몰릴 수 있는 상황에 가장 빈번히 발생하며, 트래픽이 몰리지 않는 상황에서도 같은 요청이 거의 동시에 온 경우(일명 '따닥 이슈'라고도 함) 발생할 수 있다. 

동시성 문제를 해결하기 위한 방법으로는 
1. 어플리케이션 코드 단에서 제어하기 (synchronized, atmoic 등)
2. DB 락을 활용하기
3. Redis를 활용해 분산락 구현하기  

등의 방법이 있지만 이번 글에서는 다양한 DB를 사용하고 있는 환경에서 적용할 수 있으면서 성능에 영향도 가장 적은 Redis 분산락에 대해 다뤄보려고 한다. 각각의 방법에 대한 설명과 장단점은 [이 글](https://dev-alxndr.tistory.com/45)에 자세히 정리되어 있다.

## 분산락이란?
분산락(Distributed lock)이란 단일 서버가 아닌 **여러 서버를 가용중인 상황(분산 환경)에서 공유된 데이터를 제어하기 위해 사용하는 기술**이다. 데이터베이스에서 제공하는 락 기능(낙관/비관 락)과는 다르니 헷갈리지 않도록 주의하자. 데이터베이스의 특정 데이터에 대한 접근이 동시에 일어나는 경우는 낙관/비관 락을 활용해 제어할 수 있지만, 생성 개수가 제한되어있는 상황에서 트래픽이 몰리는 경우와 같이 제어해야하는 데이터가 존재하지 않는 상황이라면 이 방법으로 해결할 수 없다. 

여러 서버에서 락에 대한 정보를 공통적으로 바라보아야 하기 때문에 이 정보를 관리할 장소로 Redis를 활용할 수 있다. 각 서버들은 공통된 장소인 Redis를 통해 임계영역에 접근할 수 있는지를 확인하고, 이를 통해 분산 환경에서 원자성을 보장할 수 있다. 이 글에서는 간단히 동시성 이슈가 발생하는 상황을 가정하고, JMeter를 통해 동시 요청을 테스트 해본 뒤 Redis를 활용하여 이 문제 상황을 해결해보려고 한다.

## 테스트 환경세팅
- Apple Silicon (M1)
- Kotlin, Spring Boot 3.1
- MySQL, Redis
- JMeter

1) 새로운 Spring Boot 프로젝트를 생성한다. 

- Spring Web, Spring Data Redis, Spring Data Jpa, MySQL Driver 의존성 추가

2) 티켓 발행개수가 선착순 30명 제한이 있는 간단한 로직을 작성한다.

- TicketController
```kotlin
@RestController
@RequestMapping("/tickets")
class TicketController(
    private val ticketService: TicketService
) {
    private val logger = LoggerFactory.getLogger(Logger::class.java)

    @PostMapping
    fun create(): ResponseEntity<String> {
        val result = ticketService.createTicket()
        logger.info(result)
        return ResponseEntity.ok(result)
    }
}
```

- TicketService
```kotlin
@Service
class TicketService(
    private val ticketRepository: TicketRepository
) {
    @Transactional
    fun createTicket() = when {
        ticketRepository.count() < 30 -> {
            val ticket = ticketRepository.save(Ticket())
            "SUCCESS! TICKET_NO : ${ticket.id}"
        }
        else -> "SOLD OUT"
    }
}
```

- TicketRepository
```kotlin
@Repository
interface TicketRepository: JpaRepository<Ticket, Long>
```

- Ticket
```kotlin
@Entity
class Ticket {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id = 0L
}
```

3) 정상적으로 수행했을 때의 기대동작을 확인한다.

- 1개의 포트만 사용해서 어플리케이션을 한 개만 띄우고, 포스트맨을 이용해 수동으로 30번의 요청을 보내보았다.
- 티켓이 30개까지 순차적으로 발행된 뒤, 이후부터는 발행되지 않는다.

<div style="width:50% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/b92d844e-c7d8-4a37-96ce-2ac0efa53abb">
</div>

4) 실제 분산 환경과 유사하게 만들어 문제동작을 확인한다.

- 2개의 포트를 사용해서 어플리케이션을 2대 띄우고, JMeter를 활용해 동시요청을 보내보자.  
Spring boot 어플리케이션을 두 개의 다른 포트에서 동시에 실행하려면, `application.properties` 를 이용해 서로 다른 프로파일을 사용한 인스턴스를 실행해도 되고, 간단하게는 터미널에서 포트를 지정해 동시에 서버를 띄우면 된다.

```bash
$ ./gradlew bootRun --args='--server.port=8080'
$ ./gradlew bootRun --args='--server.port=8081'
```

- 아래와 같이 터미널 창을 원하는 서버대수만큼 띄워 서로 다른 포트로 지정해주면 된다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/cc840d3a-5647-408e-bb26-fb74accde55c">
</div>

- brew를 이용해 JMeter를 설치하고 (윈도우라면 [링크](https://jmeter.apache.org/download_jmeter.cgi)에서 다운로드) 실행해준다.

```bash
$ brew install jmeter // 설치 명령어
$ jmeter              // 실행 명령어
```

- JMeter를 실행하고 'File > New > Test Plan' 선택한 뒤, 왼쪽 플랜 창에서 우클릭 후 'Add > Threads(Users) > Thread Goup'을 선택해 새로운 Thread Group을 추가한다.

<div style="width:90% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/5dad0283-a9dd-4285-bc13-b420d5303cd7">
</div>

- Thread Group 설정에서 Number of Threads 에 원하는 유저수를, Loop Count에 원하는 횟수를 적어주면 된다.

<div style="width:90% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/087d0cf9-6ecf-477f-93a7-f1220971344d">
</div>

- 테스트할 요청 정보를 설정하기 위해 'Add > Sampler > HTTP Request' 를 선택한다.

<div style="width:90% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/96eacb0e-88b2-499a-ab45-5a3035ea6b71">
</div>

- Server IP 정보와, 포트번호, 하단에 API 요청 정보를 입력한다. 나의 경우, 2가지 포트를 사용해 서버를 띄워줬기 때문에 2가지 포트에 대해 HTTP Request를 각각 생성해줬다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/c72abd09-88c5-4471-9236-9c7285897da6">
</div>

- 실행 후 결과가 보고싶다면 'Add > Listener > Summary Reposrt' 를 추가해주고 상단의 초록색 start 버튼을 누르면 테스트가 시작된다.

<div style="width:60% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/2515eea1-2e2d-4c0b-98ad-9cc30cf303c4">
</div>

- 실행 결과, 선착순 30개까지만 발행되어야 하는 티켓이 35개까지 발행된 것을 확인할 수 있다. 우려했던 동시성 문제가 발생한 것이다. 이렇게 서버가 여러 대인 분산환경에서는 어플리케이션 코드 단에서 강제로 동기화를 걸어줌으로써 해결하려는 것은 의미가 없다. Redis의 분산락을 구현해 문제를 해결해보자.

<div style="width:50% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/7531e5c1-45f9-4058-9db1-3ac6791c3e49">
</div>
<br>
<div style="width:80% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/e7cff729-69f6-47f6-9caa-b8030ba6c892">
</div>

## Redis 세팅
- brew를 이용해 Redis를 설치하고 (윈도우라면 [링크](https://github.com/microsoftarchive/redis)를 통해 다운로드) 실행해준다. 아래 그림과 같이 나오면 Redis가 정상적으로 실행된 것이다. Redis의 기본 포트번호는 6379이다.

```bash
$ brew install redis  // 설치 명령어
$ redis-server        // 실행 명령어
```

<div style="width:90% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/4d9cbd82-4765-4832-a2ee-c8eacfdb1a0a">
</div>

- Spring 프로젝트 `application.properties`에 redis 설정과 Redis Config를 작성해준다.

```properties
spring.data.redis.host=localhost
spring.data.redis.port=6379
```

- RedisConfig

```kotlin
@Configuration
class RedisConfig(
    @Value("\${spring.data.redis.host}")
    val redisHost: String,

    @Value("\${spring.data.redis.port}")
    val redisPort: Int
) {

    @Bean
    fun redisConnectionFactory() = LettuceConnectionFactory(redisHost, redisPort)

    @Bean
    fun redisTemplate(): RedisTemplate<String, String> {
        val redisTemplate = RedisTemplate<String, String>()
        redisTemplate.connectionFactory = redisConnectionFactory()
        return redisTemplate
    }

    @Bean
    fun redissonClient(): RedissonClient {
        val config = Config()
        config.useSingleServer().address = "redis://" + redisHost + ":" + redisPort
        return Redisson.create(config)
    }
}
```

## 방법 1: Lettuce와 SETNX를 이용한 스핀 락
- 첫 번째로 Redis Client 중 Lettuce를 이용해 분산락을 구현해보자. 이 방식은 스레드들이 스핀락을 활용해서 순차적으로 락을 획득하게 된다. 스핀락이란 락을 획득할 때까지 무한루프를 돌며 계속 락을 얻기 위해 요청을 보내는 동기화 기법이다. Spring 프로젝트에 아래 의존성을 추가해준다.

```bash
implementation('org.springframework.boot:spring-boot-starter-cache')
```

- 락에 관한 정보를 Redis 에 저장하기 위해 SETNX 명령어를 이용할 수 있다. SETNX는 해당 키 값이 존재하지 않으면 값을 세팅하는 명령어이며, 값이 세팅되었는지 여부를 반환한다. (Redis GUI 툴은 Another Redis Desktop Manager를 사용했다.)

<div style="width:70% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/8d6be210-b5e9-4e36-a6bc-8134187b54c2">
</div>

- 이제 이 원리를 이용해서 Spring 코드에 루프를 돌며 락을 획득하는 코드를 작성해보자.

- RedisLockRepository

```kotlin
@Component
class RedisLockRepository(
    private val redisTemplate: RedisTemplate<String, String>,
) {
    fun lock(key: String) = redisTemplate.opsForValue()
        .setIfAbsent(key, "true", Duration.ofMillis(3_000))

    fun unlock(key: String) = redisTemplate.delete(key)
}
```

- TicketService

```kotlin
@Service
class TicketService(
    private val ticketRepository: TicketRepository,
    private val redisLockRepository: RedisLockRepository
) {
    fun createTicketWithSpinLock(): String {
        while (!redisLockRepository.lock("lock")!!) {
            Thread.sleep(100) // 락을 획득할 때까지 대기
        }

        return try {
            when {
                ticketRepository.count() < 30 -> {
                    val ticket = ticketRepository.save(Ticket())
                    "SUCCESS! TICKET_NO : ${ticket.id}"
                }
                else -> "SOLD OUT"
            }
        } finally {
            redisLockRepository.unlock("lock")
        }
    }
}
```

- 이렇게 Redis SETNX 명령어를 통해 Redis에 락 정보를 관리하도록 하고 JMeter를 통해 서버 2대에 아까와 똑같은 요청을 보내보았더니 정확히 30개까지의 티켓만 발행된 것을 확인할 수 있었다. 하지만, 이 방법은 락을 획득할 때까지 계속해서 레디스 서버에게 요청을 날리기 때문에 레디스 서버에 부하를 줄 수 있다.

<div style="width:50% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/ff4c1187-eda1-4164-a409-57f69de89693">
</div>

## 방법 2: Redisson을 활용한 pubsub
- 다음으로는 스핀락을 사용하지 않고 Redisson을 활용해서 분산락을 구현해보자. 이 방법은 pubsub 방식으로 동작하기 때문에 락을 획득할 때까지 레디스에게 끊임없이 요청을 보내 부하를 주는 것이 아니라, 락을 해제하는 쪽에서 대기하는 프로세스에게 신호를 주는 방식으로 동작한다. Redisson 라이브러리에 이 로직이 구현이 되어있기 때문에 따로 구현할 필요는 없다. Spring 프로젝트에 아래 의존성을 추가해준다.

```bash
implementation("org.redisson:redisson-spring-boot-starter:3.17.7")
```

- Redis에서 제공하는 PUBLISH와 SUBSCRIBE 명령어를 통해 대기하는 채널에 메시지를 발행하는 로직을 구현할 수 있다. 아래와 같이 `subscribe [채널명]` 을 입력하면 특정 채널을 구독할 수 있고 `publish [채널명] [메시지]` 를 입력하면 해당 채널을 구독중인 프로세스들에게 메시지가 발행된다. 

<div style="width:70% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/567c7ad8-67a5-4863-8d42-9b13ea068a1e">
</div>

<div style="width:70% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/f82ef15f-084d-4f38-9526-1a904778acb9">
</div>

<div style="width:70% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/f3f1d7b3-c84d-4a8d-93a9-9239d5f34f28">
</div>

- TicketService

```kotlin
@Service
class TicketService(
    private val ticketRepository: TicketRepository,
    private val redissonClient: RedissonClient
) {
    @Transactional
    fun createTicketWithRedisson(): String {
        val lock = redissonClient.getLock("lock")

        try {
            val available = lock.tryLock(100, 2, TimeUnit.SECONDS)
            if (!available) {
                throw RuntimeException("Failed to get lock")
            }

            return if (ticketRepository.count() < 30) {
                val ticket = ticketRepository.save(Ticket())
                "SUCCESS! TICKET_NO : ${ticket.id}"
            } else {
                "SOLD OUT"
            }
        } catch (e: InterruptedException) {
            throw RuntimeException(e)
        } finally {
            lock.unlock()
        }
    }
}

```

- Redisson 라이브러리를 사용하면 레디스 서버에 부하를 주는 while 문 대신에 락 획득이 가능하다는 신호가 올 때까지 대기하는 방식으로 동작한다. getLock()을 통해 락을 획득할 수 있고 tryLock()을 통해 락 획득을 시도한다. 이번에도 같은 방법으로 JMeter를 통해 서버 2대에 동일한 요청을 보내보았더니 정확히 30개까지의 티켓만 발행된 것을 확인할 수 있었다. 

<div style="width:50% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/ff4c1187-eda1-4164-a409-57f69de89693">
</div>

## 결론
동시성 이슈는 실제로 현업에서 빈번히 발생한다. QA 과정에서 발견된다고 해도 재현이 어렵기 때문에 병목지점을 파악하고 해결하기가 쉽지 않다. 이직 면접에서 단골 질문으로 등장하는 키워드인 만큼 한 번쯤은 직접 동시성 이슈가 발생하는 상황을 만들어보고 해결해보면 실제 문제가 발생했을 때 더 유연하게 대처할 수 있겠다. 이 글에서 사용한 예제코드는 [이 레포지토리](에서 확인할 수 있다. 

## References
- [분산락 테스트 소스코드 Github](github.com)
- [Redis 공식문서 - distributed locks](https://redis.io/docs/manual/patterns/distributed-locks/)
- [동시성 이슈를 해결하는 여러가지 방법](https://dev-alxndr.tistory.com/45)
- [Redis 분산락(Distribution Lock)을 구현해 다중화 서버에서 발생하는 동시성 문제 제어하기](https://velog.io/@msung99/Redis-%EB%B6%84%EC%82%B0-%EB%9D%BD%EC%9D%84-%EA%B5%AC%ED%98%84%ED%95%B4-race-condition-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)
- [Redis로 분산 락을 구현해 동시성 이슈를 해결해보자](https://hudi.blog/distributed-lock-with-redis/)
- [JMeter 사용법](https://effortguy.tistory.com/164)