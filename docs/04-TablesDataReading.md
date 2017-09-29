# Чтение и обработка таблиц {#tables}

Данный модуль посвящен введению в работу с таблицами. В модуле рассмотрены важные процедуры предварительной обработки таблиц, такие как фильтрация, исправление ошибок, преобразование типов --- необходимые для дальнейшей визуализации и анализа данных

Выполните установку и подключение пакета `openxlsx`.




```r
library(openxlsx)
```

## Установка рабочей директории {#set_directory}

Прежде чем мы начнем работать с данными, необходимо установить рабочую директорию. Это папка, в которой лежат необходимые вам таблицы. Для этого используем функцию `setwd()`. Аргумент функции нужно заменить на адрес каталога на вашем компьютере, в который вы положили присланные файлы:

```r
setwd("/Volumes/Data/GitHub/r-geo-course/data")
```

> Внимание: если вы выполняете данный модуль на своем компьютере, замените вышеуказанный путь на путь к каталогу, в который вы положили исходные данные.

Теоретически рабочую директорию можно и не устанавливать, однако тогда при чтении файлов вам придется каждый раз указывать полный путь, что неудобно.

> Рабочая директория и местоположение скрипта могут не совпадать. Вы можете хранить их в разных местах. Однако рекомендуется держать их вместе, что облегчит передачу вашей программы вместе с данными другим пользователям.

## Чтение таблиц CSV {#reading_csv}

Таблицы в формате __CSV__ (Comma-Separated Values) можно прочесть с помощью универсальной функции `read.table()`. Следующие ее параметры важно указать:

- `file` --- название файла
- `sep` --- разделитель ячеек
- `dec` --- десятичный разделитель
- `header` --- содержится ли в первой строке заголовок
— `encoding` --- кодировка символов, в которой сохранен файл (чаще всего `UTF-8` или `CP1251`)

> Стандартной кодировкой для представления текста в UNIX-подобных системах (_Ubuntu_, _macOS_ и т.д.) является __UTF-8 (Unicode)__, в русскоязычных версиях _Windows_ --- __CP1251 (Windows-1251)__. Текстовый файл __CSV__, созданный в разных операционных системах, будет по умолчанию сохраняться в соответствующей кодировке, если вы не указали ее явным образом. Если при загрузке таблицы в __R__ вы видите вместо текста нечитаемые символы --- _кракозябры_ --- то, скорее всего, вы читаете файл не в той кодировке, в которой он был сохранен. Попробуйте поменять `UTF-8` на `CP1251` или наоборот. Если вы не знаете, что такое кодировка и Юникод, то вам [сюда](https://ru.wikipedia.org/wiki/Набор_символов).

Прочтем таблицу с данными Росстата по объему сброса сточных вод в бассейны некоторых морей России (в млн. м$^3$):

```r
tab <- read.table("oxr_vod.csv",
                  sep = ';',
                  dec = ',',
                  header = TRUE,
                  encoding = 'UTF-8')
```

Для просмотра таблицы в привычном виде воспользуйтесь функцией `View()`. В этом представлении вы можете фильтровать и сортировать данные:

```r
View(tab)
```

![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-4-1.png)<!-- -->

Существуют более специальные функции для чтения таблиц __CSV__: `read.csv()` и `read.csv2()`. По сути они являются "обертками" (_wrappers_) функции `read.table()` и выполняют ее вызов с автоматической подстановкой параметров `sep`, `dec` и `header`. Обе функции по умолчанию предполагают, что в файле имеется заголовок. `read.csv()` удобна для чтения таблиц с десятичной точкой и запятой-разделителем, а `read.csv2()` --- для таблиц с десятичной запятой и точкой-с-запятой в качестве разделителя.

Используем для чтения `read.csv2()`:

```r
tab2<-read.csv2("oxr_vod.csv", encoding = 'UTF-8')
```

```r
View(tab2)
```

![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-7-1.png)<!-- -->
Как видно, данная таблица не отличается от предыдущей, но ее чтение более компактно.

## Фильтрация, сортировка, работа с элементами таблицы {#filtering_sorting}

Распространенные операции с таблицами — это упорядочение по определенному столбцу и фильтрация по значениям. Мы уже знаем что из вектора, матрицы или таблицы можно извлекать элементы: `tab[V, ]`, где `tab` --- имя таблицы, `V` --- это вектор из номеров элементов. Например, извлечь 5, 2 и 4 строку таблицы можно так:

```r
tab[c(5,2,4), ]
##    Год Всего Балтийское Черное Азовское Каспийское Карское Белое Прочие
## 5 1997  23.0        2.2    0.3      3.8        9.8     4.4   0.8    1.7
## 2 1994  24.6        2.3    0.4      3.2       11.0     5.0   0.9    1.8
## 4 1996  22.4        2.2    0.3      3.1        9.8     4.7   0.8    1.5
```
Логично предположить, что таким же образом можно извлечь элементы таблицы в порядке, обеспечивающем возрастание или убывание значений в каком-то столбце. Для этого нужно правильным образом расставить индексы в векторе `c(...)`. Существует специальная функция `order()`, которая позволяет это сделать. Например, отсортируем таблицу по возрастанию сбросов в Каспийское море:

```r
indexes<-order(tab$Каспийское)
head(tab[indexes, ])
##     Год Всего Балтийское Черное Азовское Каспийское Карское Белое Прочие
## 22 2014  14.8        1.7    0.2      1.5        6.4     3.2   0.6    1.2
## 17 2009  15.9        1.8    0.2      1.5        6.8     3.5   0.7    1.4
## 21 2013  15.2        1.8    0.2      1.6        6.9     3.0   0.6    1.1
## 20 2012  15.7        1.8    0.2      1.6        7.0     3.0   0.7    1.4
## 19 2011  16.0        1.9    0.2      1.6        7.1     3.2   0.7    1.3
## 18 2010  16.5        2.0    0.2      1.6        7.3     3.3   0.7    1.4
```

> Используйте функцию `head()`, чтобы отобразить первые несколько строк таблицы. Эта возможность особенно полезна при работе с большими таблицами

Если упорядочение несложное, программист его скорее всего вставит непосредственно в инструкцию обращения к таблице:

```r
head(tab[order(tab$Каспийское), ])
##     Год Всего Балтийское Черное Азовское Каспийское Карское Белое Прочие
## 22 2014  14.8        1.7    0.2      1.5        6.4     3.2   0.6    1.2
## 17 2009  15.9        1.8    0.2      1.5        6.8     3.5   0.7    1.4
## 21 2013  15.2        1.8    0.2      1.6        6.9     3.0   0.6    1.1
## 20 2012  15.7        1.8    0.2      1.6        7.0     3.0   0.7    1.4
## 19 2011  16.0        1.9    0.2      1.6        7.1     3.2   0.7    1.3
## 18 2010  16.5        2.0    0.2      1.6        7.3     3.3   0.7    1.4
```
Схожим образом реализована _фильтрация данных_ по значению. Например, вы хотите извлечь из таблицы только те года, в которых объем сбросов в Каспийское море составил более 10 млн м$^3$. Здесь используется еще одна возможность извлечения элементов таблицы — с помощью вектора логических значений `TRUE/FALSE`. Число элементов в этом векторе должно быть равно числу элементов в индексируемом векторе, а значение указывает на то, нужно ли извлекать (`TRUE`) или нет (`FALSE`) элемент с текущим индексом. Вектор логических значений получается естественным путем с помощью операции сравнения:

```r
condition <- tab$Каспийское > 10
condition  # посмотрим что получилось
##  [1]  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
tab[condition, ] # используем его для фильтрации строк таблицы:
##    Год Всего Балтийское Черное Азовское Каспийское Карское Белое Прочие
## 1 1993  27.2        2.5    0.4      4.3       12.1     5.3   1.0    1.6
## 2 1994  24.6        2.3    0.4      3.2       11.0     5.0   0.9    1.8
## 3 1995  24.5        2.3    0.4      3.5       10.4     5.2   0.9    1.8
```
Опять же, весьма часто используется запись одним выражением:

```r
tab[tab$Каспийское > 10, ]
##    Год Всего Балтийское Черное Азовское Каспийское Карское Белое Прочие
## 1 1993  27.2        2.5    0.4      4.3       12.1     5.3   1.0    1.6
## 2 1994  24.6        2.3    0.4      3.2       11.0     5.0   0.9    1.8
## 3 1995  24.5        2.3    0.4      3.5       10.4     5.2   0.9    1.8
```
Можно создать новую таблицу, выбрав необходимые столбцы:

```r
caspian <- data.frame(tab$Год, tab$Всего, tab$Каспийское)
```
Следует заметить, что не рекомендуется использовать кириллические названия столбцов (см. [Правила подготовки таблиц для чтения в R] в конце данного модуля), поэтому переименуем их:

```r
colnames(caspian)<-c("Year", "Total", "Caspian")
```
Предположим, что теперь нам необходимо вычислить долю сбросов в Каспийское море в общем объеме и записать ее в новый столбец с точностью до 3 знаков после запятой. Для этого сначала произведем вычисления:

```r
ratio <- caspian$Caspian / caspian$Total
ratio
##  [1] 0.4448529 0.4471545 0.4244898 0.4375000 0.4260870 0.4318182 0.4396135
##  [8] 0.4532020 0.4494949 0.4646465 0.4421053 0.4486486 0.4519774 0.4457143
## [15] 0.4302326 0.4385965 0.4276730 0.4424242 0.4437500 0.4458599 0.4539474
## [22] 0.4324324
```
Далее округлим результат с помощью функции `round()` с параметром `digits`, указывающим число значащих цифр в ответе:

```r
ratio <- round(ratio, digits = 3)
ratio
##  [1] 0.445 0.447 0.424 0.438 0.426 0.432 0.440 0.453 0.449 0.465 0.442
## [12] 0.449 0.452 0.446 0.430 0.439 0.428 0.442 0.444 0.446 0.454 0.432
```
Существует простой и элегантный способ создать новый столбец в таблице --- достаточно указать его название после значка `$`. Если среда R не обнаруживает столбец с таким названием, она его создаст:

```r
caspian$CaspianRatio <- ratio
```

```r
View(caspian)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-19-1.png)<!-- -->
К столбцу таблицы можно обращаться по номеру, а не названию. Если вы указываете в квадратных скобках номер без запятой, он трактуется как номер столбца. При этом возвращаемый столбец имеет тип `data.frame`:

```r
head(caspian[2])  # второй столбец (фактически — таблица из одного столбца)
##   Total
## 1  27.2
## 2  24.6
## 3  24.5
## 4  22.4
## 5  23.0
## 6  22.0
head(caspian[c(1,4)])  # первый и четвертый столбец
##   Year CaspianRatio
## 1 1993        0.445
## 2 1994        0.447
## 3 1995        0.424
## 4 1996        0.438
## 5 1997        0.426
## 6 1998        0.432
```
В противоположность этому, более привычная форма обращения к двумерным данным через запятую приведет к тому, что столбец будет возвращен как вектор:

```r
caspian[,2]
##  [1] 27.2 24.6 24.5 22.4 23.0 22.0 20.7 20.3 19.8 19.8 19.0 18.5 17.7 17.5
## [15] 17.2 17.1 15.9 16.5 16.0 15.7 15.2 14.8
```
Использование той или иной формы зависит от контекста.

## Чтение таблиц Microsoft Excel {#excel_reading}

Чтение таблиц __Microsoft Excel__ производится с помощью функции `read.xlsx()` из пакета `openxlsx`. В качестве обязательных параметров они принимают следующие аргументы:

- `xlsxFile` — название файла
- `sheet` — номер листа

Убедитесь, что у вас установлена и подключена библиотека `openxlsx`.

Откроем таблицу с данными Росстата по сбросу загрязненных сточных вод в поверхностные водные объекты (млн м$^3$).

```r
sewage<-read.xlsx("sewage.xlsx",1) # Читаем таблицу из первого листа
## Warning in read.xlsx.default("sewage.xlsx", 1): '.Random.seed' не является
## целочисленным вектором, он типа 'NULL', и поэтому пропущен
```

```r
View(sewage)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-24-1.png)<!-- -->
Следует дать адекватные названия столбцам таблицы:

```r
colnames(sewage) <- c("Region", "Year05", "Year10", "Year11", "Year12", "Year13")
```

```r
View(sewage)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-27-1.png)<!-- -->

## Пропущенные значения {#missed_values}

Можно ли осуществлять обработку таблицы `sewage`? Попробуем в качестве примера найти минимум сбросов за 2012 год:

```r
max(sewage$Year12)
## [1] NA
```
Результат имеет тип `NA`, потому что в данном столбце имеются пропуски. В некоторых статистических задачах это недопустимо. Если вы хотите проигнорировать значения пропусков, следует в вызываемой статистической функции указать дополнительный параметр `na.rm = TRUE`:

```r
max(sewage$Year13, na.rm = TRUE)
## [1] 15189
```
Еще один вариант --- исключить из таблицы те строки, в которых имеются пропущенные значения (хотя бы одно!). Для этого существует функция `complete.cases()`, возвращающая вектор логических значений:

```r
filter<-complete.cases(sewage)
filter  # посмотрим что получилось. Там где видим FALSE - есть пропуски в строках
##  [1]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [12]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [23]  TRUE  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [34]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [45]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [56]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [67]  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [78]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [89]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE FALSE FALSE

sewage.complete <- sewage[filter, ] # отфильтруем полные строки
```

```r
View(sewage.complete)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-32-1.png)<!-- -->

## Фильтрация по текстовым полям {#filtering_text}

Часто бывает необходимо отобрать данные из таблицы, содержащей разнородные данные. В частности, в нашей таблице смешаны данные по субъектам и федеральным округам. Предположим, необходимо выгрузить в отдельную таблицу данные по федеральным округам. Для этого нужно найти строки, в которых столбец `Region` содержит фразу `"федеральный округ"`. Для поиска по текстовым эталонам импользуется функция `grep()`, выдающая номера элементов, или ее разновидность `grepl()`, выдающая список логических констант

```r
# Первый параметр - искомое выражение, второй параметр - где искть
rows <- grep("федеральный округ",sewage$Region)
rows  # посмотрим, какие элементы столбца Region ему соответствуют
## [1]  2 21 35 42 49 64 73 86
okruga <- sewage[rows,] # отфильтруем найденные строки
```

```r
View(okruga)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-35-1.png)<!-- -->
Наоборот --- для __исключения__ найденных объектов удобнее воспользоваться разновидностью `grepl()`, которая возвращает вектор из логических значений:

```r
rows2 <- grepl("федеральный округ",sewage$Region)
rows2 # вот так выглядит результат grepl
##  [1] FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE
## [23] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [34] FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
## [45] FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
## [56] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
## [67] FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE
## [78] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
## [89] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE

neokruga <- sewage[!rows2,]
```

```r
View(neokruga)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-38-1.png)<!-- -->
Обратите внимание на восклицательный знак перед `rows2`. Он меняет все значения `TRUE` на `FALSE` и наоборот, что позволяет исключить найденные объекты

В полученной таблице все еще содержится текстовая шелуха типа `"в том числе"`, `"Данные за..."`, а также строка `"Российская Федерация"`. К счастью, функция `grep()` достаточо умна и возволяет искать сразу по нескольким образцам строк. Для этого их нужно разделить вертикальной чертой — _пайпом_ (`|`):

```r
rows2 <- grepl("федеральный|числе|Российская|за|ѕ",sewage$Region)
rows2
##  [1]  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE
## [23] FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [34] FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
## [45] FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
## [56] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
## [67] FALSE  TRUE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE
## [78] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
## [89] FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE
neokruga <- sewage[!rows2,] # обратите внимание на восклицательный знак перед rows2
```

```r
View(neokruga)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-41-1.png)<!-- -->

## Преобразование типов данных и исправление ошибок {#data_conversion}

Достаточно часто при работе с реальными данными возникает необходимость преобразования их типов. Например, вам необходимо перевести строки в даты, чтобы оперировать ими соответствующим образом. Или принудительным образом указать, что столбец со строками не хранит номинальную переменную (фактор), а его нужно интерпретировать именно как строковый столбец (обычно это полезно, когда столбец содержит какую-то текстовую информацию в виде комментариев по каждому измерению). Наконец, в данных могут быть ошибки, опечатки и так далее, которые могут препятствовать правильному их чтению.

В этом разделе мы рассмотрим, как можно:

1. Найти и исправить множественные варианты одного названия с опечатками
2. Исправить ошибки в числовых данных
3. Преобразовать факторы в строки и наоборот
4. Преобразовать строки в числа и наоборот

Рассмотрим возможные манипуляции с данными на примере таблицы о землепользовании на территории Сатинского учебного полигоны Географического факультета МГУ:

```r
tab <- read.csv2("SatinoLanduse.csv", encoding = 'UTF-8')
str(tab) # посмотрим, какова структура данных
## 'data.frame':	160 obs. of  6 variables:
##  $ ID            : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ Type          : Factor w/ 12 levels "Выгоны","Вырубки",..: 11 1 1 9 5 1 5 12 9 10 ...
##  $ Administration: Factor w/ 7 levels "","РФ","Совьяковская администрация",..: 5 5 5 1 1 5 1 5 5 6 ...
##  $ Comment       : Factor w/ 35 levels "","АО \"Родина\"",..: 30 1 1 1 1 1 1 3 2 1 ...
##  $ Perimeter     : Factor w/ 160 levels "1014.155593894044800",..: 67 155 51 104 78 153 17 19 108 57 ...
##  $ Area          : Factor w/ 160 levels "0.238070145845919",..: 73 49 121 100 63 72 88 128 99 24 ...
```

```r
View(tab)
```
![](04-TablesDataReading_files/figure-epub3/unnamed-chunk-44-1.png)<!-- -->
Видно, что все столбцы, кроме двух, хранящих идентификаторы, были прочитаны как строки и преобразованы в факторы (номинальные переменные). Это означает, что мы не сможем работать привычным образом со столбцами периметра и площади, а столбец комментариев теперь также является номинальной переменной, что противоречит здравому смыслу (он вообще переменной не является).

Когда вы отображаете таблицу в консоли или графическом интерфейсе, факторы выглядят и ведут себя как обычные строки. Подвох заключается в том, что хранятся они в виде пар "ключ — значение" (об этом мы говорили выше) и все операции преобразования осуществляются __над ключами__, а не значениями. Рассмотрим, как следует правильно преобразовывать номинальные переменные в __R__.

Чтобы привести столбцы к нужному типу, необходимо использовать преобразования типов. Для этого в __R__ существует множество функций семейства `as(object,class)`, где в качестве первого параметра `object` вы указываете преобразуемый объект, а в качестве второго параметра `class` --- тип, к которому вы хотите его привести. Например:

```r
s <- "5456.788"
s + 1
## Error in s + 1: нечисловой аргумент для бинарного оператора
n <- as(s, "numeric")
## Error in as(s, "numeric"): не могу найти функцию "as"
n + 1
## Error in eval(expr, envir, enclos): объект 'n' не найден
s <- as(n, "character")
## Error in as(n, "character"): не могу найти функцию "as"
s
## [1] "5456.788"
nchar(s)
## [1] 8
```
На практике обычно пользуются не функцией `as()`, а ее обертками (_wrappers_), которые имеют вид `as.numeric()`, `as.character()`, `as.Date()` и так далее:

```r
as.numeric(s) # то же самое, что и as(s, "numeric")
## [1] 5456.788
```
Для начала преобразуем столбец `Comment` к обычному символьному представлению:

```r
tab$Comment <- as.character(tab$Comment)
str(tab)
## 'data.frame':	160 obs. of  6 variables:
##  $ ID            : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ Type          : Factor w/ 12 levels "Выгоны","Вырубки",..: 11 1 1 9 5 1 5 12 9 10 ...
##  $ Administration: Factor w/ 7 levels "","РФ","Совьяковская администрация",..: 5 5 5 1 1 5 1 5 5 6 ...
##  $ Comment       : chr  "Село Беницы" "" "" "" ...
##  $ Perimeter     : Factor w/ 160 levels "1014.155593894044800",..: 67 155 51 104 78 153 17 19 108 57 ...
##  $ Area          : Factor w/ 160 levels "0.238070145845919",..: 73 49 121 100 63 72 88 128 99 24 ...
```
Посмотрим теперь, что произойдет, если мы попытаемся преобразовать столбец `Perimeter` к числовому виду:

```r
as.numeric(tab$Perimeter)
##   [1]  67 155  51 104  78 153  17  19 108  57   7   3 158 159 156  50  91
##  [18] 143   6  58   4   5 131 148 113 128 147 114   9  18 118 132  84 134
##  [35]  81  40 130  98  83 157  42  95  71 141   8 100  34   1  87  77 160
##  [52]  93 119  90  74  35 125 150 101 136  31 109 110 103  75  14  32  63
##  [69] 145  56 102  25  65  88  72  53  92  30 117  73  43  54 121  44  52
##  [86]  27 115 149 120  45  26  41   2  60  36 123  29 151 144 106 127  12
## [103] 116  94  82 146 142  69  21  48 139 105 154 124  47  61  33  80  97
## [120]  64  10  76 111  11 112  89  28 129  68  39  49  86  96  59  24 137
## [137]  46 152  55  15  99  85  22 126  16 122  79  66 133  23 107  38 138
## [154]  13 135  37 140  70  20  62
```
Вместо значений перметра мы получили загадочные числа, которых в таблице нет. Это и есть ключи факторов. Чтобы получить их значения, необходимо использовать функцию `levels()` (для краткости выведем первые 10 значений):

```r
levels(tab$Perimeter)[1:10]
##  [1] "1014.155593894044800" "1019.457949256323400" "1020.278536197552200"
##  [4] "1021.109926202218700" "1041.122684298658400" "1060.678503301135200"
##  [7] "1081.964408568060900" "1094.945610298295600" "114.701418496307100" 
## [10] "1155.916232728818800"
```
Обратите внимание на то, что значения фактора отсортированы в алфавитном порядке, без учете порядка их встречаемости в исходной таблице. Для корректного преобразования факторов в числа необходимо сначала привести их к обычному строковому виду:

```r
tab$Perimeter <- as.numeric(as.character(tab$Perimeter))
str(tab)
## 'data.frame':	160 obs. of  6 variables:
##  $ ID            : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ Type          : Factor w/ 12 levels "Выгоны","Вырубки",..: 11 1 1 9 5 1 5 12 9 10 ...
##  $ Administration: Factor w/ 7 levels "","РФ","Совьяковская администрация",..: 5 5 5 1 1 5 1 5 5 6 ...
##  $ Comment       : chr  "Село Беницы" "" "" "" ...
##  $ Perimeter     : num  2396 922 2181 3948 279 ...
##  $ Area          : Factor w/ 160 levels "0.238070145845919",..: 73 49 121 100 63 72 88 128 99 24 ...

# Теперь попробуем преобразовать столбец Area
temp <- as.numeric(as.character(tab$Area))
## Warning: в результате преобразования созданы NA
temp[1:10]
##  [1]  286159.159   21651.964   56826.463  450293.759    2612.615
##  [6]   28608.401 3469445.793   62299.631  450291.261  147943.134
```
Все прошло вроде бы успешно, но с предупреждением, что некоторые значения были преобразованы в `NA` (_Not Available_) --- отсутствующие значения. По всей видимости, данные в соответствущих ячейках не соответствуют представлениям __R__ о том, как должно выглядеть число: ячейка или пустая, или число набрано с ошибкой/опечаткой.

Чтобы найти и исправить все неверно заданные данные, необходимо выполнить следующие действия:

1. Получить индексы всех элементов, имеющих значение `NA`.
2. Просмотреть, какие значения были в исходных данных под этими индексами
3. Исправить ошибки в этих значениях, если это поддается автоматизации
4. Повторить конвертацию в числовой тип данных

Проверку на отсутствующие данные осуществляют с помощью функции `is.na()`. Передав ей в качестве аргумента вектор значений, вы получите вектор булевых значений, в котором `TRUE` будет стоять для пустых элементов. Проверим с помощью него, какие элементы столюца `Area` привели к ошибкам конвертации данных:

```r
tab[is.na(temp), "Area"]
## [1] 89499,573298880117000 11922,638460079328000 5153,570673500797100 
## 160 Levels: 0.238070145845919 ... 9865.323033935605100
```
Видно, что __R__ не справился с преобразованием типов там, где содержится опечатка в десятичном разделителе --- вместо точки указана запятая. 

Для исправления этой ошибки мы можем воспользоваться стандартной функцией замены символа `gsub(pattern, replacement, x)`. Ее стандартные параметры означают соответственно: что искать, на что заменять, где искать:

```r
tab$Area <- gsub(',', '.', tab$Area) # заменим запятые на точки
tab$Area <- as.numeric(as.character(tab$Area)) # Теперь можно преобразовать в числа
str(tab)
## 'data.frame':	160 obs. of  6 variables:
##  $ ID            : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ Type          : Factor w/ 12 levels "Выгоны","Вырубки",..: 11 1 1 9 5 1 5 12 9 10 ...
##  $ Administration: Factor w/ 7 levels "","РФ","Совьяковская администрация",..: 5 5 5 1 1 5 1 5 5 6 ...
##  $ Comment       : chr  "Село Беницы" "" "" "" ...
##  $ Perimeter     : num  2396 922 2181 3948 279 ...
##  $ Area          : num  286159 21652 56826 450294 2613 ...
```
Теперь необходимо навести порядок в значениях факторов, убедившись, что и там нет опечаток. Выведем все уникальные значения с помощью функции `levels()`:

```r
levels(tab$Type)
##  [1] "Выгоны"                        "Вырубки"                      
##  [3] "Гидрологические объекты"       "Заболоченные земли"           
##  [5] "Леса"                          "Лесные поляны"                
##  [7] "Луга"                          "Нет данных"                   
##  [9] "Пашни"                         "Сады"                         
## [11] "Территории населенных пунктов" "Фермерские хозяйства"
levels(tab$Administration)
## [1] ""                                    
## [2] "РФ"                                  
## [3] "Совьяковская администрация"          
## [4] "Совьяковская сельскаая администрация"
## [5] "Совьяковская сельская администрация" 
## [6] "Совьяковская сельская Администрация" 
## [7] "Совьяковская сельская админитрация"
```
Видно, что если с типами все в порядке, то в данных об административном подчинении содержится 5 вариантов названия одной и той же Совьяковской сельской администрации. Помимо этого, пустые ячейки хорошо бы заменить на значение `"Прочее"`.

Чтобы найти все строчки, относящиеся к одному и тому же объекту, можно воспользоваться уже знакомой нам функцией `grep()`, передав ей подстроку, которая является для них общей. Например, `"Совьяковская"` (хотя в данном случае было бы вообще достаточно одной буквы `"с"`).

```r
filter <- grep("Совьяковская", tab$Administration) # Найдем все записи
tab[filter, "Administration"] <- "Совьяковская сельская администрация" # Заменим их одним значением
tab$Administration <- droplevels(tab$Administration) # Удаляем неиспользуемые уровни
levels(tab$Administration)
## [1] ""                                   
## [2] "РФ"                                 
## [3] "Совьяковская сельская администрация"
```
Пустые строки можно также найти c помощью `grep()`, но мы этого делать не будем, так как это требует дополнительных знаний о регулярных выражениях. Вместо этого воспользуемся тем, что пустые строки имеют длину 0. Обратите внимание ниже, что преобразование в вектор столбца `Administration` необходимо, т.к. `nchar()` не понимает объекты типа `data.frame`, которыми являются не только таблицы, но и их столбцы:

```r
filter <- nchar(as.vector(tab$Administration)) == 0 # TRUE если длина равна 0
# Пробуем заменить:
tab[filter, "Administration"] <- "Прочее"
## Warning in `[<-.factor`(`*tmp*`, iseq, value = c("Прочее", "Прочее", :
## неправильный уровень фактора, получились NA
```
Ошибка выше связана с тем, что __R__ строго следит за неизменностью набора значений фактора для того чтобы избежать всевозможных ошибок при работе с данными (опечаток и т.д.). Предыдущий раз мы заменили все значаниея одним из существующих. В данном случае необходимо ввести новое значение фактора. Чтобы это сделать, придется преобразовать данные в символьные, произвести замену срок и после этого снова конвертировать столбец в фактор:

```r
tab$Administration <- as.character(tab$Administration)
tab[filter, "Administration"] <- "Прочее"
tab$Administration <- as.factor(tab$Administration)
levels(tab$Administration)
## [1] "Прочее"                             
## [2] "РФ"                                 
## [3] "Совьяковская сельская администрация"
```
Теперь таблица готова к работе. Можно, например, подсчитать по ней сводную статистику:

```r
summary(tab)
##        ID                                    Type   
##  Min.   :  1.00   Леса                         :52  
##  1st Qu.: 40.75   Выгоны                       :27  
##  Median : 80.50   Фермерские хозяйства         :22  
##  Mean   : 80.50   Пашни                        :15  
##  3rd Qu.:120.25   Луга                         :11  
##  Max.   :160.00   Территории населенных пунктов: 8  
##                   (Other)                      :25  
##                              Administration   Comment         
##  Прочее                             :76     Length:160        
##  РФ                                 : 3     Class :character  
##  Совьяковская сельская администрация:81     Mode  :character  
##                                                               
##                                                               
##                                                               
##                                                               
##    Perimeter              Area        
##  Min.   :    3.087   Min.   :      0  
##  1st Qu.:  421.431   1st Qu.:   5087  
##  Median :  939.369   Median :  21260  
##  Mean   : 1761.654   Mean   : 125002  
##  3rd Qu.: 2135.987   3rd Qu.:  83019  
##  Max.   :23920.945   Max.   :3469446  
## 
```
Обратите внимание, что строки, интервальные и номинальные (факторы) переменные обрабатываются функцией `summary()` по-разному.

## Сохранение таблиц CSV и Microsoft Excel {#table_writing}

Одной из завершающих стадий анализа данных, помимо графиков и отчетов, часто являются новые табличные представления, которые было бы неплохо сохранить в виде файлов. К счастью, сохранение таблиц в __R__ столь же просто, как и чтение. Для текстовых файлов в формате __CSV__ можно использовать функции `write.table()`, `write.csv()` и `write.csv2()`. Для файлов __Microsoft Excel__ используйте функцию `write.xlsx()` из пакета `openxlsx` соответственно. 

> По умолчанию функции `write.table()`, `write.csv()` и `write.csv2()` записывают в таблицы в качестве первого столбца названия (номера) строк таблиц. Если вы не хотите, чтобы это происходило, укажите дополнительный параметр `row.names=FALSE`.

Сохраним таблицы `okruga` и `neokruga`, раздельно хранящие статистику по объему сброса сточных в поверхностные водные объекты по федеральным округам и субъектам соответственно:

```r
write.csv2(okruga, "okruga.csv", fileEncoding = 'UTF-8') # Сохраним первую таблицу в CSV в кодировке Unicode
write.xlsx(neokruga, "neokruga.xlsx") # Сохраним вторую таблицу в XLSX без названий строк

# Проверим, все ли в порядке с сохраненными таблицами:

okruga.saved <- read.csv2("okruga.csv", encoding = 'UTF-8')
head(okruga.saved)
##    X                               Region Year05 Year10 Year11 Year12
## 1  2       Центральный федеральный округ    4341   3761   3613   3651
## 2 21   Северо-Западный федеральный округ    3192   3088   2866   2877
## 3 35             Южный федеральный округ    1409   1446   1436   1394
## 4 42 Северо-Кавказский федеральный округ     496    390    397    395
## 5 49       Приволжский федеральный округ    3162   2883   2857   2854
## 6 64         Уральский федеральный округ    1681   1860   1834   1665
##   Year13
## 1   3570
## 2   2796
## 3   1321
## 4    374
## 5   2849
## 6   1624

neokruga.saved <- read.xlsx("neokruga.xlsx",1)
head(neokruga.saved)
##                 Region Year05 Year10 Year11 Year12 Year13
## 1 Белгородская область     11     77     72     71     71
## 2     Брянская область     89     78     75     71     68
## 3 Владимирская область    155    129    126    124    120
## 4  Воронежская область    169    134    135    131    129
## 5   Ивановская область    144    102     99     97     88
## 6    Калужская область     99     92     88     84     93
```

Видно, что в файле __CSV__ присутствует также дополнительный столбец с названиями строк, а в файле __XLSX__ его нет. Если вы не задавали названия строк явным образом и они не несут какого-то смысла, всегда указывайте параметр `row.names=FALSE`

> Вы можете дать строкам таблицы названия и извлечь их, используя функцию `row.names()` аналогично функции `colnames()` для столбцов.

## Правила подготовки таблиц для чтения в R {#table_rules}

С таблицами, которые мы использовали в настоящем модуле, все прошло гладко, поскольку они были подготовлены специальным образом. Несмотря на то, что каких-то четких правил подготовки таблиц для программной обработки не существует, можно дать несколько полезных рекомендаций по данному поводу:

1. В первой строке таблицы должны располагаться названия столбцов.
2. В названиях столбцов недопустимы объединенные ячейки, покрывающие несколько столбцов. Это может привести к неверному подсчету количества столбцов и, как следствие, некорректному чтению таблицы в целом.
3. Названия столбцов должны состоять из латинских букв и цифр, начинаться с буквы и не содержать пробелов. Сложносочиненные названия выделяйте прописными буквами. Плохое название столбца: `Валовый внутренний продукт за 2015 г.`. Хорошее название столбца: `GDP2015`.
4. Во второй строке таблицы должны начинаться данные. Не допускайте многострочных заголовков.
5. Некоторые ошибки данных в таблицах (такие как неверные десятичные разделители), проще найти и исправить в табличном/текстовом редакторе, нежели после загрузки в R.

Следование этим правилам значительно облегчит работу с табличными данными.


## Контрольные вопросы {#questions_tables}

----
_Самсонов Т.Е._ **Визуализация и анализ географических данных на языке R.** М.: Географический факультет МГУ, 2017. DOI: 10.5281/zenodo.901911
----