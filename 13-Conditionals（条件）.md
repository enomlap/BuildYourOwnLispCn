# 第13章 • 条件

## 自己动手

------

目前我们已经做得好了。你的C语言基础肯定相当不错，不然你也不可能看到这里（笑）。如果你有信心，本章可是你展示本领的好机会了。这一章不长，仅仅加入了一些新的内置函数用于处理比较.   

![小狗](https://www.buildyourownlisp.com/static/img/pug.png)

小狗Pug • 只有它睡着了的时候它才可爱

如果感受到了鼓舞，现在来试试在你自己这个语言中加入`比较`吧。仿照 C 语言里的逻辑操作，来为 *greater than（大于）*, *less than（小于）*, *equal to（相等）* 之类的比较运算定义一些新的内置函数。首先，先试着定义一个 `if` 函数，和 C 语言中一样，它对某个条件进行测试，然后依据测试的结果执行相应的代码。如果你写出来了，那就来和我给的代码对比一下。看看有什么区别，你更喜欢哪一个呢？
如果你不太有信心或者根本犯了懒病的话也无妨，接下来我将会逐步解释我的代码。

## 比较（Ordering）

------
为简单起见，我打算重用数字类型作为比较的结果。仿照 C 语言，用非 `0` 值表示 `真` ，而 `0` 用来表示 `假` 。这样我们的比较函数就简化成为了数学函数，它能且仅能处理两个参数。

如果遇见错误，那么也很简单，由于我们让它通过对比两个值来返回一个 `lval` 类型的 `0` 或者 `1` 的结果，因此我们可以使用 C 语言中的比较运算符，此外，我们打算使用单一的函数来完成所有的比较工作。

首先检查错误情况，然后对参数进行比较，并返回一个数字作为结果，程序如下：
```
lval* builtin_gt(lenv* e, lval* a) {
  return builtin_ord(e, a, ">");
}
lval* builtin_lt(lenv* e, lval* a) {
  return builtin_ord(e, a, "<");
}
lval* builtin_ge(lenv* e, lval* a) {
  return builtin_ord(e, a, ">=");
}
lval* builtin_le(lenv* e, lval* a) {
  return builtin_ord(e, a, "<=");
}
lval* builtin_ord(lenv* e, lval* a, char* op) {
  LASSERT_NUM(op, a, 2);
  LASSERT_TYPE(op, a, 0, LVAL_NUM);
  LASSERT_TYPE(op, a, 1, LVAL_NUM);

  int r;
  if (strcmp(op, ">")  == 0) {
    r = (a->cell[0]->num >  a->cell[1]->num);
  }
  if (strcmp(op, "<")  == 0) {
    r = (a->cell[0]->num <  a->cell[1]->num);
  }
  if (strcmp(op, ">=") == 0) {
    r = (a->cell[0]->num >= a->cell[1]->num);
  }
  if (strcmp(op, "<=") == 0) {
    r = (a->cell[0]->num <= a->cell[1]->num);
  }
  lval_del(a);
  return lval_num(r);
}
```

## 相等
------
比较相等略为复杂，因为我们打算让它不仅仅能够用于对比数字之间的相等，也能对 `空表` 、 `函数` 进行判断。可以定义一个函数用来对两个 `lval` 类型进行测试。

实质上，这个函数将对`lval`的所有域逐一进行测试，只有全部测试结果都为真才返回真，否则返回假。
```
int lval_eq(lval* x, lval* y) {
  /* Different Types are always unequal 不同的类型则直接判为不等*/
  if (x->type != y->type) { return 0; }

  /* Compare Based upon type 对于相同类型*/
  switch (x->type) {
    /* Compare Number Value 数值型*/
    case LVAL_NUM: return (x->num == y->num);

    /* Compare String Values 字符串*/
    case LVAL_ERR: return (strcmp(x->err, y->err) == 0);
    case LVAL_SYM: return (strcmp(x->sym, y->sym) == 0);

    /* If builtin compare, otherwise compare formals and body 如果是builtin则直接对比，否则分别对比形参和函数体*/
    case LVAL_FUN:
      if (x->builtin || y->builtin) {
        return x->builtin == y->builtin;
      } else {
        return lval_eq(x->formals, y->formals)
          && lval_eq(x->body, y->body);
      }

    /* If list compare every individual element 如果是列表，则分别对比每一个元素*/
    case LVAL_QEXPR:
    case LVAL_SEXPR:
      if (x->count != y->count) { return 0; }
      for (int i = 0; i < x->count; i++) {
        /* If any element not equal then whole list not equal 只要有一个元素不等，整个列表则不等*/
        if (!lval_eq(x->cell[i], y->cell[i])) { return 0; }
      }
      /* Otherwise lists must be equal */
      return 1;
    break;
  }
  return 0;
}
```
这样再定义比较相等的内置函数就简单了。给定两个参数，再测试相等性，比较的结果以 `lval` 类型返回。 
```
lval* builtin_cmp(lenv* e, lval* a, char* op) {
  LASSERT_NUM(op, a, 2);
  int r;
  if (strcmp(op, "==") == 0) {
    r =  lval_eq(a->cell[0], a->cell[1]);
  }
  if (strcmp(op, "!=") == 0) {
    r = !lval_eq(a->cell[0], a->cell[1]);
  }
  lval_del(a);
  return lval_num(r);
}
lval* builtin_eq(lenv* e, lval* a) {
  return builtin_cmp(e, a, "==");
}
lval* builtin_ne(lenv* e, lval* a) {
  return builtin_cmp(e, a, "!=");
}
```
## `if`函数 
------
我们的 `if`函数有点像 C 中的三元运算。在条件为真时，它对一段代码求值，若为假，则执行另一部分代码。 
可以继续使用 `Q-Expressions` 代表执行的代码。函数接受两个 Q 表达式，在条件为真或假时分别执行对应的。 
```
lval* builtin_if(lenv* e, lval* a) {
  LASSERT_NUM("if", a, 3);
  LASSERT_TYPE("if", a, 0, LVAL_NUM);
  LASSERT_TYPE("if", a, 1, LVAL_QEXPR);
  LASSERT_TYPE("if", a, 2, LVAL_QEXPR);

  /* Mark Both Expressions as evaluable */
  lval* x;
  a->cell[1]->type = LVAL_SEXPR;
  a->cell[2]->type = LVAL_SEXPR;

  if (a->cell[0]->num) {
    /* If condition is true evaluate first expression 为真则对第一个表达式求值*/
    x = lval_eval(e, lval_pop(a, 1));
  } else {
    /* Otherwise evaluate second expression 否则对第二个进行求值*/
    x = lval_eval(e, lval_pop(a, 2));
  }

  /* Delete argument list and return */
  lval_del(a);
  return x;
}
```
注册一下这个新函数： 
```
/* Comparison Functions */
lenv_add_builtin(e, "if", builtin_if);
lenv_add_builtin(e, "==", builtin_eq);
lenv_add_builtin(e, "!=", builtin_ne);
lenv_add_builtin(e, ">",  builtin_gt);
lenv_add_builtin(e, "<",  builtin_lt);
lenv_add_builtin(e, ">=", builtin_ge);
lenv_add_builtin(e, "<=", builtin_le);
```
下面是测试：
```
lispy> > 10 5
1
lispy> <= 88 5
0
lispy> == 5 6
0
lispy> == 5 {}
0
lispy> == 1 1
1
lispy> != {} 56
1
lispy> == {1 2 3 {5 6}} {1   2  3   {5 6}}
1
lispy> def {x y} 100 200
()
lispy> if (== x y) {+ x y} {- x y}
-100
```

## 递归函数

------

引入分支无疑使得我们这个新语言强大了许多，因为这个功能的加入，使得函数递归也变得有可能了。递归函数指的是函数会调用它们自己. 在实现读取和表达式求值的时候我们大量使用过 C 语言的这个功能。必须使用条件比较的原因是，递归必须能够终结。
举个例子，可以使用条件测试来实现一个 `len` 函数，用于返回例表中元素的数量。对于空表则返回 `0` ，否则返回除*第一个元素以外*的剩下的部分的 `长度` ，然后再加上 `1`（注意，这里再次使用了`len`函数）。思考一下它的原理，它会不断调用len函数，直到遇到空表，这时len将返回0，并返回所有其它元素的个数之和。

------

```
(fun {len l} {
  if (== l {})
    {0}
    {+ 1 (len (tail l))}
})
```

如同 C 语言一样，递归函数蕴含了一种有趣的对称在其中。首先我们对空表（基础情形）进行处理，如果发现了除空表以外的元素，就去掉头元素，然后对剩下的进行处理。最终就处理了所有元素。

------

下面是另一个反转列表的函数。与前例相似，先检查空列表，但这次它返回空列表。这是有道理的。空列表的倒序仍然是空列表。如果列表不是空列表，它就把除了第一个元素之外的部分反转，并粘在第一个元素之 *前* 。

```
(fun {reverse l} {
  if (== l {})
    {{}}
    {join (reverse (tail l)) (head l)}
})
```

可以使用此项技术构建很多函数，当前语言中的循环大都用此法实现。

## 参考Reference

------

####         [           conditionals.c         ](https://www.buildyourownlisp.com/chapter13_conditionals#collapseOne)      

## 彩蛋（课后作业）

------

- › 仿照给出的程序实现 *or* `||`, *and* `&&` 和 *not* `!`。
- › 定义一个 Lisp 语言递归函数实现 `nth` 。
- › 定义一个递归函数，检查元素是否存在于列表中，存在则返回 `1`，否则返回 `0`。
- › 定义一个 Lisp 函数返回列表的最后一个元素。
- › 定义一个 Lisp 语言逻辑操作符函数，实现`or`, `and` 和 `not`的功能。
- › 在这个语言中添加一个特殊的布尔类型，其值非 `true` 即 `false`。

## 导航Navigation

| [‹ Functions](https://www.buildyourownlisp.com/chapter12_functions) | [• Contents •](https://www.buildyourownlisp.com/contents) | [Strings ›](https://www.buildyourownlisp.com/chapter14_strings) |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
|                                                              |                                                           |                                                              |
