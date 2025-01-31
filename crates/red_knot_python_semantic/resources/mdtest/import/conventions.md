# Import conventions

Tests that validates the import conventions.

Reference:
* <https://typing.readthedocs.io/en/latest/spec/distributing.html#import-conventions>

## Builtins scope

When looking up for a name, red knot will fallback to using the builtins scope
if the name is not found in the current scope. The `builtins.pyi` file, that will
be used to resolve any symbol in the builtins scope, contains multiple symbols
from other modules (e.g., `typing`) but those are not being re-exported.

```py
# These symbols are being imported in `builtins.pyi` but shouldn't be considered as being
# available in the builtins scope.

# error: "Name `Literal` used when not defined"
a: Literal[1] = 1

# error: "Name `sys` used when not defined"
reveal_type(sys.executable)  # revealed: str
```

## Builtins import

Try to import the symbols from the module which aren't explicitly exporting them.

```py
# error: "Module `builtins` does not explicitly export attribute `Literal`"
# error: "Module `builtins` does not explicitly export attribute `sys`"
from builtins import Literal, sys

# TODO: This should be an error but we don't understand `*` imports yet and
# the `collections.abc` uses `from _collections_abc import *`.
from math import Iterable
```

## Explicitly re-exported symbols

When explicitly re-exporting a symbol or a module, it should not raise an error
when importing it. This tests both `import ...` and `from ... import ...` forms.

Note: Submodule imports in `import ...` form doesn't work because it's a syntax
error. For example, in `import os.path as os.path` the `os.path` is not a valid
identifier.

```py
from b import Any, Literal, ast

reveal_type(Any)  # revealed: typing.Any
reveal_type(Literal)  # revealed: typing.Literal
reveal_type(ast)  # revealed: <module 'ast'>
```

```pyi path=b.pyi
import ast as ast
from typing import Any as Any, Literal as Literal
```

## Implicitly re-exported symbols

Here, none of the symbols are being re-exported in the stub file. Regardless,
we should still infer the types correctly.

```py
# error: 15 [implicit-reexport] "Module `b` does not explicitly export attribute `ast`"
# error: 20 [implicit-reexport] "Module `b` does not explicitly export attribute `Any`"
# error: 25 [implicit-reexport] "Module `b` does not explicitly export attribute `Literal`"
from b import ast, Any, Literal

reveal_type(Any)  # revealed: typing.Any
reveal_type(Literal)  # revealed: typing.Literal
reveal_type(ast)  # revealed: <module 'ast'>
```

```pyi path=b.pyi
import ast
from typing import Any, Literal
```

TODO:
* Conditional imports
* `__init__` files even though we don't special case it
* Submodule imports
