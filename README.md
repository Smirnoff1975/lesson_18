# lesson_18
ДЗ. Vagrant

# Домашнее задание
Расширенная настройка дисков и сетей

# Исполнитель
Павел Смирнов

# Цель
научиться добавлять диски и настраивать сетевые соединения;

1. Подготовка окружения:
- Убедитесть, что установлено VirtualBox и Vagrant.
- Создайте директорию для проекта.
2. Создать базовую виртуальную машину:
- Использовать можно любой образ.
- Настроите память ВМ: 1024 МБ.
3. Добавление дисков:
- Добавьте пару виртуальных диска размером 1 ГБ каждый.
4. Настройка сети:
- Настройте проброс 80 порта с гостевой системы на порт 8080 хостовой системы.
5. Провижининг:
Напишите провижининг, который:
- Форматирует добавленные диски в файловую систему ext4.
- Создает точки монтирования /mnt/disk1 и /mnt/disk2.
- Монтирует диски в указанные директории.
- Добавляет записи в /etc/fstab для автоматического монтирования при загрузке.
Вы можете использовать пример, продемонстрированный на занятии

Формат сдачи

Сылка на git репозиторий с проектом
Репозиторий должен содержать Vagrantfile
Скриншот вывода команды df -h с запущенной ВМ
Скриншот с хостовой машины вывода команды netstat -tulpn | grep 8080 с запущенной ВМ

# Среда выполнения
  Хост машина - OS Windows 10 c Vagrant 2.4.9, VirtualBox 7.0.10, ВМ - образ OS Ubuntu 22.04

# Команды и описание действий

> через vi создаем файл скрипта pps со следующим содержимым, описание по тексту программы
```
#!/bin/bash


awk 'BEGIN {printf "%7s %7s %-6s %s\n","PID","PPID","STAT","COMMAND"}' > pps.tmp
# Формирование "шапки", для парсинга строк и выравнивания полей использую awk и его функцию printf которая позволяет задать ширину и
# выравнивание для каждого поля. Информацию для вывода будем накапливать во временном файле pps.tmp.
# Информацию для вывода буду брать из виртуальной фс /proc где отражается текущая информация о работе системы. В нашем случае нас интересуют
# числовые каталоги которые соответствуют PID запущенным процессам, внутри которых есть файлы cmdline - командная строка если есть и stat -
# строка с информацией о процессе
# PID - идентификатор процесса, поле №1
# PPID - идентификатор процесса родителя, поле №4
# STAT - статус процесса, поле №3
# COMM - имя процесса, поле №2, имя процесса будем выводить вместе с командой из cmdline разделитель ":"

# делаем цикл по папкам proc начинающихся с цифры, обычно этого достаточно чтобы выделить папки процессов, но введем дополнительную проверку

for i in /proc/[0-9]*
do

# выделим вы имени папки только ее имя и проверим на соответствие шаблону - между началом и концом строки произвольное количество цифр
# если удовлетворяет идем дальше

        pid=${i##*/}
        if [[ $pid =~ ^[0-9]+$ ]]
        then

# читаем содержимое cmdline (команды) и на лету заменяем "\0" на " " иначе лезло предупреждение, что bash не любит "\0"
# параллельно отключаем вывод ошибок

                comm=$(tr '\0' ' ' < "$i/cmdline" 2>/dev/null)
                if [[ -n $comm ]]
                then
                        comm=" :${comm}"
# если команда не пустая , добавляем ей в начало разделитель ":"

                fi
                s=$(awk -v cm="$comm" 'gsub(/[()]/,"",$2) {printf "%7s %7s %-6s %s %s\n", $1, $4, $3, $2, cm}' $i/stat 2>/dev/null)

# парсим stat используя awk, передаем команду через параметр cm, поле №2 (имя процесса ) идет в скобках, заменяем их на пусто

                if [[ -n "$s" ]]
                then
                        echo "$s" >> pps.tmp
                fi
# если полученная строка не пустая скидываем ее во временный файл pps.tmp

        fi
done
cat pps.tmp

# выводим результат на экран

```
> 
> далее включаем ему атрибут "x" для всех и запускаем, результат фиксируем


# Протокол работы
```
usr1@srv2:~$
usr1@srv2:~$
usr1@srv2:~$
usr1@srv2:~$ vi pps
#!/bin/bash


awk 'BEGIN {printf "%7s %7s %-6s %s\n","PID","PPID","STAT","COMMAND"}' > pps.tmp
for i in /proc/[0-9]*
do
        pid=${i##*/}
        if [[ $pid =~ ^[0-9]+$ ]]
        then
                comm=$(tr '\0' ' ' < "$i/cmdline" 2>/dev/null)
                if [[ -n $comm ]]
                then
                        comm=" :${comm}"
                fi
                s=$(awk -v cm="$comm" 'gsub(/[()]/,"",$2) {printf "%7s %7s %-6s %s %s\n", $1, $4, $3, $2, cm}' $i/stat 2>/dev/null)
                if [[ -n "$s" ]]
                then
                        echo "$s" >> pps.tmp
                fi

        fi
done
cat pps.tmp
~
~
~
~
~
~
"pps" 23L, 464B                                        
usr1@srv2:~$
usr1@srv2:~$
usr1@srv2:~$ chmod +x pps
usr1@srv2:~$ ./pps
    PID    PPID STAT   COMMAND
      1       0 S      systemd  :/sbin/init
     10       2 I      kworker/0:0H-events_highpri
   1059       2 S      zvol
   1060       2 S      arc_prune
   1061       2 S      arc_evict
   1062       2 S      arc_reap
   1063       2 S      dbu_evict
   1065       2 S      dbuf_evict
   1079       2 S      z_vdev_file
   1083       2 S      jbd2/sda2-8
   1084       2 I      kworker/R-ext4-
   1091       2 S      l2arc_feed
   1118       2 S      z_vdev_file
   1119       2 S      z_vdev_file
   1120       2 S      z_vdev_file
   1121       2 S      z_vdev_file
   1125       2 S      z_vdev_file
   1126       2 S      z_vdev_file
   1127       2 S      z_vdev_file
   1128       2 S      z_vdev_file
   1129       2 S      z_vdev_file
   1141       2 S      z_vdev_file
     12       2 I      kworker/R-mm_pe
   1295       2 S      z_null_iss
   1296       2 S      z_null_int
   1297       2 S      z_rd_iss
   1298       2 S      z_rd_int
   1299       2 S      z_wr_iss
     13       2 I      rcu_tasks_kthread
   1300       2 S      z_wr_iss_h
   1301       2 S      z_wr_int
   1302       2 S      z_wr_int_h
   1303       2 S      z_fr_iss
   1304       2 S      z_fr_int
   1305       2 S      z_cl_iss
   1306       2 S      z_cl_int
   1307       2 S      z_ioctl_iss
   1308       2 S      z_ioctl_int
   1309       2 S      z_trim_iss
   1310       2 S      z_trim_int
   1311       2 S      z_zvol
   1312       2 S      z_metaslab
   1313       2 S      z_prefetch
   1314       2 S      z_upgrade
   1321       2 S      z_vdev_file
   1322       2 S      z_vdev_file
   1323       2 S      z_vdev_file
   1324       2 S      z_vdev_file
   1325       2 S      z_vdev_file
   1326       2 S      dp_sync_taskq
   1327       2 S      dp_zil_clean_ta
   1328       2 S      dp_zil_clean_ta
   1329       2 S      z_zrele
   1330       2 S      z_unlinked_drai
   1362       2 S      txg_quiesce
   1363       2 S      txg_sync
   1364       2 S      mmp
   1366       2 S      z_indirect_cond
   1367       2 S      z_livelist_dest
   1368       2 S      z_livelist_cond
   1369       2 S      z_checkpoint_di
   1375       2 S      spl_system_task
   1376       2 S      z_null_iss
   1377       2 S      z_null_int
   1378       2 S      z_rd_iss
   1379       2 S      z_rd_int
   1380       2 S      z_wr_iss
   1381       2 S      z_wr_iss_h
   1382       2 S      z_wr_int
   1383       2 S      z_wr_int_h
   1384       2 S      z_fr_iss
   1385       2 S      z_fr_int
   1386       2 S      z_cl_iss
   1387       2 S      z_cl_int
   1388       2 S      z_ioctl_iss
   1389       2 S      z_ioctl_int
   1390       2 S      z_trim_iss
   1391       2 S      z_trim_int
   1392       2 S      z_zvol
   1393       2 S      z_metaslab
   1394       2 S      z_prefetch
   1395       2 S      z_upgrade
     14       2 I      rcu_tasks_rude_kthread
   1402       2 S      dp_sync_taskq
   1403       2 S      dp_zil_clean_ta
   1404       2 S      dp_zil_clean_ta
   1405       2 S      z_zrele
   1406       2 S      z_unlinked_drai
   1438       2 S      txg_quiesce
   1439       2 S      txg_sync
   1440       2 S      mmp
   1441       2 S      z_indirect_cond
   1442       2 S      z_livelist_dest
   1443       2 S      z_livelist_cond
   1444       2 S      z_checkpoint_di
   1448       2 S      z_null_iss
   1449       2 S      z_null_int
   1450       2 S      z_rd_iss
   1451       2 S      z_rd_int
   1452       2 S      z_wr_iss
   1453       2 S      z_wr_iss_h
   1454       2 S      z_wr_int
   1455       2 S      z_wr_int_h
   1456       2 S      z_fr_iss
   1457       2 S      z_fr_int
   1458       2 S      z_cl_iss
   1459       2 S      z_cl_int
   1460       2 S      z_ioctl_iss
   1461       2 S      z_ioctl_int
   1462       2 S      z_trim_iss
   1463       2 S      z_trim_int
   1464       2 S      z_zvol
   1465       2 S      z_metaslab
   1466       2 S      z_prefetch
   1467       2 S      z_upgrade
   1477       2 S      dp_sync_taskq
   1478       2 S      dp_zil_clean_ta
   1479       2 S      dp_zil_clean_ta
   1480       2 S      z_zrele
   1481       2 S      z_unlinked_drai
     15       2 I      rcu_tasks_trace_kthread
   1517       2 S      txg_quiesce
   1518       2 S      txg_sync
   1519       2 S      mmp
   1520       2 S      z_indirect_cond
   1521       2 S      z_livelist_dest
   1522       2 S      z_livelist_cond
   1523       2 S      z_checkpoint_di
   1525       2 S      z_null_iss
   1526       2 S      z_null_int
   1527       2 S      z_rd_iss
   1528       2 S      z_rd_int
   1529       2 S      z_wr_iss
   1530       2 S      z_wr_iss_h
   1531       2 S      z_wr_int
   1532       2 S      z_wr_int_h
   1533       2 S      z_fr_iss
   1534       2 S      z_fr_int
   1535       2 S      z_cl_iss
   1536       2 S      z_cl_int
   1537       2 S      z_ioctl_iss
   1538       2 S      z_ioctl_int
   1539       2 S      z_trim_iss
   1540       2 S      z_trim_int
   1541       2 S      z_zvol
   1542       2 S      z_metaslab
   1543       2 S      z_prefetch
   1544       2 S      z_upgrade
   1551       2 S      dp_sync_taskq
   1552       2 S      dp_zil_clean_ta
   1553       2 S      dp_zil_clean_ta
   1554       2 S      z_zrele
   1555       2 S      z_unlinked_drai
   1593       2 S      txg_quiesce
   1594       2 S      txg_sync
   1595       2 S      mmp
   1596       2 S      z_indirect_cond
   1597       2 S      z_livelist_dest
   1598       2 S      z_livelist_cond
   1599       2 S      z_checkpoint_di
     16       2 S      ksoftirqd/0
    160       2 S      scsi_eh_2
   1601       2 S      z_null_iss
   1602       2 S      z_null_int
   1603       2 S      z_rd_iss
   1604       2 S      z_rd_int
   1605       2 S      z_wr_iss
   1606       2 S      z_wr_iss_h
   1607       2 S      z_wr_int
   1608       2 S      z_wr_int_h
   1609       2 S      z_fr_iss
    161       2 I      kworker/R-scsi_
   1610       2 S      z_fr_int
   1611       2 S      z_cl_iss
   1612       2 S      z_cl_int
   1613       2 S      z_ioctl_iss
   1614       2 S      z_ioctl_int
   1615       2 S      z_trim_iss
   1616       2 S      z_trim_int
   1617       2 S      z_zvol
   1618       2 S      z_metaslab
   1619       2 S      z_prefetch
    162       2 S      scsi_eh_3
   1620       2 S      z_upgrade
   1629       2 S      dp_sync_taskq
    163       2 I      kworker/R-scsi_
   1630       2 S      dp_zil_clean_ta
   1631       2 S      dp_zil_clean_ta
   1632       2 S      z_zrele
   1633       2 S      z_unlinked_drai
    164       2 S      scsi_eh_4
    165       2 I      kworker/R-scsi_
    166       2 S      scsi_eh_5
   1669       2 S      txg_quiesce
    167       2 I      kworker/R-scsi_
   1670       2 S      txg_sync
   1671       2 S      mmp
   1672       2 S      z_indirect_cond
   1673       2 S      z_livelist_dest
   1674       2 S      z_livelist_cond
   1675       2 S      z_checkpoint_di
    168       2 S      scsi_eh_6
    169       2 I      kworker/R-scsi_
     17       2 I      rcu_preempt
    170       2 S      scsi_eh_7
    171       2 I      kworker/R-scsi_
    172       2 S      scsi_eh_8
    173       2 I      kworker/R-scsi_
   1730       1 S      systemd-network  :/usr/lib/systemd/systemd-networkd
   1738       1 S      rpcbind  :/sbin/rpcbind -f -w
   1739       1 S      systemd-resolve  :/usr/lib/systemd/systemd-resolved
    174       2 S      scsi_eh_9
   1740       1 S      systemd-timesyn  :/usr/lib/systemd/systemd-timesyncd
   1747       1 S      blkmapd  :/usr/sbin/blkmapd
    175       2 I      kworker/R-scsi_
   1752       2 I      kworker/R-cfg80
   1756       1 S      nfsdcld  :/usr/sbin/nfsdcld
    176       2 S      scsi_eh_10
    177       2 I      kworker/R-scsi_
    178       2 S      scsi_eh_11
    179       2 I      kworker/R-scsi_
     18       2 S      migration/0
    180       2 S      scsi_eh_12
    181       2 I      kworker/R-scsi_
    182       2 S      scsi_eh_13
    183       2 I      kworker/R-scsi_
    184       2 S      scsi_eh_14
    185       2 I      kworker/R-scsi_
   1873       1 S      dbus-daemon  :@dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
   1877       1 S      fsidd  :/usr/sbin/fsidd
   1881       1 S      polkitd  :/usr/lib/polkit-1/polkitd --no-debug
   1891       1 S      systemd-logind  :/usr/lib/systemd/systemd-logind
   1893       1 S      udisksd  :/usr/libexec/udisks2/udisksd
   1899       1 S      zed  :zed -F
     19       2 S      idle_inject/0
   1908       1 S      cron  :/usr/sbin/cron -f -P
   1926       1 S      rpc.idmapd  :/usr/sbin/rpc.idmapd
   1946       1 S      rsyslogd  :/usr/sbin/rsyslogd -n -iNONE
   1950       1 S      unattended-upgr  :/usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
      2       0 S      kthreadd
     20       2 S      cpuhp/0
   2002       1 S      rpc.statd  :/usr/sbin/rpc.statd
   2013       1 S      agetty  :/sbin/agetty -o -p -- u --noclear - linux
  20485       2 I      kworker/u4:2-flush-252:4
  20486       2 I      kworker/u4:3-flush-252:4
     21       2 S      cpuhp/1
   2103       1 S      nginx  :nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
   2104    2103 S      nginx  :nginx: worker process
   2105    2103 S      nginx  :nginx: worker process
   2188       1 S      rpc.mountd  :/usr/sbin/rpc.mountd
     22       2 S      idle_inject/1
   2207       1 S      ModemManager  :/usr/sbin/ModemManager
   2213       2 I      lockd
   2218       2 I      nfsd
   2219       2 I      nfsd
   2220       2 I      nfsd
   2221       2 I      nfsd
   2222       2 I      nfsd
   2223       2 I      nfsd
   2224       2 I      nfsd
   2225       2 I      nfsd
  22969       1 S      blkmapd  :/usr/sbin/blkmapd = :/usr/sbin/blkmapd
  22972       2 I      kworker/1:2H-kblockd
     23       2 S      migration/1
   2304       1 S      apache2  :/usr/sbin/apache2 -k start
   2306    2304 S      apache2  :/usr/sbin/apache2 -k start
   2308    2304 S      apache2  :/usr/sbin/apache2 -k start
  23088       1 S      rpc.idmapd  :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd
   2309    2304 S      apache2  :/usr/sbin/apache2 -k start
   2310    2304 S      apache2  :/usr/sbin/apache2 -k start
   2311    2304 S      apache2  :/usr/sbin/apache2 -k start
   2312    2304 S      apache2  :/usr/sbin/apache2 -k start
     24       2 S      ksoftirqd/1
  24056       1 S      blkmapd  :/usr/sbin/blkmapd = :/usr/sbin/blkmapd
  24174       1 S      rpc.idmapd  :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd
  25701       1 S      blkmapd  :/usr/sbin/blkmapd = :/usr/sbin/blkmapd
  25814       1 S      rpc.idmapd  :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd
  25917       2 I      kworker/1:2-cgroup_release
  25947       1 S      blkmapd  :/usr/sbin/blkmapd = :/usr/sbin/blkmapd = :/usr/sbin/blkmapd = :/usr/sbin/blkmapd
  25976       1 S      rpc.idmapd  :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd
     26       2 I      kworker/1:0H-kblockd
  26006       1 S      blkmapd  :/usr/sbin/blkmapd = :/usr/sbin/blkmapd = :/usr/sbin/blkmapd = :/usr/sbin/blkmapd
   2601       1 S      fwupd  :/usr/libexec/fwupd/fwupd
  26012       1 S      rpc.idmapd  :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd = :/usr/sbin/rpc.idmapd
   2608       1 S      upowerd  :/usr/libexec/upowerd
     27       2 S      kdevtmpfs
   2767       1 S      sshd  :sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
   2769    2767 S      sshd  :sshd: usr1 [priv]
   2777       2 S      psimon
   2779       1 S      systemd  :/usr/lib/systemd/systemd --user
   2781    2779 S      sd-pam  :(sd-pam)
     28       2 I      kworker/R-inet_
    285       2 I      kworker/R-raid5
    286       2 I      kworker/R-kdmfl
    287       2 I      kworker/R-kdmfl
    288       2 I      kworker/R-kdmfl
     29       2 S      kauditd
    290       2 I      kworker/R-kdmfl
    292       2 I      kworker/R-kdmfl
   2928    2769 S      sshd  :sshd: usr1@pts/0
   2929    2928 S      bash  :-bash
   2941    2767 S      sshd  :sshd: usr1 [priv]
   2988    2941 S      sshd  :sshd: usr1@pts/1
   2989    2988 S      bash  :-bash
      3       2 S      pool_workqueue_release
   3001    2989 S      sudo  :sudo -i
   3002    3001 S      sudo  :sudo -i
   3003    3002 S      bash  :-bash
    304       2 S      mdX_raid1
  30743       2 I      kworker/u4:0-flush-252:4
     31       2 S      khungtaskd
    316       2 I      kworker/R-kdmfl
    317       2 I      kworker/R-kdmfl
  32335       2 I      kworker/0:1-cgroup_release
  32351       2 I      kworker/1:1-cgroup_free
  32384       2 I      kworker/u4:1-events_power_efficient
  32396       2 I      kworker/1:3-events
  32398       2 I      kworker/u4:4-flush-252:4
  32424       2 I      kworker/0:0-cgroup_free
  32431    2929 S      pps  :/bin/bash ./pps
     33       2 S      oom_reaper
     35       2 I      kworker/R-write
     36       2 S      kcompactd0
     37       2 S      ksmd
     38       2 S      khugepaged
     39       2 I      kworker/R-kinte
    397       2 S      jbd2/dm-5-8
    398       2 I      kworker/R-ext4-
      4       2 I      kworker/R-rcu_g
     40       2 I      kworker/R-kbloc
     41       2 I      kworker/R-blkcg
     42       2 S      irq/9-acpi
     43       2 I      kworker/R-tpm_d
   4378       2 I      kworker/1:0-cgroup_release
     44       2 I      kworker/R-ata_s
     45       2 I      kworker/R-md
     46       2 I      kworker/R-md_bi
   4673    2767 S      sshd  :sshd: usr1 [priv]
     47       2 I      kworker/R-edac-
    470       1 S      systemd-journal  :/usr/lib/systemd/systemd-journald
     48       2 I      kworker/R-devfr
   4815    4673 S      sshd  :sshd: usr1@pts/3
   4816    4815 S      bash  :-bash
   4862       2 I      kworker/R-tls-s
   4869       2 I      kworker/0:2-events
     49       2 S      watchdogd
      5       2 I      kworker/R-rcu_p
     50       2 I      kworker/R-quota
    505       2 I      kworker/R-rpcio
    506       2 I      kworker/R-xprti
    507       2 I      kworker/R-kmpat
    508       2 I      kworker/R-kmpat
     52       2 S      kswapd0
     53       2 S      ecryptfs-kthread
    536       1 S      multipathd  :/sbin/multipathd -d -s
     54       2 I      kworker/R-kthro
    540       1 S      systemd-udevd  :/usr/lib/systemd/systemd-udevd
    542       1 S      dmeventd  :/usr/sbin/dmeventd -f
     55       2 I      kworker/R-acpi_
     56       2 S      scsi_eh_0
    565       2 S      psimon
     57       2 I      kworker/R-scsi_
     58       2 S      scsi_eh_1
     59       2 I      kworker/R-scsi_
      6       2 I      kworker/R-slub_
     62       2 I      kworker/R-mld
     63       2 I      kworker/R-ipv6_
    648       2 S      irq/18-vmwgfx
    649       2 I      kworker/R-ttm
     65       2 I      kworker/0:1H-kblockd
      7       2 I      kworker/R-netns
     72       2 I      kworker/R-kstrp
     74       2 I      kworker/u5:0
    774       2 S      jbd2/dm-6-8
    775       2 I      kworker/R-ext4-
     79       2 I      kworker/R-crypt
     89       2 I      kworker/R-charg
    894       2 S      jbd2/dm-4-8
    895       2 I      kworker/R-ext4-
    923       2 S      spl_system_task
    924       2 S      spl_delay_taskq
    925       2 S      spl_dynamic_tas
    926       2 S      spl_kmem_cache
usr1@srv2:~$
```
