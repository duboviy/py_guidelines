py_guidelines
======= 

.. contents:: Contents

.. _general:

General recommendations 
------------------ 

- Follow `PEP-8`_. To automatically check and correct some 
  issues, you can use the utility `pep8`_ and `autopep8`_ . Unfortunately, 
  they do not check everything (for example, the naming convention is not checked). 
  Alternatively, you can use the static analyzer 
  `flake8`_, which in addition to style checks will find some simple 
  mistakes. Also you can use plug-in with naming checks - `pep8-naming`_. 

- Imported modules are located at the beginning of the file. 
- Import modules located in lexicographical order. 

  .. code:: python

     # Bad 
     import gzip
     import sys
     from collections import defaultdict
     import io
     from contextlib import contextmanager
     import functools
     from urllib.request import urlopen
	 
     # Better 
     import functools
     import gzip
     import io
     import sys
     from collections import defaultdict
     from contextlib import contextmanager
     from urllib.request import urlopen

- Use operators ``is`` and ``is not`` **only** for comparison with 
  singletons, for example, ``None``. Exception: boolean value 
  ``True`` and ``False``. 
- Remember and use *False* / *True* semantics. *False* values - 
  ``None``, ``False``, zero ``0``, ``0.0``, ``0j``, blank lines and 
  bytes, and all the empty collection. All other values *True*. 

  .. code:: python

     # Bad
     if acc == []:
         # ...

     # Bad
     if len(acc) > 0:
         # ...

     # Better
     if not acc:
         # ...

     # Acceptable
     if acc == 0:
         # ...

- Do not use the names of the standard built-ins and collections to name your variables. Remember that you always can pick up more appropriate name for your variable. 

- Do not copy without the need. 

  .. code:: python

     # Bad
     set([x**2 for x in range(42)])

     for x in list(sorted(xs)):
         # ...

     # Better
     {x**2 for x in range(42)}

     for x in sorted(xs):
         # ...

- Do not use ``dict.get`` and a collection of ``dict.keys`` to check the 
  presence of the key in the dictionary: 

  .. code:: python

     # Bad
     if key in g.keys():
         # ...

     if not g.get(key, False):
         # ...

     # Better
     if key in g:
         # ...

     if key not in g:
         # ...

- Use literals to create an empty collection. Exception: 
  ``set``, there is no empty set of literals in Python. 

  .. code:: python

     # Bad
     dict(), list(), tuple()

     # Better
     {}, [], ()

.. _PEP-8: https://www.python.org/dev/peps/pep-0008
.. _pep8: https://pypi.python.org/pypi/pep8
.. _autopep8: https://pypi.python.org/pypi/autopep8
.. _flake8: https://pypi.python.org/pypi/flake8
.. _pep8-naming: https://pypi.python.org/pypi/pep8-naming


.. _structure: 

Code structure 
-------------- 

- do not emulate the ``for``, Python - is not Scala. 

  .. code:: python

     # Bad
     i = 0
     while i < n:
         ...
         i += 1

     # Better
     for i in range(n):
         ...

- prefer iteration through object over the loops with the counter. Error with 1 in 
  the index - it's a classic. If the index is required, remember about the 
  ``enumerate``. 

  .. code:: python

     # Bad
     for i in range(len(xs)) :
         x = xs[i]

     # Better
     for x in xs:
         ...

     # Or
     for i, x in enumerate(xs):
         ...

  .. code:: python

     # Bad
     for i in range(min(len(xs), len(ys))):
         f(xs[i], ys[i])

     # Better
     for x, y in zip(xs, ys):
         f(x, y)

- Do not use ``dict.keys`` to iterate over a dictionary. 

  .. code:: python

     # Bad
     for key in dict.keys():
         ...

     # Better
     for key in dict:
         ...

- Do not use the methods ``file.readline`` and ``file.readlines`` for iterate over the 
  file. 

  .. code:: python

     # Bad
     while True:
         line = file.readline()
         ...

     for line in file.readlines():
         ...


     # Better
     for line in file:
         ...

- Do not write useless operators ``if`` and ternary operators. 

  .. code:: python

     # Bad
     if condition:
        return True
     else
        return False

     # Better
     return condition

  .. code:: python

     # Bad
     if condition:
        return False
     else
        return True

     # Better
     return not condition

  .. code:: python

     # Bad
     xs = [x for x in xs if predicate]
     return True if xs else False

     # Better
     xs = [x for x in xs if predicate]
     return bool(xs)

     # Best
     return any(map(predicate, xs))


.. _functions : 

Features 
------- 

- Avoid mutable default values. 
- Do not abuse the functional idioms. Often a list/set/dict comprehension 
  is more understandable than combinations of functions ``map``, ``filter`` and ``zip``. 

  .. code:: python

     # Bad
     list(map(lambda x: x ** 2,
              filter(lambda x: x % 2 == 1,
                     range(10))))

     # Better
     [x ** 2 for x in range(10) if x % 2 == 1]

- not abuse generator expression. Often the usual cycle ``for`` 
  is clearer then very large embedded generator. 
- Do not fold function with effects. The first argument to 
  ``functools.reduce`` should not change the state of variable in the outer 
  scope or value accumulator. 

  .. code:: python

     # Bad
     funtools.reduce(lambda acc, s: acc.update(s), sets)

     # Better
     acc = set()
     for set in sets:
         acc.update(set)

- Avoid useless anonymous functions. 

  .. code:: python

     # Bad
     map(lambda x: frobnicate(x), xs)

     # Better
     map(frobnicate, xs)

  .. code:: python

     # Bad
     collections.defaultdict(lambda: [])

     # Better
     collections.defaultdict(list)

.. _decorators: 

Decorators 
---------- 

- **Always**  use the ``functools.wraps`` or 
  ``functools.update_wrapper`` when writing a decorator. 


.. _strings: 

Strings 
------- 

- Use the method ``str.startswith`` and ``str.endswith``. 

  .. code:: python

     # Bad
     s[:len(p)] == p
     s.find(p) == len(s) - len(p)

     # Better
     s.startswith(p)
     s.endswith(p)

- Use formatting strings instead of explicit call of ``str`` and concatenation. 

  .. code:: python

     # Bad
     "(+ " + str(expr1) + " " + str(expr2) + ")"

     # Better
     "(+ {} {})".format(expr1, expr2)

  exception: to bring to the string of argument. 

  .. code:: python

     # Bad
     "{}".format(value)

     # Better
     str(value)

- Do not complicate the formatting template unnecessarily. 

  .. code:: python

     # Bad
     "{}".format(value)

     # Better
     str(value)

- Remember that the method ``str.format`` converts the argument to a string. 

  .. code:: python

     # Bad 
     "(+ {} {})".format(str(expr1), str(expr2))

     # Better
     "(+ {} {})".format(expr1, expr2)


.. _Classes: 

Classes 
------ 

- Use the ``collections.namedtuple`` for classes with a fixed set of 
  immutable fields. 

  .. code:: python

     # Bad
     class Point:
         def __init__(self, x, y):
             self.x = x
             self.y = y

     # Better
     Point = namedtuple("Point", ["x", "y"])

- Do not call a "magic method" directly if there is a suitable public function or operator. 

  .. code:: python

     # Bad
     expr.__str__()
     expr.__add__(other)

     # Better
     str(expr)
     expr + other

- Do not use ``type`` to verify that the object is an instance of a 
  class. More suitable is function ``isinstance``. 

  .. code:: python

     # Bad
     type(instance) == Point
     type(instance) is Point

     # Better
     isinstance(instance, Point)

.. _exceptions: 

Exceptions 
---------- 

- Minimize the size of blocks ``try`` and ``with``. 
- To catch any exception use the ``except Exception``, instead of 
  ``except BaseException`` or just ``except``. 
- Indicate the most specific exception type in ``except`` block. 

  .. code:: python


     # Bad
     try:
         mapping[key]
     except Exception:
         ...

     # Better
     try:
         mapping[key]
     except KeyError:
         ...

- Derive own exceptions from the ``Exception``, instead of ``BaseException``. 
- Use context managers instead of ``try-finally``. 

  .. code:: python

     # Bad
     handle = open("path/to/file")
     try:
         do_something(handle)
     finally:
         handle.close()

     # Better
     with open("path/to/file") as handle:
         do_something(handle)
