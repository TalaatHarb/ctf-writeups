# Hash My Awesome Commands
## Task
I found this strange server that only has two commands. Looks like there's a command to get the flag, but I can't figure out how to get it to work. It did come with a strange note that had 9W5iVNkvnM6igjQaWlcN0JJgH+7pComiQZYdkhycKjs= written on it. Maybe that's important?

nc challenges.2020.squarectf.com 9020

File: hmac.go

The server provides the ability to execute two commands if you you can provide the hmac for them and since you are given a base64 value, trying it with the debug command shows that it is its hmac

you can also see that the verfication of the hmac has (spoiler alert) some time aspect to it. The target is to exploit this timing aspect of it to predict the valid hmac for the flag command and trigger the flag command

## Solution
If you turned on debugging, the server provides how long the verfication takes and checking the code provides the fact that the more right parts of the string you guess the longer it takes to verify it
you can actually have an exact value for the difference in timing from the code but haveing an exact value isn't that important as long as you can get a good enough approximation for the time added by each byte

You perform a timing based side channel attack by trying each byte until you get a byte that takes longer than the other with a noticable step (and due to the unstability of the calculation, you might need to do multiple attempts and then average them out)
The maximum number of attempts for this scenario is 8192 but since the attempts start taking longer and longer you might need to keep trying for about 20 minutes

The exploit implemented in python included some tolerance for the timining and confirmation steps for each byte and also backing of when it thinks it estimated something in the wrong way
but the essense of the attack is the same and follows the following pseudo code:

```
foreach byte in hash:
  assume value of byte 0-256 and attempt using it as part of the hmac
  if the time spent is a major increase over the previous time:
    confirm the timing by doing other attempts and if it passes move to the next byte
  else:
    try the next value of the byte
  if some byte fails to be detected many times:
    tell user that you failed guessing the previous byte at least and exit after printing the detected bytes correctly
```
