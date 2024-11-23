# Домашнее задание №1
## Shell Emulator
Этот проект представляет собой эмулятор командной оболочки, который работает с виртуальной файловой системой, хранящейся в ZIP-архиве.
Программа позволяет выполнять команды, такие как  `ls`, `cd`, и `rmdir`, в графическом интерфейсе с использованием библиотеки `tkinter`. Логи всех команд записываются в CSV файл для последующего анализа.

## Описание
Программа эмулирует работу с командной оболочкой, где пользователи могут вводить команды для навигации и управления файлами в виртуальной файловой системе, представленной в виде ZIP-архива.

## Возможности:
- Просмотр содержимого архивов с помощью команды `ls`.
- Навигация по папкам с помощью команды `cd`.
- Удаление пустых папок через команду `rmdir`.
- Запись логов всех команд и их результатов в CSV-файл.
## Требования
Для работы программы необходимо установить Python и следующие библиотеки:
- `tkinter` (для графического интерфейса)
- `zipfile` (для работы с ZIP-архивами)
- `xml.etree.ElementTree` (для чтения конфигурационного XML)
- `csv` (для записи логов)
## Установка
Клонируйте репозиторий:
```bash
  git clone https://github.com/yourusername/shell-emulator.git
  cd shell-emulator
```
Убедитесь, что у вас установлен Python 3.x. Если нет, загрузите и установите его с официального сайта Python.

### Установите необходимые зависимости:

Для Python 3:

```bash
  pip install tkinter
```
### Использование
Откройте конфигурационный файл `config.xml` и укажите:

- Имя компьютера `(computer_name)`.
- Путь к виртуальной файловой системе в формате ZIP `(fs_path)`.
- Путь к файлу для записи логов `(log_path)`.
  
Пример конфигурации:

```xml
<config>
    <computer_name>MyComputer</computer_name>
    <fs_path>virtual_fs.zip</fs_path>
    <log_path>commands_log.csv</log_path>
</config>
```
Запустите программу:

```bash
  python main.py
```
В графическом интерфейсе введите команды, например:

- `ls` — для отображения списка файлов.
- `cd folder_name` — для перехода в другую папку.
- `rmdir folder_name` — для удаления пустой папки.
  
Логи команд будут записываться в указанный CSV-файл.

# Описание функций
1. ### `read_config(config_path)`
Эта функция читает конфигурационный XML-файл и извлекает из него параметры, такие как `имя компьютера`, `путь к виртуальной файловой системе` и `путь к лог-файлу`.

```python
def read_config(config_path):
    tree = ET.parse(config_path)
    root = tree.getroot()
    computer_name = root.find('computer_name').text
    fs_path = root.find('fs_path').text
    log_path = root.find('log_path').text
    return computer_name, fs_path, log_path
```
### Параметры:

- `config_path` — путь к конфигурационному XML-файлу.
### Возвращаемое значение:

- Возвращает кортеж из трех строк: `computer_name`, `fs_path`, `log_path`.

2. ### `write_log(log_path, command, output)`
Функция записывает команду и её результат в лог-файл. В каждую запись добавляется временная метка.

```python
def write_log(log_path, command, output):
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    with open(log_path, 'a', newline='') as log_file:
        writer = csv.writer(log_file)
        writer.writerow([timestamp, command, output])
```
### Параметры:

- `log_path` — путь к лог-файлу.
- `command` — выполненная команда.
- `output` — результат выполнения команды.
  
### Возвращаемое значение:

- Нет (запись в лог-файл).

3. ### `execute_command(command)`
Эта функция выполняет различные команды, такие как ls для вывода списка файлов и директорий, cd для смены текущего каталога и rmdir для удаления каталога.

```python
def execute_command(command):
    global current_dir

    if command == 'clear':
        output_text.delete('1.0', tk.END)
        return ""

    output = ""

    if command == 'ls':
        # Список файлов и папок в текущем каталоге
        if current_dir:
            for name in myzip.namelist():
                if name.startswith(current_dir):
                    relative_name = name[len(current_dir):].strip('/')
                    if '/' not in relative_name:
                        output += relative_name + "\n"
        else:
            output = "\n".join(myzip.namelist()) + "\n"

    elif command.startswith('cd '):
        # Смена текущего каталога
        path = command.split(maxsplit=1)[1]
        if path == '..':
            current_dir = '/'.join(current_dir.strip('/').split('/')[:-1])
            if current_dir:
                current_dir += '/'
        else:
            new_path = (current_dir + path).strip('/') + '/'
            if any(name.startswith(new_path) for name in myzip.namelist()):
                current_dir = new_path
            else:
                output = f"No such directory: {path}\n"

    elif command == 'exit':
        root.destroy()
        return ""

    elif command.startswith('rmdir '):
        # Удаление папки
        path = command.split(maxsplit=1)[1]
        full_path = (current_dir + path).strip('/') + '/'

        # Проверяем, существует ли директория
        if full_path not in myzip.namelist():
            output = f"No such directory: {path}\n"
        else:
            # Проверяем, есть ли содержимое в директории
            has_content = any(item.startswith(full_path) and item != full_path for item in myzip.namelist())

            if has_content:
                output = f"Directory not empty: {path}\n"
            else:
                # Удаляем директорию, создавая новый zip без неё
                temp_zip_path = "temp_fs.zip"
                with ZipFile(temp_zip_path, 'w') as temp_zip:
                    for item in myzip.namelist():
                        if not item.startswith(full_path):
                            temp_zip.writestr(item, myzip.read(item))

                # Убедитесь, что myzip закрыт перед заменой
                myzip.close()  # Закрываем оригинальный zip перед заменой

                os.replace(temp_zip_path, fs_path)
                output = f"Directory {path} removed\n"

    else:
        output = f"Unknown command: {command}\n"

    output_text.insert(tk.END, f"{output}")
    return output
```
### Параметры:

- `command` — команда, которую нужно выполнить.
### Возвращаемое значение:
- Возвращает строку с результатом выполнения команды (вывод в консоль).

4. ### `on_command_enter(event=None)`
Эта функция обрабатывает ввод команды пользователем в `GUI`. Она считывает команду, вызывает её выполнение и записывает результат в лог.

```python
def on_command_enter(event=None):
    command = command_entry.get()
    command_entry.delete(0, tk.END)
    output_text.insert(tk.END, f"{computer_name}$ {command}\n")
    output = execute_command(command)
    write_log(log_path, command, output)
```
### Параметры:
- `event` — событие (необязательный параметр для обработки события в `tkinter`).
  
### Возвращаемое значение:
- Нет (обрабатывает команду и выводит результат).
  
5. ###  `Основная часть программы (инициализация GUI и запуск приложения)`
В этой части программы создается графический интерфейс с помощью `tkinter`, настраиваются виджеты для ввода команд и отображения вывода.

```python
# Чтение конфигурационного файла
config_path = "config.xml"
computer_name, fs_path, log_path = read_config(config_path)

# Открытие виртуальной файловой системы
with ZipFile(fs_path, 'a') as myzip:
    current_dir = ""

    # Настройка GUI с использованием tkinter
    root = tk.Tk()
    root.title("Shell Emulator")

    # Виджеты для ввода и вывода
    command_entry = tk.Entry(root, width=100)
    command_entry.pack(padx=10, pady=10)
    command_entry.bind('<Return>', on_command_enter)

    output_text = scrolledtext.ScrolledText(root, wrap=tk.WORD, width=100, height=30)
    output_text.pack(padx=10, pady=10)

    root.protocol("WM_DELETE_WINDOW", lambda: execute_command("exit"))
    # Запуск главного цикла приложения
    root.mainloop()
```
