1) Написать пайплайн для приложения-конверетра markdown -> pdf\
app.py
```
import argparse
from markdown_pdf import MarkdownPdf, Section

def convert_markdown_to_pdf(input_file, output_file):
    pdf = MarkdownPdf(toc_level=2)

    with open(input_file, 'r', encoding='utf-8') as file:
        markdown_content = file.read()

    pdf.add_section(Section(markdown_content))
    pdf.save(output_file)
    print(f"PDF файл создан: {output_file}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Конвертировать Markdown в PDF')
    parser.add_argument('-i', '--input', help='Входной файл Markdown', required=True)
    parser.add_argument('-o', '--output', help='Выходной файл PDF', required=True)
    args = parser.parse_args()

    convert_markdown_to_pdf(args.input, args.output)

```
requirements.txt
```
markdown-pdf
```
Пайплайн должен 
1) прогонять линтеры 
  - для markdown, если был пуш в любую ветку и изменения затронули файлы markdown
  - python, если был пуш в любую ветку и изуменения затронули app.py
  - если в commit message есть подстрока "skip linter", то НЕ запускать линтер
2) Генерировать с помощью скрипта app.py документ pdf и сохранять его в артефактах
- если прошел линтер markdown - автоматически
- если линтер markdown завалился или не запускался - оставить возможность ручного запуска
3) Отправлять сгенерированный pdf на ftp-сервер
4) Отправлять уведомление о том, что сгенерирован новый файл со ссылкой на него на ftp-сервере

2) Написать пайплайн для приложения на fastapi\
main.py
```
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import mysql.connector

# Настройки базы данных
config = {
    'user': 'inventory',
    'password': 'invpass',
    'database': 'inventory_db',
    'host': '127.0.0.1',
    'raise_on_warnings': True
}

def get_db_connection():
    return mysql.connector.connect(**config)

class Server(BaseModel):
    pub_ip: str
    priv_ip: str
    hostname: str
    location: str
    cpu: str
    mem: str
    disk: str
    os: str

app = FastAPI()

def create_table():
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS servers (
            id INT AUTO_INCREMENT PRIMARY KEY,
            pub_ip VARCHAR(255),
            priv_ip VARCHAR(255),
            hostname VARCHAR(255),
            location VARCHAR(255),
            cpu VARCHAR(255),
            mem VARCHAR(255),
            disk VARCHAR(255),
            os VARCHAR(255)
        )
    """)
    conn.commit()
    cursor.close()
    conn.close()

create_table()

@app.post("/servers/")
async def create_server(server: Server):
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("""
        INSERT INTO servers (pub_ip, priv_ip, hostname, location, cpu, mem, disk, os)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
    """, (server.pub_ip, server.priv_ip, server.hostname, server.location, server.cpu, server.mem, server.disk, server.os))
    conn.commit()
    cursor.close()
    conn.close()
    return {"message": "Server created successfully"}

@app.get("/servers/")
async def get_servers():
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM servers")
    rows = cursor.fetchall()
    servers = []
    for row in rows:
        server = {
            "id": row[0],
            "pub_ip": row[1],
            "priv_ip": row[2],
            "hostname": row[3],
            "location": row[4],
            "cpu": row[5],
            "mem": row[6],
            "disk": row[7],
            "os": row[8]
        }
        servers.append(server)
    cursor.close()
    conn.close()
    return servers

@app.put("/servers/{server_id}")
async def update_server(server_id: int, server: Server):
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("""
        UPDATE servers
        SET pub_ip = %s, priv_ip = %s, hostname = %s, location = %s, cpu = %s, mem = %s, disk = %s, os = %s
        WHERE id = %s
    """, (server.pub_ip, server.priv_ip, server.hostname, server.location, server.cpu, server.mem, server.disk, server.os, server_id))
    if cursor.rowcount == 0:
        raise HTTPException(status_code=404, detail="Server not found")
    conn.commit()
    cursor.close()
    conn.close()
    return {"message": "Server updated successfully"}

@app.delete("/servers/{server_id}")
async def delete_server(server_id: int):
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("DELETE FROM servers WHERE id = %s", (server_id,))
    if cursor.rowcount == 0:
        raise HTTPException(status_code=404, detail="Server not found")
    conn.commit()
    cursor.close()
    conn.close()
    return {"message": "Server deleted successfully"}
```
requirements.txt
```
annotated-types==0.7.0
anyio==4.9.0
certifi==2025.1.31
click==8.1.8
fastapi==0.115.11
h11==0.14.0
httpcore==1.0.7
httpx==0.28.1
idna==3.10
iniconfig==2.1.0
mysql-connector-python==9.2.0
packaging==24.2
pluggy==1.5.0
pydantic==2.10.6
pydantic_core==2.27.2
pytest==8.3.5
sniffio==1.3.1
starlette==0.46.1
typing_extensions==4.12.2
uvicorn==0.34.0
```
tests.py
```
import pytest
from fastapi.testclient import TestClient
from main import app, get_db_connection, create_table
import mysql.connector

# Конфигурация тестовой БД
TEST_DB_CONFIG = {
    'user': 'inventory',
    'password': 'invpass',
    'database': 'inventory_db_test',
    'host': '127.0.0.1',
    'raise_on_warnings': True
}

@pytest.fixture(scope="function")
def client(monkeypatch):
    # Подменяем конфиг БД на тестовый
    monkeypatch.setattr('main.config', TEST_DB_CONFIG)

    # Создаем таблицы
    conn = get_db_connection()
    #create_table()
    conn.close()

    # Создаем тестовый клиент
    with TestClient(app) as test_client:
        yield test_client

    # Очистка после теста
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("DELETE FROM servers")
    conn.commit()
    cursor.close()
    conn.close()

# Образец сервера для тестов
TEST_SERVER = {
    "pub_ip": "192.168.1.1",
    "priv_ip": "10.0.0.1",
    "hostname": "test-server",
    "location": "DC1",
    "cpu": "Intel Xeon",
    "mem": "64GB",
    "disk": "2TB SSD",
    "os": "Ubuntu 20.04"
}

def test_create_server(client):
    response = client.post("/servers/", json=TEST_SERVER)
    assert response.status_code == 200
    assert response.json() == {"message": "Server created successfully"}

def test_get_servers_empty(client):
    response = client.get("/servers/")
    assert response.status_code == 200
    assert response.json() == []

def test_get_servers_with_data(client):
    client.post("/servers/", json=TEST_SERVER)
    response = client.get("/servers/")
    assert response.status_code == 200
    assert len(response.json()) == 1
    assert response.json()[0]["hostname"] == "test-server"

def test_update_server(client):
    # Создаем сервер
    create_resp = client.post("/servers/", json=TEST_SERVER)
    server_id = client.get("/servers/").json()[0]["id"]

    # Обновляем данные
    update_data = {**TEST_SERVER, "hostname": "updated-server"}
    response = client.put(f"/servers/{server_id}", json=update_data)
    assert response.status_code == 200

    # Проверяем обновление
    get_resp = client.get("/servers/")
    assert get_resp.json()[0]["hostname"] == "updated-server"

def test_update_nonexistent_server(client):
    response = client.put("/servers/999", json=TEST_SERVER)
    assert response.status_code == 404
    assert "Server not found" in response.json()["detail"]

def test_delete_server(client):
    # Создаем сервер
    client.post("/servers/", json=TEST_SERVER)
    server_id = client.get("/servers/").json()[0]["id"]

    # Удаляем
    response = client.delete(f"/servers/{server_id}")
    assert response.status_code == 200

    # Проверяем удаление
    assert len(client.get("/servers/").json()) == 0

def test_delete_nonexistent_server(client):
    response = client.delete("/servers/999")
    assert response.status_code == 404
    assert "Server not found" in response.json()["detail"]
```
Пайплайн должен
1) прогонять линтер для .py файлов, если коммит затронул их
2) прогонять тесты
3) если тесты завалились - отправлять уведомление в TG и никогда не деплоить/релизить приложение
4) поробовать собрать модуль с помощью ```python -m build``` и сформировать релиз, если тесты прошли
