ATM will have unique number (in this case it is A1B2).
Excluding the case of unreadable card.

User scans card:

ATM's Actions:
If card is expired: Outputs an appropriate message.
Else: ATM reads the [Card], establishes the TCP connection with appropriate bank's CC and asks the user for pin.
User enters the [PIN]
The handshake:
  A1B2 to CC: 
  {SYN}
  CC to A1B2:
  {SYN/ACK}
  A1B2 to CC:
  {ACK}

A1B2 to CC:
  {[Code 10], On [Date], at [Time], for [Card] + [PIN]}

Possible responses from CC:
  {[Code 1], Yes, [Balance]: $A}
  {[Code 2], Incorrect [PIN], [Attempts]: X}
  {[Code 3], Blocked}
  {[Code 4], Trans. in progress elsewhere}

ATM's actions and responses based on the [Code]:
  
{[Code 1], Yes, [Balance]: $A}

ATM outputs the [Balance] and asks the amount to be withdrawn. User enters [Amount]
A1B2 to CC:
  {[Code 20], Withdraw Request: [Amount]}

CC verifies if balance-amount >= 0

If yes:
CC updates the [Balance] -= [Amount] in it's system.
CC to A1B2: 
  {[Code 5], Dispense, on [Date], at [Time], [Balance]: $B}
ATM dispenses the cash, outputs remaining balance, ejects the card (We could go on to ask the user for further actions, but, let's keep it simple)

If no:
CC to A1B2: 
  {[Code 6], Ah, ya poor!!}
ATM presents the appropriate message and ejects the card.


{[Code 2], Incorrect [PIN], [Attempts]: X}

If [Attempts] > 0, ATM updates [Attempts] -= 1, asks the user for [PIN] again. When entered:
A1B2 to CC:
  {[Code 30], [PIN], [Attempts]}
  If [PIN] is correct:
  CC to A1B2:
    {[Code 1], Yes, [Balance]: $A}
  And the process continues like in the [code 1].
  Else if [PIN] is incorrect again (If [Attempts] > 0, ATM updates [Attempts] -= 1, asks the user for [PIN] again. When entered:):
  A1B2 to CC:
    {[Code 30], [PIN], [Attempts]}
  CC updates [Attempts] and;
  CC to A1B2:
    {[Code 2], Incorrect [PIN], [Attempts]: X}
And this is like a while loop that repeats until no attempts are left. And when that happens:
A1B2 to CC:
  {[Code 40], [Attempts] = 0}
CC blocks the card
CC to A1B2:
  {[Code 3], Blocked}
ATM outputs the appropriate message and ejects the card.


{[Code 3], Blocked}
ATM outputs the appropriate message and ejects the card.


{[Code 4], Trans. in progress elsewhere}
This is in case of a joint account, that's why in the first message ATM sends CC, there's timestamp. Even though two simultaneous requests are possible, based on the timestamp, CC rejects the second request.
ATM outputs the appropriate message and ejects the card!
