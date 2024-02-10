# Python Type Checking

## Examples

```python
import math

def circumference(radius: float) -> float:
    return 2 * math.pi * radius
```

```python
pi: float = 3.142

def circumference(radius: float) -> float:
    return 2 * pi * radius
```

### Sequences and Mappings

```python
>>> name: str = "Guido"
>>> pi: float = 3.142
>>> centered: bool = False

>>> names: list = ["Guido", "Jukka", "Ivan"]
>>> version: tuple = (3, 7, 1)
>>> options: dict = {"centered": False, "capitalize": True}
```

You should use the special types defined in the `typing` module. These types add syntax for specifying the types of elements of composite types. You can write the following:

```python
>>> from typing import Dict, List, Tuple

>>> names: List[str] = ["Guido", "Jukka", "Ivan"]
>>> version: Tuple[int, int, int] = (3, 7, 1)
>>> options: Dict[str, bool] = {"centered": False, "capitalize": True}
```

```python
def create_deck(shuffle: bool = False) -> List[Tuple[str, str]]:
    """Create a new deck of 52 cards"""
    deck = [(s, r) for r in RANKS for s in SUITS]
    if shuffle:
        random.shuffle(deck)
    return deck
```

In many cases your functions will expect some kind of sequence, and not really care whether it is a list or a tuple. In these cases you should use typing.Sequence when annotating the function argument:

```python
from typing import List, Sequence

def square(elems: Sequence[float]) -> List[float]:
    return [x**2 for x in elems]
```

You can for instance create Card and Deck type aliases:

```python
from typing import List, Tuple

Card = Tuple[str, str]
Deck = List[Card]

def deal_hands(deck: Deck) -> Tuple[Deck, Deck, Deck, Deck]:
    """Deal the cards in the deck into four hands"""
    return (deck[0::4], deck[1::4], deck[2::4], deck[3::4])
```

```python
>>> from typing import List, Tuple
>>> Card = Tuple[str, str]
>>> Deck = List[Card]

>>> Deck
typing.List[typing.Tuple[str, str]]
```

### No return

```python
# play.py

def play(player_name: str) -> None:
    print(f"{player_name} plays")

ret_val = play("Filip")
```

```python
from typing import NoReturn

def black_hole() -> NoReturn:
    raise Exception("There is no going back ...")
```

### The Any Type

```python
import random
from typing import Any, Sequence

def choose(items: Sequence[Any]) -> Any:
    return random.choice(items)
```


### Type Variables

```python
# choose.py

import random
from typing import Sequence, TypeVar

Choosable = TypeVar("Choosable")

def choose(items: Sequence[Choosable]) -> Choosable:
    return random.choice(items)

names = ["Guido", "Jukka", "Ivan"]
reveal_type(names)

name = choose(names)
reveal_type(name)
```
```shell
$ mypy choose.py
choose.py:12: error: Revealed type is 'builtins.list[builtins.str*]'
choose.py:15: error: Revealed type is 'builtins.str*'
```

```python
# choose_examples.py

from choose import choose

reveal_type(choose(["Guido", "Jukka", "Ivan"]))
reveal_type(choose([1, 2, 3]))
reveal_type(choose([True, 42, 3.14]))
reveal_type(choose(["Python", 3, 7]))
```
```shell
$ mypy choose_examples.py
choose_examples.py:5: error: Revealed type is 'builtins.str*'
choose_examples.py:6: error: Revealed type is 'builtins.int*'
choose_examples.py:7: error: Revealed type is 'builtins.float*'
choose_examples.py:8: error: Revealed type is 'builtins.object*'
```

Is there a way to tell the type checker that choose() should accept both strings and numbers, but not both at the same time?
```python
# choose.py

import random
from typing import Sequence, TypeVar

Choosable = TypeVar("Choosable", str, float)

def choose(items: Sequence[Choosable]) -> Choosable:
    return random.choice(items)

reveal_type(choose(["Guido", "Jukka", "Ivan"]))
reveal_type(choose([1, 2, 3]))
reveal_type(choose([True, 42, 3.14]))
reveal_type(choose(["Python", 3, 7]))
```
```shell
$ mypy choose.py
choose.py:11: error: Revealed type is 'builtins.str*'
choose.py:12: error: Revealed type is 'builtins.float*'
choose.py:13: error: Revealed type is 'builtins.float*'
choose.py:14: error: Revealed type is 'builtins.object*'
choose.py:14: error: Value of type variable "Choosable" of "choose"
                     cannot be "object"
```
```python
Choosable = TypeVar("Choosable", str, Card)

def choose(items: Sequence[Choosable]) -> Choosable:
    ...
```


### Duck Types and Protocols

One way to categorize type systems is by whether they are nominal or structural:

- In a nominal system, comparisons between types are based on names and declarations. The Python type system is mostly nominal, where an int can be used in place of a float because of their subtype relationship.

- In a structural system, comparisons between types are based on structure. You could define a structural type Sized that includes all instances that define `.__len__()`, irrespective of their nominal type.


A protocol specifies one or more methods that must be implemented. For example, all classes defining `.__len__()` fulfill the typing.Sized protocol. We can therefore annotate `len()` as follows:

```python
from typing import Sized

def len(obj: Sized) -> int:
    return obj.__len__()
```

Other examples of protocols defined in the typing module include `Container`, `Iterable`, `Awaitable`, and `ContextManager`.

You can also define your own protocols. This is done by inheriting from `Protocol` and defining the function signatures (with empty function bodies) that the protocol expects. 
The following example shows how `len()` and `Sized` could have been implemented:

```python
from typing_extensions import Protocol

class Sized(Protocol):
    def __len__(self) -> int: ...

def len(obj: Sized) -> int:
    return obj.__len__()
```

### The Optional Type

```python
def player_order(names, start=None):
    """Rotate player order so that start goes first"""
    if start is None:
        start = choose(names)
    start_idx = names.index(start)
    return names[start_idx:] + names[:start_idx]

# to:
from typing import Sequence, Optional

def player_order(
    names: Sequence[str], start: Optional[str] = None
) -> Sequence[str]:
    ...
```
The `Optional` type simply says that a variable either has the type specified or is `None`. An equivalent way of specifying the same would be using the `Union` type: `Union[None, str]`


### Annotating *args and **kwargs

```python
class Game:
    def __init__(self, *names: str) -> None:
        """Set up the deck and deal cards to 4 players"""
        deck = Deck.create(shuffle=True)
        self.names = (list(names) + "P1 P2 P3 P4".split())[:4]
        self.hands = {
            n: Player(n, h) for n, h in zip(self.names, deck.deal(4))
        }
```


### Callables

Functions, as well as lambdas, methods and classes, are represented by `typing.Callable`. 
The types of the arguments and the return value are usually also represented. 
For instance, `Callable[[A1, A2, A3], Rt]` represents a function with three arguments with types `A1`, `A2`, and `A3`, respectively. The return type of the function is `Rt`.

```python
# do_twice.py

from typing import Callable

def do_twice(func: Callable[[str], str], argument: str) -> None:
    print(func(argument))
    print(func(argument))

def create_greeting(name: str) -> str:
    return f"Hello {name}"

do_twice(create_greeting, "Jekyll")
```


## Pros and Cons

### Pros 

- Type hints help document your code. Traditionally, you would use docstrings if you wanted to document the expected types of a function’s arguments. This works, but as there is no standard for docstrings (despite PEP 257 they can’t be easily used for automatic checks.

- Type hints improve IDEs and linters. They make it much easier to statically reason about your code. This in turn allows IDEs to offer better code completion and similar features. With the type annotation, PyCharm knows that text is a string, and can give specific suggestions based on this.

- Type hints help you build and maintain a cleaner architecture. The act of writing type hints forces you to think about the types in your program. While the dynamic nature of Python is one of its great assets, being conscious about relying on duck typing, overloaded methods, or multiple return types is a good thing.

### Cons

- Type hints take developer time and effort to add. Even though it probably pays off in spending less time debugging, you will spend more time entering code.

- Type hints work best in modern Pythons. Annotations were introduced in Python 3.0, and it’s possible to use type comments in Python 2.7. Still, improvements like variable annotations and postponed evaluation of type hints mean that you’ll have a better experience doing type checks using Python 3.6 or even Python 3.7.

- Type hints introduce a slight penalty in startup time. If you need to use the typing module the import time may be significant, especially in short scripts.





## Credits

- [Python Type Checking (Guide)](https://realpython.com/python-type-checking)
