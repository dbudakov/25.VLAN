## Домашнее задание  
Строим бонды и вланы
в Office1 в тестовой подсети появляется сервера с доп интерфесами и адресами
в internal сети testLAN
- testClient1 - 10.10.10.254
- testClient2 - 10.10.10.254
- testServer1- 10.10.10.1
- testServer2- 10.10.10.1

равести вланами
testClient1 <-> testServer1
testClient2 <-> testServer2

между centralRouter и inetRouter
"пробросить" 2 линка (общая inernal сеть) и объединить их в бонд
проверить работу c отключением интерфейсов

## Решение
![vlan](https://github.com/dbudakov/25.VLAN/blob/master/images/homework/VLAN.jpg)

 Доступ настроен сквозь `centralRouter` между `testClient1`<-->`testServer1` и `testClient2`<-->`testServer2` соответственно. Шлюзы не пингуются, так как для них нет правил маршрутизации, но назначаются правила маршрутизации входящего трафика, по каждому `vlan'у` и уже в таблицах указывается маршрут через какие интерфейсы этот маршрут доступен. Поэтому `testClient*` и `testServer*` не пингуют шлюзы на `centralRouter'e` и до указанных VM не пройдет трафик исходящий от `centalRouter`.  
```
ip ru add to 10.10.10.254 iif eth1.5 table 5 # означает маршрут до адреса 10.10.10.254 из интерфеса eth1.5 попадает в таблицу с идентификатором 5
ip ro add 10.10.10.254 via 10.10.10.6 dev eth2.5 table 5 # маршрут для таблицы с идентификатором 5 доступ к 10.10.10.254 через адрес 10.10.10.6 и интерфейс `eth2.5`
```

Маршрут содержащийся внутри файла `route-eth*`, должен относится к тому же интерфейсу, как в имени файлы, иначе возможен случай когда маршрут не назначится из-за не поднятого интерфейса.   
```
cat /etc/sysconfig/network-scripts/route-eth2.5
  10.10.10.254 via 10.10.10.6 dev eth2.5 table 5
```

В решение для `bonding` в настройках проставляется параметр `+bond.option fail_over_mac=1`, обеспечивающий отказоусточивость подключения, в случае отказа активного интерфейса `bond0`, резервный подхватит подключение.  


Дополнительно  
https://github.com/juju4/ansible-harden-sysctl - sysctl proxy_arp  
http://www.simtreas.ru/~dzo/peremark/ - маркировка трафика  
http://www.adminia.ru/nastroyka-vlan-v-linux/  
https://habr.com/ru/post/100919/  
https://www.opennet.ru/tips/2009_policy_route_linux.shtml  
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_static_routes_in_ifcfg_files  
[nmcli_rule_route](http://devemmeff.blogspot.com/2016/02/howto-policy-based-routing-using.html)  
