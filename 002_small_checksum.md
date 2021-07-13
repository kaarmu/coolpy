# Small checksum

It is often you have to do a checksum in applications. This is of course very simple. The implementation in python is just

``` python
def checksum(data: bytes) -> bytes:
    chk = 0
    for byte in data:
        chk ^= byte
    return bytes([chk])
```

But as engineers we like to use abstractions at unnecessary places, so why not here? In python we can make use of the stdlib
modules `functools` and `operators`. In `operators` we simply get the callable for the XOR operator `^`, i.e. `operators.xor`. 
And, in `functools` we want to use `reduce` to apply the XOR throught every byte in `data`. The result is a oneliner,

``` python
from functools import reduce
from operators import xor

def reducedChecksum(data: bytes) -> bytes:
    return bytes([reduce(xor, data, 0)])
```

`reduce` also have the nice feature to include an initilizer value as the third argument so that we don't need to set
`chk = 0` as we did previously.

So now we test them! Below I've setup some timing tests which I thought were interesting.
``` python
from timeit import timeit

funcs = 'checksum', 'reducedChecksum'

# Test 1
x = b'hello'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 0.5060404000032577
# reducedChecksum: 0.5356418000010308

# Test 2
x = b'hello checksums!'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 0.9298791999972309
# reducedChecksum: 0.9729004999971949

# Test 3
x = b'hello checksums! How are you today?'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 1.6949953000003006
# reducedChecksum: 1.6027499999981956

# Test 4
x = b'hello checksums! How are you today? You seem very happy.'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 2.5522939000002225
# reducedChecksum: 2.343182500000694
```

---

``` python
def checksum(data: bytes) -> bytes:
    chk = 0
    for byte in data:
        chk ^= byte
    return bytes([chk])
    
    
from functools import reduce
from operators import xor

def reducedChecksum(data: bytes) -> bytes:
    return bytes([reduce(xor, data, 0)])
    

from timeit import timeit

funcs = 'checksum', 'reducedChecksum'

# Test 1
x = b'hello'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 0.5060404000032577
# reducedChecksum: 0.5356418000010308

# Test 2
x = b'hello checksums!'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 0.9298791999972309
# reducedChecksum: 0.9729004999971949

# Test 3
x = b'hello checksums! How are you today?'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 1.6949953000003006
# reducedChecksum: 1.6027499999981956

# Test 4
x = b'hello checksums! How are you today? You seem very happy.'
for func in funcs:
    print(f'{func}:', timeit(f'{func}(x)', globals=globals()))
# checksum: 2.5522939000002225
# reducedChecksum: 2.343182500000694
```
