RabbitMQ
==

RabbitMQ 개요
--
### 1. RabbitMQ란?

> RabbitMQ는 MOM(Message-Oriented-Middleware)의 하나로 느슨하게 결합된 아키텍쳐를 지향하는 AMQP Message-Broker입니다.   
```
MOM은 분산시스템에서 메시지를 보내고 바을 수 있는 소프트웨어 또는 하드웨어 인프라를 말한다. RabbitMQ는 향상된 메시지 라우팅 및 분배기능을 제공하고
안정적인 분산 시스템을 지원하기 위해 광역 네트워크를 통해 다른 시스템과 손쉽게 연결할 수 있는 메시지 지향 미들웨어 중 하나다.
```

### 2. Loosely Coupled Architecture의 장점
```
부하분산을 제공하여 안정적인 성능을 낼 수 있도록 해 주며, 새로운 기능 추가 시 기존의 시스템과 의존성을 낮추게 해 줍니다.
```

### 3. AMQ 모델
> RabbitMQ의 강점과 유연성등은 대부분 AMQP(Advanced Message Queuing) 모델을 살펴보면 확인할 수 있다.   
> AMQ 모델은 메시지 라우팅 동작을 정의하는 메시지 브로커의 세 가지 추상 컴포넌트를 다음과 같이 논리적으로 정의한다.

* 익스체인지 : 메시지 브로커에서 큐에 메시지를 전달하는 컴포넌트
  * direct exchange
  * fanout exchange
  * topic exchange
  * headers exchange
* 큐 : 메시지를 저장하는 디스크상이나 메모리상의 자료구조
* 바인딩 : 익스체인지에 전달된 메시지가 어떤 큐에 저장돼야 하는지 정의하는 컴포넌트

### 4. AMQP를 통한 RabbitMQ
```
RabbitMQ는 AMQP 메지시브로커로 서버와 통신하는 거의 모든 부분에서 RPC(Remote Procedure Call) 패턴으로 엄격하게 통신한다. AMPQ 스펙에는 
클래스와 메소드를 사용하여 클라이언트와 서버 간의 공통 모델인 AMQP 명령이 정의돼 있다. RabbitMQ에서 AMQP 명령을 전송하거나 수신할 때 필요한 
인자들은 데이터 구조로 캡슐화된 프레임으로 인코딩돼 전송되며 다섯 개의 구성 요소로 이루어져있다.
```
* 프레임유형
* 프레임크기
* 프레임 페이로드
* 채널번호
* 끝 바이트 표식(ASCII값 206)
```
RabbitMQ를 이용해서 클라이언트 애플리케이션을 구현할 때는 복잡하게 너무 많은 채널을 사용하지 않는 것이 좋다고 한다. 각 채널마다 메모리 구조와
객체가 설정되어 있기 때문에 많을수록 메모리를 많이 사용하게 된다. 프레임유형에는 다섯가지 유형이 있다.
```
* Protocol Header Frame
* Method Frame
* Contetn Header Frame
* Body Frame
* Heartbeat Frame

### 5. 메시지 속성값
> Basic.Properties를 이해하여 RabbitMQ의 동작을 제어할 수 있습니다.
* content-type : application/json
* content-encoding : base64, gzip
* message-id, correlation-id
* timestamp
* exparation
* delivery-mode : 메모리저장 or 디스크저장(속도에 영향을 준다)
* app-id, user-id
* type
* reply-to
* headers
```
RabbitMQ의 다양한 용어와 설정에 대해 처음 접한다면, 메시지 속성(persistence)이 큐의 내구성(durable) 속성과 혼동될 수 있다.
큐의 durable 속성은 RabbitMQ 서버나 클러스터를 다시 시작한 후에도 큐 정의가 유지돼야 하는지를 나타내는 반면, delevery-mode는 메시지를 유지할지 여부를 나타낸다.
```

### 6. 메시지발행과 성능절충
* Publisher가 메시지를 디스크에 저장해야 하는가?
* 애플리케이션의 다양한 구성 요소는 발행된 메시지가 수신됐는지 보장해야 하는가?
* TCP back-pressure로 애플리케이션이 차단되거나 RabbitMQ에 메시지를 발행하는 동안 연결이 차단된 경우 어떻게 되는가?
* 메시지가 얼마나 중요한가? 메시지의 처리량을 높이기 위해 배달 보장을 희생할 수 있는가?
```
메시지 발행은 메시징 기반 아키텍처의 핵심 동작 중 하나로 RabbitMQ에는 메시지 발행에 대한 여러 설정이 있다. 애플리케이션에서 사용 가능한 다양한 메시지
발행 옵션은 애플리케이션의 성능과 안전성에 많은 영향을 준다. 메시지브로커는 빠른 성능이나 처리량도 중요하지만, 신뢰할 수 있는 메시지 전달도 중요한 지표 중 하나다.
```
* Goldilocks Principle (ex. trade off?)
* mandatory
* delevery-mode
* durable
* alternative exchange
* clustered HA queue
```
RabbitMQ와 원자 트랜젝션
원자성은 트랜잭션을 커밋하는 과정에서 트랜잭션의 모든 동작이 완료되는 것을 보장하는데, 이는 AMQP에서 트랜잭션의 모든 작업이 완료될 때까지 클라이언트가 TX.commitOK 응답 프레임을 받지 않는 다는 것을 의미한다. 불행하게도 RabbitMQ에 영향을 줄때만 원자 트랜잭션을 지원한다. 트랜잭션 명령이 둘 이상의 큐에 영향을 주면 커밋은 원자적으로 동작하지 않는다.

중략...

delevery-mode를 2로 설정해서 디스크에 저장해야 하는 메시지가 포함된 원자 트랙잭션은 Publisher의 성능에 문제를 일으킬 수도 있다. I/O가 부하가 많은 서버에서 RabbitMQ가 TX.CommitOK 프레임을 보내기 전에 쓰기가 완료될 때까지 기다리는 경우, 클라이언트는 트랜잭션으 ㄹ사용하지 않은 경우보다 오래 기다리게 된다.
```

> RabbitMQ 푸시백
> 3.2부터 AMQP 스팩을 확장해 연결에 대한 크래딧이 임계값에 도달했을 때 전동되는 알림을 추가하고 클라이언트에 연결이 차단됐다는 사실을 알린다.

### 7. 메시지 Consume
* 동기/비동기 (Basic.Get/Basic.Consume)
* no_ack=True
* linux socket buffer size
* prefetch size (no-ack가 설정되면 무시)
* Basic.Reject or Basic.Nack with redelivered flag
* dead_letter_exchange / x-dead-letter-exchange / x-dead-letter-routing-key
* auto-delete / x-expire
* durability


RabbitMQ 설치 및 설정
--
> 1.  사용할 버전에 대한 tar.xz 파일을 다운받는다. 
- 얼랭도 같이 설치한다.
  설치할 버전을 확인한다.brew, apt-get, port 등을 사용해서 편리하게 설치한다.https://github.com/erlang/otp/releases/tag/OTP-22.0 https://www.erlang.org/downloads
  
  https://www.rabbitmq.com/which-erlang.html
```
brew search erlang
brew install erlang@22
brew switch erlang 22.2.5
```

> 2. 압축을 풀고 원하는 디렉토리로 이동한다.

> 3. rabbitmq를 실행 및 종료한다.
```
rabbitmq_server/sbin/rabbitmq-server 명령어로 실행한다.
rabbitmq_server/sbin/rabbitmqctl shutdown 명령어로 실행한다.
```

> 4. rabbitmq 매니저 플러그인을 실행한다.
```
${HOME}/sbin/rabbitmq-plugins enable rabbitmq_management
* 별일 없으면 guest / guest 계정으로 로그인이 가능하다.
```

> 5. management UI에 로그인하기 위한 사용자를 추가한다.
```
${HOME}/sbin/rabbitmqctl add_user {username} {password}
```

> 6. 추가한 계정에 administrator 태그를 부여한다.(어드민 권한)
```
${HOME}/sbin/rabbitmqctl set_user_tags {username} administrator
```

> 7. 계정에 퍼미션을 추가한다. (UI 화면 메뉴가 잘 보인다.)
```
${HOME}/sbin/rabbitmqctl set_permissions -p / {username} ".*" ".*" ".*"
```




