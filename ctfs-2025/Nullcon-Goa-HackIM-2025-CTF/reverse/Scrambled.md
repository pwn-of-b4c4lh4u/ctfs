## Scrambled

For this challenge, it is given the following code and text:

```
result: 1e78197567121966196e757e1f69781e1e1f7e736d6d1f75196e75191b646e196f6465510b0b0b57
```

```python
import random

def encode_flag(flag, key):
    xor_result = [ord(c) ^ key for c in flag] # for char in flag transform to asci and xor with key

    chunk_size = 4 
    
    chunks = [xor_result[i:i+chunk_size] for i in range(0, len(xor_result), chunk_size)]
    seed = random.randint(0, 10)
    random.seed(seed)
    random.shuffle(chunks)
    
    scrambled_result = [item for chunk in chunks for item in chunk]
    return scrambled_result, chunks

def main():
    flag = "REDACTED"
    key = REDACTED

    scrambled_result, _ = encode_flag(flag, key)
    print("result:", "".join([format(i, '02x') for i in scrambled_result]))

if __name__ == "__main__":
    reverse()
```

- To reverse these operations, the following steps must be followed:
    1. Read bytes from the hex data - `byte_values = bytes.fromhex(file_data)`
    2. Initialize a loop to bruteforce the XOR key
    3. Verify if the flag initial values are present in the decoded data
    4. If the previous is true, create the chunks so that the flag can be reorganized

```python
def reverse():
    with open('output.txt', 'r') as file:
        file_data = file.read().split(" ")[1]
        
    byte_values = bytes.fromhex(file_data)
    
    for key in range(256):
        flag = bytes([byte ^ key for byte in byte_values]).decode(errors='ignore')
        if "ENO" in flag:
            chunk_size = 4 
            chunks = [flag[i:i+chunk_size] for i in range(0, len(flag), chunk_size)]
            break
    print(chunks)

# Result: ['4R3_', 'M83L', '3D_T', '5CR4', '45TY', 'GG5_', '3D_3', '1ND3', 'ENO{', '!!!}']
```

- Now we can just reorganize the flag - `ENO{5CR4M83L3D_3GG5_4R3_1ND33D_T45TY!!!}`