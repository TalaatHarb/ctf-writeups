Using the multiplicative modular inverse of e and multiplying it with the encrypted flag should result in a decrypted flag,
that can be converted into a string to get the flag,


```python
inve = pow(e, -1, n)
dec = (inve * enc) % n
flag = int.to_bytes(dec, byteorder='big')
```

for more info look check the code
