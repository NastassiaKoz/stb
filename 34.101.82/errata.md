# Поправка к официальной редакции

| В каком месте | Напечатано | Должно быть |
|---------------|------------|-------------|
| Подраздел 7.1, предпоследний абзац | Если идентификатор не может быть распознан СШВ или соответствует алгоритму хэширования, который не определен в действующем ТНПА, то СШВ НЕ СЛЕДУЕТ выпускать штамп времени, при этом СШВ СЛЕДУЕТ вернуть ответ с ошибкой `bad_alg` (см. 7.2). |   Если идентификатор не может быть распознан СШВ или соответствует алгоритму   хэширования, который не определен в действующем ТНПА, то СШВ НЕ СЛЕДУЕТ   выпускать штамп времени, при этом СШВ СЛЕДУЕТ вернуть ответ с ошибкой   `badAlg` (см. 7.2). |
| Раздел 9, абзац 2 | Учитывая, что с использованием скомпрометированного личного ключа СШВ могут быть выпущены штампы времени на дату в прошлом, любые штампы времени, подписанные скомпроментированным личным ключом, ДОЛЖНЫ признаваться недействительными, за исключением случаев, указанных в СТБ 34.101.80 (подраздел 8.2). | Учитывая, что с использованием скомпрометированного личного ключа СШВ могут быть выпущены штампы времени на дату в прошлом, любые штампы времени, подписанные скомпрометированным личным ключом, ДОЛЖНЫ признаваться недействительными, за исключением случаев, указанных в СТБ 34.101.80 (подраздел 8.2).|
| Раздел 9, абзац 6 | Если несколько сторон запрашивают штампы времени на один и тот же объект данных или если одна сторона запрашивает несколько штампов времени на один и тот же объект данных, то в случае использования для формировании запросов одного и того же алгоритма хэширования выпущенные штампы времени будут содержать одинаковые значения компонента `messageImprint`. | Если несколько сторон запрашивают штампы времени на один и тот же объект данных или если одна сторона запрашивает несколько штампов времени на один и тот же объект данных, то в случае использования для формирования запросов одного и того же алгоритма хэширования выпущенные штампы времени будут содержать одинаковые значения компонента `messageImprint`.|