Решаемая проблема:
необходимо получать метрику количества активных пользователей на конкретном сайте.

Сопутствующие условия:
- метрика в формате Prometheus для визуализации в Grafana
- nginx расположен на отдельном хосте
- nginx проксирует не только интересующий сайт, но и другие ресурсы
- на сайт из офиса (известный постоянный IP) заходят его разработчики, тестировщики и поддержка (их хотелось бы не учитывать)
- лог ротируется 1 раз в сутки
- лог довольно большой по размеру


Скрипт запускается в контейнере. Располагается на машине мониторинга, находится в одной подсети с контейнером pushgateway.
Подключается по ssh под пользователем nginx-counter на хост nginx по ключу, который прокинут в контейнер.

Скрипт передает метрику количества уникальных ip за последнюю минуту из лога nginx посетителей по адресам, включающим "mysite.ru.". Если посетителей не было, передается нулевая метрика.

Скрипт учитывает "позицию" чтения лога. Так как цикл - 1 минута, чтобы каждый раз не читался весь лог-файл, то записывается размер файла в position.txt.
Сравнивается текущий размер файла с размером, записанным в файле позиции, чтобы текущий размер был больше, и когда лог ротируется (и соответственно обнуляется его размер), скрипт понимает, что нужно читать файл с начала и обнулить позицию в файле, так как текущий размер стал меньше, чем позиция.

При остановке и перезапуске скрипта необходимо пересобрать образ, чтобы учлась последняя позиция чтения и лог-файл не читался с начала.
docker compose up -d --build

Если нужно в логах контейнера отобразить информацию об обработке каждой строки лога, нужно включить уровень логирования (файл скрипта uniq.py) - DEBUG: level=logging.DEBUG (пересоберать образ).
(улучшение: вынести в переменную)
