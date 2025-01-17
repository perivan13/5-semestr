# Средство контейнерной виртуализации Docker

Факт правильного выполнения всех пунктов задания должен быть подтверждён **хорошими** скриншотами (без чёрной заливки фона). Требуемый по заданию шаблон должен быть крупно виден (пользуйтесь "ножницами" в windows или shift+printscreen в ubuntu). Рисунки должны быть подписаны. [Шаблон документа для ЛР](http://gitlabnto/anetto/wiki/wikis/%D1%88%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD-%D0%B4%D0%BE%D0%BA%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%BE%D0%B2)

Отчёт по работе в электронном виде, соответствующем кафедральному форматированию, должен быть загружен в \\\\fs\\prepods\\anetto\\for write\\БОС\\ЛР1\\N с названием по шаблону ЛР1-N-Фамилия, где N-номер группы. Преподавателю необходимо сдать печатный вариант в сокращённом виде (титульный лист, задание, ход работы с некоторыми, наиболее существенными скриншотами, выводы).

 
В отчете должно быть:

* Код программы-сумматора
* Скриншот для подтверждения наличия/отсутствия динамических библиотек в исполняемом файле программы-сумматора
* Код Dockerfile
* Скриншот Docker build, docker run, docker logs с нужными аргументами и выводом
* Скриншот ls -la, в результате выполнения которого видно output.txt (НЕ В КОНТЕЙНЕРЕ)
(на 5) Код Dockerfile с update, скриншот docker build с наличием компиляции

Задания:
1. На С разработать программу, выводящую сумму произвольного числа аргументов (код возврата 0) в файл FULLNAME.txt в текущем каталоге (FULLNAME заменить на ваш fullname). Если среди аргументов есть не числа, выводить ошибку (в правильный поток) и обеспечивать код возврата -1.  В стандартный поток вывода вывести все введенные аргументы (даже если они не числа), каждый аргумент на новой строке. После аргументов должна быть пустая строка, а последняя строка должна содержать сумму аргументов, если они все числовые. Пример работы:
    ```bash
    user$ ./summator 15 10 2
    15
    10
    2
    
    27
    user$ echo $?
    0
    user$ ./summator 15 hello world
    Error: some args are not int
    15
    hello
    world
    user$ ./summator 15 hello world 2>/dev/null
    15
    hello
    world
    user$ echo $?
    -1
    ```
2. Сформировать Dockerfile. Строить образ на базе ubuntu, скопировать в контейнер исходный код. При построении контейнера скомпилировать разработанную программу с динамическими библиотеками.
3. Построить и запустить собственный docker-образ, задав ему ваш fullname в качестве имени. С помощью отображения (проброса, маппинга) каталогов обеспечить наличие результата (файла output.txt) в домашнем каталоге пользователя в виртуальной машине (НЕ внутри докер контейнера). Запускаемые команды и их вывод должны входить в отчет. С помощью команды ls -la подтвердить наличие файла output.txt
4. Продемонстрировать лог работы контейнера.
5. Задание на "отлично". Модифицировать Dockerfile так, чтобы в нем выполнился apt-get update, был получен список пакетов из локального репозитория. Установить компилятор gcc. Построить образ на базе ubuntu, скопировать в контейнер только исходный код. При построении контейнера скомпилировать разработанную программу со статическими библиотеками.