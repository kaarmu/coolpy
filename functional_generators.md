# Functional generators

Think of the following example. We want to find all the indices for which the bit is high in an int. A solution
to this problem is rather simple.

``` python
def whichBits(n: int) -> list[int]:
    i, res = 0, []
    while 1<<i <= n:
        if 1<<i & n:
            res.append(i)
        i += 1
    return res
```

However, we have some loops and stuff and this does feel like a function we can compress by list comprehension or generators.

``` python
from itertools import count, takewhile

def compressedWhichBits(n: int) -> list[int]:
    return [i for i in takewhile(lambda i: 1<<i <= n, count()) if 1<<i & n] # can also write if-statement as a filter
```

Very pretty. But this is rather difficult to read, could we perhaps make some thing more readable? We clearly
see here that we have a data flow that goes like

1. Create indefinite incrementally increasing indices (who doesn't like a little alliteration 😉 )
2. Take values while `1<<i` isn't larger than the value (no point in continuing then)
3. Filter out high bits
4. Put values in a list

When working with generators like this it is sometimes very seducing to try to pipe them together. The functional programmer 
might think this is trivial, we are just speaking of composition, right?  Well in python, composition can be acheived the
following way...

``` python
from functools import reduce

def compose(*fs):
    """
    Functional composition of N functions.

    Can also be used to pipe generators.
    """
    if not fs:
        raise TypeError('compose expected at least 1 argument, got 0')
    return reduce(lambda f, g: lambda *a, **kw: f(g(*a, **kw)), reversed(fs))
```

... and it isn't really pretty. You have lambda function for each pairing and thata creates some overhead. Composition
isn't really built into the language as it is for other languages. Well well, we'll see what we can do with this. 

Using `compose` we can write the pipeline in order (due to the `reversed`) and the data flow should be much more readable.

``` python
from functools import partial

def composedWhichBits(n: int) -> list[int]:
    return compose(
        count,
        partial(takewhile, lambda i: 1<<i <= n),
        partial(filter, lambda i: 1<<i & n),
        list,
    )()
```

Notice how we must also call the composed pipeline in the end (using arguments to `count`).

Now, the fun part is timing our functions and comparing them!

``` python 
from random import random
from timeit import timeit

funcs = whichBits, compressedWhichBits, composedWhichBits

for f in funcs:
    print(
        f'{f.__name__}:',
        timeit(f'{f.__name__}(x)', 'x = int(100 * random())', globals=globals())
    )
```

Rather unsurprisingly we can see that the composed version was considerably slower. Again this is mainly due to the large overhead
that we get when having multiple layers of function calls like this, compared to just a single while-loop. But hey, readability matters also.

---

``` python
def whichBits(n: int) -> list[int]:
    i, res = 0, []
    while 1<<i <= n:
        if 1<<i & n:
            res.append(i)
        i += 1
    return res
    
from itertools import count, takewhile

def compressedWhichBits(n: int) -> list[int]:
    return [i for i in takewhile(lambda i: 1<<i <= n, count()) if 1<<i & n] # can also write if-statement as a filter

from functools import reduce

def compose(*fs):
    """
    Functional composition of N functions.

    Can also be used to pipe generators.
    """
    if not fs:
        raise TypeError('compose expected at least 1 argument, got 0')
    return reduce(lambda f, g: lambda *a, **kw: f(g(*a, **kw)), reversed(fs))
    
from functools import partial

def composedWhichBits(n: int) -> list[int]:
    return compose(
        count,
        partial(takewhile, lambda i: 1<<i <= n),
        partial(filter, lambda i: 1<<i & n),
        list,
    )()
    
from random import random
from timeit import timeit

funcs = whichBits, compressedWhichBits, composedWhichBits

for f in funcs:
    print(
        f'{f.__name__}:',
        timeit(f'{f.__name__}(x)', 'x = int(100 * random())', globals=globals())
    )
```
