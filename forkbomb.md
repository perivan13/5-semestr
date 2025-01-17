# Fork Bomb #

fork() — системный вызов в Unix-подобных операционных системах, создающий новый процесс (потомок), который является практически полной копией процесса-родителя, выполняющего этот вызов.

Fork Bomb — маленькая программа, которая порождает себя n-раз. 
Действует Fork bomb по весьма простому пути: для начала программа загружает себя в память, где создает несколько копий себя самой (обычно две). Далее каждая из этих копий создает столько же копий, как и оригинал, и так далее пока память не будет полностью забита, что приводит к отказу системы, ресурсы системы быстро исчерпываются. 

Fork Bomb'ы работают как за счет использования процессорного времени в процессе разветвления, так и за счет заполнения таблицы процессов операционной системы. Базовая реализация Fork Bomb'ы - это бесконечный цикл, который многократно запускает новые копии самого себя.

## Примеры создания  Fork Bomb в Unix/Linux ##

**ВНИМАНИЕ!!!** Эти примеры могут привести к сбою вашей виртуальной машины/компьтера в случае его выполнения.

### Пример создания  Fork Bomb с использованием bash ###

Итак, код выглядит так:

	# :(){ :|:& };:

Где:
   *	:() — Определение функции.
   *	{  — Открытие функции.
   *    : — загрузка копии функции «:» в память
   *    |: — перенаправление вывода в следующую копию функции
   *	& — Помещает вызов функции в фоновом режиме, чтобы fork (дочерний процесс) не мог «умереть» вообще, тем самым это начнет есть системные ресурсы.
   *	} — Закрытие функции.
   *	; — Завершите определение функции. Т.е является разделителем команд, (командный сепаратор).
   *	: — Запускает функцию которая порождает fork bomb().
	 
Это рабочий код, но не очень читабельный. Вот пример нормального, читаемого кода:
	 
	#!/usr/bin/env bash -x
	 bomb() {
	 bomb | bomb &
	 };
	 bomb

### Пример создания  Fork Bomb с использованием perl ###

	# perl -e "fork while fork" &

### Пример создания  Fork Bomb с использованием C/C++ ###

	#include <unistd.h>
	int main(void)
	{
		while(1)
		fork();
	}

## Способ предотвращения Fork Bomb'ы ##

Избегайте использования Fork в любом, выражении, которое может оказаться в бесконечном цикле.
Можно ограничить процесс разветвления, как показано ниже:

	# sudo nano /etc/security/limits.conf
	
Отредактируйте файл как:

	 user_name hard nproc 10
	 
теперь *user_name* может создать 10 процессов.

Или ограничьте количество процессов пользователя с помощью утилиты:

	# ulimit -u count_of_process
	
Проверьте результат:

	#ulimit -a

Необходимо отметить, что ограничение количества процессов само по себе не предотвращает запуск fork-бомбы, а лишь направлено на минимизацию возможного вреда в случае её срабатывания.

Вы можете попробовать запустить команду в VMWare, если хотите убедиться в том, что все повиснет.


## Примеры Defusing Fork Bomb в Unix/Linux ##

Defusing — так называемое разминирование fork bomb. Из-за их характера, такие бомбы трудно оставновить после их запуска. Чтобы остановить такую бомбу, нужно  завершить все рабочие копии, чего трудно достичь. Одна из проблем заключается в том, что данная команда не может быть выполнена из-за того, что таблица процессов полностью забита. Вторая серьезная проблема заключается в том,  на момент поиска процессов для прекрашения потратилось время за которое могло создатся еще пару форков программы.

Для удаления такой бомбы, можно использовать одну из перечисленных команд:

	# killall -STOP processWithBombName
	# killall -KILL processWithBombName
	
Когда у системы мало свободных PID (в Linux максимальное количество PID-ов можно получить получено комндой ``` cat /proc/sys/kernel/pid_max ```), «разрядка» bomb-ы становится более сложной:

	# killall -9 processWithBombName

Можно получить ошибку:

	bash: fork: Cannot allocate memory

В этом случае разрядка бомбы возможна только в том случае, если хотя бы одна оболочка открыта. Процессы могут не разветвляться, но можно выполнить любую программу из текущей оболочки. Как правило, возможна только одна попытка.

Команда «killall -9» не выполняется непосредственно из оболочки, потому что команда не является атомарной и не удерживает блокировки в списке процессов, поэтому к тому времени когда она закончится, fork bomb наплодит еще PID-ов. Поэтому нужно запустить несколько процессов killall, например:

	# while :; do killall -9 processWithBombName; done
	
Еще пример, — т.к таблица процессов достапна в Linux через ФС (/proc), то можно обезвредить данный форк бомбы с помощью встроенных функций bash, которые не требуют развертывания новых процессов.

Следующий пример идентифицирует процессы, связанные с нарушением и приостанавливает их, чтобы предотвратить данный форк, до того момента, пока они убиты по одному за раз. Это позволяет избежать состояния гонки других примеров, которые могут потерпеть неудачу, если нарушающие процессы могут развиваться быстрее, чем они убиты:

	# cd /proc && for p in [0-9]*; do read cmd < "$p/cmdline"; if [[ $cmd = processWithBombName ]]; then kill -s 
	
Отключите питание системы на случай, если вы запустили ее и не нашли выход для продолжения.

Как-то так!
