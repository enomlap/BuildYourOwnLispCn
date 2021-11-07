# 字符串 • 第 14 章 

## 库 

------

![细绳](https://www.buildyourownlisp.com/static/img/string.png)

字符串（绳子） • 有多长。 

我们的 Lisp 终于可以运行了。我们应该能够编写几乎任何我们想要的函数。我们可以使用它构建一些非常复杂的结构，甚至可以做一些其他重量级和流行语言都无法完成的很酷的事情； 

每次我们更新我们的程序并再次运行它时，不得不输入我们所有的函数是很烦人的。  在本章中，我们将添加从文件加载代码并运行它的功能。  这将允许我们开始建立一个标准库了。  在此过程中，我们还将添加对代码注释、字符串和打印的支持。 

## 字符串类型 

------

为了让用户加载文件，我们必须让他们提供一个由文件名组成的字符串。  我们的语言支持符号，不过还不支持字符串，字符串可以包含空格和其他字符。  我们需要添加这个 `lval`类型 来指定我们需要的文件名。 

与其他章节一样，我们首先添加一个条目，并使用`lval`来表示类型的数据。 

```c
enum { LVAL_ERR, LVAL_NUM,   LVAL_SYM, LVAL_STR,
       LVAL_FUN, LVAL_SEXPR, LVAL_QEXPR };
/* Basic */
long num;
char* err;
char* sym;
char* str;
```

接下来我们可以添加一个构造字符串型 `lval`的函数，这很类似于我们构造符号的方式。 

```c
lval* lval_str(char* s) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_STR;
  v->str = malloc(strlen(s) + 1);
  strcpy(v->str, s);
  return v;
}
```

把处理 此`lval`类型的相关功能添加到进去. 

 **删除** ... 

```c
case LVAL_STR: free(v->str); break;
```

 **复制** ... 

```c
case LVAL_STR: x->str = malloc(strlen(v->str) + 1);
  strcpy(x->str, v->str); break;
```

判断 **相等** ... 

```c
case LVAL_STR: return (strcmp(x->str, y->str) == 0);
```

处理 **类型名称** ... 

```c
case LVAL_STR: return "String";
```

对于 **打印** ，有些额外的工作。 我们内部存储的字符串与我们要打印的字符串是不一样的。 我们打印出来的可能会和用户输入的不一样，例如使用转义字符`\n`来表示新的一行。 

这样，需要在打印之前对其进行转义。  幸运的是，我们可以利用 `mpc`为我们做这件事的函数。 

在打印功能中，我们添加以下内容... 

```c
case LVAL_STR:   lval_print_str(v); break;
```

位置：... 

```c
void lval_print_str(lval* v) {
  /* Make a Copy of the string */
  char* escaped = malloc(strlen(v->str)+1);
  strcpy(escaped, v->str);
  /* Pass it through the escape function */
  escaped = mpcf_escape(escaped);
  /* Print it between " characters */
  printf("\"%s\"", escaped);
  /* free the copied string */
  free(escaped);
}
```

## 读取字符串 

------

现在我们需要添加对字符串解析的支持。  像往常一样，首先需要添加一个新的语法规则，称为 `string`并将其添加到我们的解析器中。 

我们将要使用的字符串表达规则与 C 语言的相同。  就是说，字符串本质上是两个引号之间的一系列转义字符或普通字符 `""`.  我们可以将其指定为语法字符串中的正则表达式，如下所示。 

```c
string  : /\"(\\\\.|[^\"])*\"/ ;
```

这看起来很复杂，不过分开解释一下你就懂了。 它是这样子的。  一个字符串以一个`"`字符开头并以另一个结束 `"`，里面可以包含零个或多个任何其他字符 `.`。这里反斜杠 `\\`也是可以的，不过如果第一个代表转义（译注：参考C语言的字符串规则）。 

在 `lval_read`还需要添加对应的代码。 

```c
if (strstr(t->tag, "string")) { return lval_read_str(t); }
```

由于输入字符串是以转义形式输入的，所以我们需要创建一个函数 `lval_read_str`处理。  这个函数稍稍棘手，首先它要去掉两头的 `"`。  然后它要对字符串进行转义，转换一系列字符，例如 `\n`到他们的实际编码字符。  最后它创建一个新的 `lval`并清理中间产生的垃圾（译注：小心处理，当心内存泄露）。 

```c
lval* lval_read_str(mpc_ast_t* t) {
  /* Cut off the final quote character */
  t->contents[strlen(t->contents)-1] = '\0';
  /* Copy the string missing out the first quote character */
  char* unescaped = malloc(strlen(t->contents+1)+1);
  strcpy(unescaped, t->contents+1);
  /* Pass through the unescape function */
  unescaped = mpcf_unescape(unescaped);
  /* Construct a new lval using the string */
  lval* str = lval_str(unescaped);
  /* Free the string and return */
  free(unescaped);
  return str;
}
```

如果这一切正常，我们应该能够在提示中使用字符串了。  接下来，我们将添加一些有实际作用函数了。 

```lisp
lispy> "hello"
"hello"
lispy> "hello\n"
"hello\n"
lispy> "hello\""
"hello\""
lispy> head {"hello" "world"}
{"hello"}
lispy> eval (head {"hello" "world"})
"hello"
lispy>
```

## 注释 

任何语言总需要包含注释。就像C语言一样，我们可以使用注释来告知其他人（或我们自己）代码的含义或编写代码的原因。 C语言中， "/*"和"*/"之间的部分被认为是注释。Lisp中，每行以"; "开始的直到行尾的部分是注释。

我曾试图研究一下为什么 Lisps 使用 ";" 来作为注释的开头，但由于历史的原因，根本搞不清楚。我猜想这可能是对C和Java语言的一个小小的调侃：这些语言使用分号来表示语句的结束，而在 Lisp 看来，“你们太小儿科了，这些就像注释一样没用”。

在lisp中，由 `lisp`开头直到行末尾（`\r` 或者 `\n`）的部分都是注释，用正则表达式可以这样表示：
```c
comment : /;[^\\r\\n]*/ ;
```

与字符串一样，我们也需要创建一个新的解析器，并在 `mpca_lang` 中更新。别忘了将解析器添加到"mpc_cleanup"中，并更新第一个整数参数。

最后的语法现在看起来像这样。

```c
mpca_lang(MPCA_LANG_DEFAULT,
  "                                              \
    number  : /-?[0-9]+/ ;                       \
    symbol  : /[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/ ; \
    string  : /\"(\\\\.|[^\"])*\"/ ;             \
    comment : /;[^\\r\\n]*/ ;                    \
    sexpr   : '(' <expr>* ')' ;                  \
    qexpr   : '{' <expr>* '}' ;                  \
    expr    : <number>  | <symbol> | <string>    \
            | <comment> | <sexpr>  | <qexpr>;    \
    lispy   : /^/ <expr>* /$/ ;                  \
  ",
  Number, Symbol, String, Comment, Sexpr, Qexpr, Expr, Lispy);
```
清理函数：
```c
mpc_cleanup(8,
  Number, Symbol, String, Comment,
  Sexpr,  Qexpr,  Expr,   Lispy);
```

由于注释仅仅是供程序员阅读的，所以内部实现的方法仅仅是 `忽略` 而已。类似于中括号和圆括号，我们可以添加如下的语句来处理， `lval_read`.... 
```c
if (strstr(t->children[i]->tag, "comment")) { continue; }
```
注释在交互式提示上没有多大用处，但在代码文件中进行注释非常必要。 

## 加载函数 

我们想要构建一个函数，当传递文件名称的字符串时，该函数可以加载和并对文件内容求值。为了实现这个函数，我们需要使用我们的语法来实现读取文件、解析和求值。加载函数将依赖于我们的 Lispy可调用元素`mpc_parser*`. 

因此，和函数一样，需要前置声明解析器指针，并将它们放在文件的开头部分。 

```c
mpc_parser_t* Number;
mpc_parser_t* Symbol;
mpc_parser_t* String;
mpc_parser_t* Comment;
mpc_parser_t* Sexpr;
mpc_parser_t* Qexpr;
mpc_parser_t* Expr;
mpc_parser_t* Lispy;
```

`load`函数将就像任何其他内置函数一样，首先确认输入参数是否是一个字符串。  然后使用 `mpc_parse_contents`使用语法树解析 文件内容。和 `mpc_parse`类似，将文件的内容解析为`mpc_result`对象，即 *抽象语法树* 或者返回 *错误* 。 

与命令提示符略有不同，在成功解析文件后，我们不应将其视为单个表达式。在输入文件时，我们让用户列出多个表达式并分别对所有表达式求值。  这就需要遍历文件内容中的每个表达式并一一求值。  如果没有任何错误，我们就打印错结果后继续。 

如果解析错误，就把消息放入 `lval` 型的错误报告并返回。没有错误的话这个内置函数就返回一个空表达式。完整代码如下 ... 

```c
lval* builtin_load(lenv* e, lval* a) {
  LASSERT_NUM("load", a, 1);
  LASSERT_TYPE("load", a, 0, LVAL_STR);

  /* Parse File given by string name */
  mpc_result_t r;
  if (mpc_parse_contents(a->cell[0]->str, Lispy, &r)) {

    /* Read contents */
    lval* expr = lval_read(r.output);
    mpc_ast_delete(r.output);

    /* Evaluate each Expression */
    while (expr->count) {
      lval* x = lval_eval(e, lval_pop(expr, 0));
      /* If Evaluation leads to error print it */
      if (x->type == LVAL_ERR) { lval_println(x); }
      lval_del(x);
    }

    /* Delete expressions and arguments */
    lval_del(expr);
    lval_del(a);

    /* Return empty list */
    return lval_sexpr();

  } else {
    /* Get Parse Error as String */
    char* err_msg = mpc_err_string(r.error);
    mpc_err_delete(r.error);

    /* Create new error message using it */
    lval* err = lval_err("Could not load Library %s", err_msg);
    free(err_msg);
    lval_del(a);

    /* Cleanup and return error */
    return err;
  }
}
```

## 命令行参数 

有了加载文件的能力，我们就可以借此机会添加一些编程语言都有的典型功能。当文件名作为参数提供给命令行时，我们可以尝试运行这些文件。例如运行一个 python 文件，一个人可能会写 `python filename.py`. 

这些命令行参数由 `main`函数的 `argc`和 `argv`变量访问。`argc`变量给出参数的数量，和 `argv` 指向参数字符串数组。这 `argc` 至少为1，因为第一个参数永远是调用的命令本身。 

也就是说，如果 `argc`是`1`，就仅仅调用解释器给出一个提示符，否则的话，通过 `builtin_load`函数执行脚本文件（译注：这语言越加越像个真的语言了）。 

```c
/* Supplied with list of files */
if (argc >= 2) {

  /* loop over each supplied filename (starting from 1) */
  for (int i = 1; i < argc; i++) {

    /* Argument list with a single argument, the filename */
    lval* args = lval_add(lval_sexpr(), lval_str(argv[i]));

    /* Pass to builtin load and get the result */
    lval* x = builtin_load(e, args);

    /* If the result is an error be sure to print it */
    if (x->type == LVAL_ERR) { lval_println(x); }
    lval_del(x);
  }
}
```

例如编写一些程序，并存为文件，使用此方法调用它。 

```
lispy example.lspy
```

## 打印功能 

如果我们从命令行运行程序，我们可能希望它们输出一些数据，而不仅仅是定义函数和其他值。利用我们现有的 `lval_print` 功能我们可以在我们的 Lisp添加一个 `print`函数。 

此函数打印由空格分隔的每个参数，然后打印一个换行符以结束。返回值是个空表达式。 

```c
lval* builtin_print(lenv* e, lval* a) {

  /* Print each argument followed by a space */
  for (int i = 0; i < a->count; i++) {
    lval_print(a->cell[i]); putchar(' ');
  }

  /* Print a newline and delete arguments */
  putchar('\n');
  lval_del(a);

  return lval_sexpr();
}
```

## 错误函数 

可以利用字符串来为错误报告添加功能。它接受一个字符串作为输入，并将其作为错误消息提供给 `lval_err`. 

```c
lval* builtin_error(lenv* e, lval* a) {
  LASSERT_NUM("error", a, 1);
  LASSERT_TYPE("error", a, 0, LVAL_STR);

  /* Construct Error from first argument */
  lval* err = lval_err(a->cell[0]->str);

  /* Delete arguments and return */
  lval_del(a);
  return err;
}
```

最后一步是将这些注册为内置函数。现在可以开始构建库并将它们写入文件了。 

```c
/* String Functions */
lenv_add_builtin(e, "load",  builtin_load);
lenv_add_builtin(e, "error", builtin_error);
lenv_add_builtin(e, "print", builtin_print);
```
```lisp
lispy> print "Hello World!"
"Hello World!"
()
lispy> error "This is an error"
Error: This is an error
lispy> load "hello.lspy"
"Hello World!"
()
lispy>
```

## 总结

这是最后一章了，我们使用C语言一行一行的实现了一个自己的 Lisp 。 

、总行数大约有1000 行。编写这么多代码绝非易事。如果您已经做到了这一点，那么您就已经编写了一个真正的程序并完成了一个不错的项目。你在这里学到的技能当然可以用于其它工作，让你有信心去寻找自己的目标和目标。您已经拥有了一个复杂而漂亮的程序，您可以与之互动和玩耍。这是你应该引以为豪的事情。去向你的朋友和家人炫耀吧！ 

下一章，我们将用 Lisp 来构建一些常用函数的标准库。然后，我还会提示你一些这个语言需要改进的地方和之后的发展方向。虽然再往后我不会再参与了，但这真的只是开始。感谢您的关注，祝您将来编写的任何 C 语言都好运！ 

## 参考 

------

####         [           字符串.c          ](https://www.buildyourownlisp.com/chapter14_strings#collapseOne) 
完成的string.c版本：     
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

/* Parser Declariations */

mpc_parser_t* Number; 
mpc_parser_t* Symbol; 
mpc_parser_t* String; 
mpc_parser_t* Comment;
mpc_parser_t* Sexpr;  
mpc_parser_t* Qexpr;  
mpc_parser_t* Expr; 
mpc_parser_t* Lispy;

/* Forward Declarations */

struct lval;
struct lenv;
typedef struct lval lval;
typedef struct lenv lenv;

/* Lisp Value */

enum { LVAL_ERR, LVAL_NUM,   LVAL_SYM, LVAL_STR, 
       LVAL_FUN, LVAL_SEXPR, LVAL_QEXPR };
       
typedef lval*(*lbuiltin)(lenv*, lval*);

struct lval {
  int type;

  /* Basic */
  long num;
  char* err;
  char* sym;
  char* str;
  
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

lval* lval_str(char* s) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_STR;
  v->str = malloc(strlen(s) + 1);
  strcpy(v->str, s);
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
  v->builtin = NULL;  
  v->env = lenv_new();  
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
    case LVAL_STR: free(v->str); break;
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
    case LVAL_STR: x->str = malloc(strlen(v->str) + 1);
      strcpy(x->str, v->str);
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

void lval_print_str(lval* v) {
  /* Make a Copy of the string */
  char* escaped = malloc(strlen(v->str)+1);
  strcpy(escaped, v->str);
  /* Pass it through the escape function */
  escaped = mpcf_escape(escaped);
  /* Print it between " characters */
  printf("\"%s\"", escaped);
  /* free the copied string */
  free(escaped);
}

void lval_print(lval* v) {
  switch (v->type) {
    case LVAL_FUN:
      if (v->builtin) {
        printf("<builtin>");
      } else {
        printf("(\\ ");
        lval_print(v->formals);
        putchar(' ');
        lval_print(v->body);
        putchar(')');
      }
    break;
    case LVAL_NUM:   printf("%li", v->num); break;
    case LVAL_ERR:   printf("Error: %s", v->err); break;
    case LVAL_SYM:   printf("%s", v->sym); break;
    case LVAL_STR:   lval_print_str(v); break;
    case LVAL_SEXPR: lval_print_expr(v, '(', ')'); break;
    case LVAL_QEXPR: lval_print_expr(v, '{', '}'); break;
  }
}

void lval_println(lval* v) { lval_print(v); putchar('\n'); }

int lval_eq(lval* x, lval* y) {
  
  if (x->type != y->type) { return 0; }
  
  switch (x->type) {
    case LVAL_NUM: return (x->num == y->num);    
    case LVAL_ERR: return (strcmp(x->err, y->err) == 0);
    case LVAL_SYM: return (strcmp(x->sym, y->sym) == 0);    
    case LVAL_STR: return (strcmp(x->str, y->str) == 0);    
    case LVAL_FUN: 
      if (x->builtin || y->builtin) {
        return x->builtin == y->builtin;
      } else {
        return lval_eq(x->formals, y->formals) && lval_eq(x->body, y->body);
      }    
    case LVAL_QEXPR:
    case LVAL_SEXPR:
      if (x->count != y->count) { return 0; }
      for (int i = 0; i < x->count; i++) {
        if (!lval_eq(x->cell[i], y->cell[i])) { return 0; }
      }
      return 1;
    break;
  }
  return 0;
}

char* ltype_name(int t) {
  switch(t) {
    case LVAL_FUN: return "Function";
    case LVAL_NUM: return "Number";
    case LVAL_ERR: return "Error";
    case LVAL_SYM: return "Symbol";
    case LVAL_STR: return "String";
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
    if (strcmp(e->syms[i], k->sym) == 0) { return lval_copy(e->vals[i]); }
  }
  
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
  while (e->par) { e = e->par; }
  lenv_put(e, k, v);
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

lval* builtin_lambda(lenv* e, lval* a) {
  LASSERT_NUM("\\", a, 2);
  LASSERT_TYPE("\\", a, 0, LVAL_QEXPR);
  LASSERT_TYPE("\\", a, 1, LVAL_QEXPR);
  
  for (int i = 0; i < a->cell[0]->count; i++) {
    LASSERT(a, (a->cell[0]->cell[i]->type == LVAL_SYM),
      "Cannot define non-symbol. Got %s, Expected %s.",
      ltype_name(a->cell[0]->cell[i]->type), ltype_name(LVAL_SYM));
  }
  
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
      "Got %s, Expected %s.",
      func, ltype_name(syms->cell[i]->type), ltype_name(LVAL_SYM));
  }
  
  LASSERT(a, (syms->count == a->count-1),
    "Function '%s' passed too many arguments for symbols. "
    "Got %i, Expected %i.",
    func, syms->count, a->count-1);
    
  for (int i = 0; i < syms->count; i++) {
    if (strcmp(func, "def") == 0) { lenv_def(e, syms->cell[i], a->cell[i+1]); }
    if (strcmp(func, "=")   == 0) { lenv_put(e, syms->cell[i], a->cell[i+1]); } 
  }
  
  lval_del(a);
  return lval_sexpr();
}

lval* builtin_def(lenv* e, lval* a) { return builtin_var(e, a, "def"); }
lval* builtin_put(lenv* e, lval* a) { return builtin_var(e, a, "="); }

lval* builtin_ord(lenv* e, lval* a, char* op) {
  LASSERT_NUM(op, a, 2);
  LASSERT_TYPE(op, a, 0, LVAL_NUM);
  LASSERT_TYPE(op, a, 1, LVAL_NUM);
  
  int r;
  if (strcmp(op, ">")  == 0) { r = (a->cell[0]->num >  a->cell[1]->num); }
  if (strcmp(op, "<")  == 0) { r = (a->cell[0]->num <  a->cell[1]->num); }
  if (strcmp(op, ">=") == 0) { r = (a->cell[0]->num >= a->cell[1]->num); }
  if (strcmp(op, "<=") == 0) { r = (a->cell[0]->num <= a->cell[1]->num); }
  lval_del(a);
  return lval_num(r);
}

lval* builtin_gt(lenv* e, lval* a) { return builtin_ord(e, a, ">");  }
lval* builtin_lt(lenv* e, lval* a) { return builtin_ord(e, a, "<");  }
lval* builtin_ge(lenv* e, lval* a) { return builtin_ord(e, a, ">="); }
lval* builtin_le(lenv* e, lval* a) { return builtin_ord(e, a, "<="); }

lval* builtin_cmp(lenv* e, lval* a, char* op) {
  LASSERT_NUM(op, a, 2);
  int r;
  if (strcmp(op, "==") == 0) { r =  lval_eq(a->cell[0], a->cell[1]); }
  if (strcmp(op, "!=") == 0) { r = !lval_eq(a->cell[0], a->cell[1]); }
  lval_del(a);
  return lval_num(r);
}

lval* builtin_eq(lenv* e, lval* a) { return builtin_cmp(e, a, "=="); }
lval* builtin_ne(lenv* e, lval* a) { return builtin_cmp(e, a, "!="); }

lval* builtin_if(lenv* e, lval* a) {
  LASSERT_NUM("if", a, 3);
  LASSERT_TYPE("if", a, 0, LVAL_NUM);
  LASSERT_TYPE("if", a, 1, LVAL_QEXPR);
  LASSERT_TYPE("if", a, 2, LVAL_QEXPR);
  
  lval* x;
  a->cell[1]->type = LVAL_SEXPR;
  a->cell[2]->type = LVAL_SEXPR;
  
  if (a->cell[0]->num) {
    x = lval_eval(e, lval_pop(a, 1));
  } else {
    x = lval_eval(e, lval_pop(a, 2));
  }
  
  lval_del(a);
  return x;
}

lval* lval_read(mpc_ast_t* t);

lval* builtin_load(lenv* e, lval* a) {
  LASSERT_NUM("load", a, 1);
  LASSERT_TYPE("load", a, 0, LVAL_STR);
  
  /* Parse File given by string name */
  mpc_result_t r;
  if (mpc_parse_contents(a->cell[0]->str, Lispy, &r)) {
    
    /* Read contents */
    lval* expr = lval_read(r.output);
    mpc_ast_delete(r.output);

    /* Evaluate each Expression */
    while (expr->count) {
      lval* x = lval_eval(e, lval_pop(expr, 0));
      /* If Evaluation leads to error print it */
      if (x->type == LVAL_ERR) { lval_println(x); }
      lval_del(x);
    }
    
    /* Delete expressions and arguments */
    lval_del(expr);    
    lval_del(a);
    
    /* Return empty list */
    return lval_sexpr();
    
  } else {
    /* Get Parse Error as String */
    char* err_msg = mpc_err_string(r.error);
    mpc_err_delete(r.error);
    
    /* Create new error message using it */
    lval* err = lval_err("Could not load Library %s", err_msg);
    free(err_msg);
    lval_del(a);
    
    /* Cleanup and return error */
    return err;
  }
}

lval* builtin_print(lenv* e, lval* a) {
  
  /* Print each argument followed by a space */
  for (int i = 0; i < a->count; i++) {
    lval_print(a->cell[i]); putchar(' ');
  }
  
  /* Print a newline and delete arguments */
  putchar('\n');
  lval_del(a);
  
  return lval_sexpr();
}

lval* builtin_error(lenv* e, lval* a) {
  LASSERT_NUM("error", a, 1);
  LASSERT_TYPE("error", a, 0, LVAL_STR);
  
  /* Construct Error from first argument */
  lval* err = lval_err(a->cell[0]->str);
  
  /* Delete arguments and return */
  lval_del(a);
  return err;
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
  
  /* Comparison Functions */
  lenv_add_builtin(e, "if", builtin_if);
  lenv_add_builtin(e, "==", builtin_eq);
  lenv_add_builtin(e, "!=", builtin_ne);
  lenv_add_builtin(e, ">",  builtin_gt);
  lenv_add_builtin(e, "<",  builtin_lt);
  lenv_add_builtin(e, ">=", builtin_ge);
  lenv_add_builtin(e, "<=", builtin_le);
  
  /* String Functions */
  lenv_add_builtin(e, "load",  builtin_load); 
  lenv_add_builtin(e, "error", builtin_error);
  lenv_add_builtin(e, "print", builtin_print);
}

/* Evaluation */

lval* lval_call(lenv* e, lval* f, lval* a) {
  
  if (f->builtin) { return f->builtin(e, a); }
  
  int given = a->count;
  int total = f->formals->count;
  
  while (a->count) {
    
    if (f->formals->count == 0) {
      lval_del(a);
      return lval_err("Function passed too many arguments. "
        "Got %i, Expected %i.", given, total); 
    }
    
    lval* sym = lval_pop(f->formals, 0);
    
    if (strcmp(sym->sym, "&") == 0) {
      
      if (f->formals->count != 1) {
        lval_del(a);
        return lval_err("Function format invalid. "
          "Symbol '&' not followed by single symbol.");
      }
      
      lval* nsym = lval_pop(f->formals, 0);
      lenv_put(f->env, nsym, builtin_list(e, a));
      lval_del(sym); lval_del(nsym);
      break;
    }
    
    lval* val = lval_pop(a, 0);    
    lenv_put(f->env, sym, val);    
    lval_del(sym); lval_del(val);
  }
  
  lval_del(a);
  
  if (f->formals->count > 0 &&
    strcmp(f->formals->cell[0]->sym, "&") == 0) {
    
    if (f->formals->count != 2) {
      return lval_err("Function format invalid. "
        "Symbol '&' not followed by single symbol.");
    }
    
    lval_del(lval_pop(f->formals, 0));
    
    lval* sym = lval_pop(f->formals, 0);
    lval* val = lval_qexpr();    
    lenv_put(f->env, sym, val);
    lval_del(sym); lval_del(val);
  }
  
  if (f->formals->count == 0) {  
    f->env->par = e;    
    return builtin_eval(f->env, lval_add(lval_sexpr(), lval_copy(f->body)));
  } else {
    return lval_copy(f);
  }
  
}

lval* lval_eval_sexpr(lenv* e, lval* v) {
  
  for (int i = 0; i < v->count; i++) { v->cell[i] = lval_eval(e, v->cell[i]); }
  for (int i = 0; i < v->count; i++) { if (v->cell[i]->type == LVAL_ERR) { return lval_take(v, i); } }
  
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

lval* lval_read_str(mpc_ast_t* t) {
  /* Cut off the final quote character */
  t->contents[strlen(t->contents)-1] = '\0';
  /* Copy the string missing out the first quote character */
  char* unescaped = malloc(strlen(t->contents+1)+1);
  strcpy(unescaped, t->contents+1);
  /* Pass through the unescape function */
  unescaped = mpcf_unescape(unescaped);
  /* Construct a new lval using the string */
  lval* str = lval_str(unescaped);
  /* Free the string and return */
  free(unescaped);
  return str;
}

lval* lval_read(mpc_ast_t* t) {
  
  if (strstr(t->tag, "number")) { return lval_read_num(t); }
  if (strstr(t->tag, "string")) { return lval_read_str(t); }
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
    if (strstr(t->children[i]->tag, "comment")) { continue; }
    x = lval_add(x, lval_read(t->children[i]));
  }
  
  return x;
}

/* Main */

int main(int argc, char** argv) {
  
  Number  = mpc_new("number");
  Symbol  = mpc_new("symbol");
  String  = mpc_new("string");
  Comment = mpc_new("comment");
  Sexpr   = mpc_new("sexpr");
  Qexpr   = mpc_new("qexpr");
  Expr    = mpc_new("expr");
  Lispy   = mpc_new("lispy");
  
  mpca_lang(MPCA_LANG_DEFAULT,
    "                                              \
      number  : /-?[0-9]+/ ;                       \
      symbol  : /[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/ ; \
      string  : /\"(\\\\.|[^\"])*\"/ ;             \
      comment : /;[^\\r\\n]*/ ;                    \
      sexpr   : '(' <expr>* ')' ;                  \
      qexpr   : '{' <expr>* '}' ;                  \
      expr    : <number>  | <symbol> | <string>    \
              | <comment> | <sexpr>  | <qexpr>;    \
      lispy   : /^/ <expr>* /$/ ;                  \
    ",
    Number, Symbol, String, Comment, Sexpr, Qexpr, Expr, Lispy);
  
  lenv* e = lenv_new();
  lenv_add_builtins(e);
  
  /* Interactive Prompt */
  if (argc == 1) {
  
    puts("Lispy Version 0.0.0.1.0");
    puts("Press Ctrl+c to Exit\n");
  
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
  }
  
  /* Supplied with list of files */
  if (argc >= 2) {
  
    /* loop over each supplied filename (starting from 1) */
    for (int i = 1; i < argc; i++) {
      
      /* Argument list with a single argument, the filename */
      lval* args = lval_add(lval_sexpr(), lval_str(argv[i]));
      
      /* Pass to builtin load and get the result */
      lval* x = builtin_load(e, args);
      
      /* If the result is an error be sure to print it */
      if (x->type == LVAL_ERR) { lval_println(x); }
      lval_del(x);
    }
  }
  
  lenv_del(e);
  
  mpc_cleanup(8, 
    Number, Symbol, String, Comment, 
    Sexpr,  Qexpr,  Expr,   Lispy);
  
  return 0;
}
```
## 课后作业

------

- 修改内置函数 `join` 让它能够处理字符串。 
- 修改内置函数 `head` 让它能够处理字符串。 
- 修改内置函数 `tail` 让它能够处理字符串。 
- 创建一个内置函数 `read` 读入并将字符串转换为 Q 表达式。 
- 创建一个内置函数 `show` 可以按原样打印字符串的内容（未转义）。 
- 创造特殊值 `ok` 返回以替代空表达式 `()`. 
- 添加函数来包装所有 C 的文件处理函数，例如 `fopen` 和 `fgets`。 
