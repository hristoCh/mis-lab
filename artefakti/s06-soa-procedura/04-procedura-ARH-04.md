# Процедура: Достъп до папката с резервни копия само за група arhiv (ARH-04)

| Поле | Стойност |
|---|---|
| Реализира контрол | ARH-04 (ограничаване на достъпа до копията) |
| Annex A | 8.2, 8.3, 5.15 |
| R-ID | R-04 |
| Изпълнява се от | Системен администратор |
| Честота | При първоначална настройка и при всеки нов сървър за архивиране. Проверка на правата: на всеки 3 месеца, от ИТ мениджър. |
| Версия, дата | 0.1, 04.11.2026 |

## Предусловия

| Какво | Стойност |
|---|---|
| Достъп | Акаунт student на srv с право на sudo. |
| Инструменти | sudo, groupadd, useradd, usermod, chown, chmod, stat, id, getent (вградени в Debian 13). |
| Снапшот преди започване | pre-h06. Преди изпълнение от съученик srv е върнат в този снапшот: няма група arhiv, папка /srv/arhiv и акаунт erp. |
| Втора сесия | Не е нужна: контролът не променя входа в сървъра. |

## Стъпки

| # | Действие | Команда | Очакван резултат |
|---|---|---|---|
| 1 | Запиши състоянието преди | `mkdir -p ~/h06 && { date; stat -c '%A %U:%G %n' /srv/arhiv; getent group arhiv; id erp; } > ~/h06/predi.txt 2>&1; cat ~/h06/predi.txt` | В predi.txt има дата, съобщение „cannot statx '/srv/arhiv': No such file or directory“ и „id: ‘erp’: no such user“. Групата не се показва. |
| 2 | Създай групата за достъп | `sudo groupadd arhiv` | Няма изход. |
| 3 | Създай папката за копията | `sudo mkdir /srv/arhiv` | Няма изход. |
| 4 | Задай собственик и права | `sudo chown root:arhiv /srv/arhiv && sudo chmod 2770 /srv/arhiv` | Няма изход. |
| 5 | Добави администратора в групата | `sudo usermod -aG arhiv student` | Няма изход. Текущата сесия още няма новата група, затова тестовете са със sudo -u. |
| 6 | Създай акаунт, който представя ERP, извън групата | `sudo useradd -r -M -s /usr/sbin/nologin erp` | Няма изход. |
| 7 | Запиши състоянието след (от работещата система) | `{ date; stat -c '%A %U:%G %n' /srv/arhiv; getent group arhiv; id erp; } > ~/h06/sled.txt 2>&1; cat ~/h06/sled.txt` | В sled.txt: „drwxrws--- root:arhiv /srv/arhiv“, „arhiv:x:<номер>:student“ и „uid=<номер>(erp) gid=<номер>(erp) groups=<номер>(erp)“. erp не е в групата arhiv. |

## Проверка

Положителна (разрешеното работи):

```
sudo -u student bash -c 'echo test > /srv/arhiv/test-admin.txt' && sudo -u student ls -l /srv/arhiv
```

Очаквано: Редът „-rw-rw-r-- 1 student arhiv ... test-admin.txt“. Групата на файла е arhiv заради бита setgid върху папката.

Отрицателна (забраненото се отказва):

```
{ date; sudo -u erp ls /srv/arhiv; sudo -u erp touch /srv/arhiv/erp-test.txt; sudo -u erp rm /srv/arhiv/test-admin.txt; } > ~/h06/otkaz.txt 2>&1; cat ~/h06/otkaz.txt
```

Очаквано: Изпълнява се на srv, като нова команда с акаунта erp. В otkaz.txt има три реда с „Permission denied“: „ls: cannot open directory '/srv/arhiv': Permission denied“, „touch: cannot touch '/srv/arhiv/erp-test.txt': Permission denied“ и „rm: cannot remove '/srv/arhiv/test-admin.txt': Permission denied“. После „sudo -u student ls /srv/arhiv“ още показва test-admin.txt.

## При отклонение

Ако някоя стъпка даде различен резултат (например „group arhiv already exists“ или erp успее да прочете папката), спри и не продължавай към проверката. Запиши командата и съобщението, провери „stat /srv/arhiv“ и „id erp“. Ако правата и групите са верни, а erp пак чете папката, върни снапшота pre-h06 и започни отначало. Докладвай на ИТ мениджър в същия работен ден.

## Възстановяване (Rollback)

От обикновената сесия на srv: sudo rm -f /srv/arhiv/test-admin.txt && sudo rmdir /srv/arhiv && sudo userdel erp && sudo gpasswd -d student arhiv && sudo groupdel arhiv. Ако нещо не стане, върни снапшота pre-h06.

## Запис

Доказателствата са predi.txt, sled.txt и otkaz.txt в ~/h06 на srv. Копират се в хранилището от Gateway с: scp student@192.168.56.3:~/h06/*.txt ~/mis-lab/artefakti/s06-soa-procedura/ Пазят се в artefakti/s06-soa-procedura/ най-малко 12 месеца, както е в политиката за ARH-04.

## Проверка от съученик

| Поле | Стойност |
|---|---|
| Изпълнил, дата | |
| Засякъл на стъпка | |
| Неясно | |
| Отрицателният тест даде | |
| Поправено след проверката | |
