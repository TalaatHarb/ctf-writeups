# nanothorpe

## Description of challenge
Welcome back to ls-as-a-service, the most useful tool on the internet!

Download:
[nanothorpe-d2095dcfeda4b08d.tar.xz ](https://2020.ctf.link/assets/files/nanothorpe-d2095dcfeda4b08d.tar.xz)(13.3 KiB)

Connection (mirrors):
[http://157.90.22.14:8832/](http://157.90.22.14:8832/)

The UI makes a GET request to the backend at the endpoint '/api/authorize' with the parameter 'cmd' (url encoded) ; where the API checks the parameter and confirms that the command is 'ls' and not blank or anything else and then it creates a cookie with an expiration data parameter that acts as a signature to another endpoint '/api/run' which perform the execution of the command after it verifies the validity of the request signature and expiration time and returns the results of the execution in base64.

The signature is created using a hashing function made for this challenge specifically called 'octothorpe' with its custom padding and it does the signing by appending a secret data before the parameters the user sends.

## Solution
There are three problems with this implementation, the first is that the confirmation of the content of the command is done separatly from the execution of the command which suggests that if you can bypass the signature check you can trick the execution endpoint to execute random commands for you. The second is that the hashing function 'octothorpe' is (spoiler alert) vulnerable to length extension attacks. The third is that if you send multiple 'cmd' parameters the server takes the last one.

### Length extension attack
To confirm that the hashing function is vulnerable to the length extension attack, you try hashing some data and using it as the state for the next attempt of hashing and if that predicts the resulting hash of appending data to the original data.

```python
from octothorpe import octothorpe

secret = b'very secret data' # sample
decoded = b'expiry=1608460998&cmd=ls' # original data

secret_length = len(secret)
actual_length = secret_length + len(decoded)

# Predicting padding
padding = b'\x80' + b'\x00' * ((octothorpe.block_size - 9 - actual_length) % octothorpe.block_size) + (8 * actual_length).to_bytes(8, 'little')

# Appended data
b = b'; ls -ltr'

n = actual_length + len(padding)
ha = octothorpe(secret + decoded).hexdigest() # the signature

hb = octothorpe(b, _length=n, _state=bytearray.fromhex(ha)).hexdigest() # Your prediction of the hash

h = octothorpe(secret + decoded + padding + b).hexdigest() # The target hash

if hb == h:
    print('Bingo')
```

Executing this code confirms that the hashing function is vulnerable to length extension attacks

### Exploit
The exploit works as follows, hit the first end point with the right parameters that it expects and receicve the signature and then manufacture another request instead of the redirected one that contains the padding and an additional 'cmd' parameter that prints the flag for you and of course predict the signature for this new request using the length extension attack


For the actual exploit check the original write-up
