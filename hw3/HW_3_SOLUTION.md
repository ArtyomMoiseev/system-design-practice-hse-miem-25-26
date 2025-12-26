## HW 3

### Наш кластер patroni из трёх реплик и одного лидера

<img width="723" height="155" alt="Screenshot 2025-12-26 at 17 47 55" src="https://github.com/user-attachments/assets/47889229-c603-4af8-a1db-12445163f9fe" />

### HAProxy

<img width="2103" height="868" alt="image" src="https://github.com/user-attachments/assets/904d88a4-e8b9-4b35-95c1-54942787d320" />

На скриншоте видны две активные реплики (patroni1, patroni3), и мастер patroni2

### Стреляем

<img width="1067" height="511" alt="Screenshot 2025-12-26 at 18 07 16" src="https://github.com/user-attachments/assets/e70cfeed-cd4b-4652-aaea-21fd9c743aaa" />

Инсерты данных работают. Операции записи выполняются на primary узле, читать можем с реплик.

### Тест отказоустойчивости

Если остановить один из контейнеров etcd - то ничего не случиться, 2 - получаем ошибки при инсертах данных, т.к. отсутствует кворум.

<img width="880" height="445" alt="Screenshot 2025-12-27 at 00 09 33" src="https://github.com/user-attachments/assets/ff9cbb3d-8b9f-4278-885b-030b21b9e388" />

Если восстановить работоспособность двух контейнеров etcd, то работоспособность кластера восстанавливается.

Также без потери работоспособности мы можем остановить реплики Patroni. Пример с двумя остановленными репликами:

<img width="1239" height="366" alt="image" src="https://github.com/user-attachments/assets/e7965dab-743b-46cc-8f49-63bf8dc27039" />

HAPproxy явлются единой точкой отказа, при остановке контейнера - теряем возможность работать с кластером:

<img width="880" height="445" alt="Screenshot 2025-12-27 at 00 10 33" src="https://github.com/user-attachments/assets/c9fb641e-930f-44b7-91f1-39829d649e38" />

При остановке primary узла Patrony, мы теряем возможности записи данных, но можем продолжать читать сохранённые:

<img width="635" height="451" alt="image" src="https://github.com/user-attachments/assets/9d5565f7-17a3-47ca-9500-55ce87f8c921" />
