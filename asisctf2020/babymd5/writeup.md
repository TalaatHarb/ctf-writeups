# Challenge Description
Baby MD5
Your mission is to find weird hash collision!

nc 66.172.10.203 2570

# Solution
The challenge is split into two parts, the first is a proof-of-worth (PoW) and the second is the actual challenge
Booth parts are about getting partial hash collisions and by that I mean (4-6) hex digits of the hash are the same

## PoW
You are asked to provide a printable string with a defined length that when hashed with a hash function (that is defined in the question); will have the requested 6 hex digits at the end of the hash

Example: Please submit a printable string X, such that md5(X)[-6:] = 937087 and len(X) = 40
options for hash functions are SHA1, SHA224, SHA256, SHA384, SHA512, MD5 and may be others that I didn't see

### How to solve?
Generating random strings with the requested length and testing if thier hash matches the required digits is actually possible since the search is space only 6 hex digits
Something like this in a loop could work
```python
   new_str = ''.join(random.choice(string.ascii_letters) for _ in range(length))
   new_hash_seg = f(new_str) # f is your hash function
```
Another faster solution is to use existing rainbow tables for hash functions that are pre-computed to get a string that matches the request but we didn't end up going for this solution

## Actual challenge
You are asked to provide a pair of strings that will result True in a given function (babymd5) which just checks the two strings to make sure they meet certain conditions.
The conditions are that they have a pre-defined prefixes (one of them is the word 'dead'), and also that after repeatedly hashing them using MD5 they will provide the same hash

```python
def babymd5(m, n, x_head, y_head, x, y):
  if x.startswith(x_head) and y.startswith(y_head):
    for _ in range(m):
      xhash = md5(x.encode('utf-8')).hexdigest()
      x = xhash
    for _ in range(n):
      yhash = md5(y.encode('utf-8')).hexdigest()
      y = yhash
    if xhash == yhash:
      return True
return False
```
Where you are given m, n, x_head and y_head = 'dead' and you are asked to produce x and y
Example: (m, n, x_head, y_head) = (260, 103, 'Dtj', 'dead')

### How to solve? (We actually didn't solve this part during the competition)
The solution depends on the fact that the word 'dead' is a valid start of a hex digest and since the babymd5 function repetadly performs the hash on x;
then all what we need is a string x that starts with the given x_head and that hashes to a hash that starts with the digits 'dead' and then we use the resulting hash as our string y

Something like this in a loop will work:

```python
   possible_x = x_head + ''.join(random.choice(string.ascii_letters) for _ in range(length))
   x_hash = md5(possible_x.encode('utf-8')).hexdigest()
   if x_hash.startswith('dead')
    y = x_hash
    x = possible_x
    break
```
since the length isn't restricted in this scenario, you might want to choose a good enough length to maximize the possility of the partial 4 hex digit collision
