# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис
  
  ```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }, # добавлена запятая
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43" # добавлены кавычки
            }
        ]
    }
```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import time
import socket
import json
import yaml


resolve_dict = dict()
print("=======FIRST CHECKING========")
hosts = ["drive.google.com", "mail.google.com", "google.com"]
for host in hosts:
    resolve_ip = socket.gethostbyname_ex(host)
    resolve_dict[host] = resolve_ip[2][0]
    print(f"<{host}> - <{resolve_ip[2][0]}>")
with open('hosts.json', 'w') as js:
    js.write(json.dumps(resolve_dict, indent=2))
with open('hosts.yml', 'w') as ym:
    ym.write(yaml.dump(resolve_dict, explicit_start=True, explicit_end=True))
print("==========END================")
checking = True
while checking == True:
    for host in hosts:
        old_ip = resolve_dict.get(host)
        new_ip = socket.gethostbyname_ex(host)[2][0]
        if old_ip == new_ip:
            print("IP not change")
            time.sleep(5)
            continue
        else:
            resolve_dict[host] = new_ip
            print("IP was change!!!")
            print(f"[ERROR] <{host}> IP mismatch: <{old_ip}> <{new_ip}>")
            with open('hosts.json', 'w') as js:
                js.write(json.dumps(resolve_dict, indent=2))
            with open('hosts.yml', 'w') as ym:
                ym.write(yaml.dump(resolve_dict, explicit_start=True, explicit_end=True))
            checking = False
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~$ ./myscript.py
=======FIRST CHECKING========
<drive.google.com> - <173.194.222.194>
<mail.google.com> - <64.233.165.19>
<google.com> - <173.194.222.113>
==========END================
IP not change
IP not change
IP was change!!!
[ERROR] <google.com> IP mismatch: <173.194.222.113> <173.194.220.100>

В момент работы скрипта в качестве проверки даю команду:
root@vagrant:/home/vagrant# sudo systemd-resolve --flush-caches
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{
  "drive.google.com": "173.194.222.194",
  "mail.google.com": "64.233.165.19",
  "google.com": "173.194.220.100"
}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
---
drive.google.com: 173.194.222.194
google.com: 173.194.220.100
mail.google.com: 64.233.165.19
...
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:
   * Принимать на вход имя файла
   * Проверять формат исходного файла. Если файл не json или yml - скрипт должен остановить свою работу
   * Распознавать какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны
   * Перекодировать данные из исходного формата во второй доступный (из JSON в YAML, из YAML в JSON)
   * При обнаружении ошибки в исходном файле - указать в стандартном выводе строку с ошибкой синтаксиса и её номер
   * Полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов

### Ваш скрипт:
```python
???
```

### Пример работы скрипта:
???
