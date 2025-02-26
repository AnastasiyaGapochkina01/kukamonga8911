Репы с занятия
- bookstore: https://gitlab.com/devops201206/bookstore
- templates: https://gitlab.com/devops201206/deploy-templates

# Задача

Дано приложение на golang \
https://gist.github.com/AnastasiyaGapochkina01/472f54eddb8efc8a3397a9949d54c2bb \
Необходимо написать модульный pipeline, который будет
- прогонять форматирование ```go fmt```
- делать проверку с помощью ```go vet```
- собирать бинарный файл и сохранить его в артефактах gitlab
  - для windows. Переменные для сборки ```GOOS=windows GOARCH=386``` и ```GOOS=windows GOARCH=amd64```
  - для linux. Переменные для сборки ```GOOS=linux GOARCH=amd64```
- собирать docker image с этим бинарным файлом
- деплоить с помощью docker compose (__приложению нужен postgres__)
