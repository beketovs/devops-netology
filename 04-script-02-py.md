# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | будет ошибка, т.к. пытаемся сложить разные типы данных  |
| Как получить для переменной `c` значение 12?  | c = str(a) + b  |
| Как получить для переменной `c` значение 3?  | c = a + int(b)  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
#is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
#        break
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   curl.log
        modified:   test.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .bash_history
        .bash_logout
        .bashrc
        .cache/
        .gitconfig
        .profile
        .python_history
        .ssh/
        .sudo_as_admin_successful
        .vbox_version
        .viminfo
        .wget-hsts
        4_2_task.sh
        4_3_task.sh
        script.py

no changes added to commit (use "git add" and/or "git commit -a")

vagrant@vagrant:~$ ./script.py
curl.log
test.sh
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import sys

try:
    path = sys.argv[1]
    bash_command = ["cd " + path + " 2>&1", "git status 2>&1"]
except IndexError:
    print("Задайте директорию для проверки")
    sys.exit()
result_os = os.popen(' && '.join(bash_command)).read()
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(path + prepare_result)
    elif result.find('fatal') != -1:
        print("Директория не инициализирована под git, попробуйте другую директорию")
        sys.exit()
    elif result.find('can\'t cd') != -1:
        print("Директория задана некорректно или проблема с правами доступа")
        sys.exit()
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:/tmp/test$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   1.txt
        modified:   2.txt

no changes added to commit (use "git add" and/or "git commit -a")

vagrant@vagrant:~$ ./script.py /tmp/test/
/tmp/test/1.txt
/tmp/test/2.txt
vagrant@vagrant:~$ ./script.py /root
Директория задана некорректно или проблема с правами доступа
vagrant@vagrant:~$ ./script.py
Задайте директорию для проверки
```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import time
import socket

resolve_dict = dict()
print("=======FIRST CHECKING========")
hosts = ["drive.google.com", "mail.google.com", "google.com"]
for host in hosts:
    resolve_ip = socket.gethostbyname_ex(host)
    resolve_dict[host] = resolve_ip[2][0]
    print(f"<{host}> - <{resolve_ip[2][0]}>")
print("==========END================")
checking = True
while checking == True:
    for host in hosts:
        old_ip = resolve_dict.get(host)
        new_ip = socket.gethostbyname_ex(host)[2][0]
        if old_ip == new_ip:
            print("IP not changed")
            time.sleep(5)
            continue
        else:
            print("IP был изменен!!!")
            print(f"[ERROR] <{host}> IP mismatch: <{old_ip}> <{new_ip}>")
            checking = False
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~$ ./myscript.py
=======FIRST CHECKING========
<drive.google.com> - <173.194.222.194>
<mail.google.com> - <74.125.131.83>
<google.com> - <216.58.210.174>
==========END================
IP not changed
IP not changed
IP not changed
IP not changed
IP был изменен!!!
[ERROR] <mail.google.com> IP mismatch: <74.125.131.83> <142.251.1.18>
IP был изменен!!!
[ERROR] <google.com> IP mismatch: <216.58.210.174> <216.58.210.142>

В момент работы скрипта в качестве проверки даю команду:
root@vagrant:/home/vagrant# sudo systemd-resolve --flush-caches

```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 

### Ваш скрипт:
```python
???
```

### Вывод скрипта при запуске при тестировании:
```
???
```
