### Possible future features
This is a non-exhaustive list of features that could be implemented in Dijon:

 - Shorthand "string mode" and function call/definition shorthand
   - `string "Hello, world!"`
     - `string "Hi"` equivalent to `sb_clear ~ arg0 uh ~ sb ~ arg0 li ~ string sb`
   - `return_val function(1 2)` equivalent to `arg0 1 ~ arg1 2 ~ return_val function`
 - Trigger flags `:varname|flag ... ;`
   - `set` flag: runs trigger when variable is set, rather than fetched
   - `self` flag: trigger can run itself
 - `#` command with 1 argument (rebase)
   - Let `var` be a reference
     - calling `var #` will set var to the Variable it's referencing