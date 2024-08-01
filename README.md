# ADJUVANT

## Synopsis

Utility functions used in other packages.

## API

### List Manipulation
* `hash-to-assoc` -- Build a car/cadr assoc-list from the entries in a given hash table.

* `assoc-to-hash` -- Build an assoc-list from the entries in a given hash table.

* `exists-tail` -- iterate a variable across a list, as if by `CL:MAPL`, until an element is found to verify
the `BODY`, at which point the tail, including the verifying element, is returned.
The values of the variable are the cons cells of the given list, not the individual elements.
Otherwise, `NIL` is returned.
```lisp
(exists-tail x '(1 3 5 2 x x x)
  (evenp (car x)))
==> (2 x x x)
```

* `sort-unique` -- sort a list and remove duplicate elements.  This is
more efficient than sorting and calling `CL:REMOVE-DUPLICATES`,
because duplicates can be removed in linear time when it is known
that the list is already sorted. E.g.,
```lisp
(sort-unique '(1 4 3 5 3 4 1) #'< #'=)
==> '(1 3 4 5)
```

* `remfq` -- remove (with `CL:REMOVE`) element from place destructivly using `EQ` for equivalence.

* `find-duplicates` -- Return a uniquified list of elements, each of which appear more than once in the given DATA list.

* `group-by` -- Create an alist by applying a key function to every element of a sequence
E.g., to group the lists in an array by length.

```lisp
PKG> (group-by #((1) (1 2) (3) (1 2 3) (3 4)) :key #'length)
==> ((3 ((1 2 3)))
     (2 ((3 4) (1 2)))
     (1 ((3) (1))))
```

 E.g., to group strings together in `string-equal` case-independent equal lists.

```lisp
PKG> (group-by '("aaa" "aAA" "AAA" "b" "BA" "bA" "AaA") :key #'identity :test #'string-equal)
==> (("BA"  ("bA" "BA"))
     ("b"   ("b"))
     ("aaa" ("AaA" "AAA" "aAA" "aaa")))
```

* `tconc` -- Standard function missing from Common Lisp, adds an item to the END of a conc structure.
1. Initialize a conc structure by calling `(tconc nil)`, this returns a structure which should be reused
in successive calls to `tconc` or `lconc`.  E.g., `(setf *buf* (tconc nil))`.
2. Destructively add one item to the end of the list with `tconc`, e.g.,
`(tconc *buf* 'the-item)`.
3. Destructively add muliple explicit items: `(tconc *buf* 'item1 'item2 'item3)`.
4. Destructively splice in multiple items: E.g., `(lconc *buf* '(item1 item2 item3)`.  Beware, the given list becomes the tail of the conc list.  Therefore successives calls to `tconc` or `lconc` will modify this list.
```lisp
PKG> (tconc *buf* *x*)
PKG> (tconc *buf* 'item) ;; as a side effect, *x* has been destructively to contain `item`.
```
5. Non-destructively extract the collected list with `car`. E.g., `(car *buf*)`
```lisp
PKG> (defvar *buf* (tconc nil))
*BUF*
PKG> *buf*
(NIL)
PKG> (tconc *buf* 'a)
((A) A)
PKG> (tconc *buf* 'b)
((A B) B)
PKG> (tconc *buf* 'c 'd 'e)
((A B C D E) E)
PKG> (car *buf*)
(A B C D E)
PKG> (lconc *buf* '(u v w))
((A B C D E U V W) W)
PKG> (lconc *buf* '(x y z))
((A B C D E U V W X Y Z) Z)
PKG> (car *buf*)
(A B C D E U V W X Y Z)
```

* `lconc` -- Standard function missing from Common Lisp; like tconc but adds multiple items to the end of a conc structure.
See `tconc` for example.

* `map-pairs` -- Call a given function over all the x,y pairs from a given list
```lisp
PKG> (let (pairs)
       (map-pairs (lambda (a b) (push (list a b) pairs))
                  '(a b c d))
       pairs)
===> ((C D) (B D) (B C) (A D) (A C) (A B))
```

* `dolist-tconc` -- like `DOLIST` but used for iterating over a TCONC structure.
An advantage of useing `DOLIST-TCONC` rather than `DOLIST` is that you can call TCONC
within the loop an be sure that the new elements are visited before the loop terminates.
```
PKG> (defvar *BUF* (list nil))
PKG> (tconc *BUF* 10)
PKG> (dolist-tconc (item *BUF*)
       (when (and (not (member (1- item) (car *BUF*)))
                  (plusp (1- item)))
          (tconc *BUF* (1- item))))
PKG> (car *BUF*)
==> (10 9 8 7 6 5 4 3 2 1)
```

* `linear-reduce` -- The same as `CL:REDUCE`, except that `LINEAR-REDUCE` allows an extra keyword argument, `:STOP-WHEN`
 which it ignores.   This is so `LINEAR-REDUCE` may be used as a drop-in replacement for `TREE-REDUCE`
 for the purpose of testing and profiling."

* `tree-reduce` -- Same semantics as `CL:REDUCE`, but does the evaluation tree-wise rather than left-to-right.
I.e., it attempts to evaluate as `(+ (+ (+ x0 x1) (+ x2 x3)) (+ (+ x4 x5) (+ x6 x7)))`, rather than 
`(+ (+ (+ (+ (+ (+ (+ x0 x1) x2) x3) x4) x5) z6) z7)`.
Of course this is only possible if the number of objects given is a power of 2.
Otherwise, it will tree-fold what it can, and use a simple `cl:reduce` with the remaining elements.

* `unionf` -- destructive union operator.
```lisp
PKG> (setf *x* '(1 2 3 4))
(1 2 3 4)
PKG> (unionf *x* '(2 4 6 8))
(3 1 2 4 6 8)
PKG> *x*
(3 1 2 4 6 8)
```

* `shuffle-list` -- Return a new list with the elements of the given list in randomized order.

* `choose-randomly` --  Return a list of `N` elements from `DATA` chosen at random, (in random order).
If `N > (length of data)` then a permutation of `DATA` is returned.

* `rnd-element` -- Returns a list of two elements 1) a randomly selected element of `DATA`
  and 2) a copy of DATA with the element removed, sharing a tail of `DATA`.

* `topological-sort` -- Graph is an association list whose keys are objects and whose
values are lists of objects on which the corresponding key depends.
Test is used to compare elements, and should be a suitable test for
hash-tables.  Topological-sort returns two values.  The first is a
list of objects sorted toplogically.  The second is a boolean
indicating whether all of the objects in the input graph are present
in the topological ordering (i.e., the first value).

* `take-while` -- Returns a copy of the head of the given list of `ITEMS` containing all the element of `ITEMS`
 for which the given `PREDICATE` returns true.  The list returns terminates (and does not include
 the first item which satisfies the given `PREDICATE`.
 This function returns two values, first is the head of the list as described above,
 second is the rest of the list from that point.
"
```lisp
PKG> (take-while #evenp '(2 4 6 8 9 10 11 12))
(2 4 6 8), (9 10 11 12)
```

* `delimit-on` -- Nondestructively chop a given list of `ITEMS` into consecutive sublists such that if you append them back together
 you get the original list.  The chopping occurs on each element for which the given `PREDICATE`
 returns `true`.  A list is returned whose first element is a possibly `NIL` list of the leading
 elements for which the `PREDICATE` is false, thereafter the elements are lists whose first element
 satisfies the `PREDICATE`, and whose trailing elements do not satisfy the `PREDICATE`. e.g.,
```lisp
 (delimit-on #'evenp '(1 3 5 2 5 7 10 12 3 5 7)
 =>   ((1 3 5) (2 5 7) (10) (12 3 5 7))
 (delimit-on #'evenp '(2 5 7 10 12 3 5 7)
 =>   (nil (2 5 7) (10) (12 3 5 7))"
```

### Iterators

* `map-subsets` --  call the given VISITOR function once for each subset of the list DATA

* `map-permutations` -- call the given VISITOR function once for each permutation of the given list DATA

* `exists` -- Tests whether there exists an element which satisfies an expression.  E.g., `(exists x '(1 2 3) (evenp x))`
* `forall` -- Tests whether all the elements in a given list satisfies an expression.  E.g., `(forall x '(2 4 6 8 10) (evenp x))`
* `setof` -- Construct a new list of all the elements which satisfy an expression.  E.g., `(setof x '(2 3 5 6 7) (evenp x))`
* `while` -- Loop while condition true. E.g., `(while (evenp x) (setf x (g x)))`


### Implementation independent interfaces sbcl/Allegro

* `valid-type-p` -- Predicate to determine whether the given object is
a valid type specifier.


* `type-expand` -- expand a type specifier into base types. E.g.,
```lisp
PKG> (deftype and-not (x y)
       `(and ,x (not ,y)))
PKG> (type-expand '(and-not integer fixnum))
==> (AND INTEGER (NOT FIXNUM))
```

* `process-kill` -- Kill a process started by `run-program` if it was started with `(run-program ... :wait t)`

* `*dot-path*` -- Special variable containing the full path to the UNIX `dot` (graphviz) program.

* `run-program` -- Wrapper around the implementation dependent (sbcl/Allegro) shell command function.

* `getenv` -- Wrapper around the implementation dependent (sbcl/Allegro) UNIX environment variable reader.

* `demand-env-var` --  Signal an error if the named environment variable is missing, otherwise return its value. 

* `garbage-collect` -- run the garbage collector

### Other

* `count-1-bits` -- given an non-negative integer, count the number of 1 bits in its binary representation

* `count-bit-diffs` -- Given two unsigned integers, count the numbers of bits their binary representations differ by.


* `prog1-let` -- This macro declares the given variable, and returns its value after the body has been evaluated. E.g.,
```lisp
(prog1-let (A 100)
   ...)
```
This expression binds A to 100 and then evaluates the body.  It returns 100
unless the body modifies the value of A, otherwise that new value of A is
returned.

* `diff-files` -- Given two file names, acceptable as 2nd argument of `CL:WITH-OPEN-FILE`, return TRUE
if the files differ and return FALSE if they are the same.
Same ==> FALSE
Different ==> TRUE
Same vs different are judged by the content.  If the files are different lengths
the files are different, otherwise when reading the files in parallel one character at
a time using `CL:READ-CHAR`, if all each sequence of characters are the same according to
EQL, then the files are judged to be the same.

* `replace-all` -- find an replace all occurances of once string in another.
```lisp
(replace-all "abc++def++ghi++" "++" "---")
==> "abc---def---ghi---"
  (replace-all (format nil "~A/~A.~A" dir-name base extension)
                                "//" "/")
```

* `fixed-point` -- find the fixed point of a function.  I.e. call the given function
on the given intial value to produce the next value.  Continue producing new values
in this manner until two successive equal values have been returned according to a :TEST function.
   

* `boolean-expr-to-latex` -- Generate LaTeX code, and print to `*standard-output*` to post into a tex document to represent a Boolean expression  E.g.,
```lisp
PKG> (boolean-expr-to-latex '(and (or a b (not c)) (not (or c d))))
((A \vee B \vee \neg C) \wedge \neg (C \vee D))
==> NIL
```

* `encode-time` -- Generate a time string in human readable format.  E.g.,
```lisp
PKG> (encode-time)
==> "Sat Sep 22 18:35:18 2018"
PKG> (let ((time (get-universal-time)))
       (sleep 5)
       (encode-time time))
==> "Sat Sep 22 18:35:43 2018"
```

* `*tmp-dir-root*` -- Special global variable used as the base directory for `make-temp-dir`

* `make-temp-dir` -- Return a string indicating the full path to a temporary directory.  E.g.,
```lisp
PKG> (make-temp-dir "mydir")
==> "/tmp/jimka/mydir/"
```

* `getter` -- Given a field name, return a unary function which will retrieve the value of the field from the given object.
```lisp
PKG> (mapcar (getter :x) '((:x 1 :y 2)
                           (:a 3 :x 2 :y 3)
                           (:a 3 :b 4 :x 4 :y 4)))
==> (1 2 4)
```

* `user-read` --  Calls `cl:read` with the specified arguments, but with `*PACKAGE*` bound to the CL-USER package.  
The effect of this is that symbols like `NIL` and `-` get read as `COMMON-LISP:NIL` and `COMMON-LISP:-` rather 
than as keywords.

* `def-cache-fun` -- macro -  Define three functions named by FUN-NAME and WITH-NAME a derived name.  The lambda list of the 
first function is given by LAM-LIST.  The semantics of the first function will be
to normally simply return the value of BODY.  However, if the call site to the
first function is within the dymamic extent of the second function, the
the return value will be cached, and the arguments are found in the cache
BODY is not evaluated but simply the cached value of the return value will be
returned.

* `compare-objects` -- Deterministic compare function:  returns a symbol in `(< > =)`.

* `locate-symbol` -- Return a list of symbols which is a collection of symbols from all packages which have the given symbol-name.

* `change-extension` -- change file name extension:
E.g., `(change-extension "/path/to/file.gnu" "png")` --> `"/path/to/file.png"`

* `insert-suffix` -- insert the given SUFFIX before the filename extension: 
E.g., `(insert-suffix "/path/to/file.gnu" "-smooth")` --> `"/path/to/file-smooth.gnu"`

* `chop-pathname` -- return the file name portion of a string, copping off the leading directory name
E.g.,  `(chop-pathname "/full/path/name/to/file.extension")` --> `"file.extension"`

* `empty-file-p` -- Predicate to determine whether a file is empty.  A limitation of the `CL:OPEN` function
is that this only works if the user has read permission on the file.  This is unfortunate
as it is not a UNIX limitation.


* `destructuring-let` -- macro -- Similar to `CL:LET`, but the variables also understand destructuring like with `CL:DESTRUCTURING-BIND`:  E.g.,
```lisp
(destructuring-let ((a 1)
                    ((b) '(2))
                    ((&key c d &allow-other-keys) '(:d 1 :a 2 :b 3 :c 4)))
 ...)
```

* `gnu-plot` -- Plot curves using the gnuplot UNIX program.
`:DATA` specifies a list of curve-specifiers, each curve-specifier is a plist
with fields `:title` and `:xys`.  `:title` specifies a string and `:xys` specifies
a list of xy points.  E.g.,
```lisp
    (gnu-plot \"/tmp/plot.gnu\" 
              :title \"my plot\"
              :data '((:title \"first\" :xys ((1 10) (2 20) (3 30)))
                      (:title \"second\" :xys ((1.5 15.6) (2.1 20.8) (3.5 34.3) (3.7 37.96)))))
   ==> \"/tmp/plot.png\"
```
If `:create-png-p nil` is specified, then just the .gnu file is created, and `nil`
is returned.

## License

~~~~
Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation
files (the "Software"), to deal in the Software without restriction,
including without limitation the rights to use, copy, modify, merge,
publish, distribute, sublicense, and/or sell copies of the Software,
and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
~~~~