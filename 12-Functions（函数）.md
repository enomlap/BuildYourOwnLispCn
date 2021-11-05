##第12章• 函数

#什么是函数？

函数是所有编程的本质。在计算机科学的早期年代，有一个幼稚的梦想，就是能够把计算一步步的分解成一段段更小的可重用的代码片段。假以时日，并且得到了合适的结构，最终我们能写出满足所有计算需求的代码。人们从此再不用编写自己的函数了，编程将变成搭积木一样。

虽然这个梦想至今也还没有变成现实（译注：但是已经有了这样的趋势，现在编写 *Python* 代码的时候已经只有极少的部分是自己根据算法来写的了，而是使用大量的现成的库），但是不管如何，很明显这的确是现在的发展方向。新出现的编程语言或范式大致都遵从这个想法，承诺能更好地重用代码，具有更好的抽象，让所有人的生活更轻松。 

回到编程的现实中来看，每个范式提供的只是 *不同的* 抽象。抽象层次越高，扔掉的肯定越多。这意味着，不管你多么努力的想要保持全部的特性，最终你总得有所取舍。不过分解成函数的办法往往被证明的确挺有效的。  

对于我们在 C 语言中使用过的很多函数，我们仅需知道函数的 *样子* ，不用知道函数到底是什么。对此可以有几种理解方式。 

其一是将其想像成 *将会发生的一件事* 。 定义一个函数等价于：“当我用 *这个* 名字，我希望会发生某件事情”。 这是一个非常实用的思想。 
这很自然并且相当形像。如同你给一个人或者宠物发一个命令。一个让人易于接受此看法的原因是，这个看法突显了函数的延迟特性。一次定义函数，处处可用。

另一个看待的角度是，将函数看作是这么一个黑匣子：你给它一定的输入，它就给你一定的输出。这个看法与第一个看法有些不同。这个看法更接近代数学思想，完全不考虑计算和指令。这样的定义已经纯粹是一个数学上的概念，并且完全与具体机器或者编程语言无关了。某些情况下，这个看法特别有用。它允许我们考虑函数的时候可以不用再虑及其内部结构或者具体实现。我们可以组合不同函数而忽略细节。这是抽像背后的核心思想，并且允许复杂调用而不会产生冲突。这种看法的强大之处也可能会成为缺点，比如说它并没有考虑具体的实现，不能回答诸如“这个函数需要多长时间来跑？”，以及“这个函数效率如何？“、“这个函数是否会改变程序的运行状态，如果是的话，怎么改？”

第三种理解则是将函数看作 *部分计算* 。如同数学模型一样能够接受一定的输入。想要函数完成计算的话，这些输入必须提前准备好。这就是为什么把这种看法叫作 *部分计算* 的原因。不过我喜欢这种模型，函数体包含具体实现。这些输入被称为 *未绑定变量* ，如果想要完成计算的话只需要简单的提供这些变量即可。就像给一台没有具体功能的机器装入“齿轮”，结果可以得到了一台能够运行的机器，然后这台机器开始运转。“部分计算”的输出就是一个未知值的变量，这个输出还能当作另一个函数的输入。产生特定的函数依赖。

这个想法的先进更甚于数学模型，原因在于将计算过程也考虑在内了。我们可以看见什么时候计算执行，以及机器内部的实际运转过程。这意味着我们认识到了随着时间流逝，特定的事情将会发生，或者说函数将会影响程序的状态，或者其它的我们不确定的事。

所有这些思想将会在后面的学习中探究，并称之为 *Lambda表达式* 。这个领域组合了逻辑、数学和计算机科学。其名称来源于希腊字母Lambda，用于代表绑定的值。使用Lambda表达式给了我们一种定义、组合、建立函数的一种简单方法。

我们将会使用前面学过的办法来定义函数。Lisp很适合实现这样的概念，对于我们来说也用不着太多的工作就能实现这样的函数。

第一步，先来写一个内置函数，用来生成用户定义函数。下面是一个具体的例子，第一个参数是一个标志列表，就像 `def` 函数一样。这个标志我们称为 `形式参数` ,同时也是一个 `未绑定变量` 。它代表了我们的输入的 *部分计算* 。第二个参数是另一个列表，当运行这个函数的时候，这一部分将会被内置函数 `eval` 求值。

目前先将这个函数简单的称为 `\`，就是一个斜杠，（向 Lambda 微积分致敬，它看起来像是小写的Lambda字母）。 像下面这样就可以创建一个加法函数了：

```lisp
\ {x y} {+ x y}
```

我们可以通过将其作为普通 S 表达式中的第一个参数来调用该函数：

```lisp
(\ {x y} {+ x y}) 10 20
```

也可以像命名普通变量一样使用内置函数 `def`给这个函数命名，并同时保存在环境中。

```lisp
def {add-together} (\ {x y} {+ x y})
```

然后我们可以用它的名字来调用 了：

```lisp
add-together 10 20
```

函数类型
想要保存一个函数为一个'lval'，我们必需确定它里面都含有什么。

由之前的定义，一个函数由三部分组成。第一个是形式参数的列表, 必须绑定值以后才能对这个函数求值。第二部分是一个Q-Expression表达式，是函数主体。最后我们还需要一个地方来存储赋给形式参数的值。很幸运，我们已经有地方可以来存变量了，那就是 `环境（environment）`。

同样可以使用类型 `LVAL_FUN` 来存储内置函数和用户函数。不过，这样的话我们需要一个内在的办法来区分它们。我们可以通过检测内置函数指针 `lbuiltin` 是否为 `NULL` 来区分。如果不是 `NULL` ，我们就知道这是一个内置函数，反之则是一个用户函数。

```c
struct lval {
  int type;

  /* Basic */
  long num;
  char* err;
  char* sym;

  /* Function */
  lbuiltin builtin;
  lenv* env;
  lval* formals;
  lval* body;

  /* Expression */
  int count;
  lval** cell;
};
```

这里，我们把 `lbuiltin` 域从 `fun` 改成了 `builtin` ，相应地，别忘了把我们代码中所有相关部分都一起改掉。

还需要创建一个构造函数用于用户定义 `lval` 函数。这里我们为函数创建一个新的 `环境` ，并且用输入的内容给形参和函数体赋值。

```c
lval* lval_lambda(lval* formals, lval* body) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_FUN;

  /* Set Builtin to Null */
  v->builtin = NULL;

  /* Build new environment */
  v->env = lenv_new();

  /* Set Formals and Body */
  v->formals = formals;
  v->body = body;
  return v;
}
```

就像每次改变的 `lval`的类型时一样，我们也需要更新一下 *删除* 、 *复制* 和 *打印* 的功能。不过对于 *求值* ，我们需要更深入地研究。 
对于 **删除** ...

```c
case LVAL_FUN:
  if (!v->builtin) {
    lenv_del(v->env);
    lval_del(v->formals);
    lval_del(v->body);
  }
break;
```

对于 **复制** ...

```c
case LVAL_FUN:
  if (v->builtin) {
    x->builtin = v->builtin;
  } else {
    x->builtin = NULL;
    x->env = lenv_copy(v->env);
    x->formals = lval_copy(v->formals);
    x->body = lval_copy(v->body);
  }
break;
```

用于 **打印** ...

```c
case LVAL_FUN:
  if (v->builtin) {
    printf("<builtin>");
  } else {
    printf("(\\ "); lval_print(v->formals);
    putchar(' '); lval_print(v->body); putchar(')');
  }
break;
```

## Lambda 函数

------

现在可以为 `lambda` 函数添加一个内置函数。我们希望它接收一些符号列表和一个表示代码的列表作为输入，之后它应该返回一个函数 `lval` 。我们现在已经定义了一些内置函数，这里将遵循相同的格式。和 `def` 一样，还需要（使用一些新定义的宏）做错误检查以确保参数类型和数目正确。然后我们从列表中弹出前两个参数并将它们传递给我们之前定义的函数 `lval_lambda` ...

```c
lval* builtin_lambda(lenv* e, lval* a) {
  /* Check Two arguments, each of which are Q-Expressions */
  LASSERT_NUM("\\", a, 2);
  LASSERT_TYPE("\\", a, 0, LVAL_QEXPR);
  LASSERT_TYPE("\\", a, 1, LVAL_QEXPR);

  /* Check first Q-Expression contains only Symbols */
  for (int i = 0; i < a->cell[0]->count; i++) {
    LASSERT(a, (a->cell[0]->cell[i]->type == LVAL_SYM),
      "Cannot define non-symbol. Got %s, Expected %s.",
      ltype_name(a->cell[0]->cell[i]->type),ltype_name(LVAL_SYM));
  }

  /* Pop first two arguments and pass them to lval_lambda */
  lval* formals = lval_pop(a, 0);
  lval* body = lval_pop(a, 0);
  lval_del(a);

  return lval_lambda(formals, body);
}
```

`LASSERT_NUM` 和 `LASSERT_TYPE` 来自何方？

我冒昧地改进了本章的错误报告宏。这是上一章彩蛋部分作业的一个。它使代码如此清晰，没办法不用。 

如果您正打算自己完成此任务，现在正是好时机。不过，您也可以参考本章中我所采用的代码，并将其集成到你自己的代码中。 我们把这个函数和其它内置函数一起注册一下...

```c
lenv_add_builtin(e, "\\", builtin_lambda);
```

## 游戏时间 • 父环境

我们为函数提供了它们私有的环境。其形参的值存储于此。得到相应的值后，便可在此环境中对函数体求值。

不过理想情况下，我们还希望这些函数能够访问全局环境中的变量，例如内置函数。 

解决此问题可以通过在环境的定义包含一条对 *父* 环境的引用来实现。当我们想要求值一个函数时，将此 *父* 环境设置为全局环境，其中定义了所有内置函数。 

这一部分要注意一下，当把此功能添加到的 `lenv` 结构中时，这一部分是 *引用* 环境 ，而不是子环境之类。当`lenv`被删除或者复制后本该删除时，不要 *删除* 它！ 

 
至于实现则很简单，如果在当前环境调用 `lenv_get` 却找不到符号变量，就一路往上 *层层遍历* 父环境，查看命名值是否存在。`NULL`可以用来表示没有父环境。 

构造函数只需稍稍改动即可 ...

```c
struct lenv {
  lenv* par;
  int count;
  char** syms;
  lval** vals;
};

lenv* lenv_new(void) {
  lenv* e = malloc(sizeof(lenv));
  e->par = NULL;
  e->count = 0;
  e->syms = NULL;
  e->vals = NULL;
  return e;
}
```

取值时如果找不到值，需要遍历搜索父环境 ...

```c
lval* lenv_get(lenv* e, lval* k) {

  for (int i = 0; i < e->count; i++) {
    if (strcmp(e->syms[i], k->sym) == 0) {
      return lval_copy(e->vals[i]);
    }
  }

  /* If no symbol check in parent otherwise error */
  if (e->par) {
    return lenv_get(e->par, k);
  } else {
    return lval_err("Unbound Symbol '%s'", k->sym);
  }
}
```

由于更新了 `lval`类型，用于复制环境的函数也要更新 ...

```c
lenv* lenv_copy(lenv* e) {
  lenv* n = malloc(sizeof(lenv));
  n->par = e->par;
  n->count = e->count;
  n->syms = malloc(sizeof(char*) * n->count);
  n->vals = malloc(sizeof(lval*) * n->count);
  for (int i = 0; i < e->count; i++) {
    n->syms[i] = malloc(strlen(e->syms[i]) + 1);
    strcpy(n->syms[i], e->syms[i]);
    n->vals[i] = lval_copy(e->vals[i]);
  }
  return n;
}
```

至此，变量 *定义* 的范围也有了新的概念 。 

目前有两种方法可以定义变量：或者在本地环境中定义，或者在全局部环境中定义。我们分别处理。`lenv_put` 方法不需要改变，依然可用于在本地环境中进行定义。不过需要一个新的函数 `lenv_def` 用于在全局环境中定义变量。这也很简单，只需要在使用 `lenv_put` 之前依照父链找到上一层环境就行了。

```c
void lenv_def(lenv* e, lval* k, lval* v) {
  /* Iterate till e has no parent */
  while (e->par) { e = e->par; }
  /* Put value in e */
  lenv_put(e, k, v);
}
```

这个区别目前似乎无用, 不过等会我们需要它来将部分结果写入函数内部的局部变量。需要添加一个内置函数用来 *局地* 绑定。在 C 语言中称之为 `put`，但在 Lisp 中我们给它加上一个 `=` 符号。就像我们的数学运算，调整一下 `builtin_def` 函数就可以重用相同公共代码了。

随后需要将这些注册为内置函数 ...

```c
lenv_add_builtin(e, "def", builtin_def);
lenv_add_builtin(e, "=",   builtin_put);

lval* builtin_def(lenv* e, lval* a) {
  return builtin_var(e, a, "def");
}

lval* builtin_put(lenv* e, lval* a) {
  return builtin_var(e, a, "=");
}

lval* builtin_var(lenv* e, lval* a, char* func) {
  LASSERT_TYPE(func, a, 0, LVAL_QEXPR);

  lval* syms = a->cell[0];
  for (int i = 0; i < syms->count; i++) {
    LASSERT(a, (syms->cell[i]->type == LVAL_SYM),
      "Function '%s' cannot define non-symbol. "
      "Got %s, Expected %s.", func,
      ltype_name(syms->cell[i]->type),
      ltype_name(LVAL_SYM));
  }

  LASSERT(a, (syms->count == a->count-1),
    "Function '%s' passed too many arguments for symbols. "
    "Got %i, Expected %i.", func, syms->count, a->count-1);

  for (int i = 0; i < syms->count; i++) {
    /* If 'def' define in globally. If 'put' define in locally */
    if (strcmp(func, "def") == 0) {
      lenv_def(e, syms->cell[i], a->cell[i+1]);
    }

    if (strcmp(func, "=")   == 0) {
      lenv_put(e, syms->cell[i], a->cell[i+1]);
    }
  }

  lval_del(a);
  return lval_sexpr();
}
```

## 函数调用 

------

当`lval`函数被调用时还不能求值，因为我们还没有编写表达式求值的代码。

如果这个函数类型是内置函数，我们可以像以前一样使用函数指针调用它。但对于用户定义函数还有额外的麻烦。我们需要将传入的每个参数分别绑定到 `formals`形参。 完成后就可以在`env`环境中对 `body`部分进行求值了，而当前环境作为这个环境的父环境。

这是没加错误检查的第一次的尝试：

```c
lval* lval_call(lenv* e, lval* f, lval* a) {

  /* If Builtin then simply call that */
  if (f->builtin) { return f->builtin(e, a); }

  /* Assign each argument to each formal in order */
  for (int i = 0; i < a->count; i++) {
      lenv_put(f->env, f->formals->cell[i], a->cell[i]);
  }

  lval_del(a);

  /* Set the parent environment */
  f->env->par = e;

  /* Evaluate the body */
  return builtin_eval(f->env,
    lval_add(lval_sexpr(), lval_copy(f->body)));
}
```

看起来不错，但是当提供的参数数量与形式参数的数量不同时， 它会崩溃! 

实际上，这是一个有趣的案例，给我们留下了几个选项。可以仅仅 `throw（抛出）` 一个错误，提示提供的参数计数不正确，也可以做得更好玩些。当提供的参数太少时，可以仅绑定函数的前几个形式参数，其余的不绑定。

这创建了一个 *部分计算* 的函数，和我们之前将函数当作 *部分计算的* 代码思想一致。假如某个函数接受两个参数但仅传入了一个，那就绑定第一个，然后返回一个新函数（译注：颠覆了 C 语言中关于函数的认识）。 

打个比方说，当一个函数面对表达式时，不断地吃掉右侧的输入。在吃进其右侧的第一个输入后，如果它吃饱了（不需要更多输入），它就求值并用新值替换自己（译注：类似雷电战机吃了足够的五星变身超级战机，或者大个子马立奥）。假如没吃饱， 就能绑定一个是一个，然后用另一个更完整的函数替换自己（译注：没吃到足够的五星还没能变身）。

想象一下吃豆人小游戏，不是一口吃掉所有豆子，而是一个一个吃，越来越大，直到它吃饱然后爆炸变身。代码中并不是这样实现，不过也差不多有趣。

```c
lval* lval_call(lenv* e, lval* f, lval* a) {

  /* If Builtin then simply apply that */
  if (f->builtin) { return f->builtin(e, a); }

  /* Record Argument Counts */
  int given = a->count;
  int total = f->formals->count;

  /* While arguments still remain to be processed */
  while (a->count) {

    /* If we've ran out of formal arguments to bind */
    if (f->formals->count == 0) {
      lval_del(a); return lval_err(
        "Function passed too many arguments. "
        "Got %i, Expected %i.", given, total);
    }

    /* Pop the first symbol from the formals */
    lval* sym = lval_pop(f->formals, 0);

    /* Pop the next argument from the list */
    lval* val = lval_pop(a, 0);

    /* Bind a copy into the function's environment */
    lenv_put(f->env, sym, val);

    /* Delete symbol and value */
    lval_del(sym); lval_del(val);
  }

  /* Argument list is now bound so can be cleaned up */
  lval_del(a);

  /* If all formals have been bound evaluate */
  if (f->formals->count == 0) {

    /* Set environment parent to evaluation environment */
    f->env->par = e;

    /* Evaluate and return */
    return builtin_eval(
      f->env, lval_add(lval_sexpr(), lval_copy(f->body)));
  } else {
    /* Otherwise return partially evaluated function */
    return lval_copy(f);
  }

}
```

上面的函数与我们所说的完全一样，此外，还添加了正确的错误处理。它先遍历每一个传入的参数，并尝试将每个参数放入环境中。然后它检查一下环境中参数是否已满，如果是，就进行求值，否则的话返回一个携带一些参数的自身的拷贝。

如果我们用 `lval_call`调用更新后的求值函数 `lval_eval_sexpr`，新系统应该可以运转了。

```c
lval* f = lval_pop(v, 0);
if (f->type != LVAL_FUN) {
  lval* err = lval_err(
    "S-Expression starts with incorrect type. "
    "Got %s, Expected %s.",
    ltype_name(f->type), ltype_name(LVAL_FUN));
  lval_del(f); lval_del(v);
  return err;
}

lval* result = lval_call(e, f, v);
```

尝试自己定义函数并试试部分求值。 

```lisp
lispy> def {add-mul} (\ {x y} {+ x (* x y)})
()
lispy> add-mul 10 20
210
lispy> add-mul 10
(\ {y} {+ x (* x y)})
lispy> def {add-mul-ten} (add-mul 10)
()
lispy> add-mul-ten 50
510
lispy>
```
## 可变参数

------

我们已经定义了一些内置函数，因此它们可以接受可变数量的参数。逻辑上，函数 `+` 和 `join` 可以接受任意数量的参数。我们应该寻求某种办法以便用户定义函数也能这样。

不幸的是，除非添加一些特殊的语法，否则并没有什么优雅的方式能够实现。既然这样只能使用特殊符号`&`来点硬核编码了。先把形式参数改成 `{x & xs}` 的形式，表示一个函数可以接受一个参数 `x` ，后跟一个列表 `xs` ，其中 `xs` 包含零个或多个其他参数，类似于 C 语言中用于声明可变参数的省略号。 

在给形式参数赋值时，查找是否存在 `&` 符号，若存在，则将剩余参数都分配给这个形式参数。必须将此参数列表转换为 Q-Expression（译注：否则会被求值）。记住，`&` 后面须跟着一个真正的符号，否则应该抛出一个错误。

当第一个符号从形参中弹出后，立即在 `lval_call` 的 `while` 循环里加入这段特殊代码。

```c
/* Special Case to deal with '&' */
if (strcmp(sym->sym, "&") == 0) {

  /* Ensure '&' is followed by another symbol */
  if (f->formals->count != 1) {
    lval_del(a);
    return lval_err("Function format invalid. "
      "Symbol '&' not followed by single symbol.");
  }

  /* Next formal should be bound to remaining arguments */
  lval* nsym = lval_pop(f->formals, 0);
  lenv_put(f->env, nsym, builtin_list(e, a));
  lval_del(sym); lval_del(nsym);
  break;
}
```

假设在调用函数时未提供可变参数，而只提供了第一个参数。在此情形下，需要将符号 `&` 置为空表。在删除参数列表之后，检查是否所有形参都已经示值之前，添加处理这个特殊情况的代码。 

```c
/* If '&' remains in formal list bind to empty list */
if (f->formals->count > 0 &&
  strcmp(f->formals->cell[0]->sym, "&") == 0) {

  /* Check to ensure that & is not passed invalidly. */
  if (f->formals->count != 2) {
    return lval_err("Function format invalid. "
      "Symbol '&' not followed by single symbol.");
  }

  /* Pop and delete '&' symbol */
  lval_del(lval_pop(f->formals, 0));

  /* Pop next symbol and create empty list */
  lval* sym = lval_pop(f->formals, 0);
  lval* val = lval_qexpr();

  /* Bind to environment and delete */
  lenv_put(f->env, sym, val);
  lval_del(sym); lval_del(val);
}
```

## 有趣的函数

------

### 函数定义 Function Definition

很显然Lambda语法是定义函数的一种简单而强大的方法。不过因为有太多的括号和符号，总归看起来有点笨。尝试使用一些更简单的语法编写一个定义函数本身的函数，也许更有意思

本质上我们只是想要一个可以一次性完成两个步骤的函数。首先它应该能够创建一个新函数，然后将此函数定义为某个名称。有个巧办法，让用户在一个列表中提供名称和形式参数，然后分开，并用于函数定义中。函数如下。此函数接受参数和函数体作为输入。以参数头作为函数名称，其余部分作为形式参数，函数体则直接传递给 lambda。

```lisp
\ {args body} {def (head args) (\ (tail args) body)}
```

我们可以照常将这个函数命名为 `fun`，并将其传递给 `def`。

```lisp
def {fun} (\ {args body} {def (head args) (\ (tail args) body)})
```

如此定义函数更简单方便了。  定义之前的 `add-together`函数如下所示，可以用函数来定义函数。在 C语言 中可能永远也做不到，这可真是太酷了！ 

```lisp
fun {add-together x y} {+ x y}
```

![curry咖喱](https://www.buildyourownlisp.com/static/img/curry.png)

柯里化 • 不像听起来那么好。 Currying • Not as good as it sounds.

### 柯里化（Currying）

目前像 `+`这类函数可以接受可变数量的参数，某些情况下的确不错，但是如果传递的参数本身就是一个列表呢？似乎难以处理。

再次创建一个函数来解决这个特例。如果以期望的格式创建一个列表，就可以使用 `eval` 来处理。对于 `+` 可以将此函数附加到列表的前面，然后进行求值。

可以定义一个 `unpack`函数来做这事。它接受一些函数和一些列表作为输入，并在求值之前将该函数附加到列表的前面。 

```lisp
fun {unpack f xs} {eval (join (list f) xs)}
```
 
在某些情况下，可能会面临相反的困境。有可能有一个将某个列表作为输入的函数，但我们希望使用可变参数来调用它。在这种情况下，解决方案其实更简单一些。可以使用 `&` 变量参数语法将变量参数打包成一个列表。

```lisp
fun {pack f & xs} {f xs}
```

在某些语言中，这分别被称为 *柯里化* 和 *非柯里化* 。这其实来源于 *Haskell语言的 Curry* 而与我们喜欢的辣味毫无关系。

```lisp
lispy> def {uncurry} pack
()
lispy> def {curry} unpack
()
lispy> curry + {5 6 7}
18
lispy> uncurry head 5 6 7
{5}
```

由于部分求值的特性，我们不需要考虑用特定的参数化进行 *柯里*（译注：不懂什么意思）。可以想见函数自己的 *柯里化* 或 *非柯里化* 的形式。

```lisp
lispy> def {add-uncurried} +
()
lispy> def {add-curried} (curry +)
()
lispy> add-curried {5 6 7}
18
lispy> add-uncurried 5 6 7
18
```

玩耍一下，看看还能想出什么有趣或者强大（有趣和强大常想伴随）的功能。  下一章，将添加条件语法，这会使我们的语言更加完备。但这并不意味着你现在不能实现一些有趣的事。我们的 Lisp 越来越丰富了。 

## 参考 Reference
functions.c
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
#include <editline/readline.h>
#include <editline/history.h>
#endif

/* Forward Declarations */

struct lval;
struct lenv;
typedef struct lval lval;
typedef struct lenv lenv;

/* Lisp Value */

enum { LVAL_ERR, LVAL_NUM, LVAL_SYM, LVAL_FUN, LVAL_SEXPR, LVAL_QEXPR };

typedef lval*(*lbuiltin)(lenv*, lval*);

struct lval {
  int type;

  /* Basic */
  long num;
  char* err;
  char* sym;
  
  /* Function */
  lbuiltin builtin;
  lenv* env;
  lval* formals;
  lval* body;
  
  /* Expression */
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
  va_list va;
  va_start(va, fmt);  
  v->err = malloc(512);  
  vsnprintf(v->err, 511, fmt, va);  
  v->err = realloc(v->err, strlen(v->err)+1);
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

lval* lval_builtin(lbuiltin func) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_FUN;
  v->builtin = func;
  return v;
}

lenv* lenv_new(void);

lval* lval_lambda(lval* formals, lval* body) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_FUN;
  
  /* Set Builtin to Null */
  v->builtin = NULL;
  
  /* Build new environment */
  v->env = lenv_new();
  
  /* Set Formals and Body */
  v->formals = formals;
  v->body = body;
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

void lenv_del(lenv* e);

void lval_del(lval* v) {

  switch (v->type) {
    case LVAL_NUM: break;
    case LVAL_FUN: 
      if (!v->builtin) {
        lenv_del(v->env);
        lval_del(v->formals);
        lval_del(v->body);
      }
    break;
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

lenv* lenv_copy(lenv* e);

lval* lval_copy(lval* v) {
  lval* x = malloc(sizeof(lval));
  x->type = v->type;
  switch (v->type) {
    case LVAL_FUN:
      if (v->builtin) {
        x->builtin = v->builtin;
      } else {
        x->builtin = NULL;
        x->env = lenv_copy(v->env);
        x->formals = lval_copy(v->formals);
        x->body = lval_copy(v->body);
      }
    break;
    case LVAL_NUM: x->num = v->num; break;
    case LVAL_ERR: x->err = malloc(strlen(v->err) + 1);
      strcpy(x->err, v->err);
    break;
    case LVAL_SYM: x->sym = malloc(strlen(v->sym) + 1);
      strcpy(x->sym, v->sym);
    break;
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
  memmove(&v->cell[i],
    &v->cell[i+1], sizeof(lval*) * (v->count-i-1));  
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
    case LVAL_FUN:
      if (v->builtin) {
        printf("<builtin>");
      } else {
        printf("(\\ "); lval_print(v->formals);
        putchar(' '); lval_print(v->body); putchar(')');
      }
    break;
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
  lenv* par;
  int count;
  char** syms;
  lval** vals;
};

lenv* lenv_new(void) {
  lenv* e = malloc(sizeof(lenv));
  e->par = NULL;
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

lenv* lenv_copy(lenv* e) {
  lenv* n = malloc(sizeof(lenv));
  n->par = e->par;
  n->count = e->count;
  n->syms = malloc(sizeof(char*) * n->count);
  n->vals = malloc(sizeof(lval*) * n->count);
  for (int i = 0; i < e->count; i++) {
    n->syms[i] = malloc(strlen(e->syms[i]) + 1);
    strcpy(n->syms[i], e->syms[i]);
    n->vals[i] = lval_copy(e->vals[i]);
  }
  return n;
}

lval* lenv_get(lenv* e, lval* k) {
  
  for (int i = 0; i < e->count; i++) {
    if (strcmp(e->syms[i], k->sym) == 0) {
      return lval_copy(e->vals[i]);
    }
  }
  
  /* If no symbol check in parent otherwise error */
  if (e->par) {
    return lenv_get(e->par, k);
  } else {
    return lval_err("Unbound Symbol '%s'", k->sym);
  }
}

void lenv_put(lenv* e, lval* k, lval* v) {
  
  for (int i = 0; i < e->count; i++) {
    if (strcmp(e->syms[i], k->sym) == 0) {
      lval_del(e->vals[i]);
      e->vals[i] = lval_copy(v);
      return;
    }
  }

  
  e->count++;
  e->vals = realloc(e->vals, sizeof(lval*) * e->count);
  e->syms = realloc(e->syms, sizeof(char*) * e->count);  
  e->vals[e->count-1] = lval_copy(v);
  e->syms[e->count-1] = malloc(strlen(k->sym)+1);
  strcpy(e->syms[e->count-1], k->sym);
}

void lenv_def(lenv* e, lval* k, lval* v) {
  /* Iterate till e has no parent */
  while (e->par) { e = e->par; }
  /* Put value in e */
  lenv_put(e, k, v);
}

/* Builtins */

#define LASSERT(args, cond, fmt, ...) \
  if (!(cond)) { lval* err = lval_err(fmt, ##__VA_ARGS__); lval_del(args); return err; }

#define LASSERT_TYPE(func, args, index, expect) \
  LASSERT(args, args->cell[index]->type == expect, \
    "Function '%s' passed incorrect type for argument %i. " \
    "Got %s, Expected %s.", \
    func, index, ltype_name(args->cell[index]->type), ltype_name(expect))

#define LASSERT_NUM(func, args, num) \
  LASSERT(args, args->count == num, \
    "Function '%s' passed incorrect number of arguments. " \
    "Got %i, Expected %i.", \
    func, args->count, num)

#define LASSERT_NOT_EMPTY(func, args, index) \
  LASSERT(args, args->cell[index]->count != 0, \
    "Function '%s' passed {} for argument %i.", func, index);
    
lval* lval_eval(lenv* e, lval* v);

lval* builtin_lambda(lenv* e, lval* a) {
  /* Check Two arguments, each of which are Q-Expressions */
  LASSERT_NUM("\\", a, 2);
  LASSERT_TYPE("\\", a, 0, LVAL_QEXPR);
  LASSERT_TYPE("\\", a, 1, LVAL_QEXPR);
  
  /* Check first Q-Expression contains only Symbols */
  for (int i = 0; i < a->cell[0]->count; i++) {
    LASSERT(a, (a->cell[0]->cell[i]->type == LVAL_SYM),
      "Cannot define non-symbol. Got %s, Expected %s.",
      ltype_name(a->cell[0]->cell[i]->type),ltype_name(LVAL_SYM));
  }
  
  /* Pop first two arguments and pass them to lval_lambda */
  lval* formals = lval_pop(a, 0);
  lval* body = lval_pop(a, 0);
  lval_del(a);
  
  return lval_lambda(formals, body);
}

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
  
  if ((strcmp(op, "-") == 0) && a->count == 0) { x->num = -x->num; }
  
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

lval* builtin_add(lenv* e, lval* a) { return builtin_op(e, a, "+"); }
lval* builtin_sub(lenv* e, lval* a) { return builtin_op(e, a, "-"); }
lval* builtin_mul(lenv* e, lval* a) { return builtin_op(e, a, "*"); }
lval* builtin_div(lenv* e, lval* a) { return builtin_op(e, a, "/"); }

lval* builtin_var(lenv* e, lval* a, char* func) {
  LASSERT_TYPE(func, a, 0, LVAL_QEXPR);
  
  lval* syms = a->cell[0];
  for (int i = 0; i < syms->count; i++) {
    LASSERT(a, (syms->cell[i]->type == LVAL_SYM),
      "Function '%s' cannot define non-symbol. "
      "Got %s, Expected %s.", func, 
      ltype_name(syms->cell[i]->type), ltype_name(LVAL_SYM));
  }
  
  LASSERT(a, (syms->count == a->count-1),
    "Function '%s' passed too many arguments for symbols. "
    "Got %i, Expected %i.", func, syms->count, a->count-1);
    
  for (int i = 0; i < syms->count; i++) {
    /* If 'def' define in globally. If 'put' define in locally */
    if (strcmp(func, "def") == 0) {
      lenv_def(e, syms->cell[i], a->cell[i+1]);
    }
    
    if (strcmp(func, "=")   == 0) {
      lenv_put(e, syms->cell[i], a->cell[i+1]);
    } 
  }
  
  lval_del(a);
  return lval_sexpr();
}

lval* builtin_def(lenv* e, lval* a) {
  return builtin_var(e, a, "def");
}

lval* builtin_put(lenv* e, lval* a) {
  return builtin_var(e, a, "=");
}

void lenv_add_builtin(lenv* e, char* name, lbuiltin func) {
  lval* k = lval_sym(name);
  lval* v = lval_builtin(func);
  lenv_put(e, k, v);
  lval_del(k); lval_del(v);
}

void lenv_add_builtins(lenv* e) {
  /* Variable Functions */
  lenv_add_builtin(e, "\\",  builtin_lambda); 
  lenv_add_builtin(e, "def", builtin_def);
  lenv_add_builtin(e, "=",   builtin_put);
  
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

lval* lval_call(lenv* e, lval* f, lval* a) {
  
  /* If Builtin then simply apply that */
  if (f->builtin) { return f->builtin(e, a); }
  
  /* Record Argument Counts */
  int given = a->count;
  int total = f->formals->count;
  
  /* While arguments still remain to be processed */
  while (a->count) {
    
    /* If we've ran out of formal arguments to bind */
    if (f->formals->count == 0) {
      lval_del(a);
      return lval_err("Function passed too many arguments. "
        "Got %i, Expected %i.", given, total); 
    }
    
    /* Pop the first symbol from the formals */
    lval* sym = lval_pop(f->formals, 0);
    
    /* Special Case to deal with '&' */
    if (strcmp(sym->sym, "&") == 0) {
      
      /* Ensure '&' is followed by another symbol */
      if (f->formals->count != 1) {
        lval_del(a);
        return lval_err("Function format invalid. "
          "Symbol '&' not followed by single symbol.");
      }
      
      /* Next formal should be bound to remaining arguments */
      lval* nsym = lval_pop(f->formals, 0);
      lenv_put(f->env, nsym, builtin_list(e, a));
      lval_del(sym); lval_del(nsym);
      break;
    }
    
    /* Pop the next argument from the list */
    lval* val = lval_pop(a, 0);
    
    /* Bind a copy into the function's environment */
    lenv_put(f->env, sym, val);
    
    /* Delete symbol and value */
    lval_del(sym); lval_del(val);
  }
  
  /* Argument list is now bound so can be cleaned up */
  lval_del(a);
  
  /* If '&' remains in formal list bind to empty list */
  if (f->formals->count > 0 &&
    strcmp(f->formals->cell[0]->sym, "&") == 0) {
    
    /* Check to ensure that & is not passed invalidly. */
    if (f->formals->count != 2) {
      return lval_err("Function format invalid. "
        "Symbol '&' not followed by single symbol.");
    }
    
    /* Pop and delete '&' symbol */
    lval_del(lval_pop(f->formals, 0));
    
    /* Pop next symbol and create empty list */
    lval* sym = lval_pop(f->formals, 0);
    lval* val = lval_qexpr();
    
    /* Bind to environment and delete */
    lenv_put(f->env, sym, val);
    lval_del(sym); lval_del(val);
  }
  
  /* If all formals have been bound evaluate */
  if (f->formals->count == 0) {
  
    /* Set environment parent to evaluation environment */
    f->env->par = e;
    
    /* Evaluate and return */
    return builtin_eval(f->env, 
      lval_add(lval_sexpr(), lval_copy(f->body)));
  } else {
    /* Otherwise return partially evaluated function */
    return lval_copy(f);
  }
  
}

lval* lval_eval_sexpr(lenv* e, lval* v) {
  
  for (int i = 0; i < v->count; i++) {
    v->cell[i] = lval_eval(e, v->cell[i]);
  }
  
  for (int i = 0; i < v->count; i++) {
    if (v->cell[i]->type == LVAL_ERR) { return lval_take(v, i); }
  }
  
  if (v->count == 0) { return v; }  
  if (v->count == 1) { return lval_eval(e, lval_take(v, 0)); }
  
  lval* f = lval_pop(v, 0);
  if (f->type != LVAL_FUN) {
    lval* err = lval_err(
      "S-Expression starts with incorrect type. "
      "Got %s, Expected %s.",
      ltype_name(f->type), ltype_name(LVAL_FUN));
    lval_del(f); lval_del(v);
    return err;
  }
  
  lval* result = lval_call(e, f, v);
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
      symbol  : /[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/ ;        \
      sexpr  : '(' <expr>* ')' ;                          \
      qexpr  : '{' <expr>* '}' ;                          \
      expr   : <number> | <symbol> | <sexpr> | <qexpr> ;  \
      lispy  : /^/ <expr>* /$/ ;                          \
    ",
    Number, Symbol, Sexpr, Qexpr, Expr, Lispy);
  
  puts("Lispy Version 0.0.0.0.8");
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

## 课后练习

------

- 定义一个 Lisp 函数，它返回列表中的第一个元素。
- 定义一个 Lisp 函数，它返回列表中的第二个元素。
- 定义一个 Lisp 函数，该函数以相反的顺序调用带有两个参数的函数。
- 定义一个 Lisp 函数，它调用一个带参数的函数，然后将结果传递给另一个函数。
- 定义一个 `builtin_fun`函数，相当于 Lisp 中的 `fun`函数。
- 更改变量参数，以便在求值之前必须至少提供一个参数。
