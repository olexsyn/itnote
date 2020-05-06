## Как правильно редактировать файл /etc/fstab

<http://rus-linux.net/lib.php?name=/MyLDP/file-sys/fstab.html> (2007)

Оригинал: [How to edit and understand /etc/fstab](http://www.tuxfiles.org)

### Что такое файл fstab и для чего он нужен

Один из конфигурационных файлов в Линукс-системах носит имя **fstab**. Он содержит информацию обо всех разделах жесткого диска и других носителях информации в компьютере. Полный путь к нему **/etc/fstab**.

В **fstab** прописано, куда и как разделы винчестера и другие носители должны быть примонтированы. Если вы не имеете доступа к Windows разделу, не можете примонтировать CD, не в состоянии записать, как рядовой пользователь, файл на дискету, или испытываете трудности с CD-RW, то, скорее всего, у вас неверно сконфигурирован **fstab**.

Редактируя этот файл, обычно решают все проблемы с монтированием.

**fstab** - это обычный текстовый файл, поэтому его можно редактировать в любом текстовом редакторе. Единственное требование - наличие прав суперпользователя. Так что, прежде чем приступать, войдите в систему как root или используйте команду `su`, чтобы получить права _root_.

### Как выглядит файл /etc/fstab

В каждой конкретной системе файл /etc/fstab выглядит не так, как в другой, ведь разделы, устройства, и их свойства, различаются в разных системах. Но скелет структуры файла всегда одинаков. Вот пример содержимого файла /etc/fstab:

```
/dev/hda2   /               ext2  defaults             1 1

/dev/hdb1   /home           ext2  defaults             1 2

/dev/cdrom  /media/cdrom    auto  ro,noauto,user,exec  0 0

/dev/fd0    /media/floppy   auto  rw,noauto,user,sync  0 0

proc        /proc           proc  defaults             0 0

/dev/hda1   swap            swap  pri=42               0 0
```

Как легко заметить, каждая строка содержит информацию об одном разделе или устройстве. Первый столбец содержит имя устройства, второй - точку его монтирования, третий - тип файловой системы, четвертый - опции монтирования, пятый (число) - опции дампа, шестой (число) опции проверки файловой системы. Давайте подробно рассмотрим всю эту информацию.

### Первый и второй столбцы: Устройство и точка монтирования

Они содержат ровно то же самое, что вы пишете в командной строке, когда даете команду `mount`, то есть имя устройства (раздела) и точку его монтирования. Точка монтирования, указанная в **fstab**, является точкой монтирования по умолчанию. Эта та директория, куда будет примонтировано устройство, если вы не указали другой, когда давали команду  mount.

Большинство дистрибутивов Линукса создают специальные директории для точек монтирования. Большинство дистрибутивов создают их в каталоге **/mnt**, некоторые (в том числе и SuSE), в каталоге **/media**.

Что все это означает практически? Если я дам команду:

```$ mount /dev/fd0/```

...то моя дискета будет смонтирована в **/media/floppy**, потому что эта точка монтирования указана в **fstab** и поэтому используется по умолчанию.

Если строчки `/dev/fd0` в файле **fstab** не окажется, то команда `mount` не будет знать, куда следует монтировать дискету.

Точки монтирования по умолчанию легко изменить, если они вас почему-либо не устраивают. Для этого нужно заменить директории в файле **fstab** на любые другие, реально существующие директории. Если подходящих не существует, то просто создайте их.

Некоторые разделы и устройства монтируются автоматически, в процессе загрузки системы. Взгляните на приведенный пример.

```
/dev/hda2   /       ext2   defaults    1 1
/dev/hdb1   /home   ext2   defaults    1 2
```

Эти строки означают, что `/dev/hda2` будет примонтирован в директорию **/**, а `/dev/hdb1` - в директорию **/home**. Это произойдет автоматически, когда система загружается.

### Третий столбец: Файловая система

Третий столбец файла **fstab** указывает тип файловой системы раздела или устройства. Поддерживается множество различных файловых систем, но мы рассмотрим только наиболее распрострненные.

- **ext2** и **ext3** - c большой вероятностью ваши Линукс-разделы отформатированы в Ext3 (![!](/i/w.png) - внимание, статья 2007 г.) Раньше стандартом была система Ext2, но в наши дни почти все дистрибутивы используют по умолчанию Ext3 или ReiserFS. Ext3 более современная система, чем Ext2 и отличается от нее своей журналируемостью. Это, в практическом плане, означает, что, если вы обесточите ваш компьютер, вместо того, чтобы выключить его по всем правилам, то вы не потеряете информацию, и не будете долго ждать при следующем включении, пока ваш компьютер проверяет файловую систему.
- **reiserfs** -вполне возможно, что ваши Линукс-разделы отформатированы в  ReiserFS. Подобно Ext3,  ReiserFS тоже журналируемая файловая система, но она является гораздо более "продвинутой". Многие дистрибутивы Линукс используют ReiserFS по умолчанию.
- **swap** - раздел подкачки.
- **vfat** и **ntfs** - Windows разделы используют либо Vfat, либо NTFS. В 9х сериях (95, 98, МЕ) применялась Vfat, более известная как FAT32, в сериях NT (NT, 2000, XP) используется NTFS. В 2000 и XP можно применять и Vfat тоже. Если вы хотите иметь возможность писать в свои Windows-разделы из Линукса, советую отформатировать их в Vfat, потому что в Линуксе запись в NTFS-разделы до сих пор может причинить головную боль.
- **auto** - нет, это не тип файловой системы! Опция "auto" означает, что тип файловой системы определяется автоматически. Если снова взглянете на пример файла fstab, приведенный выше, то увидите, что и floppy и CD-ROM - оба - имеют вместо типа файловой системы опцию "auto". Почему? - Дело в том, что в этих устройствах могут применяться различные типы файловых систем. Одна дискета может быть отформатирована для Windows, другая - для Линукс (Ext2). Довольно разумно позволить системе самой определить тип файловой системы на носителях вроде дискет и оптических дисков.

### Четвертый столбец: Опции монтирования

В четвертом столбце перечислены все опции, с которыми устройство или раздел будут смонтированы. По совместительству, это еще и самый сложный для заполнения столбец, но, зная некоторые самые употребительные опции, вы избежите большинства недоразумений. Я рассмотрю только наиболее широко распространенные опции, а по поводу остальных -  смотрите ман-страницу mount.

- **auto** и **noauto** - если задана опция `auto`, то устройство будет смонтировано автоматически во время запуска компьютера (или по команде  mount -a). Эта опция включена по умолчанию. Если вам не нужно, чтобы устройство монтировалось автоматически, вы должны прописать опцию `noauto` в **fstab**. С опцией `noauto`, устройство или раздел могут быть смонтированы только явно.
- **exec** и **noexec** - если хотите запускать программы, которые находятся в данном разделе, то применяйте опцию `exec`, а если нет - то `noexec`. Последнее может быть полезно, если на разделе содержатся программы, которые не могут работать в вашей системе, например Windows-приложения, либо программы, нежелательные к запуску по той или иной причине. Опция `exec` включена по умолчанию.
- **ro** - монтирует файловую систему в режиме "только чтение".
- **rw** - монтирует файловую систему в режиме "чтение и запись".
- **sync** и **async** - эти опции определяют как осуществляется ввод/вывод в данную файловую систему: синхронно или асинхронно. Обратите внимание, что в примере опция `sync` применена с дискетой. Попросту говоря, когда вы копируете файл на дискету, то запись физически происходит в тот самый момент, когда дана команда копировать. Если же применяется опция `async` ввод и вывод происходят неодновременно (асинхронно). В случае с дискетой это означает, что физически запись может произойти много позже команды. В этом нет ничего плохого, и во многих случаях даже предпочтительно, но может иметь неприятные побочные следствия: если вытащить дискету из дисковода, не отмонтировав ее, скопированного файла на ней может не оказаться. По умолчанию применяется опция `async`. Но, может быть, стоит для дискеты прописать `sync`, особенно если вы привыкли вытаскивать неотмонтированные дискеты, подобно тому, как это делается в Windows.
- **defaults** - по умолчанию включены следующие опции: `rw, suid, dev, exec, auto, nouser, async`.


### Пятый и шестой столбцы: Опции dump и fsck

Дамп - это опция резервного копирования, а fsck - опция проверки файловой системы. Я не стану слишком много о них распространяться, так как для этого
может понадобиться отдельная статья, но скажу несколько слов, чтобы вы не гадали, что они могут означать.

Пятый столбец файла **fstab** - это опция дампа, выраженная числом. От значения этого числа зависит, будет ли создаваться резервная копия данной
файловой системы. Если это ноль, программа dump проигнорирует такую файловую систему. Как видно из примера, в большинстве строк в пятом столбце нули.

В шестой колонке опция программы fsck (filesystem check- проверка файловой системы). Программа fsck использует значение чисел в этом столбце, чтобы определить, в каком порядке проверять файловые системы. Если там ноль, то файловая система вообще не будет проверяться.

## Примеры записей в файл fstab

Для примера мы разберем два случая, которые чаще прочих расстраивают новых пользователей Линукса: дискета и CD-ROM (хотя дискеты в последнее время употребляются все реже).

```
/dev/fd0 /media/floppy auto rw,noauto,user,sync 0 0
```

Эта строка означает, что дискета монтируется по умолчанию с директорию /media/floppy и что тип файловой системы при этом определяется автоматически. Это полезно, так как тип файловой системы на дискетах может быть различным. Особое внимание обратите на опции <b>rw</b> и <b>user</b>: они обязательно должны быть прописаны, если вы хотите монтировать дискету и записывать на нее, будучи рядовым пользователем. Если это не получается, проверьте файл /etc/fstab на предмет наличия этих опций. Еще обратите внимание на опцию <b>sync</b>. С таким же успехом может быть и async, по причинам, которые мы уже обсудили.

```
/dev/cdrom /media/cdrom auto ro,noauto,user,exec 0 0
```

Снова отметьте опцию <b>user</b>, позволяющую рядовому пользователю монтировать компакт диски. Опция ro установлена потому, что нет смысла монтировать CD-ROM в режиме "чтение-запись", ведь на него все равно ничего не запишешь. А вот опция `exec` очень кстати, если надо запустить какую-либо программу с компакт-диска.

Обратите также внимание на применение опции `noauto` как с дискетой, так и с CD-ROM, это означает, что они не будут автоматически смонтированы при запуске системы. Это очень разумно для съемных носителей, которых при запуске может просто не быть в дисководах, ведь нет смысла пытаться монтировать то, чего нет.

{% include f.htm f="article01.md" %}