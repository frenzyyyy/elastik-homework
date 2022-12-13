
<h1>
Домашнее задание к занятию "6.5. Elasticsearch"
</h1>

____

<h2>Задача 1</h2>
____

В этом задании вы потренируетесь в:

 - установке elasticsearch
 - первоначальном конфигурировании elastcisearch
 - запуске elasticsearch в docker

Используя докер образ centos:7 как базовый и документацию по установке и запуску Elastcisearch:

 - составьте Dockerfile-манифест для elasticsearch
 - соберите docker-образ и сделайте push в ваш docker.io репозиторий
 - запустите контейнер из получившегося образа и выполните запрос пути / c хост-машины

Требования к elasticsearch.yml:

 - данные path должны сохраняться в /var/lib
 - имя ноды должно быть netology_test
В ответе приведите:

 - текст Dockerfile манифеста
 - ссылку на образ в репозитории dockerhub 
 - ответ elasticsearch на запрос пути / в json виде
   
Подсказки:

 - возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
 - при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
 - при некоторых проблемах вам поможет docker директива ulimit
 - elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

<h2>Решение</h2>
____

 - Текст Dockerfile:

   
    FROM centos:7
    LABEL maintainer="frenzyyyy1@gmail.com"
    ENV PATH=/usr/lib:/usr/lib/jvm/jre-11/bin:$PATH

    RUN yum install java-11-openjdk -y 
    RUN yum install wget -y 


    RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz
    RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz.sha512 
	
    RUN yum install perl-Digest-SHA -y 
    RUN shasum -a 512 -c elasticsearch-7.11.1-linux-x86_64.tar.gz.sha512 \ 
        && tar -xzf elasticsearch-7.11.1-linux-x86_64.tar.gz \
        && yum upgrade -y
	
	
    ADD elasticsearch.yml /elasticsearch-7.11.1/config/
    ENV JAVA_HOME=/elasticsearch-7.11.1/jdk/
    ENV ES_HOME=/elasticsearch-7.11.1
    RUN groupadd elasticsearch \
        && useradd -g elasticsearch elasticsearch
    
    RUN mkdir /var/lib/logs \
    && chown elasticsearch:elasticsearch /var/lib/logs \
    && mkdir /var/lib/data \
    && chown elasticsearch:elasticsearch /var/lib/data \
    && chown -R elasticsearch:elasticsearch /elasticsearch-7.11.1/
    RUN mkdir /elasticsearch-7.11.1/snapshots &&\
        chown elasticsearch:elasticsearch /elasticsearch-7.11.1/snapshots
    
    USER elasticsearch
    CMD ["/usr/sbin/init"]
    CMD ["/elasticsearch-7.11.1/bin/elasticsearch"]

 - Ссылка
~~~~
https://hub.docker.com/repository/docker/frenzyyyy/elstk001
~~~~
 - Ответ сервера


    {
    "name" : "cb69dbddb8c8",
    "cluster_name" : "netology_test",
    "cluster_uuid" : "CPl_jLsVTB60nya-bwnI_g",
    "version" : {
    "number" : "7.11.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "ff17057114c2199c9c1bbecc727003a907c0db7a",
    "build_date" : "2021-02-15T13:44:09.394032Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
    },
    "tagline" : "You Know, for Search"
    }
<h2>Задача 2</h2>

____


В этом задании вы научитесь:

 - создавать и удалять индексы
 - изучать состояние кластера 
 - обосновывать причину деградации доступности данных
 - Ознакомтесь с документацией и добавьте в elasticsearch 3 индекса, в соответствии со таблицей:


    Имя	     Количество реплик    	Количество шард

    ind-1	           0	                    1

    ind-2	           1                        2

    ind-3	           2                        4

Получите список индексов и их статусов, используя API и приведите в ответе на задание.

Получите состояние кластера elasticsearch, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард, иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

<h2>Решение</h2>
____

 - Создание и список индексов

~~~~
C:\Users\Павел>curl -X PUT 127.0.0.1:9200/ind-1 -H "Content-Type: application/json" -d"{ \"settings\": { \"number_of_shards\": 1,  \"number_of_replicas\": 0 }}"
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-1"}
C:\Users\Павел>curl -X PUT 127.0.0.1:9200/ind-2 -H "Content-Type: application/json" -d"{ \"settings\": { \"number_of_shards\": 2,  \"number_of_replicas\": 1 }}"
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-2"}
C:\Users\Павел>curl -X PUT 127.0.0.1:9200/ind-3 -H "Content-Type: application/json" -d"{ \"settings\": { \"number_of_shards\": 4,  \"number_of_replicas\": 2 }}"
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-3"}

health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 NXUW2VJlTty84B-tf8VG3A   1   0          0            0       208b           208b
yellow open   ind-3 n44E1i-pRqCpfgf5cTT79Q   4   2          0            0       832b           832b
yellow open   ind-2 Sa6xPqmwS-CiZQu5p8U40Q   2   1          0            0       416b           416b

~~~~
 - Состояние индексов:
 
ind-1
~~~~
{"cluster_name":"netology_test",
"status":"green",
"timed_out":false,
"number_of_nodes":1,
"number_of_data_nodes":1,
"active_primary_shards":1,
"active_shards":1,
"relocating_shards":0,
"initializing_shards":0,
"unassigned_shards":0,
"delayed_unassigned_shards":0,
"number_of_pending_tasks":0,
"number_of_in_flight_fetch":0,
"task_max_waiting_in_queue_millis":0,
"active_shards_percent_as_number":100.0}
~~~~

ind-2
~~~~
{"cluster_name":"netology_test",
"status":"yellow",
"timed_out":false,
"number_of_nodes":1,
"number_of_data_nodes":1,
"active_primary_shards":2,
"active_shards":2,
"relocating_shards":0,
"initializing_shards":0,
"unassigned_shards":2,
"delayed_unassigned_shards":0,
"number_of_pending_tasks":0,
"number_of_in_flight_fetch":0,
"task_max_waiting_in_queue_millis":0,
"active_shards_percent_as_number":41.17647058823529}
~~~~
ind-3
~~~~
{"cluster_name":"netology_test",
"status":"yellow",
"timed_out":false,
"number_of_nodes":1,
"number_of_data_nodes":1,
"active_primary_shards":4,
"active_shards":4,
"relocating_shards":0,
"initializing_shards":0,
"unassigned_shards":8,
"delayed_unassigned_shards":0,
"number_of_pending_tasks":0,
"number_of_in_flight_fetch":0,
"task_max_waiting_in_queue_millis":0,
"active_shards_percent_as_number":41.17647058823529}
~~~~

Кластер
~~~~
{"cluster_name":"netology_test",
"status":"yellow",
"timed_out":false,
"number_of_nodes":1,
"number_of_data_nodes":1,
"active_primary_shards":7,
"active_shards":7,
"relocating_shards":0,
"initializing_shards":0,
"unassigned_shards":10,
"delayed_unassigned_shards":0,
"number_of_pending_tasks":0,
"number_of_in_flight_fetch":0,
"task_max_waiting_in_queue_millis":0,
"active_shards_percent_as_number":41.17647058823529}
~~~~

 - Удаление индексов:

~~~~
C:\Users\Павел>curl -X DELETE 'http://127.0.0.1:9200/ind-1?pretty'
curl: (3) URL using bad/illegal format or missing URL

C:\Users\Павел>curl -X DELETE "http://127.0.0.1:9200/ind-1?pretty"
{
  "acknowledged" : true
}

C:\Users\Павел>curl -X DELETE "http://127.0.0.1:9200/ind-2?pretty"
{
  "acknowledged" : true
}

C:\Users\Павел>curl -X DELETE "http://127.0.0.1:9200/ind-3?pretty"
{
  "acknowledged" : true
}

~~~~

 - У индексов статус Yelow из-за того, что указано число реплик, нонет других серверов и не куда реплицироваться.
<h2>Задача 3</h2>

____

В данном задании вы научитесь:

 - создавать бэкапы данных 
 - восстанавливать индексы из бэкапов
 - Создайте директорию {путь до корневой директории с elasticsearch в образе}/snapshots.

Используя API зарегистрируйте данную директорию как snapshot repository c именем netology_backup.

Приведите в ответе запрос API и результат вызова API для создания репозитория.

Создайте индекс test с 0 реплик и 1 шардом и приведите в ответе список индексов.

Создайте snapshot состояния кластера elasticsearch.

Приведите в ответе список файлов в директории со snapshotами.

Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.

Восстановите состояние кластера elasticsearch из snapshot, созданного ранее.

Приведите в ответе запрос к API восстановления и итоговый список индексов.

Подсказки:

возможно вам понадобится доработать elasticsearch.yml в части директивы path.repo и перезапустить elasticsearch

____


<h2>Решение</h2>

____

 - Создание директории для бэкапов

~~~~
C:\Users\Павел>curl -XPOST 127.0.0.1:9200/_snapshot/netology_backup?pretty -H "Content-Type: application/json" -d"{\"type\": \"fs\", \"settings\": { \"location\":\"/elasticsearch-7.11.1/snapshots\" }}"
{
  "acknowledged" : true
}
~~~~

~~~~
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "/elasticsearch-7.11.1/snapshots"
    }
  }
}
~~~~

 - Создание индекса

~~~~
C:\Users\Павел>curl -X PUT 127.0.0.1:9200/test -H "Content-Type: application/json" -d"{ \"settings\": { \"number_of_shards\": 1,  \"number_of_replicas\": 0 }}"
{"acknowledged":true,"shards_acknowledged":true,"index":"test"}
~~~~
~~~~
{"test":{"aliases":{},"mappings":{},"settings":{"index":{"routing":{"allocation":{"include":{"_tier_preference":"data_content"}}},"number_of_shards":"1","provided_name":"test","creation_date":"1670930052793","number_of_replicas":"0","uuid":"mmLGtWRZRf-fUNdG-TSN0w","version":{"created":"7110199"}}}}}
~~~~

 - Создание Snapshot

~~~~
C:\Users\Павел>curl -X PUT 127.0.0.1:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true
{"snapshot":{"snapshot":"elasticsearch","uuid":"Nn32dMI3Qaiq84ZHwiOuXQ","version_id":7110199,"version":"7.11.1","indices":["test"],"data_streams":[],"include_global_state":true,"state":"SUCCESS","start_time":"2022-12-13T11:17:59.334Z","start_time_in_millis":1670930279334,"end_time":"2022-12-13T11:17:59.334Z","end_time_in_millis":1670930279334,"duration_in_millis":0,"failures":[],"shards":{"total":1,"failed":0,"successful":1}}}
~~~~
 - Список файлов

~~~~
[elasticsearch@cb69dbddb8c8 /]$ cd elasticsearch-7.11.1/
LICENSE.txt      README.asciidoc  config/          lib/             modules/         snapshots/
NOTICE.txt       bin/             jdk/             logs/            plugins/
[elasticsearch@cb69dbddb8c8 /]$ cd elasticsearch-7.11.1/snapshots/
[elasticsearch@cb69dbddb8c8 snapshots]$ ls
index-0  index.latest  indices  meta-Nn32dMI3Qaiq84ZHwiOuXQ.dat  snap-Nn32dMI3Qaiq84ZHwiOuXQ.dat
[elasticsearch@cb69dbddb8c8 snapshots]$ ls -alg
total 60
drwxr-xr-x 1 elasticsearch  4096 Dec 13 11:17 .
drwxr-xr-x 1 elasticsearch  4096 Dec 13 07:18 ..
-rw-r--r-- 1 elasticsearch   437 Dec 13 11:17 index-0
-rw-r--r-- 1 elasticsearch     8 Dec 13 11:17 index.latest
drwxr-xr-x 3 elasticsearch  4096 Dec 13 11:17 indices
-rw-r--r-- 1 elasticsearch 30935 Dec 13 11:17 meta-Nn32dMI3Qaiq84ZHwiOuXQ.dat
-rw-r--r-- 1 elasticsearch   269 Dec 13 11:17 snap-Nn32dMI3Qaiq84ZHwiOuXQ.dat
~~~~

 - Удаление test
~~~~
C:\Users\Павел>curl -X DELETE "http://127.0.0.1:9200/test?pretty
{
  "acknowledged" : true
}
~~~~
 - Создание test-2
~~~~
C:\Users\Павел>curl -X PUT 127.0.0.1:9200/test2 -H "Content-Type: application/json" -d"{ \"settings\": { \"number_of_shards\": 2,  \"number_of_replicas\": 0 }}"
{"acknowledged":true,"shards_acknowledged":true,"index":"test2"}

{"test2":{"aliases":{},"mappings":{},"settings":{"index":{"routing":{"allocation":{"include":{"_tier_preference":"data_content"}}},"number_of_shards":"2","provided_name":"test2","creation_date":"1670930703064","number_of_replicas":"0","uuid":"P-S1aW_cSC232duDDWSsdw","version":{"created":"7110199"}}}}}
~~~~

 - Восстановление индекса:

~~~~
C:\Users\Павел>curl -X POST 127.0.0.1:9200/_snapshot/netology_backup/elasticsearch/_restore?pretty -H "Content-Type: application/json" -d"{\"include_global_state\":true}"
{
  "accepted" : true
}

C:\Users\Павел>curl http://127.0.0.1:9200/_cat/indices
green open test2 P-S1aW_cSC232duDDWSsdw 2 0 0 0 416b 416b
green open test  GESRZNqKSiOZViZZg1MO8Q 1 0 0 0 208b 208b
~~~~