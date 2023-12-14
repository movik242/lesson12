# lesson12

**Домашнее задание**

1. Запустить nginx на нестандартном порту 3-мя разными способами:
  - переключатели setsebool;
  - добавление нестандартного порта в имеющийся тип;
  - формирование и установка модуля SELinux.

К сдаче:
 -- README с описанием каждого решения (скриншоты и демонстрация приветствуются).
 
2. Обеспечить работоспособность приложения при включенном selinux.
  - развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems;
  - выяснить причину неработоспособности механизма обновления зоны (см. README);
  - предложить решение (или решения) для данной проблемы;
  - выбрать одно из решений для реализации, предварительно обосновав выбор;
  - реализовать выбранное решение и продемонстрировать его работоспособность.

К сдаче:
 -- README с анализом причины неработоспособности, возможными способами решения и обоснованием выбора одного из них;
исправленный стенд или демонстрация работоспособной системы скриншотами и описанием.

**Основные команды и расположения файлов**

Для того чтобы работать с политиками сразу установим semanage: yum -y install policycoreutils-python

Просмотреть контекст безопасности на файлах, директориях и процессах можем командами ps -Z и ls -Z.

Узнать режим работы можно командой sestatus и getenforce

Файлы политик размещаются по пути - /etc/selinux/targeted/contexts/files

Лог аудита храниться в файле /var/log/audit/audit.log

Просмотреть текущие правила - sesearch -A -s httpd_t

Восстановить контекст (а также необходимо если надо окончательно применить новые метки файлов в файловой системе, чтобы заработало после перезагрузки) - restorecon /html

Проверка по портам - semanage port -l | grep ssh. Назначить доп. порты в тип правил (ssh) - semanage port -a -t ssh_port_t -p -tcp 5022. Удалить прописанный порт - semanage port -d -p tcp 5022. Порты по-умолчанию описанные в политике удалить нельзя.

Итак, 1 задание

После установки порта 4881 - nginx не стартовал

![image](https://github.com/movik242/lesson12/assets/143793993/bc7287df-9b53-4ca7-90c9-93ffce7bef46)

Далее смотрим лог почему не запустился, меняем значение nis_enabled=on

![image](https://github.com/movik242/lesson12/assets/143793993/c451937e-d57e-42cd-b9f5-1a3df84b3a69)

![image](https://github.com/movik242/lesson12/assets/143793993/ddf25d8d-5448-4072-af02-c211a7dfc297)

Выключаем обратно nis_enabled=off, и пробуем добавить порт в правила selinux

![image](https://github.com/movik242/lesson12/assets/143793993/6655273b-14d6-4939-ae8f-9c8378f3c9f6)

![image](https://github.com/movik242/lesson12/assets/143793993/0105b5f8-f760-477c-8130-d2f85658aefe)

Убираем порт из разрешенных и даем разрешение модулю

![image](https://github.com/movik242/lesson12/assets/143793993/b7365c53-54df-47ce-9dfd-6e978007ad47)

![image](https://github.com/movik242/lesson12/assets/143793993/05c7f08f-6446-4be7-8f27-6dabc1dcbdad)

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

2 задание

Развернул приложенный стенд  https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems

Запустил виртуальные машины командой vagrant up

Подключаемся к клиентской машине     vagrant ssh client

![image](https://github.com/movik242/lesson12/assets/143793993/0e80eaa1-316f-4ab9-8dcb-09d4fb6fe2f5)

![image](https://github.com/movik242/lesson12/assets/143793993/a15b599b-5355-4544-b023-bde8d15a5421)

Получаем ошибку    update failed: SERVFAIL

Смотрим логи SELinux на сервере

      sudo -i
      cat /var/log/audit/audit.log | audit2why

В логах мы видим, что ошибка в контексте безопасности. Вместо типа named_t используется тип etc_t.

![image](https://github.com/movik242/lesson12/assets/143793993/68933b15-1d59-4f4d-a183-8f42ff346629)

![image](https://github.com/movik242/lesson12/assets/143793993/4ef2fb5c-b76d-4b38-af4d-4d51c0e70961)

Изменим тип контекста безопасности для каталога /etc/named

      chcon -R -t named_zone_t /etc/named

![image](https://github.com/movik242/lesson12/assets/143793993/8ce23c4c-fdf1-41e8-a2fa-5e2ee2eccfbd)

Попробуем снова внести изменения с клиента:

      nsupdate -k /etc/named.zonetransfer.key

![image](https://github.com/movik242/lesson12/assets/143793993/bc2a3231-0051-437d-8a27-15962f6386e5)

Причина неработоспособности механизма обновления заключается в том что Selinux блокировал доступ к обновлению файлов динамического обновления для DNS сервера, а также к некоторым файлам ОС, к которым DNS сервер обращается во время своей работы


