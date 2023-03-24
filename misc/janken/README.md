# Cyber Apocalypse 2023

## Janken

> As you approach an ancient tomb, you're met with a wise Guru who guards its entrance. In order to proceed, he challenges you to a game of Janken, a variation of rock paper scissors with a unique twist. But there's a catch: you must win 100 rounds in a row to pass. Fail to do so, and you'll be denied entry.

>
>  README Author: [bobAKAbill](https://github.com/bobakabill)
>

Tags: _misc__

## Time to Flag


## Initial looksie doodle

When you start the docker and connect, you're met with a colorful little interface (you'll need to imagine the colors) that is very much asking for input
```


                         ▛▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▜
                         ▌   じ ゃ ん 拳  ▐
                         ▙▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▟


1. ℙ ∟ ₳ Ұ
2. ℜ ℧ ∟ Ӗ ⅀

>> 

```

Since I had no idea how to play the game, I asked for the rules

```
>> 2

▛▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▜
▚  [*] Rock     is called "Guu"   (ぐう).                                ▞
▞  [*] Scissors is called "Choki" (ちょき).                              ▚
▚  [*] Paper    is called "Paa"   (ぱあ).                                ▞
▞                                                                        ▚
▚  1. Rock > scissors, scissors > paper, paper > rock.                   ▞
▞  2. You have to win [100] times in a row in order to get the prize. 🏆 ▚
▙▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▟

1. ℙ ∟ ₳ Ұ
2. ℜ ℧ ∟ Ӗ ⅀

>> 
```

# oh
I have to win rock/paper/scissors 100 times in a row... I'm going to try and see if the files included have any kind of gimmick (like a pattern in the way The Guru plays) that I can take advantage of


## GHIDRA TIME
Opening it up in Ghidra (it's just an exe written in C), I find that the choices are about as random as one can make them. The Guru calls upon the powers of time(), srand(), and rand() to decide how she's going to play.

Wait a minute...

## A light at the end of the tunnel

Upon some discussion with my teammates (and me going back in time to my C 101 class), I decided the way that The Guru wants me to proceed isn't by attempting the sisyphean task of actually trying to beat them 100 times in a row at rock/paper/scissors (Guu/Paa/Choki?), which has odds of 1.9403252\*10^-48 of happening, but by taking advantage of being able to reproduce their results.

## The Black-Banded Krait comes out to play

My first order of business is to throw together a script that connects to the server and can play the game, so I go ahead and whip that one up first. I borrowed some code online that will read from a buffer until a certain delimeter, just to make my code a little happier about playing (so that I'd only send input when The Guru is ready to receive). I included this code below for fellow Pythonists, as I'd never thought of doing things this way and it's pretty neat
```
def read_until(s, delim=b':'):
    buf = b''
    s.settimeout(1)
    while not buf.endswith(delim):
        try:
            buf += s.recv(1)
        except:
            break

    s.settimeout(None)

    return buf.decode("utf8")
```

Once I can connect happily to the domain of The Guru, I have to make sure my script has the same randomness as The Guru does. This is achieved through `ctypes:CDLL` and `ctypes.util:find_library`

If you put the line `libc = CDLL("libc.so.6")` into your Python code, it imports the correct implementation of C functions for the program (I knew the correct libc version since I saw it in Ghidra (...and it was included in the files that I downloaded for the problem... I didn't look there))

Now I can finally use the same powers of the almighty libc.time(), libc.srand(), and libc.rand() as the wise Guru. Let's see what this looks like!
```
import socket
import time
import random
from ctypes import CDLL
from ctypes.util import find_library

def read_until(s, delim=b':'):
    buf = b''
    s.settimeout(1)
    while not buf.endswith(delim):
        try:
            buf += s.recv(1)
        except:
            break

    s.settimeout(None)

    return buf.decode("utf8")

libc = CDLL("libc.so.6")

guru_choices = ["rock", "scissors", "paper"]
player_choices = ["paper", "rock", "scissors"]

host = "68.183.37.122"
port = 30000

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((host, port))
time.sleep(0.5)
printout = bytearray.fromhex(s.recv(2048).hex()).decode()
print(printout)

if(">>" in printout):
	print("connection successful")
	s.sendall(b'1')
	printout = read_until(s, b">>")
	print(printout)
	firstTime = libc.time()
	libc.srand(firstTime)
	firstRand = libc.rand()%3
else:
	exit()

win_count = 0
print("starting game")
while(True):
	if(win_count == 0):
		randint = firstRand
	else:
		timeVar = libc.time()
		libc.srand(timeVar)
		randint = libc.rand()%3

	guru_choice = guru_choices[randint]
	choice = player_choices[randint]
	s.send(choice.encode())
	printout = read_until(s, b">>")
	print(printout)

	if("lost" in printout):
		break
	if("won" in printout):
		win_count = win_count + 1
		continue

```
Important note: The order that guru_choices[] and player_choices[] are initialized is important. The exe determines if the player won by checking the player choice against guru_choices[(i-1)%3]. This means it knows if you won at rock/paper/scissors by comparing against only one result! The way my script is running, I just need to grab player_choices[i], because it will beat guru_choices[i].

I also definitely could've coded this better by providing the host address and port using command line arguments... but reinitializing the Docker happened rarely

## The apprentice becomes the master!

With my mighty script in hand, I march up the mountain! er... to the door of the tomb!

And I discover that my connection between the docker and my VM is spotty. Meaning that I get to run the script repeatedly to try and get a happy result without breaking the pipe. Goody.
(Authors note: After I had all of this python code written, it took me a good 20 minutes of running to actually end up with the flag. I got above 70 wins multiple times.)

## An alternate way!

I discovered afterwards that the (probably) intended method to solve this problem was to take advantage of the game() function in janken using strstr() and not strcmp(). Because it uses strstr(), if you add the line `choice = "rockpaperscissors"` in my code right after `choice = player_choices[randint]`, it just wins every time.

I feel silly

## Flag
HTB{r0ck_p4p3R_5tr5tr_l0g1c_buG}