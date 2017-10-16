# Lists and tuples

- [Common list functions](#common-list-functions)
- [Ranges](#ranges)
- [Infinite lists](#infinite-lists)
- [List comprehensions](#list-comprehensions)
- [Tuples](#tuples)

Lists in Haskell are homogeneous, so they store elements all of the same type:

```
ghci> let lostNumbers = [4,8,15,16,23,42]
ghci> lostNumbers
[4,8,15,16,23,42]
```

List concatenation is performed with the `++` operator:

```
ghci> [1,2,3,4] ++ [9,10,11,12]
[1,2,3,4,9,10,11,12]
ghci> "Hello" ++ " " ++ ['W','o','r','l','d','!']
"Hello World!"
```

Notice that strings are actually lists of characters.

Appending a list `b` to a list `a` causes Haskell to go through the whole list `a` to be able to perform the operation. Instead, adding a value to the head of the list, which is called a *cons* operation, is instantaneous, and is done with the `:` operator:

```
ghci> 'A' : " SMALL CAT"
"A SMALL CAT"
```

As the string `"Hello"` is just syntactic sugar for the list `['H','e','l','l','o']`, it turns out that `['H','e','l','l','o']` is just syntactic sugar for `'H':'e':'l':'l':'o':[]`, since this is the actual operation performed by Haskell. A new list is always created adding elements to the head of the empty list `[]`.

Indexing in Haskell is done via the `!!` operator:

```
ghci> "Steve Buscemi" !! 6
'B'
```

Multidimensional lists can contain lists of different sizes, but all of them must have elements of the same type. Comparing lists with operators `<`, `<=` and `>=` means comparing their values, and this can be done only if the elements are comparable. For example, a list is greater than another if its first element is greater than the first element of the other. If they are equal, the second elements are compared, and so on:

```
ghci> [3,2,1] > [2,1,0]
True
ghci> [3,2,1] > [2,10,100]
True
ghci> [3,4,2] > [3,4]
True
ghci> [3,4,2] > [2,4]
True
ghci> [3,4,2] == [3,4,2]
True
```


## Common list functions

- `head`: returns the head of a list

```
ghci> head [5,4,3,2,1]
5
```

- `tail`: returns the tail of a list, which means, all elements except for the head

```
ghci> tail [5,4,3,2,1]
[4,3,2,1]
```

- `last`: returns the last element of a list

```
ghci> last [5,4,3,2,1]
1
```

- `init`: returns the first elements of a list, which means, all elements except for the first

```
ghci> init [5,4,3,2,1]
[5,4,3,2]
```

By the way, applying these functions to empty lists always throws an error. Furthermore, these kinds of errors can't be caught at compile time, because there's no way the compiler can know what value a list will contain during execution.

- `length`: returns the number of elements contained in a list

```
ghci> length [5,4,3,2,1]
5
```

- `null`: tells if a list is empty

```
ghci> null [1,2,3]
False
ghci> null []
True
```

The preferred method to check if a list is empty is `null myList` rather than `myList == []`.

- `reverse`: reverses a list

```
ghci> reverse [5,4,3,2,1]
[1,2,3,4,5]
```

- `take`: returns the first `n` elements of a list

```
ghci> take 3 [5,4,3,2,1]
[5,4,3]
ghci> take 1 [3,9,3]
[3]
ghci> take 5 [1,2]
[1,2]
ghci> take 0 [1,2,3]
[]
```

- `drop`: returns the given list, after having removed the first `n` elements

```
ghci> drop 3 [8,4,2,1,5,6]
[1,5,6]
ghci> drop 0 [1,2,3,4]
[1,2,3,4]
ghci> drop 100 [1,2,3,4]
[]
```

- `maximum`, `minimum`: return the maximum and minimum among the values of the list, respectively

```
ghci> maximum [1,9,2,3,4]
9
ghci> minimum [8,4,2,1,5,6]
1
```

- `sum`, `product`: return the sum or the product of the list elements, respectively

```
ghci> sum [5,2,1,6,3,2,5,7]
31
ghci> product [6,2,1,2]
24
```

- `elem`: tells if an object is an element of a list

```
ghci> elem 4 [3,4,5,6]
True
ghci> 10 `elem` [3,4,5,6]
False
```


## Ranges

Ranges are a way to conveniently define sorted lists of enumerable objects, indicating only the first and last elements:

```
ghci> [1..5]
[1,2,3,4,5]
ghci> ['f'..'i']
"fghi"
```

It's possible to specify a step in the range, giving "examples" of what the list should look like:

```
ghci> [2,4..20]
[2,4,6,8,10,12,14,16,18,20]
ghci> [3,6..20]
[3,6,9,12,15,18]
```

This is useful to make backwards ranges, since things like `[20..10]` don't work:

```
ghci> [20,19..10]
[20,19,18,17,16,15,14,13,12,11,10]
```


## Infinite lists

Since Haskell is a lazy language, it won't evaluate an expression until it's required. This means we can define infinite lists, for instance, if they're used inside an expression where only a part of them will be used:

```
ghci> take 24 [13,26..]
```

This is a better way to define `[13,26..24*13]`. Here's some functions that produce infinite lists:

- `cycle`: takes a list and repeats it an infinite number of times

```
ghci> take 10 (cycle [1,2,3])
[1,2,3,1,2,3,1,2,3,1]
```

- `repeat`: takes an object and repeats it an infinite number of times

```
ghci> take 10 (repeat 5)
[5,5,5,5,5,5,5,5,5,5]
```

- `replicate`: repeats an object a certain number of times

```
ghci> replicate 3 10
[10,10,10]
```


## List comprehensions

A common way to represent sets of object in mathematics is *set comprehension*, with which we describe a relation among elements, like "S is the set of all `2 * x` where `x` is included in `N` and `x <= 10`". The first part, like `2 * x`, is called *output function*, and represents the relation among set objects; the second part is the *domain* of the output function. This can be written in Haskell as:

```
ghci> [ x * 2 | x <- [1..10] ]
[2,4,6,8,10,12,14,16,18,20]
```

In this example we are just using an output function and an *input set*, which is a set of values bound to the variable (or variables) of the output function. We can also add a *predicate*, that adds a condition to the input set:

```
ghci> [ x * 2 | x <- [1..10], x * 2 >= 12 ]
[12,14,16,18,20]
ghci> [ x | x <- [50..100], x `mod` 7 == 3 ]
[52,59,66,73,80,87,94]
```

The output function can be as complex as we need, and it can contain anything that can go into a function, not necessarily mathematical expressions. For instance, we could want to write a function that takes a list of numbers, and replaces each odd number lower than 10 with "BOOM!", and each odd number higher than 10 with "BANG!":

```haskell
boomBangs xs = [ if x < 10 then "BOOM!" else "BANG" | x <- xs, odd x ]
```

```
ghci> boomBangs [7..13]
["BOOM!", "BOOM!", "BANG!", "BANG!"]
```

Multiple predicates can be defined, as long as they are separated by a comma:

```
ghci> [ x | x <- [10..20], x /= 13, x /= 15, x /= 19 ]
[10, 11, 12, 14, 16, 17, 18, 20]
```

When using more than one variable in the output function, the elements of the resulting list are taken from the Cartesian product of the input sets:

```
ghci> [ x * y | x <- [2,5,10], y <- [8,10,11] ]
[16,20,22,40,50,55,80,100,110]
```

For example, `16` is calculated from the list `[2,8]`, which in turn is the first element of the Cartesian product between `[2,5,10]` and `[8,10,11]`. This also means that the size of the output list is the product of the sizes of the input sets, provided that we don't filter the output.

To filter the output we need to add some predicate:

```
ghci> [ x * y | x <- [2,5,10], y <- [8,10,11], x * y > 50 ]
[55,80,100,110]
```

The `_` character is used in place of a variable name, to indicate that we don't want to use the elements from the input set, either for filtering or in the output function, so there's no point in assigning them to a variable:

```
ghci> let length' xs = sum [1 | _ <- xs ]
ghci> length' [1,2,3,4,5]
5
```

Here we are taking all elements from the input set `xs`, but not assigning them to any variable, thanks to `_`. Then we take a `1` for each of these elements. So, basically we are creating a list of `1`'s, one for each element in the input set. Finally, we sum all these ones, getting the length of the original list.

List comprehension also works with lists of lists:

```
ghci> let xxs = [ [1,3,5,2,3,1,2,4,5], [1,2,3,4,5,6,7,8,9], [1,2,4,2,1,6,3,1,3,2,3,6] ]
ghci> [ [ x | x <- xs, even x  | xs <- xxs ]
[[2,2,4],[2,4,6,8],[2,4,2,6,2,6]]
```

Here the output function is a list comprehension itself, using `xs` as the input set, and extracting all even numbers from it. This `xs` is actually the variable to which the input set `xxs` is assigned in the outer list comprehension. So, in the outer list comprehension, `xs` represents each element of `xxs`, and the output function is another list comprehension that extracts even numbers from it. The end result is the original list, where all its contained lists have only even numbers.

Let's say we want to find all the right triangles with sides smaller than or equal to `10`, and with perimeter equal to `24`. We first generate all possible triangles with sides smaller than or equal to `10`:

```haskell
triangles = [ (a,b,c) | c <- [1..10], b <- [1..10], c <- [1..10] ]
```

Here the output function is an expression using values from the three input sets to create a triple. These values are actually the result of the Cartesian product among all input sets. Next, we want these triangles to be right:

```haskell
triangles = [ (a,b,c) | c <- [1..10], b <- [1..10], c <- [1..10], a^2 + b^2 == c^2 ]
```

Finally, the perimeter should be equal to `24`:

```haskell
triangles = [ (a,b,c) | c <- [1..10], b <- [1..10], c <- [1..10], a^2 + b^2 == c^2, a + b + c == 24 ]
```

```
ghci> triangles
[(6,8,10)]
```


## Tuples

Tuples are another way to represent sets of elements, just like lists. Key differences are that tuples can contain different types of values, but the actual type of the tuple is defined by both the type of values contained, and their number.

This means that `(1,2)` and `(5,2)` are objects of the same type, while `(1,2)` and `(1,2,3)` are tuples of different types. So a list of tuples can only contain tuples of the same number of elements of the same type.

Since tuples of different sizes are all of different types, it's impossible to write a function manipulating tuples of generic sizes. So, for example, if we wanted to write a function to append elements to tuples, we should write a function for couples, another one for triples, and so on.

Here are some functions that work with tuples:

- `fst`, `snd`: return the first and second element of a pair, respectively

```
ghci> fst (8,11)
8
ghci> snd (8,11)
11
```

These functions are defined to work only with pairs, so an error is raised when trying to use them with tuples having size other than two.

- `zip`: combines two lists into a list of pairs, containing matching elements from the lists:

```
ghci> zip [1..5]["one","two","three","four","five"]
[(1,"one"),(2,"two"),(3,"three"),(4,"four"),(5,"five")]
```

If the two lists aren't of the same length, the bigger one get shrunk to match the smaller one:

```
ghci> zip [1..] ["apple","orange","cherry","mango"]
[(1,"apple"),(2,"orange"),(3,"cherry"),(4,"mango")]
```
