## Baby REV

- We are given a file with ofuscated code using freetools.org
    - it is a reversed, base64 encoded and compressed with zlib function
    - We can run the functions in the following order
        - Reverse
        - Base64 decode
        - zlib decompress

```python
def deobfuscate(code):
    try:
        reversed_code = code[::-1]
        decoded_data = base64.b64decode(reversed_code)
        decompressed_data = zlib.decompress(decoded_data)
        return decompressed_data.decode()
    except Exception as e:
        return None
```

- Flag - `BITSCTF{obfuscation_and_then_some_more_obfuscation_4a4a4a4a}`
