# 第15章 • 标准库

## 极简主义
------

![图书馆](https://www.buildyourownlisp.com/static/img/library.png)

图书馆Library • 只用皮革、纸、木头和墨水建造

我们构建了一个最简Lisp，只加入了最少化的核心结构和内置函数。只要小心选择，可以为这个语言加入任何想要的功能。


极简主义出于两方面的动机，第一是使得语言核心易于学习并排除错误。这同样有利于用户和开发者。就像[Occam's Razor](http://en.wikipedia.org/wiki/Occam's_razor) 奥卡姆剃刀，把非必要部分当作废物剔除掉通常是一个好办法。第二个原因是，臃肿的语言丑得要死！能够造出来一个小巧却完备的核心语言绝对很有意思、很有趣、很有用，并且只有聪明人才能做到！就像历史上的黑客一样，我们乐在其中，因为我们能！

## 原子Atoms
------
处理条件分支的时候，我们并没有加入新的布尔类型。因此也没有加入`true` or `false`。取而代之的是用了数字。可读性永远很重要，因此我们可以定义一些常量来代替这些值。

类似的，在Lisp家族中很多时候使用 `nil` 来代替空表`{}`。我们当然也可以这么做。这些值保持不变并且是基础常量，一般被称作 *原子 atoms*。

使用这些命名常量不是必须的，使用数字和空表也可以。我们信奉最小干涉原理：尽量为用户保留最大可能性的自由，用户可以是任何人（包括我们自己）。

```
; Atoms
(def {nil} {})
(def {true} 1)
(def {false} 0)
```

## 构建模块 

示例中已经展示一些很酷的函数。`fun`函数便是其中之一，它允许我们以相当简洁的方式声明函数。它绝对应该入选我们的标准库。 

```lisp
; Function Definitions
(def {fun} (\ {f b} {
  def (head f) (\ (tail f) b)
}))
```

还有 `unpack`和 `pack`，对于用户来说同样必不可少，同时包含它们的 `curry`和 `uncurry`别名。 

```lisp
; Unpack List for Function
(fun {unpack f l} {
  eval (join (list f) l)
})

; Pack List for Function
(fun {pack f & xs} {f xs})

; Curried and Uncurried calling
(def {curry} unpack)
(def {uncurry} pack)
```

假设我们想按顺序做几件事，一种方法是将每件事都作为某个函数的参数。我们知道参数是按从左到右的顺序计算的，这本质上是对事件进行排序。对于 `print`和 `load`，我们不太关心它的求值结果，但关心它发生的顺序。 

可以创建一个 `do`函数，它按顺序对多个表达式求值并返回最后一个值。这个函数依赖 `last`函数，用它返回列表的最后一个元素，我们稍后再定义。 

```lisp
; Perform Several things in Sequence
(fun {do & l} {
  if (== l nil)
    {nil}
    {last l}
})
```
使用 `=` 操作符可以将结果保存于本地变量中，如果是在函数中，默认保存于本地，但有时可能需要保存在一个更小的范围。为此可以创建一个函数 `let`，它创建一个空函数环境以供代码求值。 

```
; Open new scope
(fun {let b} {
  ((\ {_} b) ())
})
```

结合`do`使用，以确保变量不会泄漏到其范围之外。 

```
lispy> let {do (= {x} 100) (x)}
100
lispy> x
Error: Unbound Symbol 'x'
lispy>
```

## 逻辑运算符Logical Operators
------
目前为止我们还没有定义任何诸如`and` and `or`这样的逻辑运算符。可能以后再加入会更好，现在我们就先使用算术运算符来模拟。思考一下，碰见逻辑值`0` 或者 `1`的话该怎么用算术运算实现逻辑运算呢？ 

```
; Logical Functions
(fun {not x}   {- 1 x})
(fun {or x y}  {+ x y})
(fun {and x y} {* x y})
```

## 杂项函数
------
有一些不太好归类的杂项函数，没什么适合的地方可以归类。猜猜它们的预期功能。

```lisp
(fun {flip f a b} {f b a})
(fun {ghost & xs} {eval xs})
(fun {comp f g x} {f (g x)})
```

`flip` 函数接受一个函数`f`和两个参数`a`，`b`，然后以相反的顺序使用这两个参数调用`f`。对于 *partially evaluated部分执行* 比较有用（译注：其它情形有个鬼用啊）：如果只提供第二个参数对函数进行部分求值，就得到一个接受接受第一个参数的新函数（译注：如果提供了第一个参数的话不需要flip）。

也就是说，如果只能提供第二个参数，那只要使用`flip`来翻转函数就行了，

```
lispy> (flip def) 1 {x}
()
lispy> x
1
lispy> def {define-one} ((flip def) 1)
()
lispy> define-one {y}
()
lispy> y
1
lispy>
```

我也想不出来`ghost`函数有什么用，不过这玩意看起来挺有趣。它接受任意数量的参数，然后就像这些参数本来的样子一样求值一遍。如同影子一样，除了看起来有点唬人以外蛋用没有。如果你发现了这玩意真有什么用途，家祭无忘告及翁。

```
lispy> ghost + 2 2
4
```

 `comp` 函数组合两个函数。它接受两个函数`f`、`g`以及一个给`g`的参数`x`，返回`f(g(x))`。像以前一样，这东西往往与部分求值结合使用，从而以简单函数构建复杂函数（译注：对比一下场论中的delta算子）。 

例如，对一个列表中的数字求累积，再求反，可以用`comp`组合成一个函数： 

```
lispy> (unpack *) {2 2}
4
lispy> - ((unpack *) {2 2})
-4
lispy> comp - (unpack *)
(\ {x} {f (g x)})
lispy> def {mul-neg} (comp - (unpack *))
()
lispy> mul-neg {2 8}
-16
lispy>
```

## 列表函数 
------
  这 `head`函数用于获取列表的第一个元素，但它返回的内容仍然包含在列表中。  如果我们想真正从这个列表中取出元素，我们需要以某种方式提取它。 

单元素列表仅计算该元素，因此我们可以使用 `eval`函数来提取。我们也可以定义几个辅助函数来帮助提取列表的第一个、第二个和第三个元素。我们稍后会更多地使用这些功能。 
```lisp
; First, Second, or Third Item in List
(fun {fst l} { eval (head l) })
(fun {snd l} { eval (head (tail l)) })
(fun {trd l} { eval (head (tail (tail l))) })
```
前面我们简要地研究了一些递归列表函数。很显然当然，递归的威力不止于此。 

例如，要求列表长度，我们可以对其进行递归：`1`加上列表尾部（译注：列表尾部指除了第一个元素的部分）的长度。要实现 `nth`我们可以对列表元素进行 `tail`操作`n`次.  要获取列表最后一个元素，我们可以用`nth`访问`长度减一`的元素。 

```lisp
; List Length
(fun {len l} {
  if (== l nil)
    {0}
    {+ 1 (len (tail l))}
})
; Nth item in List
(fun {nth n l} {
  if (== n 0)
    {fst l}
    {nth (- n 1) (tail l)}
})
; Last item in List
(fun {last l} {nth (- (len l) 1) l})
```
很多有用的函数遵循这样模式。可以定义函数来获取和删除列表的第一个元素，或者检查一个值是否存在列表中。 
```
; Take N items
(fun {take n l} {
  if (== n 0)
    {nil}
    {join (head l) (take (- n 1) (tail l))}
})
; Drop N items
(fun {drop n l} {
  if (== n 0)
    {l}
    {drop (- n 1) (tail l)}
})
; Split at N
(fun {split n l} {list (take n l) (drop n l)})
; Element of List
(fun {elem x l} {
  if (== l nil)
    {false}
    {if (== x (fst l)) {true} {elem x (tail l)}}
})
```

这些函数都遵循同样的模式。希望有某种方法可以对这样的模式进行抽象，以后就不必每次都输入了。想要对列表的每个元素执行某个函数，可以定义一个函数`map`。它接受一个函数`f`和一个列表作为输入。对于列表中每个元素，使用 `f`进行运算，将结果组合成新列表返回（译注：注意递归方式的实现：对第一个元素用`f`调用，对剩下的部分递归使用`map f`）。

```lisp
; Apply Function to List
(fun {map f l} {
  if (== l nil)
    {nil}
    {join (list (f (fst l))) (map f (tail l))}
})
```

用这样，我们可以用一种精巧的方式实现循环（译注：威力展现时刻--循环通过一个副作用就可以实现，所以新手在lisp中经常连do，for这样的loop结构都找不到）。 从某个角度讲，这个概念强于循环。 可以想象为一次性对所有元素执行了操作，而非依次操作。 我们实现的是数学上的 *映射* 而不是编程语言中的*遍历*（译注：这玩意儿越来越像个真正的Lisp了，参考CL，Clojure） 。 
```bash
lispy> map - {5 6 7 8 2 22 44}
{-5 -6 -7 -8 -2 -22 -44}
lispy> map (\ {x} {+ x 10}) {5 2 11}
{15 12 21}
lispy> print {"hello" "world"}
{"hello" "world"}
()
lispy> map print {"hello" "world"}
"hello"
"world"
{() ()}
lispy>
```

以此思想改编一个 `filter`函数，它接受某个条件，返回列表中满足此条件的元素列表。 

```lisp
; Apply Filter to List
(fun {filter f l} {
  if (== l nil)
    {nil}
    {join (if (f (fst l)) {head l} {nil}) (filter f (tail l))}
})
```
应用举例：
```bash
lispy> filter (\ {x} {> x 2}) {5 2 11 -7 8 1}
{5 11 8}
```

某些循环并不完全作用于列表元素，而是将列表压缩为单个值，例如对列表元素做累积运算、求和运算。这类循环非常类似于 `len`函数。 

这些被称为 *折叠（译注：fold，我更愿称为’坍塌‘、’降维‘）* 。其工作原理如下，接受一个函数 `f`、 一个 *基值*  `z`和一个列表 `l`，以基值为基础，将列表中的每个元素使用`f`运算。 
```lisp
; Fold Left
(fun {foldl f z l} {
  if (== l nil)
    {z}
    {foldl f (f z (fst l)) (tail l)}
})
```

使用优雅的`fold*降维*`方式来定义 `sum`和 `product`： 

```lisp
(fun {sum l} {foldl + 0 l})
(fun {product l} {foldl * 1 l})
```
## 条件函数 
------
定义 `fun`函数展示了我们的语言有多么强大，轻而易举就能用函数定义新的*语法*（译注：在C语言中只能通过宏非常有限的定义新语法）。下面给出另一个模拟 C语言`switch`- `case`语法的例子。C 语言中必须内置这个语法，但对于我们这个语言，定义为库的一部分即可。 

定义一个函数 `select`，它接受零个或多个二元素列表作为输入。对于每个二元列表，首先对第一个元素求值，若为真就对第二个元素求值并返回该值，否则就对列表的其余部分再次执行相同操作。 

```lisp
(fun {select & cs} {
  if (== cs nil)
    {error "No Selection Found"}
    {if (fst (fst cs)) {snd (fst cs)} {unpack select (tail cs)}}
})
```
再定义一个函数 `otherwise`让它永远为 `true`.  就实现了C中的 `default`。 
```lisp
; Default Case
(def {otherwise} true)

; Print Day of Month suffix
(fun {month-day-suffix i} {
  select
    {(== i 0)  "st"}
    {(== i 1)  "nd"}
    {(== i 3)  "rd"}
    {otherwise "th"}
})
```

这实际上比C的 `switch`语句更强大。在 C 中，你不能传入条件，仅能与多个常量候选者进行相等性比较。在 Lisp 如果想实现同样的功能则可以这样定义函数，接受一个值 `x`和零个或多个二元素列表。如果二元素列表中的第一个元素等于 `x`，就对第二个元素求值并返回该值，否则该对其余元素继续。 

```lisp
(fun {case x & cs} {
  if (== cs nil)
    {error "No Case Found"}
    {if (== x (fst (fst cs))) {snd (fst cs)} {
      unpack case (join (list x) (tail cs))}}
})
```

这样的语法真又简单又好，试试看你还有什么类似的想法。
```lisp
(fun {day-name x} {
  case x
    {0 "Monday"}
    {1 "Tuesday"}
    {2 "Wednesday"}
    {3 "Thursday"}
    {4 "Friday"}
    {5 "Saturday"}
    {6 "Sunday"}
})
```

## 斐波那契数列
------
就像编程语言该以“Hello,World!"为入门第一课一样，如果一个标准库未包含 Fibonacci 函数的定义显然很丢脸。让我们写一个可爱又简洁、可读且清晰的 `fib`函数。 
```
; Fibonacci
(fun {fib n} {
  select
    { (== n 0) 0 }
    { (== n 1) 1 }
    { otherwise (+ (fib (- n 1)) (fib (- n 2))) }
})
```

我写的标准库就是这样。设计标准库是语言设计中有趣的部分，你得发挥自己的创造力和决断力，懂得取舍。自己玩玩，探索新世界其乐无穷。

## 参考程序
prelude.lspy 
```lisp
;;;
;;;   Lispy Standard Prelude
;;;

;;; Atoms
(def {nil} {})
(def {true} 1)
(def {false} 0)

;;; Functional Functions

; Function Definitions
(def {fun} (\ {f b} {
  def (head f) (\ (tail f) b)
}))

; Open new scope
(fun {let b} {
  ((\ {_} b) ())
})

; Unpack List to Function
(fun {unpack f l} {
  eval (join (list f) l)
})

; Unapply List to Function
(fun {pack f & xs} {f xs})

; Curried and Uncurried calling
(def {curry} unpack)
(def {uncurry} pack)

; Perform Several things in Sequence
(fun {do & l} {
  if (== l nil)
    {nil}
    {last l}
})

;;; Logical Functions

; Logical Functions
(fun {not x}   {- 1 x})
(fun {or x y}  {+ x y})
(fun {and x y} {* x y})


;;; Numeric Functions

; Minimum of Arguments
(fun {min & xs} {
  if (== (tail xs) nil) {fst xs}
    {do 
      (= {rest} (unpack min (tail xs)))
      (= {item} (fst xs))
      (if (< item rest) {item} {rest})
    }
})

; Maximum of Arguments
(fun {max & xs} {
  if (== (tail xs) nil) {fst xs}
    {do 
      (= {rest} (unpack max (tail xs)))
      (= {item} (fst xs))
      (if (> item rest) {item} {rest})
    }  
})

;;; Conditional Functions

(fun {select & cs} {
  if (== cs nil)
    {error "No Selection Found"}
    {if (fst (fst cs)) {snd (fst cs)} {unpack select (tail cs)}}
})

(fun {case x & cs} {
  if (== cs nil)
    {error "No Case Found"}
    {if (== x (fst (fst cs))) {snd (fst cs)} {
          unpack case (join (list x) (tail cs))}}
})

(def {otherwise} true)


;;; Misc Functions

(fun {flip f a b} {f b a})
(fun {ghost & xs} {eval xs})
(fun {comp f g x} {f (g x)})

;;; List Functions

; First, Second, or Third Item in List
(fun {fst l} { eval (head l) })
(fun {snd l} { eval (head (tail l)) })
(fun {trd l} { eval (head (tail (tail l))) })

; List Length
(fun {len l} {
  if (== l nil)
    {0}
    {+ 1 (len (tail l))}
})

; Nth item in List
(fun {nth n l} {
  if (== n 0)
    {fst l}
    {nth (- n 1) (tail l)}
})

; Last item in List
(fun {last l} {nth (- (len l) 1) l})

; Apply Function to List
(fun {map f l} {
  if (== l nil)
    {nil}
    {join (list (f (fst l))) (map f (tail l))}
})

; Apply Filter to List
(fun {filter f l} {
  if (== l nil)
    {nil}
    {join (if (f (fst l)) {head l} {nil}) (filter f (tail l))}
})

; Return all of list but last element
(fun {init l} {
  if (== (tail l) nil)
    {nil}
    {join (head l) (init (tail l))}
})

; Reverse List
(fun {reverse l} {
  if (== l nil)
    {nil}
    {join (reverse (tail l)) (head l)}
})

; Fold Left
(fun {foldl f z l} {
  if (== l nil) 
    {z}
    {foldl f (f z (fst l)) (tail l)}
})

; Fold Right
(fun {foldr f z l} {
  if (== l nil) 
    {z}
    {f (fst l) (foldr f z (tail l))}
})

(fun {sum l} {foldl + 0 l})
(fun {product l} {foldl * 1 l})

; Take N items
(fun {take n l} {
  if (== n 0)
    {nil}
    {join (head l) (take (- n 1) (tail l))}
})

; Drop N items
(fun {drop n l} {
  if (== n 0)
    {l}
    {drop (- n 1) (tail l)}
})

; Split at N
(fun {split n l} {list (take n l) (drop n l)})

; Take While
(fun {take-while f l} {
  if (not (unpack f (head l)))
    {nil}
    {join (head l) (take-while f (tail l))}
})

; Drop While
(fun {drop-while f l} {
  if (not (unpack f (head l)))
    {l}
    {drop-while f (tail l)}
})

; Element of List
(fun {elem x l} {
  if (== l nil)
    {false}
    {if (== x (fst l)) {true} {elem x (tail l)}}
})

; Find element in list of pairs
(fun {lookup x l} {
  if (== l nil)
    {error "No Element Found"}
    {do
      (= {key} (fst (fst l)))
      (= {val} (snd (fst l)))
      (if (== key x) {val} {lookup x (tail l)})
    }
})

; Zip two lists together into a list of pairs
(fun {zip x y} {
  if (or (== x nil) (== y nil))
    {nil}
    {join (list (join (head x) (head y))) (zip (tail x) (tail y))}
})

; Unzip a list of pairs into two lists
(fun {unzip l} {
  if (== l nil)
    {{nil nil}}
    {do
      (= {x} (fst l))
      (= {xs} (unzip (tail l)))
      (list (join (head x) (fst xs)) (join (tail x) (snd xs)))
    }
})

;;; Other Fun

; Fibonacci
(fun {fib n} {
  select
    { (== n 0) 0 }
    { (== n 1) 1 }
    { otherwise (+ (fib (- n 1)) (fib (- n 2))) }
})
```
## 课后作业
› 使用`foldl`重写`len`函数。
› 使用`foldl`重写`elem`函数。
› 将标准库直接融入语言中。并在启动时加载。
› 为您的标准库编写文档，解释每个函数的作用。
› 使用标准库编写一些示例程序，供用户学习。
› 为标准库中的每个功能编写一些测试案例。


