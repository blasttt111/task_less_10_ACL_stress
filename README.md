# ACL
Создадим директорию `/var/www/server-app`
  - (`sudo mkdir -p /var/www/server-app/uploads`)

Создаем файлы `readme.md` и `app.log`

  - (`sudo touch /var/www/server-app/readme.md`)
  - (`sudo touch /var/www/server-app/app.log`)

<img width="531" height="127" alt="image" src="https://github.com/user-attachments/assets/ba1c0248-2e87-4747-a07d-9220cf3ea610" />


Создаем группы по заданию
  - (`sudo groupadd ftpusers`)
  - (`sudo groupadd admins`)
  - (`sudo groupadd auditors`)
  - (`sudo useradd -m -d /opt/logger -s /usr/sbin/nologin logger`)

<img width="818" height="304" alt="image" src="https://github.com/user-attachments/assets/3d20323e-9410-40bc-992d-826e94e1ce22" />

Выдаем разрешения с помощью ACL
  - (`sudo setfacl -R -m u:www-data:r /var/www/server-app/uploads`)
  - (`sudo setfacl -m g:ftpusers:w /var/www/server-app/uploads`)
  - (`sudo setfacl -R -m g:admins:rwx /var/www/server-app`)
  - (`sudo setfacl -m u:logger:rwx /var/www/server-app/app.log`)
  - (`sudo setfacl -m g:auditors:r /var/www/server-app/app.log`)
<img width="784" height="169" alt="image" src="https://github.com/user-attachments/assets/d79db2b1-aa3a-4cda-ad6b-82cd22f6ba30" />

<img width="677" height="810" alt="image" src="https://github.com/user-attachments/assets/5377eee3-58ad-465a-a51f-dd48bf2b306d" />

Чтобы настроить создание файлов с `rw-r-----`, а директории - `rwxr-x---` нужно поставить umask на значение `027`
Создадим директорию `testing` и файл `test.txt`

<img width="682" height="314" alt="image" src="https://github.com/user-attachments/assets/e35e75ad-b511-4042-8bee-b08808011044" />

Разрешаем пользователю `logger` вход в систему

  - (`sudo usermod -s /bin/bash logger`)

Чтобы переключиться на него, `СОХРАНИВ` окружение выполним:
  - (`su logger -m`)
Создаем файл с переменными окружения:
  - (`env > /tmp/logger-envs`)

Теперь выполним переключение с чистым окружением и сравним вывод env
<img width="1438" height="915" alt="image" src="https://github.com/user-attachments/assets/e7f166ae-168f-4408-894a-990b41e53933" />

Без переключения пользователя создаем директорию от имени `logger`
  - (`sudo -u logger mkdir /tmp/logger-dir`)

<img width="775" height="103" alt="image" src="https://github.com/user-attachments/assets/25cf6a20-d854-4ec5-b700-7727b7629d8f" />

# Анализ производительности

Определите текущую `load average`; установить пакет `sysstat`

<img width="695" height="271" alt="image" src="https://github.com/user-attachments/assets/c30af51e-56ca-4998-af8c-046866ac331c" />

Проведем тесты по заданию
<img width="787" height="206" alt="image" src="https://github.com/user-attachments/assets/a5d0845a-ada5-4e48-ad7f-66243274f782" />
<img width="1118" height="225" alt="image" src="https://github.com/user-attachments/assets/4b819557-865d-4f0d-9765-ed64aea85bc0" />
<img width="881" height="206" alt="image" src="https://github.com/user-attachments/assets/5e9d74a8-52e4-4232-b327-f3462576bcfe" />

Посмотрим вывод `Dstat` и `mpstat`
<img width="1173" height="440" alt="image" src="https://github.com/user-attachments/assets/fd4de00e-bcd3-4f3a-beca-a963ac0210af" />

<img width="1044" height="187" alt="image" src="https://github.com/user-attachments/assets/1073a057-698e-4c0d-a0ac-5bec2a055578" />

Запустить `dd if=/dev/zero of=/tmp/testfile bs=1M count=500` и параллельно с помощью `iostat` найдем нужные метрики

  - (`iostat -xz 1`)
<img width="1526" height="261" alt="image" src="https://github.com/user-attachments/assets/c0015f52-b1ed-4c27-b99a-3ee38a83ee31" />

Снова дал нагрузку и проанализировал `sar`  и `atop`
  - (`stress-ng --cpu 2 --vm 1 --vm-bytes 1G --timeout 300`)
  - (`dd if=/dev/zero of=/tmp/testfile2 bs=1M count=1000`)

`sar -u` CPU

`sar -r` RAM

`sar -d` DISK

<img width="1253" height="720" alt="image" src="https://github.com/user-attachments/assets/753431f0-5604-4777-9a14-3bea6cf9181f" />

<img width="1134" height="294" alt="image" src="https://github.com/user-attachments/assets/f77735ee-48dc-4ba4-a9d7-c4cca3cf07f1" />

`Atop` без нагрузки
<img width="1332" height="795" alt="image" src="https://github.com/user-attachments/assets/8a6a8e1f-5a1b-4964-b5f2-17d5ae08735c" />

`stress-ng` в статусе `running`
<img width="1634" height="378" alt="image" src="https://github.com/user-attachments/assets/792f0aeb-2a15-4994-88e3-6614000c1c72" />

Запустить `openssl speed -seconds 30 2>& 1 > /dev/null &` и с помощью `pidstat` найти PID и % usr и sys CPU time

  - (`pidstat -G openssl`)

<img width="850" height="91" alt="image" src="https://github.com/user-attachments/assets/1cbc91a7-cc32-42e1-be51-aae4e4397a8b" />

Запустить `sleep 5; ls -R /usr` и после ее завершения посмотреть метрики

  - (`/usr/bin/time -v sh -c "sleep 5; ls -R /usr"`)

<img width="1148" height="492" alt="image" src="https://github.com/user-attachments/assets/3e058d77-8968-4657-8909-58a3705458c2" />

Снова дадим нагрузку
  - (`stress-ng --cpu 2 --vm 1 --hdd 1 --timeout 300 &`)

Выведем с помощью `ps` топ-5 процессов с наибольшим потреблением
`CPU`
`MEM`
`DISK`
<img width="1122" height="312" alt="image" src="https://github.com/user-attachments/assets/f70fe8d0-bfb5-4995-9f7f-49877768c77c" />
<img width="699" height="172" alt="image" src="https://github.com/user-attachments/assets/3d6bfa84-a554-4266-a22e-15efc4a376c1" />
