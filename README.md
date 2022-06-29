# **Введение**

В данном домашнем задании нам необходимо изучить основы DNS, научиться работать с технологией Split-DNS в Linux-based системах.

---

# Работа со стендом и настройка DNS
Для выполнения данного ДЗ, необходимо скачать стенд и запустить его.
```
git clone https://github.com/Dmitriy-Iv/Otus-Pr-linux-HW23.git
cd Otus-Pr-linux-HW23
vagrant up
```
В результате развернётся стенд, в котором 4 виртуальных сервера. Два из них - это name servers (DNS), а два - это клиенты. В качестве DNS сервера у нас будет выступать Bind.
Конфигурирование серверов происходит с помощью ansible. Все конфигруационные файлы сконфигурированы согласно методички:
- resolv.conf - файлы отвечающий для настройки преобразователя системы доменных имен,
- named.* - файлы отвечающие за зоны на DNS серверах - ns1 (master), ns2 (slave),
- named.conf - основные конфигурационные файлы DNS сервера.

# Настройка Split-DNS
Согласно задания, client1 должен видеть две настроенные зоны - dns.lab и newdns.lab, но в зоне dns.lab видеть только A запись web1. Второй же клиент - client2 должен видеть обе A записи - web1 и web2, однако зона newdns.lab ему должна быть не доступна.
Для выполнения этого задания, необходимо создать дополнительную зону  - аналогичную dns.lab, убрав из неё A запись web2. Для этого создаём отдельный файл зоны - в нашем случае это файл с названием - named.dns.lab.client.
После этого сгенерировать 2 ключа для хостов client и client2 с помощью команды tsig-keygen.
Далее в конфигурационных файлах DNS серверов сделать разграничение с помощью так называемых настроек `view`, которые делают match по `acl` - где описывается клиент и его ключ.
Блок данных настроек внесён в соответствующие файлы. 
В конце playbook есть проверка, которая проверяет досутпность данных хостов. Выглядит она вот так: 
```
TASK [Ping Executing] **********************************************************
failed: [client2] (item=None) => {"censored": "the output has been hidden due to the fact that 'no_log: true' was specified for this result", "changed": false}
ok: [client] => (item=None)
ok: [client2] => (item=None)
ok: [client] => (item=None)
failed: [client] (item=None) => {"censored": "the output has been hidden due to the fact that 'no_log: true' was specified for this result", "changed": false}
fatal: [client]: FAILED! => {"censored": "the output has been hidden due to the fact that 'no_log: true' was specified for this result", "changed": false}
...ignoring
ok: [client2] => (item=None)
fatal: [client2]: FAILED! => {"censored": "the output has been hidden due to the fact that 'no_log: true' was specified for this result", "changed": false}
...ignoring

TASK [Ping Report] *************************************************************
ok: [client] => (item={'server': 'www.newdns.lab'}) => {
    "msg": "PING SUCCESS"
}
ok: [client] => (item={'server': 'web1.dns.lab'}) => {
    "msg": "PING SUCCESS"
}
skipping: [client] => (item={'server': 'web2.dns.lab'}) 
skipping: [client2] => (item={'server': 'www.newdns.lab'}) 
ok: [client2] => (item={'server': 'web1.dns.lab'}) => {
    "msg": "PING SUCCESS"
}
ok: [client2] => (item={'server': 'web2.dns.lab'}) => {
    "msg": "PING SUCCESS"
}
```
Из этого вывода видно, что client1 не видит web2.dns.lab, а client2 - не видит www.newdns.lab, что соотвествует требованиям задания.
