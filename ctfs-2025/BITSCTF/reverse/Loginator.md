## Loginator

- we are given a binary and some encoded bytes

- When running the binary we can see the letter get encoded to 2 bytes hexadecimal.
    - If we test the input with BITSCTF we see the same output as the encoded bytes given to us

- So we can bruteforce the flag by trying the different chars of the charset
    - Retrieve the output and compare for each letter with the current expected char

```python
import subprocess
import string

expected_output = "02 92 a8 06 77 a8 32 3f 15 68 c9 77 de 86 99 7d 08 60 8e 64 77 be ba 74 26 96 e7 4e".split()
flag = "BITSCTF{"
flag_size = 28
charset = string.ascii_letters + string.digits + string.punctuation

def get_encoded_output(candidate):
    result = subprocess.run(["./loginator.out", candidate], capture_output=True, text=True)
    return result.stdout.strip().split()

while len(flag) < flag_size:
    for char in charset:
        test_flag = flag + char
        output = get_encoded_output(test_flag)
        
        if output[:len(test_flag)] == expected_output[:len(test_flag)]:
            flag = test_flag
            print(f"Found: {flag}")
            break

print(f"Recovered flag: {flag}")
```

- Flag - `BITSCTF{C4ND4C3_L0G1C_W0RK?}`