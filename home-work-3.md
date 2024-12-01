1) Запустить в docker элементарное приложение на nodejs
```
// app.js
const http = require('http');

const requestHandler = (req, res) => {
    res.end('Hello from the Node.js application!');
};

const server = http.createServer(requestHandler);

server.listen(3000, () => {
    console.log('Server is listening on port 3000');
});
```
2) Настроить nginx так чтобы
- запросы по типу /node он отправлял на контейнер с приложением
- запросы на / отдавали index.html
```
<html><body><h1>Welcome to node 1!</h1></body></html>
```
- логи vhost`a записывались в файлы /var/log/nginx/node-1-access.log и /var/log/nginx/node-1-error.log
- домен сайта - node1.local
3) Запустить еще один контейнер с приложением на другом порту и настроить сайт node2.local аналогичный node1.local.
4) Настроить ssl для обоих сайтов на самоподписанных сертификатах
