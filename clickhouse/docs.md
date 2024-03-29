<details>
<summary>Общая информация про clickhouse</summary>

ClickHouse - это колоночная СУБД с открытым исходным кодом, разработанная для аналитики и обработки больших объемов данных. Его архитектура основана на сильной компрессии данных в колоночном формате, что обеспечивает высокую скорость выполнения аналитических запросов и эффективное использование ресурсов. Преимущества ClickHouse включают высокую производительность при обработке больших объемов данных, масштабируемость, поддержку SQL-запросов, гибкую настройку агрегации данных и поддержку распределенной архитектуры для работы с кластерами.

</details>

<details>
<summary>Зачем нужен zookeeper для clickhouse</summary>

 - Для настройки репликации используется zookeeper. Зукипер содержит в себе информацию о реплицируемых таблицах, его можно поднимать на отдельных серверах или размещать на сервере с ClickHouse.
 - Он постоянно забирает себе актуальные данные с реплик, по этому все репликации смотрят на него. Когда реплика видит что в зукипере появились новые данные, она идет на прямую в реплику и скачивает их себе. Когда реплика данные забрала, зукипер отмечает в себе что данные благополучно забрали.

- Таким образом если реплика вышла из строя, она подключается к зукиперу, находит все необходимые данные которые ей нужны. Когда есть нужные данные или их части, реплика идет на работающую реплику (в которой данные есть) и скачивает их себе.

- Репликация работает только для отдельных таблиц с использованием движка MergeTree

Основные возможности MergeTree

Основная идея, заложенная в основу движков семейства MergeTree следующая. Когда у вас есть огромное количество данных, которые должны быть вставлены в таблицу, вы должны быстро записать их по частям, а затем объединить части по некоторым правилам в фоновом режиме. Этот метод намного эффективнее, чем постоянная перезапись данных в хранилище при вставке.

- Хранит данные, отсортированные по первичному ключу. Это позволяет создавать разреженный индекс небольшого объёма, который позволяет быстрее находить данные.

- Позволяет оперировать партициями, если задан ключ партиционирования. ClickHouse поддерживает отдельные операции с партициями, которые работают эффективнее, чем общие операции с этим же результатом над этими же данными. Также, ClickHouse автоматически отсекает данные по партициям там, где ключ партиционирования указан в запросе. Это также увеличивает эффективность выполнения запросов.

- Поддерживает репликацию данных. Для этого используется семейство таблиц ReplicatedMergeTree.

- Поддерживает сэмплирование данных. При необходимости можно задать способ сэмплирования данных в таблице.

</details>

<details>
<summary>Шардирование в clickhouse</summary>

 - Shard в кластере ClickHouse - это часть данных, которая физически хранится на определенной ноде (инстансе сервера) в кластере. Кластер ClickHouse может быть разделен на несколько шардов, что позволяет распределить нагрузку и обеспечить более эффективное хранение и обработку данных. Каждый шард обычно содержит часть данных и может быть реплицирован для обеспечения отказоустойчивости
 
 - Шардирование — это стратегия горизонтального масштабирования кластера, при которой части одной базы данных ClickHouse® размещаются на разных шардах. Шард состоит из одного или нескольких хостов-реплик. Запрос на запись или чтение в шард может быть отправлен на любую его реплику, выделенного мастера нет. При вставке данных они будут скопированы с реплики, на которой был выполнен INSERT-запрос, на другие реплики шарда в асинхронном режиме.

 Для реализации шардирования существует Engine=Distributed дистрибьютор таблиц. В ClickHouse таблицы можно разделить на виртуальнее или реальные.

- реальные: хранят физически данные на дисках
  Engine = MergeTree

- виртуальные: ничего не хранят
  Engine = Distributor

Движок дистрибьютор позволяет собрать данные с разных шардов, обєдинить их, посчитать количество и отдать пользователю, подробнее про движки можно почитать здесь. Стоит помнить, что он в себе ничего не хранит.

</details>

<details>
<summary>Более подробно про репликацию</summary>

Репликация в ClickHouse может быть настроена для каждой таблицы отдельно. Вы можете иметь несколько реплицированных и несколько не реплицированных таблиц на одном сервере. Вы также можете реплицировать таблицы по-разному, например, одну с двухфакторной репликацией и другую с трехфакторной.

Репликация реализована в движке таблицы ReplicatedMergeTree. Путь в ZooKeeper указывается в качестве параметра движка. Все таблицы с одинаковым путем в ZooKeeper становятся репликами друг друга: они синхронизируют свои данные и поддерживают согласованность. Реплики можно добавлять и удалять динамически, просто создавая или удаляя таблицу.

Репликация использует асинхронную multi-master-схему. Вы можете вставить данные в любую реплику, которая имеет открытую сессию в ZooKeeper, и данные реплицируются на все другие реплики асинхронно. Поскольку ClickHouse не поддерживает UPDATE, репликация исключает конфликты (conflict-free replication). Поскольку подтверждение вставок кворумом не реализовано, только что вставленные данные могут быть потеряны в случае сбоя одного узла.

Метаданные для репликации хранятся в ZooKeeper. Существует журнал репликации, в котором перечислены действия, которые необходимо выполнить. Среди этих действий: получить часть (get the part); объединить части (merge parts); удалить партицию (drop a partition) и так далее. Каждая реплика копирует журнал репликации в свою очередь, а затем выполняет действия из очереди. Например, при вставке в журнале создается действие “получить часть” (get the part), и каждая реплика загружает эту часть. Слияния координируются между репликами, чтобы получить идентичные до байта результаты. Все части объединяются одинаково на всех репликах. Одна из реплик-лидеров инициирует новое слияние кусков первой и записывает действия “слияния частей” в журнал. Несколько реплик (или все) могут быть лидерами одновременно. Реплике можно запретить быть лидером с помощью merge_tree настройки replicated_can_become_leader.

Репликация является физической: между узлами передаются только сжатые части, а не запросы. Слияния обрабатываются на каждой реплике независимо, в большинстве случаев, чтобы снизить затраты на сеть, во избежание усиления роли сети. Крупные объединенные части отправляются по сети только в случае значительной задержки репликации.

Кроме того, каждая реплика сохраняет свое состояние в ZooKeeper в виде набора частей и его контрольных сумм. Когда состояние в локальной файловой системе расходится с эталонным состоянием в ZooKeeper, реплика восстанавливает свою согласованность путем загрузки отсутствующих и поврежденных частей из других реплик. Когда в локальной файловой системе есть неожиданные или испорченные данные, ClickHouse не удаляет их, а перемещает в отдельный каталог и забывает об этом.

</details>
