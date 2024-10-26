# Incident reporting (4 points)

Hi, TCC-CSIRT analyst,

our automatic incident recording system has captured about an hour of traffic
originating from the IP range within the AI-CSIRT constituency. Analyze whether
there are any incidents present and report all of them through the AI-CSIRT web
interface.

* [Download the pcap file (cca 53 MB)](incident_reporting.zip)
  (sha256 checksum: `e38550ba2cc32931d7c75856589a26e652f2f64e5be8cafaa34d5c191fc0fd1c`)
* The web interface is at http://incident-report.csirt.ai.tcc.

See you in the next incident!

## Hints

* The IP ranges in constituency of AI-CSIRT are `10.99.224.0/24` and
  `2001:db8:7cc::a1:0/112`.
* Based on our previous experience, the AI handling reports is very rigid and
  refuses to acknowledge incompletely described incidents. Make sure that you
  have described the incident accurately, with nothing missing or excessive.

## Solution

We can use [Wireshark] to analyze the extracted pcap file. Since the AI-CSIRT
web interface requires UTC times when submitting the data, it is important to
switch the display setting accordingly (`View -> Time Display Format -> UTC Date and Time of Day`).

Then we just need to analyze the provided data to find 4 incidents. After
submitting each of the reports, the AI-CSIRT web interface yields 1/4 of the
FLAG.

### Incident 1: Brute force attack

Wireshark filter: `http && ipv6.src == 2001:db8:7cc::a1:210 && ipv6.dst == 2001:db8:7cc::acdc:24:beef`

* Offending IP address: `2001:db8:7cc::a1:210`
* Target IP address: `2001:db8:7cc::acdc:24:beef`
* Datetime of the first attempt (UTC): `2024-09-26 08:55:20`
* Datetime of the last attempt (UTC): `2024-09-26 08:55:43`
* Number of attempts: `1000-4999`
* Result: `Success`

> AI response:
>
> `{"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}`
>
> Your incident ID is MS80OiBGTEFHe2xFOA==, please keep it.

```console
$ echo MS80OiBGTEFHe2xFOA== | base64 -d
1/4: FLAG{lE8
```

### Incident 2: (D)DOS

Wireshark filter: `http && ipv6.src == 2001:db8:7cc::a1:d055`

* Offending IP address: `2001:db8:7cc::a1:d055`
* Target IP address: `2001:db8:7cc::acdc:24:911`
* Datetime of the first attempt (UTC): `2024-09-26 08:58:44`
* Datetime of the last attempt (UTC): `2024-09-26 09:46:44`
* Service affected: `HTTP`

> AI response:
>
> `{"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}`
>
> Your incident ID is Mi80OiBzLVVrb3g=, please keep it.

```console
$ echo Mi80OiBzLVVrb3g= | base64 -d
2/4: s-Ukox
```

### Incident 3: Scanning

Wireshark filter: `ipv6.addr == 2001:db8:7cc::a1:42`

* Offending IP address: `2001:db8:7cc::a1:42`
* Target scan range – CIDR: `2001:db8:7cc::acdc:24:0/112`
* Datetime of the first attempt (UTC): `2024-09-26 08:44:29`
* Datetime of the last attempt (UTC): `2024-09-26 09:17:09`
* Target ports: `21, 22, 53, 80, 443, 8080`
* Number of found and scanned targets: `20-99`

> AI response:
>
> `{"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}`
>
> Your incident ID is My80OiAtYTBRZi0=, please keep it.

```console
$ echo My80OiAtYTBRZi0= | base64 -d
3/4: -a0Qf-
```

### Incident 4: Web service enumeration

Wireshark filter: `http && ipv6.addr == 2001:db8:7cc::a1:210 && ipv6.addr == 2001:db8:7cc::acdc:24:a160`

* Offending IP address: `2001:db8:7cc::a1:210`
* Target scan range – CIDR: `2001:db8:7cc::acdc:24:a160`
* Datetime of the first attempt (UTC): `2024-09-26 08:48:18`
* Datetime of the last attempt (UTC): `2024-09-26 08:49:45`
* Number of enumerated URL: `10000-49999`

> AI response:
>
> `{"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}`
>
> Your incident ID is NC80OiBkNWtNfQ==, please keep it.

```console
$ echo NC80OiBkNWtNfQ== | base64 -d
4/4: d5kM}
```

### Summary

If we combine all four parts we get `FLAG{lE8s-Ukox-a0Qf-d5kM}`.

[Wireshark]: https://www.wireshark.org/
