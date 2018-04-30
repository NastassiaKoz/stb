# 11 <a name="Pades"></a>Формат PAdES

## 11.1 <a name="Pades1"></a>Общие положения

PDF, переносимый формат данных, определен базовой спецификацией [[PDF]](99Biblio.md#PDF).
PDF описывает структуру цифровых документов, которые можно обрабатывать на
разных платформах. На самом низком уровне PDF-документ представляет собой
последовательность октетов. Из октетов согласно определенным синтаксическим
правилам составляются объекты. Объекты являются базовыми логическими
элементами PDF, они задают структуру и содержание документа. Объекты могут
вкладываться в другие объекты или ссылаться по имени на другие объекты.
Определены контейнеры двух типов: массив - упорядоченная последовательность
объектов и словарь - ассоциативная таблица пар объектов "ключ - значение".

В PDF предусмотрено расширение формата, т.е. добавление в базовый формат 
новых объектов и изменение существующих. Использованные расширения 
указываются в специальном словаре  `extensions`.

PDF-документы могут подписываться. ЭЦП добавляется в документ, т.е.
является вложенной. Формат подписи описывается в подразделе [11.2](#Pades2). 
PAdES расширяет данный формат и налагает на него ряд ограничений.
Расширения и уточнения PAdES описываются в подразделах [11.3](#Pades3), [11.4](#Pades4).

## 11.2 <a name="Pades2"></a>Электронная цифровая подпись PDF

### 11.2.1 <a name="Pades21"></a>Словарь `Sig`

ЭЦП хранится внутри документа в специальном словаре `Sig`. 
В поле `Contents` словаря хранится базовая ЭЦП и возможно некоторые 
атрибуты РЭЦП, в других полях словаря - другие атрибуты.

Хэш-значение, используемое для выработки базовой ЭЦП, вычисляется от 
набора октетов, составляющих определенную часть документа. Способы задания 
целевого набора октетов описаны в п. [11.3.4](#Pades34). 

Структура словаря `Sig` определена в таблице 1. В данной таблице, как и во
всех последующих таблицах раздела, в столбце "Обязательность" значение
"опц." означает, что поле при каких-то условиях может отсутствовать, а
значение "обяз." - то, что поле присутствует всегда и имеет непустое
значение.

**Таблица 1**. Словарь `Sig`

| Ключ (поле) |  Тип |Обязательность| Значение  |
|-------------|------|-----------|-----------|
| Type        | name | опц.      | `"Sig"`      |
| Filter      | name | обяз.     | Идентификатор обработчика подписи (см. п. [11.3.5](#Pades35)) |
| SubFilter   | name | опц.      | Идентификатор кодировки подписи (например, `"ETSI.CAdES.detached"`) |
| Contents    | byte string|обяз.| Базовая ЭЦП |
| Cert|array или byte string | опц.| Используется в случае, если СОК не содержится в конверте, хранящемся в `Contents`|
| ByteRange| array | обяз.         | Массив пар чисел, определяющий набор фрагментов документа, подлежащих включению к выработке хэш-значения|
| Reference| array| опц.           | Массив словарей `SigRef`, определяющих с помощью механизма трансформаций данные, которые будут подписываться     |
| Changes | array | опц.           | Специальным образом определяет изменения в документе, сделанные на момент подписания по сравнению с состоянием на момент предыдущего подписания (применяется в случае контрподписей)|
| Name    | text string| опц.      | Имя подписанта (если его нельзя получить из других источников, например из СОК)|
| M       | date     |   опц.      | Время подписания (если его нельзя получить из конверта, хранящегося в поле `Contents`)|
| Location| text string| опц.      | Место подписания (имя хоста, географическое расположение)|
| Reason  | text string| опц.      | Причины (мотивы), побудившие к подписанию (например, "Я согласен, ....") |
| ContactInfo| text string| опц.   | Контактные данные подписанта|
| V          | integer | опц.      | Версия формата словаря `Sig` (значение `1` означает что поле `Reference` обязательное, по умолчанию - `0`) |
| Prop_Build | dictionary| опц.    | Информация о состоянии системы во время подписания, например идентификатор обработчика подписи, версия програмного обеспечения, операционная система |
| Prop_AuthTime| integer| опц.     | Количество секунд, прошедших с момента последней аутентификации подписанта  |
| Prop_AuthType| name| опц.        | Метод аутентификации подписанта (например `"PIN"`, `"Password"`, `"Fingerprint"`) |

В таблице 1 и других таблицах раздела используются следующие базовые типы 
PDF:

* integer - целое число;
* string - строка;
* name - строка, которая является именем (идентификатором) объекта;
* array - массив;
* dictionary - словарь;
* stream - поток, последовательность октетов, подходящая для данных 
большого объема, например картинок.

Базовый тип `string` уточняется следующими подтипами:

* text string - текст;
* byte string - строка октетов;
* date - время и дата.

Словарь `Sig` может размещаться в документе различными способами. 
Например, словарь `Sig` может быть значением определенного поля PDF-формы, 
или определенный объект документа может содержать ссылку на словарь.
Полная информация о вариантах размещения словаря `Sig` в документе PDF 
содержится в [[PDF]](99Biblio.md#PDF), подраздел 12.8. 

### 11.2.2 <a name="Pades22"></a>Задание подписываемых данных

Существует два способа задания той части документа, которая будет
подписываться.
 
Основной cпособ состоит в использовании поля `ByteRange` словаря `Sig`. Это
поле представляет собой массив пар целых чисел. Каждая пара задает фрагмент
документа. Первое число пары является номером первого октета фрагмента,
второе число - длиной фрагмента. Заданные парами фрагменты конкатенируются
и подписываются.

Альтернативный способ состоит в использовании специального словаря
`SigRef`, в котором указывается так называемый метод трансформации.
Этот метод применяется к документу или его части, на выходе получается
набор октетов, подлежащих подписанию. Детали использования словаря
`SigRef` определены в [[PDF]](99Biblio.md#PDF), подраздел 12.8.

И в первом, и во втором случаях, обработчик подписи при выработке ЭЦП
добивается, чтобы поле `Contents`, в котором будет содержаться базовая ЭЦП,
было исключено из подписываемых данных. Так, в случае использования
`ByteRange`, обработчику подписи следует сначала зарезервировать
определенное количество октетов под поле `Contents`, а только затем указать
`ByteRange` и подписать данные.

### 11.2.3 <a name="Pades23"></a>Обработчик подписи

Обработчик подписи - это программа, которая отвечает за выработку, проверку
и дополнение РЭЦП. Подписи различных видов (например, различных профилей)
могут обрабатываться различными обработчиками.

Идентификатор обработчика (или его префикс) регистрируется в 
специальном реестре согласно требованиям [[PDF]](99Biblio.md#PDF), приложение E. 
Идентификатор может выбираться разработчиком программного обеспечения 
с учетом требований уникальности и однозначности определения. 

В поле `Filter` словаря `Sig` указывается идентификатор обработчика подписи.
Это может быть либо обработчик, с помощью которого была выработана РЭЦП,
либо обработчик, который рекомендуется использовать при проверке и 
дополнении подписи. Несмотря на рекомендации, верификатор может 
использовать любой другой обработчик подписи, поддерживающий 
кодировку, указанную в поле `SubFilter` (см. п. [11.2.4](#Pades24)). 

### 11.2.4 <a name="Pades24"></a>Кодировка подписи PDF

Кодировка подписи PDF - это формат значения поля `Contents` в словаре 
`Sig`.  Идентификатор кодировки указывается обработчиком подписи в поле 
`SubFilter` во время выработки ЭЦП. При последующей работе с подписью 
(дополнение, проверка) должен использоваться подходящий обработчик подписи - 
тот, который поддерживает заданную кодировку. 

Обычно при реализации подписи PDF используются такие кодировки, что поле
`Contents` содержит только базовую ЭЦП (без атрибутов). Однако в PAdES
используется кодировка CAdES, и это значит, что в `Contents` хранится не
просто базовая подпись, а РЭЦП формата CAdES. Вложенная в `Contents` 
подпись CAdES называется конвертом.

## 11.3 <a name="Pades3"></a>PAdES на основе CAdES

### 11.3.1 <a name="Pades31"></a>Основные положения

Формат PAdES расширяет базовый формат подписи PDF-документов. Расширение
фиксируется добавлением в словарь `extensions` следущего элемента:
 
```
<</ESIC
  <</BaseVersion /1.7
    /ExtensionLevel 2
  >>
>>

```

Здесь и далее используется синтаксис словаря PDF. Согласно этому
синтаксису, строка `"ESIC"` - это ключ элемента словаря. Значением элемента
является вложенный словарь с двумя ключами `"BaseVersion"` и
`"ExtensionLevel"` и их значениями - строкой `"/1.7"` и числом 2
соответственно.

### 11.3.2 <a name="Pades32"></a>Кодировка CAdES

Кодировкой подписи для PAdES является формат CAdES. Это означает, что поле
`Contents` ДОЛЖНО содержать кодовое представление подписи CAdES одного из
профилей. Соответственно, в поле `SubFilter` ДОЛЖНА быть указана кодировка,
определяющая использование CAdES: `"ETSI.CAdES.detached"`.

При использовании CAdES многие атрибуты РЭЦП (например, СОК) содержатся в
конверте, хранящемся в поле `Contents`, и могут не указываться в
соответствующих полях словаря `Sig`. В частности, поле `Cert` словаря `Sig`
НЕ ДОЛЖНО использоваться.

Использование CAdES в PAdES имеет следующие особенности:
 
- атрибут `SigningTime` НЕ ДОЛЖЕН использоваться. Для определения времени 
служит поле `M` словаря `Sig`;
- атрибут `ContentType` ДОЛЖЕН принимать значение `"id-data"`;
- атрибут `ContentReference` НЕ ДОЛЖЕН использоваться;
- атрибут `ContentHints` НЕ ДОЛЖЕН использоваться;
- атрибут `SignerLocation` НЕ ДОЛЖЕН использоваться. Вместо него 
используется поле `Location` словаря `Sig`;
- атрибут `Countersignature` НЕ ДОЛЖЕН использоваться. Для управления 
контрподписями используется другой механизм, описанный в п. 11.3.4.

### 11.3.3 <a name="Pades33"></a>Ограничения на `ByteRange`

В PAdES разрешается использовать только 2 фрагмента в поле `ByteRange`
словаря `Sig`:

<!-- Хотя в ISO 32000-1 это только рекомендуется -->  

- от начала документа до поля `Contents` словаря `Sig`;
- от поля `Contents` до конца документа.

Таким образом, подписываются все октеты, задающие содержимое документа и 
атрибуты РЭЦП, хранящиеся в словаре `Sig`.

### 11.3.4 <a name="Pades34"></a>Контрподписи

В PAdES контрподписи реализуются следующим образом. Каждый из подписантов 
создает в документе свой словарь `Sig`, содержимое которого заполняется в 
соответствии с текущим состоянием документа на момент подписания (PDF 
позволяет иметь несколько словарей `Sig`).

Каждый из словарей задает свой `ByteRange` так, что фрагмент для каждой
следующей подписи включает в себя фрагмент для предыдущей. 
Каждый новый словарь добавляется в конец документа. Таким образом
удается избежать ситуации, когда предыдущий фрагмент покрывает словарь
следующей подписи.

### 11.3.5 <a name="Pades35"></a>Дополнительные объекты для долгосрочных подписей

### 11.3.5.1 <a name="Pades351"></a>Расширения

В долгосрочных РЭЦП профилей B-LT и B-LTA используются специальные
расширения формата PDF: `"Document security store"` (DSS) и `"Document
Time-stamp"` (DTS). Расширение DSS служит для хранения аттестатов. 
Расширение DST служит для хранения штампа времени. Аттестаты СЛЕДУЕТ 
хранить в объектах расширений, а не в атрибутах CAdES. При этом упрощается 
процесс дополнения подписи, учитывается специфика управления 
PDF-документами. 

Объекты расширений DSS и DST создаются в документе в процессе дополнения 
подписи. Дополнений может быть несколько, при каждом дополнении в документе 
создается определенный набор объектов:

- при дополнении подписи до профиля B-LT создается набор объектов 
расширения DSS;
- при дополнении подписи профиля B-LT до подписи профиля B-LTA создаются 
объекты расширения DTS;
- при дополнении подписи профиля B-LTA до новой подписи профиля B-LTA 
создаются новые объекты DSS (с аттестатами для предыдущих штампов времени) 
и новые объекты DTS.

Таким образом, документ с подписью профиля B-LT ДОЛЖЕН содержать объекты 
расширения DSS, а документ с подписью профиля B-LTA ДОЛЖЕН содержать один 
или несколько наборов объектов DSS и DTS. 

Для указания на использование расширений DSS и DST в словарь 
`extensions` (см. п. [11.2.3](#Pades23)) необходимо добавить следущий элемент:
 
```
<</ESIC
  <</BaseVersion /1.7
    /ExtensionLevel 1
  >>
>>
```

#### 11.3.5.2 <a name="Pades352"></a>Расширение DSS

DSS представляет собой словарь внутри документа, служащий хранилищем для 
различных аттестатов. Аттестаты могут относиться к одной или нескольким 
подписям PAdES на основе CAdES, к одной или нескольким подписям XAdES в 
XML-вставках (см. подраздел 11.4), к встречающимся в документе штампам 
времени.
 
Расширение DSS описывается словарем `DSS` и словарями `Signature VRI`. 
Словарь `DSS` является единственным для всего документа, 
тогда как каждый из словарей `Signature VRI` служит для описания 
определенной подписи (если таковых в документе несколько, например в 
случае контрподписей). 
 
Поле `VRI` словаря `DSS` ДОЛЖНО храниться в специальном месте документа - 
так называемой инкрементальной секции (`"incremental section"`, см.  [[PDF]](99Biblio.md#PDF), 
п. 7.5.6). 
 
При проверке подписи, верификатор имеет несколько источников аттестатов. 
При поиске аттестатов РЕКОМЕНДУЕТСЯ последовательно проверять
сначала словари `Signature VRI`, затем словарь `DSS`,
затем данные, хранящиеся в `Contents`, или подписи XAdES в XML-вставках.
 
Структуры словарей `DSS` и `Signature VRI` определены в таблицах 2 и 3.

**Таблица 2**. Словарь `DSS`
 
| Ключ (поле) |  Тип |Обязательность| Значение  |
|-------------|------|-----------|-----------|
| Type        | name | опц.      | DSS      |
| VRI      | dictionary | опц.     | словарь, ключами которого являются  SHA1-хэши подписей, значениями - соответствующие подписям словари `Signature VRI` |
| Certs|        array|  опц.| массив объектов (или ссылок на объекты) типа stream, каждый из которых задает сертификат |
| OCSPs|        array|  опц.| массив объектов (или ссылок на объекты) типа stream, каждый из которых задает OCSP-ответ |
| CRLs|        array|  опц.| массив объектов (или ссылок на объекты) типа stream, каждый из которых задает СОС |

**Таблица 3**. Словарь `Signature VRI`

| Ключ (поле) |  Тип |Обязательность| Значение  |
|-------------|------|-----------|-----------|
| Type        | name | опц.      | VRI      |
| Cert|        array|  опц.| массив объектов (или ссылок на объекты) типа stream, каждый из которых задает СОК |
| OCSP|        array|  опц.| массив объектов (или ссылок на объекты) типа stream, каждый из которых задает OCSP-ответ |
| CRL|        array|  опц.| массив объектов (или ссылок на объекты) типа stream, каждый из которых задает СОС |
|TU| date| опц. | время создания словаря |
|TS| stream| опц. | штамп времени, соответствующий времени создания данного `Signature VRI` |


#### 11.3.5.3 <a name="Pades353"></a>Расширение DST

Расширение DST задается словарем `DocTimeStamp`, который хранится в 
документе после словаря `DSS`.
 
Словарь `DocTimeStamp` представляет собой измененный словарь `Sig`.
Изменения описаны в таблице 4.

**Таблица 4.** Словарь `DocTimeStamp` (изменения относительно `Sig`)

| Ключ (поле) |  Тип |Обязательность| Значение  |
|-------------|------|-----------|-----------|
| Type        | name | опц.      | `"DocTimeStamp"`      |
| SubFilter   | name | обяз.     | Идентификатор кодировки (`"ETSI.RFC3161"`, или другой, определенный разработчиками системы) |
| Contents    | byte string | обяз.| Штамп времени в формате, определенном в СТБ 34.101.ts |
|V            | integer     | опц. | 0 | 

Словарь `DocTimeStamp` НЕ ДОЛЖЕН содержать поля `Cert`, `Reference`,
`Changes`, `R`, `Prop_AuthTime`, `Prop_AuthType`.

В словарь НЕ РЕКОМЕНДУЕТСЯ включать поля `Name`, `M`, `Location`, `Reason`,
`ContactInfo` (соответствующие данные присутствуют в конверте, хранящемся в
`Contents`). Верификатору СЛЕДУЕТ игнорировать перечисленные поля при
проверке подписи.

## 11.4 <a name="Pades4"></a>PAdES для XML-вставок

Документ PDF может содержать в себе данные в формате XML. Эти данные 
представляют собой XML-документ, вложенный в определенный объект PDF.

Вложенный XML-документ МОЖЕТ подписываться. Его подпись XAdES также будет
вложена в PDF-документ как часть XML-документа. 

Уже подписанный XML-документ МОЖЕТ быть повторно подписан как часть 
PDF-документа. При этом ДОЛЖНА вырабатываться описанная выше подпись PAdES 
на основе CAdES.

Для долгосрочных подписей XAdES (профили B-LT и B-LTA) МОГУТ 
использоваться расширения DSS и DST, описанные в п. [11.3.5](#Pades35). Тогда 
объекты этих расширений ДОЛЖНЫ включать аттестаты для подписей XAdES. 
