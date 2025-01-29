1) Упаковать в docker приложения
- https://github.com/AnastasiyaGapochkina01/monitoring-app
- https://github.com/AnastasiyaGapochkina01/node-app
- https://github.com/AnastasiyaGapochkina01/book-landing/tree/main
- https://github.com/AnastasiyaGapochkina01/apps/tree/main/django-docker
> [!NOTE]  
> - для всех контейнеров в Dockerfile придумать и реализовать healthcheck
2) Запустить все, что собрал в задаче №1, проверить что работает
3) Написать скрипт, который будет собирать информацию по всем запущенным контейнерам
- ip адреса контейнеров
- имена контейнеров
- id контейнеров
- имя приложения
- image
вывод скрипта долже перенаправляться в файл docker_projects_info в формате
```
=== <container_name> ===
from <image>
<container_id> <container_ip> 
```
