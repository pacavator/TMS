HW_5 
**#1. Распределить основные сетевые протоколы (перечислены ниже) по уровням модели TCP/IP**

	UDP Транспортный 3
	TCP Транспортный 3
	FTP Прикладной 4
	RTP Прикладной 4
	DNS Прикладной 4
	ICMP Сетевой 2
	HTTP Прикладной 4
	NTP Прикладной 4
	SSH Прикладной 4

**#2. Узнать pid процесса и длительность подключения ssh с помощью утилиты ss**

	pacavaca@ubuntu-vm:~$ sudo ss -atop | grep ssh | grep ESTAB
	ESTAB  0      0          10.0.2.11:ssh      10.153.1.9:57037 users:(("sshd",pid=152571,fd=4),("sshd",pid=152524,fd=4)) timer:(keepalive,18sec,0)
	ESTAB  0      64         10.0.2.11:ssh      10.153.1.9:64797 users:(("sshd",pid=177229,fd=4),("sshd",pid=177132,fd=4)) timer:(on,376ms,0)

**#3. Закрыть все порты для входящих подключений, кроме ssh**

	pacavaca@ubuntu-vm:~$ sudo iptables -L
	Chain INPUT (policy DROP)
	target     prot opt source               destination
	ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
	
	Chain FORWARD (policy ACCEPT)
	target     prot opt source               destination
	
	Chain OUTPUT (policy ACCEPT)
	target     prot opt source               destination

**#4. Установить telnetd на ВМ, зайти на нее с другой ВМ с помощью telnet и отловить вводимый пароль и вводимые команды при входе c помощью tcpdump**

	[pacavaca@centos7-vm ~]$ telnet 10.0.2.11
	Trying 10.0.2.11...
	Connected to 10.0.2.11.
	Escape character is '^]'.
	Ubuntu 22.04.2 LTS
	ubuntu-vm login: pacavaca
	Password:
	Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)
	Privet poslannikam inoplanetnih civilizaciy
	pacavaca@ubuntu-vm:~$ 

*Рассматриваемый ниже пример охватывает частный случай, когда в ходе telnet-подключения логин и пароль вводятся посимвольно. При анализе дампа было выявлено, что 		вводимые символы находятся на последней позиции строки фиксированной длины в 9 символов, при этом других строк такой длины в дампе нет. Это наблюдение позволяет 		применить к дампу следующие действия для выявления вводимой информации:
 	
	sudo tcpdump  port 23 -Alq -Q in > rawdump.txt              # Собираем текстовый дамп 
	cat rawdump.txt | egrep "^.{9}$" > rawinput.txt             # Выбираем строки длиной 9 символов
	cut -с 9 rawinput.txt                                       # Получаем столбец введенных символов на 9 позиции, которые и есть то, что мы вводили.

**#5.Открыть порт 222/tcp и  обеспечить прослушивание порта с помощью netcat, проверить доступность порта 222 с помощью telnet и nmap**
	
	sudo iptables -A INPUT -p tcp --dport 222 -j ACCEPT
	
	pacavaca@ubuntu-vm:~$ sudo nc -lp 222
	hello
	hi
	centos online
	ubuntu here
	
	[pacavaca@centos7-vm ~]$ telnet 10.0.2.11 222
	Trying 10.0.2.11...
	Connected to 10.0.2.11.
	Escape character is '^]'.
	hello
	hi
	centos online
	ubuntu here

	[pacavaca@centos7-vm ~]$ sudo nmap 10.0.2.11
	
	Starting Nmap 6.40 ( http://nmap.org ) at 2023-07-19 14:50 +03
	Nmap scan report for 10.0.2.11
	Host is up (0.0012s latency).
	Not shown: 996 filtered ports
	PORT     STATE  SERVICE
	22/tcp   open   ssh
	23/tcp   open   telnet
	222/tcp  open   rsh-spx
	9000/tcp closed cslistener
	MAC Address: 08:00:27:3C:45:2E (Cadmus Computer Systems)
	
	Nmap done: 1 IP address (1 host up) scanned in 5.15 seconds


 и ***не обязательные  Дедлайн: 27/07/2023
