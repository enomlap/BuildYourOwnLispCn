# 第十一章 •变量
## 不可变性
------
![turtle](http://buildyourownlisp.com/static/img/turtle.png)
十几岁的忍者龟•总是在变。
在前些章节里，我们的这个新语言里已经取得了不错的进展。
其中一些特性真的很酷，少有编程语言可以轻易做到， 例如说把代码放在列表中（译注：至少强大如C也极难把代码放进数据中动态去执行！）。现在我们准备加上一些让这个语言变得更加实用的*特性*，首先我们要加的就是*变量*。

虽然称作“变量”，但这其实是一个有误导性的词，因为我们的变量其实并不会变。我们的变量是*不可变*的，意即是它们不能被改变。目前为止，这个语言中的每一个部分都是*不可变*的。当我们对一个表达式求值的时候，我们应该想像成这样：之前的一切被销毁了，只是返回了新的值而已。 虽然在实现的时候，复用已有的往往比新建更容易，但是概念上来讲，这样理解我们的这个语言比较合适。
实际上，我们的变量可以简单地理解为一个代表了某个*值的名字*而已，它允许我们为这个*名字*赋一个*值*，并且允许需要的时候得到这个*值*的［拷贝］（译注：务必注意这时得到的仅仅是个［拷贝］而已，原值仍然是［不可变］的）。

为了能够给值*命名*，我们首先来创建一个结构（structure），用来保存有关变量名和值的各种信息。 这些信息我们称之为*环境(environment)*。当开始一个新的提示符的时候，我们应该创建一个新的与之对应的*环境*，所有的输入将在这个环境里被求值。这样我们就可以在程序里保存、调用这些变量了。

**等等，当我们对一个名字重新赋值的时候，发生了什么？这不刚好是变了吗？**

好吧，在我们这个Lisp里面，当给一个变量名赋新值的时候其实是删除了旧的绑定，建立了一个新的绑定而已。认为变量改变了其实是一个错觉，实际上旧的东西已经被销毁了。这点和C语言很不一样，在C语言中我们真的可以改变一个指针指向的值（译者按：C语言中有严重的*内存条*的概念，Lisp中完全没有，就像数学家对于*空间*的理解是无限的，根本不关心具体大小，但是工程师眼里任何一个房间里的*空间*总有个长宽高体积），或者在struct里面保存新的值，并没有销毁和创建的过程。

## 词法•Symbol Syntax
------
现在我们想要允许用户定义变量（译者按：原著这里并非`symbol`而是`variables`，应该不是原著者笔误，而是因为在Lisp里面变量只是绑定到某个值上面，而这个值可以是任何东西包括各种运算符、函数以及其它），这就需要更新原来的语法解析器了。 除了能够匹配内建的函数，解析器还需要能够匹配任何合法的标识。不像C语言， 我们准备让这个语言对变量名只有最低限度的限制。

下面是我们新创建的用于代表可用字符的正则表达式：
```
/[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/
```
乍一样这有点像某人不小心把手肘压到了键盘上打出来的东西，不过实际上，这是一个合法的正则表达式，Linux用户可能会很熟悉这个东西。主要的部分放在了一个中括号`[]`里面，全部正则表达式用一对反斜杠‘/’包含起来。正则表达里面的字符多数已经不是本意，不过另有一些特别的字符额外需要使用斜杠‘\’转义。此外，由于我们当前所用的是C语言环境，所以这一串字符中的斜杠‘\’还需要用斜杠本身来转义斜杠！！（译注：别急着抓逛，这事儿在使用正则表达式的时候常有）

这个规则除了可以匹配C语言里面允许的`a-zA-Z0-9_`字符之外，另外还可以匹配数学运算符号加减乘除‘+’、‘-’、‘*’、‘/’，还有斜杠‘\’，当然这需要用四个斜杠`\\\\`来表示（第一个、第三个`\`是C语言里面的，用来转义其后的`\`，这样总共给正则表达式提供了两个`\`，正则表达式中，第一个`\`用来转意第二个`\`，这样算来总共其实就简单的匹配一个`\`），这样就能匹配`a-zA-Z0-9_` 和计算符号`+\\-*\\/` 以及`\\\\`，比较符`=<>!`或者`and`符号`&`。这样在定义的时候更加灵活（译注：一百万条腿的LISP大象真不是盖的）。

```c
mpca_lang(MPCA_LANG_DEFAULT,
  "                                                     \
    number : /-?[0-9]+/ ;                               \
    symbol : /[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/ ;         \
    sexpr  : '(' <expr>* ')' ;                          \
    qexpr  : '{' <expr>* '}' ;                          \
    expr   : <number> | <symbol> | <sexpr> | <qexpr> ;  \
    lispy  : /^/ <expr>* /$/ ;                          \
  ",
  Number, Symbol, Sexpr, Qexpr, Expr, Lispy);
```

## 函数指针

------

一旦我们引入了变量，在这个语言中标识就不能再代表函数了，而是代表了环境中的某个名字，可以通过这个名字去查找、返回名字代表的值。

于是我们就需要用一个新的值用来代表函数，每次返回一个内建标志。为创建这个新的`lval`类型这次我们将用到C语言中的`函数指针（function pointer）`。

函数指针是C语言中的一个特性，其实就是一个指向函数的指针（按：从内存角度来讲就是某段程序的入口值），可以用来指向不同的函数。改变这个值本身没有什么意义，但我们可以用它来访问需要的函数。

如同普通指针一样， 函数指针也有其类型，这个类型就是它所指向的函数的类型。如同函数需要有类型一样，编译器也需要用到这个类型信息，这样才能正确的调用所指向的函数（译注：如果学过汇编就知道调用的时候要提前填好寄存器EX，DX等待位置，一样的），比如用到几个参数以及参数类型、返回值的类型。

在之前的章节里面我们的内建函数使用一个`lval`类型参数作为输入并且返回一个`lval`类型值。在这一章里我们的函数将额外需要一个指向环境的`lenv`型的指针作为输入。我们可以声明一个新的函数指针，其类型定为`lbuiltin`：

```c
typedef lval*(*lbuiltin)(lenv*, lval*);
```

**等等，怎么这个语法这么奇怪？**
在某些地方，C语言的一些语法看起来特别古怪。如果我们知道为什么要这么做那就好了。让我们来分解一下上面这个语法。
（译注：此处译略，学过C语言的都会了，无非是些关于`typedef`的东西，以及指针，参见任何一本C语言教科书即可）First the `typedef`. This can be put before any standard variable declaration. It results in the name of the variable, being declared a new type, matching what would be the inferred type of that variable. This is why in the above declaration what looks like the function name becomes the new type name.

Next all those `*`. Pointer types in C are actually meant to be written with the star `*` on the left hand side of the variable name, not the right hand side of the type `int *x;`. This is because C type syntax works by a kind of inference. Instead of reading *"Create a new `int` pointer `x`"*. It is meant to read *"Create a new variable `x` where to dereference `x` results in an `int`."* Therefore `x` is inferred to be a pointer to an `int`.
This idea is extended to function pointers. We can read the above declaration as follows. *"To get an `lval\*` we dereference `lbuiltin` and call it with a `lenv\*` and a `lval\*`."* Therefore `lbuiltin` must be a function pointer that takes an `lenv*` and a `lval*` and returns a `lval*`.

## 关于类型的循环引用

------
（译注：这一部分也不译了，同上，参见普通的C语言教科书即可，与本文的LISP没甚关系。另注：这部分涉及到编译原理的部分内容，明白编译器会多次扫描即可，就像你考试的时候碰到某题不会做结果试卷后面有提示，你又回到前面把不会的那部分填上。对于编译器，它只需要知道这是一个指针即可，具体是什么类型，反正都可以，都是指向某内存地址的值而已）。
The `lbuiltin` type references the `lval` type and the `lenv` type. This means that they should be declared first in the source file.
But we want to make a `lbuiltin` field in our `lval` struct so we can create function values. So therefore our `lbuiltin` declaration must go before our `lval` declaration. This leads to what is called a cyclic type dependency, where two types depend on each other.
We've come across this problem before with functions which depend on each other. The solution was to create a *forward declaration* which declared a function but left the body of it empty.
In C we can do exactly the same with types. First we declare two `struct` types without a body. Secondly we typedef these to the names `lval` and `lenv`. Then we can define our `lbuiltin` function pointer type. And finally we can define the body of our `lval` struct. Now all our type issues are resolved and the compiler won't complain any more.
```c
/* Forward Declarations 前置定义*/
struct lval;
struct lenv;
typedef struct lval lval;/*没有定义体的定义*/
typedef struct lenv lenv;
/* Lisp Value */
enum { LVAL_ERR, LVAL_NUM,   LVAL_SYM,
       LVAL_FUN, LVAL_SEXPR, LVAL_QEXPR };
typedef lval*(*lbuiltin)(lenv*, lval*);/*理解提示：定义一个[lbuiltin]类型,这个类型指向一个函数，其函数类型为 lval，参数列表是(lenv*, lval*)*/
struct lval {/*lval的实际定义*/
  int type;
  long num;
  char* err;
  char* sym;
  lbuiltin fun;
  int count;
  lval** cell;
};
```

## 函数类型

------
我们新定义的`lval`类型在内部加入了一个枚举类型`LVAL_FUN`。其它相应的部分也要更新以便能够处理这个新的类型。多数情况下仅需要在switch部分加入相应的case判断机制就行了。现在可以为这个新类型制作一个构建函数了：
```c
lval* lval_fun(lbuiltin func) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_FUN;
  v->fun = func;
  return v;
}
```
在**deleting**中无须对函数指针做任何特别的处理。

```c
case LVAL_FUN: break;
```

在**printing**中我们仅打印一串普通的字符就行。

```c
case LVAL_FUN:   printf("<function>"); break;
```

此外我们再把对`lval`的**拷贝**功能加上，以便用它来把东西从环境中放入、取出。对于数字和函数，我们只需要直接拷贝相应域即可。对于字符串，需要用到`malloc`和`strcpy`以免内存泄漏。拷贝列表时需要单独为每个成员申请合适大小的空间。

```c
lval* lval_copy(lval* v) {
  lval* x = malloc(sizeof(lval));
  x->type = v->type;
  switch (v->type) {
    /* Copy Functions and Numbers Directly */
    case LVAL_FUN: x->fun = v->fun; break;
    case LVAL_NUM: x->num = v->num; break;
    /* Copy Strings using malloc and strcpy */
    case LVAL_ERR:
      x->err = malloc(strlen(v->err) + 1);
      strcpy(x->err, v->err); break;
    case LVAL_SYM:
      x->sym = malloc(strlen(v->sym) + 1);
      strcpy(x->sym, v->sym); break;
    /* Copy Lists by copying each sub-expression */
    case LVAL_SEXPR:
    case LVAL_QEXPR:
      x->count = v->count;
      x->cell = malloc(sizeof(lval*) * x->count);
      for (int i = 0; i < x->count; i++) {
        x->cell[i] = lval_copy(v->cell[i]);
      }
    break;
  }
  return x;
}
```

## 环境Environment

------

我们的环境结构必须维持*命名names*与*值values*之间对应关系的一个列表。有很多办法可以构建这样一个结构。我们选个最简单的办法，就是使用两个具有相同长度的列表。其中一个列表是 `lval*`类型，而另一个是`char*`。两个列表中相同位置的元素一一对应。

我们已经前置声明了`lenv`结构，只需要再加上下面的部分就可以了。

```c
struct lenv {
  int count;
  char** syms;
  lval** vals;
};
```

我们还需要几个函数用来生成和销毁这个结构才行。这很简单，创建的时候初始化相应的域，而销毁的时候需要把所有的项目释放（译注：千万注意，因为C语言是没有垃圾回收机制的，处理不好很容易内存泄漏）。

```c
lenv* lenv_new(void) {
  lenv* e = malloc(sizeof(lenv));
  e->count = 0;
  e->syms = NULL;
  e->vals = NULL;
  return e;
}
void lenv_del(lenv* e) {
  for (int i = 0; i < e->count; i++) {
    free(e->syms[i]);
    lval_del(e->vals[i]);
  }
  free(e->syms);
  free(e->vals);
  free(e);
}
```

然后我们就可以写两个函数分别用来存、取。

为得到环境中的变量值，我们可以循环查找所有的元素，看看它的标识是否与已经当前的标识相同。如果相同就返回保存的值，如果找不到匹配的就返回一个错误。

往环境中存入新值的函数要复杂一点，首先要检查一下有没有相同标识的变量已经存在。如果是，就拿新值代替，通过遍历所有已存在的变量的变量名就可以了。如果存在，那就把它的值删掉，然后存入新的输入值。
如果找不到重名的，就申请一块内存来放它，在此情况下可以用`realloc`函数重新申请内存，然后把`lval`类型的值以及名字复制进去。

```c
lval* lenv_get(lenv* e, lval* k) {
  /* Iterate over all items in environment */
  for (int i = 0; i < e->count; i++) {
    /* Check if the stored string matches the symbol string */
    /* If it does, return a copy of the value */
    if (strcmp(e->syms[i], k->sym) == 0) {
      return lval_copy(e->vals[i]);
    }
  }
  /* If no symbol found return error */
  return lval_err("unbound symbol!");
}
void lenv_put(lenv* e, lval* k, lval* v) {
  /* Iterate over all items in environment */
  /* This is to see if variable already exists */
  for (int i = 0; i < e->count; i++) {
    /* If variable is found delete item at that position */
    /* And replace with variable supplied by user */
    if (strcmp(e->syms[i], k->sym) == 0) {
      lval_del(e->vals[i]);
      e->vals[i] = lval_copy(v);
      return;
    }
  }
  /* If no existing entry found allocate space for new entry */
  e->count++;
  e->vals = realloc(e->vals, sizeof(lval*) * e->count);
  e->syms = realloc(e->syms, sizeof(char*) * e->count);
  /* Copy contents of lval and symbol string into new location */
  e->vals[e->count-1] = lval_copy(v);
  e->syms[e->count-1] = malloc(strlen(k->sym)+1);
  strcpy(e->syms[e->count-1], k->sym);
}
```

## 变量求值

------

现在，求值函数变复杂了，因为求值的时候是环境相关的。所以还必须把指向环境的指针一起传递给函数。记住，我们的环境只是返回一个值的拷贝，所以别忘了还需要额外删除输入的`lval`型标记。

```c
lval* lval_eval(lenv* e, lval* v) {
  if (v->type == LVAL_SYM) {
    lval* x = lenv_get(e, v);
    lval_del(v);
    return x;
  }
  if (v->type == LVAL_SEXPR) { return lval_eval_sexpr(e, v); }
  return v;
}
```

由于我们增加了函数类型，所以S表达式的求值部分也需要做相应的修改。事先要确保这是函数型而不是标志类型。如果这个条件满足的话就可以通过这个`lval`的`fun` 成员，像调用普通函数一样调用了。

```c
lval* lval_eval_sexpr(lenv* e, lval* v) {
  for (int i = 0; i < v->count; i++) {
    v->cell[i] = lval_eval(e, v->cell[i]);
  }
  for (int i = 0; i < v->count; i++) {
    if (v->cell[i]->type == LVAL_ERR) { return lval_take(v, i); }
  }
  if (v->count == 0) { return v; }
  if (v->count == 1) { return lval_take(v, 0); }
  /* Ensure first element is a function after evaluation */
  lval* f = lval_pop(v, 0);
  if (f->type != LVAL_FUN) {
    lval_del(v); lval_del(f);
    return lval_err("first element is not a function");
  }
  /* If so call function to get result */
  lval* result = f->fun(e, v);
  lval_del(f);
  return result;
}
```

## 内置函数Builtins

------
现在，求值依赖于新的函数类型，因此要确保在交互之前注册所有的内置函数。目前我们的内置函数还没有正确的类型。我们需要改变它们的类型签名，例如说明它们需要环境，以及传递环境的规则 。这里就不给出代码了，大家试试自己改变内置函数的类型签名，增加`lenv*`部分为第一个参数。如果你还有点困惑的话可以参考一下本章给出的示例代码。

作为一个例子，我们可以使用`builtin_in`函数分别定义几个数学函数。

```c
lval* builtin_add(lenv* e, lval* a) {
  return builtin_op(e, a, "+");
}
lval* builtin_sub(lenv* e, lval* a) {
  return builtin_op(e, a, "-");
}
lval* builtin_mul(lenv* e, lval* a) {
  return builtin_op(e, a, "*");
}
lval* builtin_div(lenv* e, lval* a) {
  return builtin_op(e, a, "/");
}
```

一旦我们把内置函数类型改好了我们就可以创建一个函数，用来把我们所有的内置函数注册到环境中。对于每一个内置函数我们想要根据给定的名字分别创建一个`lval`函数和一个 `lval`标记，然后使用`lenv_put`注册到环境中。环境总是接收、返回*值的拷贝*，所以别忘了在注册完成后不再需要它们的时候额外删除这两个`lval`。

如果我们把这个任务分解成两个功能我们就可以简洁地注册环境的所有内置函数了。
```c
void lenv_add_builtin(lenv* e, char* name, lbuiltin func) {
  lval* k = lval_sym(name);
  lval* v = lval_fun(func);
  lenv_put(e, k, v);
  lval_del(k); lval_del(v);
}
void lenv_add_builtins(lenv* e) {
  /* List Functions */
  lenv_add_builtin(e, "list", builtin_list);
  lenv_add_builtin(e, "head", builtin_head);
  lenv_add_builtin(e, "tail", builtin_tail);
  lenv_add_builtin(e, "eval", builtin_eval);
  lenv_add_builtin(e, "join", builtin_join);
  /* Mathematical Functions */
  lenv_add_builtin(e, "+", builtin_add);
  lenv_add_builtin(e, "-", builtin_sub);
  lenv_add_builtin(e, "*", builtin_mul);
  lenv_add_builtin(e, "/", builtin_div);
}
```

最后一步是在创建交互之前调用一下这个函数。再提示一下，不要忘记使用完毕时销毁环境。

```c
lenv* e = lenv_new();
lenv_add_builtins(e);
while (1) {
  char* input = readline("lispy> ");
  add_history(input);
  mpc_result_t r;
  if (mpc_parse("<stdin>", input, Lispy, &r)) {
    lval* x = lval_eval(e, lval_read(r.output));
    lval_println(x);
    lval_del(x);
    mpc_ast_delete(r.output);
  } else {
    mpc_err_print(r.error);
    mpc_err_delete(r.error);
  }
  free(input);
}
lenv_del(e);
```

如果一切顺利，现在交互环境已经可以使用了，可以发现函数已经是新类型值了，而不再是标记（Symbols）。

```
lispy> +
<function>
lispy> eval (head {5 10 11 15})
5
lispy> eval (head {+ - + - * /})
<function>
lispy> (eval (head {+ - + - * /})) 10 20
30
lispy> hello
Error: unbound symbol!
lispy>
```

## 定义函数

------

我们已经能够将内置函数像变量一样注册了，不过用户还没有办法定义自己的变量。
这有点尴尬，因为我们需要让用户能够传入标记的名字，和用户给这个名字赋的值。不过标记不可以代表他们自己，否则求值函数将会尝试在环境只取得其值。

传入变量标记并避免求值的唯一办法是把它们放在引用`{}`里面（Q表达式），。用这个技术来定义函数，可以分别取得标记列表和值的列表， 然后把他们对应赋值。

这个函数行为应与其它的内置函数相似T。首先需要检查错误成立条件，然后执行一些命令，再返回。这种情况下它需要先检查输入参数是否是正确的类型。然后遍历每一个标记和值，放入环境。如果出现错误，那么就返回错误，如果一切正常的话就返回一个空表达式`()`。

```c
lval* builtin_def(lenv* e, lval* a) {
  LASSERT(a, a->cell[0]->type == LVAL_QEXPR,
    "Function 'def' passed incorrect type!");
  /* First argument is symbol list */
  lval* syms = a->cell[0];
  /* Ensure all elements of first list are symbols */
  for (int i = 0; i < syms->count; i++) {
    LASSERT(a, syms->cell[i]->type == LVAL_SYM,
      "Function 'def' cannot define non-symbol");
  }
  /* Check correct number of symbols and values */
  LASSERT(a, syms->count == a->count-1,
    "Function 'def' cannot define incorrect "
    "number of values to symbols");
  /* Assign copies of values to symbols */
  for (int i = 0; i < syms->count; i++) {
    lenv_put(e, syms->cell[i], a->cell[i+1]);
  }
  lval_del(a);
  return lval_sexpr();
}
```
我们需要把这个新的内置函数用内置函数`lenv_add_builtins`注册一下（译注：可以看出，用户定义函数与内置函数地位相同）。

```c
/* Variable Functions */
lenv_add_builtin(e, "def",  builtin_def);
```

目前我们已经可以支持用户定义变量了。由于`def`函数可以接收一个标记列表，所以我们可以做一点酷酷的事，例如在输入它们之前，存储和处理标志列表。试用一下我们的这个交互环境，并保证一切工作正常。应该可以得到设想的结果了，可以探索一下定义和求值变量的其它复杂用法。一旦我们有一定义函数，我们会发现这还挺有用的。

```
lispy> def {x} 100
()
lispy> def {y} 200
()
lispy> x
100
lispy> y
200
lispy> + x y
300
lispy> def {a b} 5 6
()
lispy> + a b
11
lispy> def {arglist} {a b x y}
()
lispy> arglist
{a b x y}
lispy> def arglist 1 2 3 4
()
lispy> list a b x y
{1 2 3 4}
lispy>
```
## 错误报告机制
------
目前为止，我们错误报告机制还不怎么样。虽然当错误发生时，的确已经可以报告错误，但是只能给出一个模糊的提示，并不能给出更多的关于错误如何发生的信息。比如说如果发生标志未绑定错误时，我们应该能够报精确告哪个标志未被绑定。这将有助于用于追踪错误、输入失误以及其它琐碎的问题。

月食• 省略号

![eclipses](http://buildyourownlisp.com/static/img/eclipses.png)

如果我们写一个函数可以像`printf`一样报告错误和其它的问题，那肯定棒极了。能够接受字符串、整数和其它类型的数据丰富错误信息肯定很理想。C语言中的`printf`是一个很特殊的函数，它就可以接受可变数量的参数。依照它我们也可以创建我们自己的*可变参数*函数，用于改善我们的这个错误报告机制。

让我们来改造一下`lval_err`让它有与`printf`相似的功能：接受一个格式化字符串，之后接受可变数量的参数用于匹配前面的格式字符串。
创建过程要使用到省略号`...`，它用来代表不定数量的参数（译注：这个机制绝大多数普通C语言教程都没有说过！很酷，对不对？）。

```c
lval* lval_err(char* fmt, ...);
```

能这么做的原因是，在函数内部有一些标准函数可以用来确定调用者提供了什么参数（译注：C语言还是相当强大）。第一步是创建`va_list`结构并将其初始化为 `va_start`，传给最末的命名参数。出于其它的目的，检查每一个用`va_arg`传入的参数也是有可能的，不过我们打算将参数变量列表做为一个整体直接传给`vsnprintf`函数。这个函数表现的如同`printf`函数，不同的是它接受一个`va_list`并写入到一个字符串中 。准备好了变量参数之后就可以调用`va_end`来清理已用的资源。这个`vsnprintf`函数输出到一个字符串，因此提前应该申请好内存。但由于事先无法知道它的大小，我们就只能先申请`512`字节用着，定下大小输出完毕之后再换成小一点的合适大小。 若有错误信息长度大于512字符，这里就直接砍掉了，希望这种事不会发生吧。

综上，如下是新的错误处理函数：
```c
lval* lval_err(char* fmt, ...) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_ERR;
  /* Create a va list and initialize it */
  va_list va;
  va_start(va, fmt);
  /* Allocate 512 bytes of space */
  v->err = malloc(512);
  /* printf the error string with a maximum of 511 characters */
  vsnprintf(v->err, 511, fmt, va);
  /* Reallocate to number of bytes actually used */
  v->err = realloc(v->err, strlen(v->err)+1);
  /* Cleanup our va list */
  va_end(va);
  return v;
}
```
错误报告函数看起来不错，例如`lenv_get`，假如某个标志找不着，我们能够报告标志的名字，而不仅仅是笼统地提示一个错误。

```c
return lval_err("Unbound Symbol '%s'", k->sym);
```

还可以改造一下我们的`LASSERT`宏（译注：不定参数宏的改造，此为高阶用法），让它也能够接受可变参数。由于这是一个宏而不是一个标准函数，做法些有不同（译注：简直大有不同）。在定义的左边我们继续使用省略符号，不过在右边我们得用一个特殊变量`__VA_ARGS__`，用来粘贴（paste in）所有剩余参数的内容。

这个特殊变量打头必须是两个丼号符`##`，以确保当再没有其它参数时，它被正确的粘过来了（译注：对的，宏的调用是解析部分（CPP）的事，与函数调用完全不同，原文此处使用`pasted`）。本质上这么做只是为了确认删除起首的逗号`,`这样才能确认没有遗漏。

由于在构建错误信息的时候也许会使用`args`所以我们还得确保在生成错误值之前不要删除它。

```c
#define LASSERT(args, cond, fmt, ...) \
  if (!(cond)) { \
    lval* err = lval_err(fmt, ##__VA_ARGS__); \
    lval_del(args); \
    return err; \
  }
```

这下就可以更新错误提示来显示更多信息了。举例来说如果错误数量的参数被传进来我们就可以确定我们需要多少个而实际传来了多少个（译注：这肯定挺有用的，这么高级的功能我从来没有使用过）。

```c
LASSERT(a, a->count == 1,
  "Function 'head' passed too many arguments. "
  "Got %i, Expected %i.",
  a->count, 1);
```
还能把类型错误提示再改进一下，报告函数需要什么类型而实际得到了什么类型。在此之前还要写一个函数，这个函数接收某个类型的参数并返回代表这种类型名称的字符串。

```c
char* ltype_name(int t) {
  switch(t) {
    case LVAL_FUN: return "Function";
    case LVAL_NUM: return "Number";
    case LVAL_ERR: return "Error";
    case LVAL_SYM: return "Symbol";
    case LVAL_SEXPR: return "S-Expression";
    case LVAL_QEXPR: return "Q-Expression";
    default: return "Unknown";
  }
}
LASSERT(a, a->cell[0]->type == LVAL_QEXPR,
  "Function 'head' passed incorrect type for argument 0. "
  "Got %s, Expected %s.",
  ltype_name(a->cell[0]->type), ltype_name(LVAL_QEXPR));
```
在代码中试下这个`LASSERT`吧，这样的提示比之前好太多了。下一步用我们这个新语言写复杂代码时，debug起来肯定更容易。试试看你能否使用宏来写自动生成错误报告的通用代码（译注：脑子说我可以试试，手说：我不行！）
```
lispy> + 1 {5 6 7}
Error: Function '+' passed incorrect type for argument 1. Got Q-Expression, Expected Number.
lispy> head {1 2 3} {4 5 6}
Error: Function 'head' passed incorrect number of arguments. Got 2, Expected 1.
lispy>
```
## 参考：

------
#### [variables.c](http://buildyourownlisp.com/chapter11_variables#collapseOne)
```c
#include "mpc.h"

#ifdef _WIN32

static char buffer[2048];

char* readline(char* prompt) {
  fputs(prompt, stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

void add_history(char* unused) {}

#else
#include <readline/readline.h>
#include <readline/history.h>
#endif

/* Forward Declarations */

struct lval;
struct lenv;
typedef struct lval lval;
typedef struct lenv lenv;

/* Lisp Value */

enum { LVAL_ERR, LVAL_NUM,   LVAL_SYM, 
       LVAL_FUN, LVAL_SEXPR, LVAL_QEXPR };

typedef lval*(*lbuiltin)(lenv*, lval*);

struct lval {
  int type;
  long num;
  char* err;
  char* sym;
  lbuiltin fun;
  int count;
  lval** cell;
};

lval* lval_num(long x) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_NUM;
  v->num = x;
  return v;
}

lval* lval_err(char* fmt, ...) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_ERR;
  
  /* Create a va list and initialize it */
  va_list va;
  va_start(va, fmt);
  
  /* Allocate 512 bytes of space */
  v->err = malloc(512);
  
  /* printf the error string with a maximum of 511 characters */
  vsnprintf(v->err, 511, fmt, va);
  
  /* Reallocate to number of bytes actually used */
  v->err = realloc(v->err, strlen(v->err)+1);
  
  /* Cleanup our va list */
  va_end(va);
  
  return v;
}

lval* lval_sym(char* s) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_SYM;
  v->sym = malloc(strlen(s) + 1);
  strcpy(v->sym, s);
  return v;
}

lval* lval_fun(lbuiltin func) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_FUN;
  v->fun = func;
  return v;
}

lval* lval_sexpr(void) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_SEXPR;
  v->count = 0;
  v->cell = NULL;
  return v;
}

lval* lval_qexpr(void) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_QEXPR;
  v->count = 0;
  v->cell = NULL;
  return v;
}

void lval_del(lval* v) {

  switch (v->type) {
    case LVAL_NUM: break;
    case LVAL_FUN: break;
    case LVAL_ERR: free(v->err); break;
    case LVAL_SYM: free(v->sym); break;
    case LVAL_QEXPR:
    case LVAL_SEXPR:
      for (int i = 0; i < v->count; i++) {
        lval_del(v->cell[i]);
      }
      free(v->cell);
    break;
  }
  
  free(v);
}

lval* lval_copy(lval* v) {

  lval* x = malloc(sizeof(lval));
  x->type = v->type;
  
  switch (v->type) {
    
    /* Copy Functions and Numbers Directly */
    case LVAL_FUN: x->fun = v->fun; break;
    case LVAL_NUM: x->num = v->num; break;
    
    /* Copy Strings using malloc and strcpy */
    case LVAL_ERR:
      x->err = malloc(strlen(v->err) + 1);
      strcpy(x->err, v->err); break;
      
    case LVAL_SYM:
      x->sym = malloc(strlen(v->sym) + 1);
      strcpy(x->sym, v->sym); break;
    
    /* Copy Lists by copying each sub-expression */
    case LVAL_SEXPR:
    case LVAL_QEXPR:
      x->count = v->count;
      x->cell = malloc(sizeof(lval*) * x->count);
      for (int i = 0; i < x->count; i++) {
        x->cell[i] = lval_copy(v->cell[i]);
      }
    break;
  }
  
  return x;
}

lval* lval_add(lval* v, lval* x) {
  v->count++;
  v->cell = realloc(v->cell, sizeof(lval*) * v->count);
  v->cell[v->count-1] = x;
  return v;
}

lval* lval_join(lval* x, lval* y) {  
  for (int i = 0; i < y->count; i++) {
    x = lval_add(x, y->cell[i]);
  }
  free(y->cell);
  free(y);  
  return x;
}

lval* lval_pop(lval* v, int i) {
  lval* x = v->cell[i];  
  memmove(&v->cell[i], &v->cell[i+1],
    sizeof(lval*) * (v->count-i-1));  
  v->count--;  
  v->cell = realloc(v->cell, sizeof(lval*) * v->count);
  return x;
}

lval* lval_take(lval* v, int i) {
  lval* x = lval_pop(v, i);
  lval_del(v);
  return x;
}

void lval_print(lval* v);

void lval_print_expr(lval* v, char open, char close) {
  putchar(open);
  for (int i = 0; i < v->count; i++) {
    lval_print(v->cell[i]);    
    if (i != (v->count-1)) {
      putchar(' ');
    }
  }
  putchar(close);
}

void lval_print(lval* v) {
  switch (v->type) {
    case LVAL_FUN:   printf("<function>"); break;
    case LVAL_NUM:   printf("%li", v->num); break;
    case LVAL_ERR:   printf("Error: %s", v->err); break;
    case LVAL_SYM:   printf("%s", v->sym); break;
    case LVAL_SEXPR: lval_print_expr(v, '(', ')'); break;
    case LVAL_QEXPR: lval_print_expr(v, '{', '}'); break;
  }
}

void lval_println(lval* v) { lval_print(v); putchar('\n'); }

char* ltype_name(int t) {
  switch(t) {
    case LVAL_FUN: return "Function";
    case LVAL_NUM: return "Number";
    case LVAL_ERR: return "Error";
    case LVAL_SYM: return "Symbol";
    case LVAL_SEXPR: return "S-Expression";
    case LVAL_QEXPR: return "Q-Expression";
    default: return "Unknown";
  }
}

/* Lisp Environment */

struct lenv {
  int count;
  char** syms;
  lval** vals;
};

lenv* lenv_new(void) {

  /* Initialize struct */
  lenv* e = malloc(sizeof(lenv));
  e->count = 0;
  e->syms = NULL;
  e->vals = NULL;
  return e;
  
}

void lenv_del(lenv* e) {
  
  /* Iterate over all items in environment deleting them */
  for (int i = 0; i < e->count; i++) {
    free(e->syms[i]);
    lval_del(e->vals[i]);
  }
  
  /* Free allocated memory for lists */
  free(e->syms);
  free(e->vals);
  free(e);
}

lval* lenv_get(lenv* e, lval* k) {
  
  /* Iterate over all items in environment */
  for (int i = 0; i < e->count; i++) {
    /* Check if the stored string matches the symbol string */
    /* If it does, return a copy of the value */
    if (strcmp(e->syms[i], k->sym) == 0) {
      return lval_copy(e->vals[i]);
    }
  }
  /* If no symbol found return error */
  return lval_err("Unbound Symbol '%s'", k->sym);
}

void lenv_put(lenv* e, lval* k, lval* v) {
  
  /* Iterate over all items in environment */
  /* This is to see if variable already exists */
  for (int i = 0; i < e->count; i++) {
  
    /* If variable is found delete item at that position */
    /* And replace with variable supplied by user */
    if (strcmp(e->syms[i], k->sym) == 0) {
      lval_del(e->vals[i]);
      e->vals[i] = lval_copy(v);
      return;
    }
  }
  
  /* If no existing entry found allocate space for new entry */
  e->count++;
  e->vals = realloc(e->vals, sizeof(lval*) * e->count);
  e->syms = realloc(e->syms, sizeof(char*) * e->count);
  
  /* Copy contents of lval and symbol string into new location */
  e->vals[e->count-1] = lval_copy(v);
  e->syms[e->count-1] = malloc(strlen(k->sym)+1);
  strcpy(e->syms[e->count-1], k->sym);
}

/* Builtins */

#define LASSERT(args, cond, fmt, ...) \
  if (!(cond)) { lval* err = lval_err(fmt, ##__VA_ARGS__); lval_del(args); return err; }

#define LASSERT_TYPE(func, args, index, expect) \
  LASSERT(args, args->cell[index]->type == expect, \
    "Function '%s' passed incorrect type for argument %i. Got %s, Expected %s.", \
    func, index, ltype_name(args->cell[index]->type), ltype_name(expect))

#define LASSERT_NUM(func, args, num) \
  LASSERT(args, args->count == num, \
    "Function '%s' passed incorrect number of arguments. Got %i, Expected %i.", \
    func, args->count, num)

#define LASSERT_NOT_EMPTY(func, args, index) \
  LASSERT(args, args->cell[index]->count != 0, \
    "Function '%s' passed {} for argument %i.", func, index);


lval* lval_eval(lenv* e, lval* v);

lval* builtin_list(lenv* e, lval* a) {
  a->type = LVAL_QEXPR;
  return a;
}

lval* builtin_head(lenv* e, lval* a) {
  LASSERT_NUM("head", a, 1);
  LASSERT_TYPE("head", a, 0, LVAL_QEXPR);
  LASSERT_NOT_EMPTY("head", a, 0);
  
  lval* v = lval_take(a, 0);  
  while (v->count > 1) { lval_del(lval_pop(v, 1)); }
  return v;
}

lval* builtin_tail(lenv* e, lval* a) {
  LASSERT_NUM("tail", a, 1);
  LASSERT_TYPE("tail", a, 0, LVAL_QEXPR);
  LASSERT_NOT_EMPTY("tail", a, 0);

  lval* v = lval_take(a, 0);  
  lval_del(lval_pop(v, 0));
  return v;
}

lval* builtin_eval(lenv* e, lval* a) {
  LASSERT_NUM("eval", a, 1);
  LASSERT_TYPE("eval", a, 0, LVAL_QEXPR);
  
  lval* x = lval_take(a, 0);
  x->type = LVAL_SEXPR;
  return lval_eval(e, x);
}

lval* builtin_join(lenv* e, lval* a) {
  
  for (int i = 0; i < a->count; i++) {
    LASSERT_TYPE("join", a, i, LVAL_QEXPR);
  }
  
  lval* x = lval_pop(a, 0);
  
  while (a->count) {
    lval* y = lval_pop(a, 0);
    x = lval_join(x, y);
  }
  
  lval_del(a);
  return x;
}

lval* builtin_op(lenv* e, lval* a, char* op) {
  
  for (int i = 0; i < a->count; i++) {
    LASSERT_TYPE(op, a, i, LVAL_NUM);
  }
  
  lval* x = lval_pop(a, 0);
  
  if ((strcmp(op, "-") == 0) && a->count == 0) {
    x->num = -x->num;
  }
  
  while (a->count > 0) {  
    lval* y = lval_pop(a, 0);
    
    if (strcmp(op, "+") == 0) { x->num += y->num; }
    if (strcmp(op, "-") == 0) { x->num -= y->num; }
    if (strcmp(op, "*") == 0) { x->num *= y->num; }
    if (strcmp(op, "/") == 0) {
      if (y->num == 0) {
        lval_del(x); lval_del(y);
        x = lval_err("Division By Zero.");
        break;
      }
      x->num /= y->num;
    }
    
    lval_del(y);
  }
  
  lval_del(a);
  return x;
}

lval* builtin_add(lenv* e, lval* a) {
  return builtin_op(e, a, "+");
}

lval* builtin_sub(lenv* e, lval* a) {
  return builtin_op(e, a, "-");
}

lval* builtin_mul(lenv* e, lval* a) {
  return builtin_op(e, a, "*");
}

lval* builtin_div(lenv* e, lval* a) {
  return builtin_op(e, a, "/");
}

lval* builtin_def(lenv* e, lval* a) {

  LASSERT_TYPE("def", a, 0, LVAL_QEXPR);
  
  /* First argument is symbol list */
  lval* syms = a->cell[0];
  
  /* Ensure all elements of first list are symbols */
  for (int i = 0; i < syms->count; i++) {
    LASSERT(a, (syms->cell[i]->type == LVAL_SYM),
      "Function 'def' cannot define non-symbol. "
      "Got %s, Expected %s.",
      ltype_name(syms->cell[i]->type), ltype_name(LVAL_SYM));
  }
  
  /* Check correct number of symbols and values */
  LASSERT(a, (syms->count == a->count-1),
    "Function 'def' passed too many arguments for symbols. "
    "Got %i, Expected %i.",
    syms->count, a->count-1);
  
  /* Assign copies of values to symbols */
  for (int i = 0; i < syms->count; i++) {
    lenv_put(e, syms->cell[i], a->cell[i+1]);
  }
  
  lval_del(a);
  return lval_sexpr();
}

void lenv_add_builtin(lenv* e, char* name, lbuiltin func) {
  lval* k = lval_sym(name);
  lval* v = lval_fun(func);
  lenv_put(e, k, v);
  lval_del(k); lval_del(v);
}

void lenv_add_builtins(lenv* e) {
  /* Variable Functions */
  lenv_add_builtin(e, "def", builtin_def);
  
  /* List Functions */
  lenv_add_builtin(e, "list", builtin_list);
  lenv_add_builtin(e, "head", builtin_head);
  lenv_add_builtin(e, "tail", builtin_tail);
  lenv_add_builtin(e, "eval", builtin_eval);
  lenv_add_builtin(e, "join", builtin_join);
  
  /* Mathematical Functions */
  lenv_add_builtin(e, "+", builtin_add);
  lenv_add_builtin(e, "-", builtin_sub);
  lenv_add_builtin(e, "*", builtin_mul);
  lenv_add_builtin(e, "/", builtin_div);
}

/* Evaluation */

lval* lval_eval_sexpr(lenv* e, lval* v) {
  
  for (int i = 0; i < v->count; i++) {
    v->cell[i] = lval_eval(e, v->cell[i]);
  }
  
  for (int i = 0; i < v->count; i++) {
    if (v->cell[i]->type == LVAL_ERR) { return lval_take(v, i); }
  }
  
  if (v->count == 0) { return v; }  
  if (v->count == 1) { return lval_take(v, 0); }
  
  /* Ensure first element is a function after evaluation */
  lval* f = lval_pop(v, 0);
  if (f->type != LVAL_FUN) {
    lval* err = lval_err(
      "S-Expression starts with incorrect type. "
      "Got %s, Expected %s.",
      ltype_name(f->type), ltype_name(LVAL_FUN));
    lval_del(f); lval_del(v);
    return err;
  }
  
  /* If so call function to get result */
  lval* result = f->fun(e, v);
  lval_del(f);
  return result;
}

lval* lval_eval(lenv* e, lval* v) {
  if (v->type == LVAL_SYM) {
    lval* x = lenv_get(e, v);
    lval_del(v);
    return x;
  }
  if (v->type == LVAL_SEXPR) { return lval_eval_sexpr(e, v); }
  return v;
}

/* Reading */

lval* lval_read_num(mpc_ast_t* t) {
  errno = 0;
  long x = strtol(t->contents, NULL, 10);
  return errno != ERANGE ? lval_num(x) : lval_err("Invalid Number.");
}

lval* lval_read(mpc_ast_t* t) {
  
  if (strstr(t->tag, "number")) { return lval_read_num(t); }
  if (strstr(t->tag, "symbol")) { return lval_sym(t->contents); }
  
  lval* x = NULL;
  if (strcmp(t->tag, ">") == 0) { x = lval_sexpr(); } 
  if (strstr(t->tag, "sexpr"))  { x = lval_sexpr(); }
  if (strstr(t->tag, "qexpr"))  { x = lval_qexpr(); }
  
  for (int i = 0; i < t->children_num; i++) {
    if (strcmp(t->children[i]->contents, "(") == 0) { continue; }
    if (strcmp(t->children[i]->contents, ")") == 0) { continue; }
    if (strcmp(t->children[i]->contents, "}") == 0) { continue; }
    if (strcmp(t->children[i]->contents, "{") == 0) { continue; }
    if (strcmp(t->children[i]->tag,  "regex") == 0) { continue; }
    x = lval_add(x, lval_read(t->children[i]));
  }
  
  return x;
}

/* Main */

int main(int argc, char** argv) {
  
  mpc_parser_t* Number = mpc_new("number");
  mpc_parser_t* Symbol = mpc_new("symbol");
  mpc_parser_t* Sexpr  = mpc_new("sexpr");
  mpc_parser_t* Qexpr  = mpc_new("qexpr");
  mpc_parser_t* Expr   = mpc_new("expr");
  mpc_parser_t* Lispy  = mpc_new("lispy");
  
  mpca_lang(MPCA_LANG_DEFAULT,
    "                                                     \
      number : /-?[0-9]+/ ;                               \
      symbol : /[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/ ;         \
      sexpr  : '(' <expr>* ')' ;                          \
      qexpr  : '{' <expr>* '}' ;                          \
      expr   : <number> | <symbol> | <sexpr> | <qexpr> ;  \
      lispy  : /^/ <expr>* /$/ ;                          \
    ",
    Number, Symbol, Sexpr, Qexpr, Expr, Lispy);
  
  puts("Lispy Version 0.0.0.0.7");
  puts("Press Ctrl+c to Exit\n");
  
  lenv* e = lenv_new();
  lenv_add_builtins(e);
  
  while (1) {
  
    char* input = readline("lispy> ");
    add_history(input);
    
    mpc_result_t r;
    if (mpc_parse("<stdin>", input, Lispy, &r)) {
      lval* x = lval_eval(e, lval_read(r.output));
      lval_println(x);
      lval_del(x);
      mpc_ast_delete(r.output);
    } else {    
      mpc_err_print(r.error);
      mpc_err_delete(r.error);
    }
    
    free(input);
    
  }
  
  lenv_del(e);
  
  mpc_cleanup(6, Number, Symbol, Sexpr, Qexpr, Expr, Lispy);
  
  return 0;
}

```
## 课后作业
- 创建一个宏（*Macro*）以便能够报告参数类型错误。
- 创建一个宏（*Macro*）以便能够报告参数个数不匹配错误。
- 创建一个宏（*Macro*）以便能够报告空表错误。
- 更改内置printing函数用于打印函数的名字。
- 写一个函数，打印出当前环境中所有的变量名。
- 将一个内置变量重新定义一下。
- 重新定义一个内置变量，专门用于处理错误。   › Change redefinition of one of the builtin variables to something different an error.
- 建立正常退出函数，可以不使用Ctrl-c中止程序。
