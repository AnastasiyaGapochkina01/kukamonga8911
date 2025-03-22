Написать пайплайн для приложения-конверетра markdown -> pdf\
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
