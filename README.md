# RabbitMQ

### O que é RabbitMQ?

RabbitMQ é um software de Message Broker amplamente utilizado que facilita a comunicação entre diferentes partes de um sistema distribuído. Ele atua como intermediário de mensagens, permitindo que os aplicativos enviem e recebam dados de forma assíncrona através de filas de mensagens.

Desenvolvido inicialmente pela Rabbit Technologies em 2007, o RabbitMQ é baseado no protocolo AMQP (Advanced Message Queuing Protocol), mas também suporta outros protocolos como STOMP, MQTT, e HTTP, proporcionando flexibilidade e integração com diferentes tipos de sistemas.

### Vantagens do RabbitMQ

1. **Escalabilidade e Flexibilidade**: RabbitMQ pode ser configurado em um cluster de servidores para garantir alta disponibilidade e distribuição de carga, permitindo a escalabilidade horizontal.
2. **Suporte a Múltiplos Protocolos**: Além do AMQP, RabbitMQ suporta outros protocolos como MQTT e STOMP, o que facilita a integração com diversos tipos de aplicações.
3. **Alta Disponibilidade**: Oferece mecanismos para replicação de mensagens e failover, garantindo que as mensagens não sejam perdidas em caso de falhas no servidor.
4. **Mensagens Persistentes**: RabbitMQ permite a persistência de mensagens, garantindo que elas sejam salvas no disco e não perdidas em caso de falhas.
5. **Mecanismos de Roteamento**: Com recursos como Exchanges, é possível roteamento avançado de mensagens, permitindo distribuir mensagens para diferentes filas com base em regras específicas.
6. **Comunicação Assíncrona**: RabbitMQ facilita a comunicação assíncrona entre serviços, o que é essencial para sistemas distribuídos e microservices.

### Desvantagens do RabbitMQ

1. **Complexidade de Configuração**: Configurar RabbitMQ pode ser complexo, especialmente em ambientes de alta disponibilidade com clusters e replicação de dados.
2. **Latência**: Em comparação com outros brokers de mensagens, RabbitMQ pode introduzir mais latência, especialmente em cenários de alta carga ou quando configurado para garantir alta durabilidade das mensagens.
3. **Gerenciamento de Filas**: O gerenciamento de filas pode se tornar complexo à medida que o número de filas e a quantidade de mensagens aumentam.
4. **Sobrecarga Operacional**: Requer monitoramento e manutenção contínuos, especialmente em ambientes críticos onde a perda de mensagens não é tolerada.
5. **Curva de Aprendizado**: Pode ter uma curva de aprendizado íngreme para novos usuários, especialmente para aqueles que não estão familiarizados com o conceito de message brokers.

### **Tipos de exchange**

### 1. **Direct Exchange**

- **Roteamento**: Baseado em uma correspondência exata entre a chave de roteamento (`routing key`) da mensagem e a chave associada à fila.
- **Uso**: Quando se deseja que a mensagem vá para uma fila específica, como logs categorizados por nível de severidade.

### 2. **Fanout Exchange**

- **Roteamento**: Envia a mensagem para todas as filas vinculadas, independentemente da chave de roteamento.
- **Uso**: Ideal para broadcasting, onde todas as filas recebem a mesma mensagem, como em notificações ou eventos globais.

### 3. **Topic Exchange**

- **Roteamento**: Baseado em padrões de correspondência com a chave de roteamento, usando curingas ('*' para uma palavra, '#' para zero ou mais palavras).
- **Uso**: Para roteamento flexível e dinâmico, como no encaminhamento de mensagens em sistemas com tópicos ou categorias.

### 4. **Headers Exchange**

- **Roteamento**: Baseado nos headers das mensagens em vez da chave de roteamento.
- **Uso**: Para casos onde o roteamento depende de múltiplos critérios ou atributos específicos das mensagens.

### Docker Compose para inicializar servidor RabbitMQ:

```yaml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq_server
    ports:
      - "5672:5672"    # Porta do RabbitMQ
      - "15672:15672"  # Porta do Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq    # Persistência de dados
    environment:
      RABBITMQ_DEFAULT_USER: rabbitmquser # Usuário padrão
      RABBITMQ_DEFAULT_PASS: rabbitmqpassword # Senha padrão
    command: >
      sh -c "rabbitmq-plugins enable --offline rabbitmq_management &&
             rabbitmq-plugins enable --offline rabbitmq_shovel &&
             rabbitmq-plugins enable --offline rabbitmq_shovel_management &&
             rabbitmq-plugins enable --offline rabbitmq_prometheus &&
             rabbitmq-server"
    restart: unless-stopped

volumes:
  rabbitmq_data:

```

### Passos para Utilizar

1. **Instalar Docker e Docker Compose**:
    - Certifique-se de que o Docker e o Docker Compose estão instalados no seu sistema.
2. **Executar o Docker Compose**:
    - Navegue até o diretório onde o arquivo `docker-compose.yml` está localizado.
    - Execute o comando `docker-compose up -d` para iniciar o serviço RabbitMQ em segundo plano.
3. **Acessar a Interface de Gerenciamento**:
    - Abra o navegador e acesse http://localhost:15672.
    - Faça login com o usuário e senha definidos (`user` e `password` neste exemplo).
    - Agora, você deve ver as opções para mover mensagens entre filas diretamente pela interface.

### Como Implementar o RabbitMQ

1. **Implementação em um Sistema Spring Boot**
- Adicione as dependências no `pom.xml`:
    
    ```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    
    ```
    
- Configure as propriedades do RabbitMQ no `application.properties`:
    
    ```
    spring.rabbitmq.host=localhost
    spring.rabbitmq.port=5672
    spring.rabbitmq.username=guest
    spring.rabbitmq.password=guest
    
    ```
    
- Crie uma classe de configuração:
    
    ```java
    @Configuration
    public class RabbitConfig {
        @Bean
        public Queue myQueue() {
            return new Queue("myQueue", true);
        }
        @Bean
        public TopicExchange exchange() {
            return new TopicExchange("myExchange");
        }
        @Bean
        public Binding binding(Queue queue, TopicExchange exchange) {
            return BindingBuilder.bind(queue).to(exchange).with("routing.key");
        }
    }
    
    ```
    
- Crie um listener para consumir as mensagens:
    
    ```java
    @Service
    public class MessageListener {
        @RabbitListener(queues = "myQueue")
        public void listen(String message) {
            System.out.println("Received: " + message);
        }
    }
    
    ```
    
- **Monitoramento e Manutenção**
    - Use o painel de administração do RabbitMQ para monitorar filas, exchanges e consumidores.
    - Configure alertas para filas não processadas ou para mensagens não entregues.

### Como Implementar o reprocessamento automático com RabbitMQ

### 1. **Dependências no `pom.xml`**

Adicione as seguintes dependências ao seu projeto Spring Boot:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>

```

### 2. **Configuração do RabbitMQ**

Crie uma classe de configuração para o RabbitMQ:

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

    public static final String ORIGINAL_QUEUE = "originalQueue";
    public static final String DEAD_LETTER_QUEUE = "deadLetterQueue";
    public static final String EXCHANGE = "exchange";
    public static final String DEAD_LETTER_EXCHANGE = "deadLetterExchange";

    @Bean
    public Queue originalQueue() {
        return QueueBuilder.durable(ORIGINAL_QUEUE)
                .withArgument("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE)
                .withArgument("x-dead-letter-routing-key", DEAD_LETTER_QUEUE)
                .build();
    }

    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
    }

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE);
    }

    @Bean
    public TopicExchange deadLetterExchange() {
        return new TopicExchange(DEAD_LETTER_EXCHANGE);
    }

    @Bean
    public Binding bindingOriginalQueue() {
        return BindingBuilder.bind(originalQueue()).to(exchange())
        .with(ORIGINAL_QUEUE);
    }

    @Bean
    public Binding bindingDeadLetterQueue() {
        return BindingBuilder.bind(deadLetterQueue()).to(deadLetterExchange())
        .with(DEAD_LETTER_QUEUE);
    }
}

```

### 3. **Reprocessamento com Retry e Dead Letter Queue**

Crie um listener que tentará reprocessar as mensagens. Caso todas as tentativas falhem, a mensagem será movida para a Dead Letter Queue:

```java
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.retry.RepublishMessageRecoverer;
import org.springframework.amqp.rabbit.support.ListenerExecutionFailedException;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Service;

@Service
public class MessageListener {

    private final RabbitTemplate rabbitTemplate;

    public MessageListener(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    @RabbitListener(queues = RabbitConfig.ORIGINAL_QUEUE)
    public void receiveMessage(String message, @Header(AmqpHeaders.DELIVERY_TAG) 
    long tag) {
        try {
            // Processa a mensagem
            System.out.println("Processing message: " + message);
            if (someConditionFails()) {
                throw new RuntimeException("Failed to process message");
            }
        } catch (Exception e) {
            // Em caso de falha, a mensagem será movida para a Dead Letter Queue após 5 tentativas
            throw new ListenerExecutionFailedException("Retry failed for message", e, message);
        }
    }

    private boolean someConditionFails() {
        // Lógica para simular falha no processamento
        return true;
    }
}

```

### 4. **Configuração de Retry no `application.properties`**

Para definir o número de tentativas e o delay entre elas, configure o Spring Retry no `application.properties`:

```
spring.rabbitmq.listener.simple.retry.enabled=true
spring.rabbitmq.listener.simple.retry.max-attempts=5
spring.rabbitmq.listener.simple.retry.initial-interval=2000  # 2 segundos de delay
spring.rabbitmq.listener.simple.retry.multiplier=1.0
spring.rabbitmq.listener.simple.retry.max-interval=3000  # Máximo de 3 segundos de delay

```

### 5. **Descrição do Processo**

- **Original Queue**: É a fila onde as mensagens são enviadas inicialmente.
- **Dead Letter Queue**: Se uma mensagem não puder ser processada após todas as tentativas, ela é enviada para a Dead Letter Queue.
- **Retry Mechanism**: O Spring Retry é configurado para tentar reprocessar a mensagem até 5 vezes, com um delay de 2 a 3 segundos entre as tentativas.
- **Reprocessamento**: O listener tenta processar a mensagem. Se falhar, uma exceção é lançada e a mensagem é reencaminhada para a fila original. Após 5 tentativas, a mensagem é enviada para a Dead Letter Queue.
