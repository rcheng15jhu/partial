# partial



## Code Tutorial

To use the command line program, run `./partial -i <path/to/.jil>`. The result of the partial evaluation will be printed to console as another valid `.jil` program. To use in utop, first run the following commands: `# open PartialEvaluator; open PartialEval;;`. This will expose helper functions to run in the top-loop: for a full list of functions, see code structure below. Use `# rep <path/to/.jil>` to have the result of partial evaluation printed to console, and use  `# drep <path/to/.jil>` to have the end environment (“penv”) printed to console. There is also a `SimpleEval` module that can be used to run the dependency accumulating evaluator, which has the same helper functions available in the top loop; it is created from the same functor. The functor may be modified to take programs by string instead of file, see code structure below. (If there is access to the surrounding jaylang repository, test cases may be found in `/test/partial/`)



## Code Summary

The code is split between two files, `partial_evaluator.ml`, and `types_and_printing.ml`. Both files reference Batteries (for data structure functors), Jhupllib (for pp & yojson), Jayil.Ast (for type definitions that expr uses), and Dj_common (for file utils).

`types_and_printing.ml` contains all the new types and modules used by the partial evaluator, including `lexadr`, `identline`, `pvalue`, `presidual`, and `penv`. Their associated pretty printing functions are also included. The option+set monad is also included here. As `penv` is defined here, helper functions to add to and retrieve from the environment are also included, as well as to create more complicated data structures from an existing environment. Finally, the file contains also contains the functor to facilitate easy utop usage with useful commands, that takes in a parser module, as well as a eval/peval module.

`partial_evaluator.ml` contains the main functions for the dependency-accumulating testing evaluator, as well as the partial evaluator. It also contains helper functions more closely related to evaluation and expression manipulation, including prepending captured variables, reconstructing an expression from a dependency set, capturing analysis, and identifier renaming throughout an expression. It also contains helper functions that process the final monad result during partial evaluation of a line to decide what entry to emit for that line.



## Code Description

### `types_and_printing.ml` (Key Types)

#### `type lexadr`
A 2-tuple representing a “lexical address”, whose first integer entry denotes the environment depth, and whose second integer entry denotes the line number.

#### `type identline`
A variant type that acts as the type of the key for the interpreter environment. It either holds an identifier, or a lexadr.

#### `type pvalue`
This types augments values from the AST to include values with closures, like functions and records.

#### `type presidual`
A variant type that is part of the entry of the environment. The entry assigned to each identifier or line number is either a pvalue, the original clause body from the ast, or an expression to be inlined. Additional variants may be added as needed to signal different behavior of the partial evaluator.

#### `type presidual_with_ident_lexadr_deps`
This tuple type is the entry of the environment. The first entry is the identifier being assigned to, the second entry is a presidual, and the last entry is a set of lexical addresses that represent that lines previous dependencies (i.e., if the line remains in the residual program, which previous lines are in the def/use chain, and are required to also remain?)

#### `type penv`
This map type acts as the environment for partial evaluation. It takes identline as keys, and presidual_with_ident_lexadr_deps as entries. Helper functions are included to facilitate add to and retrieve from the environment, as well as to create more complicated data structures from an existing environment.

#### `module OptionSetSyntax`
This module contains applicative and monad composition functions that allow OCaml's new monad syntax to be used. The specific monad, a Cartesian product of an option and set union monad, hides None checking, and allows line dependencies to be accumulated.

#### `module PETopLoop`
This functor creates utop helper functions given a parser module and a peval module. These helper functions include printing evaluation results, and also the environment at the end of evaluation.


### `partial_evaluator.ml` (Key Functions)

#### `bail_with, bail_compose`
These two functions take the output of the monad and process it as the final result of partially evaluating a line in the program. If evaluation succeeds, then the accumulated dependencies are removed (with the exception of closure values keeping dependencies while still being values). Otherwise, the fallback clause body is returned with the accumulated dependencies.

#### `prepend_vars_if_pvalue_else_env_deps`
This function prepares a function closure by iterating through its captured variables. If the captured variables are known at peval time, they are prepended to the function body. Otherwise, they are added to the function environment and line dependencies.

#### `reconstruct_expr_from_lexadr`
This function reconstructs an expression given some environment and a environment number. It searches for the last line in the environment, and iterates through its dependencies in ascending order to reconstruct each line.

#### `simple_cval`
This function performs capturing analysis on an expression to return a set of all free variables. This is useful for constructing function closures.

#### `simple_eval`
This function performs dependency-accumulating evaluation on an expression, returning a value and a penv.

#### `simple_peval`
This function performs partial evaluation on an expression, returning an expr and a penv.



## Additional Comments for future work

