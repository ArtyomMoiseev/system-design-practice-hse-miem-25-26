## HW2
### Запущенная Kafka

#### Брокеры
<img width="1578" height="611" alt="image" src="https://github.com/user-attachments/assets/a3bc57c4-2d8e-43fb-a2cd-eab9365fac28" />

#### Тестовый топик
<img width="1454" height="893" alt="image" src="https://github.com/user-attachments/assets/e3c86551-bc0d-4a37-a2cb-ceadc8c93961" />

### Основные параметры

Кластер из трех брокеров, в текущий момент времени все брокеры онлайн, у лидера идентификатор - 3. Мы создали топик с восьмью партициями и фактором реплицирования 3. Соответственно наш партиции реплицировались равномерно на три наших брокера.
Как минимум будет всё хорошо, если один из брокеров упадёт.

### Запускаем нагрузочное

<img width="1067" height="511" alt="image" src="https://github.com/user-attachments/assets/6d328a3b-cf6d-466b-838f-5ca3787de393" />
<img width="1578" height="940" alt="Screenshot 2025-12-26 at 14 17 21" src="https://github.com/user-attachments/assets/de57a2dc-1a97-476b-b1ee-e4f9d1234e66" />

При локальном запуске получил RPS 186532.4 records/sec, avg latency = 86.32 ms, max latency = 392 ms

Процентиле:
p50 = 18 ms, p95 = 279 ms, p99 = 325 ms, p99.9 = 379 ms. Для локального запуска - норм. p50 даже низкий - 18ms.

В UI видно, что все сообщения (1000000) дошли.

### Отказоустойчивость

Запустил скрипт - события в топик летят
<img width="1600" height="928" alt="image" src="https://github.com/user-attachments/assets/a13fccba-18a4-4bd9-8397-6dd30cfb388f" />

Останавливаем контейнер с контроллером:
<img width="1600" height="928" alt="Screenshot 2025-12-26 at 15 44 31" src="https://github.com/user-attachments/assets/5d7ac041-fbe9-474e-9f47-7e268e1a1eed" />
<img width="1600" height="928" alt="Screenshot 2025-12-26 at 15 44 23" src="https://github.com/user-attachments/assets/56009c28-292a-4ae6-b873-b795af9f5404" />

По итогу второй контейнер стал контроллером, сообщения продолжают лететь
<img width="1600" height="928" alt="Screenshot 2025-12-26 at 15 44 41" src="https://github.com/user-attachments/assets/1dbc2e8f-279f-4005-b4cb-cf945c455d48" />

Останавливаем второй контейнер:

Получаем следующие записи в логи первого контейнера:

```
[2025-12-26 10:46:11,279] INFO [RaftManager id=1] Election has timed out, backing off for 1000ms before becoming a candidate again (org.apache.kafka.raft.KafkaRaftClient)

[2025-12-26 10:46:11,332] INFO [BrokerLifecycleManager id=1] Unable to send a heartbeat because the RPC got timed out before it could be sent. (kafka.server.BrokerLifecycleManager)

[2025-12-26 10:46:12,284] INFO [RaftManager id=1] Re-elect as candidate after election backoff has completed (org.apache.kafka.raft.KafkaRaftClient)

[2025-12-26 10:46:12,290] INFO [RaftManager id=1] Completed transition to CandidateState(localId=1, localDirectoryId=nmzgoIIffh5JMG4KSuzfZQ,epoch=78, retries=13, voteStates={1=GRANTED, 2=UNRECORDED, 3=UNRECORDED}, highWatermark=Optional[LogOffsetMetadata(offset=64728, metadata=Optional.empty)], electionTimeoutMs=1199) from CandidateState(localId=1, localDirectoryId=nmzgoIIffh5JMG4KSuzfZQ,epoch=77, retries=12, voteStates={1=GRANTED, 2=UNRECORDED, 3=UNRECORDED}, highWatermark=Optional[LogOffsetMetadata(offset=64728, metadata=Optional.empty)], electionTimeoutMs=1527) (org.apache.kafka.raft.QuorumState)

[2025-12-26 10:46:12,291] WARN [RaftManager id=1] Error connecting to node kafka02:9093 (id: 2 rack: null) (org.apache.kafka.clients.NetworkClient)

java.net.UnknownHostException: kafka02

	at java.base/java.net.InetAddress$CachedLookup.get(InetAddress.java:998)

	at java.base/java.net.InetAddress.getAllByName0(InetAddress.java:1807)

	at java.base/java.net.InetAddress.getAllByName(InetAddress.java:1676)

	at org.apache.kafka.clients.DefaultHostResolver.resolve(DefaultHostResolver.java:27)

	at org.apache.kafka.clients.ClientUtils.resolve(ClientUtils.java:124)
```

Не можем подключиться к другим брокерам, и не можем провести выборы нового лидера кластера. Число реплик, меньше числа insync < 2. Мы перестаём принимать новые записи.

В логе приложения ловим ошибки:

<img width="1067" height="511" alt="image" src="https://github.com/user-attachments/assets/5059a36a-307a-4b40-a8ac-d8be30ca5705" />

При восстановлении брокеров, кластер переходит в нормальную работу.

<img width="1426" height="640" alt="image" src="https://github.com/user-attachments/assets/e07f78e3-654f-4642-846d-f9393709b6ed" />
<img width="1067" height="511" alt="image" src="https://github.com/user-attachments/assets/4a3aa7d5-e555-44d3-9571-4766f4b15c6c" />
<img width="1067" height="511" alt="image" src="https://github.com/user-attachments/assets/3165f48c-00be-419b-a28c-4dda0418c76b" />
