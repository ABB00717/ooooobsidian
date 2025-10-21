identifier
data type
compound data type
_c-like initialization_
_constructor initialization_
_uniform initialization_

`auto` and `decltype`

Therefore, unless you have a strong reason not to, you should always use `getline` to get input in your console programs instead of extracting from `cin`.

template <template-parameters> function-declaration

the value of template parameters is determined on compile-time to generate a different instantiation of the function `fixed_multiply`, and thus the value of that argument is never passed during runtime: The two calls to `fixed_multiply` in `main` essentially call two versions of the function: one that always multiplies by two, and one that always multiplies by three. For that same reason, the second template argument needs to be a constant expression (it cannot be passed a variable).

Namespaces allow us to group named entities that otherwise would have _global scope_ into narrower scopes, giving them _namespace scope_.

Existing namespaces can be aliased with new names, with the following syntax:  
`namespace new_name = current_name;`

But there is another substantial difference between variables with _static storage_ and variables with _automatic storage_:  
- Variables with _static storage_ (such as global variables) that are not explicitly initialized are automatically initialized to zeroes.  
- Variables with _automatic storage_ (such as local variables) that are not explicitly initialized are left uninitialized, and thus have an undetermined value.


