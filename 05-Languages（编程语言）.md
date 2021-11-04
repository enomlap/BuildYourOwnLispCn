# 第零五章 • 编程语言

## 什么是编程语言？

编程语言和自然语言非常相似，也有它背后固有的结构和规则来界定语句的正确性。当我们读写自然语言时，语言的规则就在无意中学会了。学习编程语言也是一样，需要长久的读写练习才能掌握。一旦掌握，我们就可以利用这些规则去理解其他人的代码，并写出自己的代码。

在 19 世纪 50 年代，语言学家 Noam Chomsky 定义了一系列关于语言的[重要理论](http://en.wikipedia.org/wiki/Chomsky_hierarchy)。这些理论支撑了我们今天对于语言结构的基本理解。其中重要的一条结论就是：自然语言都是建立在递归和重复的子结构之上的。

举例来说：

> `The cat walked on the carpet.`

根据英语的规则，名词 `cat` 可以被两个由 `and` 连接的名词代替：

> `The cat and dog walked on the carpet.`

我们可以像之前一样再次使用这个规则，将 `cat` 替换为两个使用 `and` 符号连接的新名词。我们还可以使用另外一个规则，将一个名词替换为一个形容词加一个名词，其中形容词作为对名词的修饰：

> `The cat and mouse and dog walked on the carpet.`

> `The white cat and black dog walked on the carpet.`

以上，我们只是简单的举两个例子。英语的语法规则远不止于此，汉语的语法规则就更复杂了，呵呵。

我们注意到，在编程语言中也有相似的规则。在 C 语言中，`if` 语句可以包含多条的新语句，新语句当然也可以是另一个 `if`语句。这些递归和重复的规则在语言的其他部分也同样是适用的。

> `if (x > 5) { return x; }`

> `if (x > 5) { if (x > 10) { return x; } }`

Chomsky 提出的理论是非常重要的。它意味着，虽然一门语言可以表达无限的内容，我们仍然可以使用有限的规则去解析所有用该门语言写就的东西。这些有限的规则就叫语法(grammar)。

对于语法，我们有多种表达方式。最容易想到的方式就是使用白话文。譬如，我们可以这样说：*"句子必须是动词短语"*、*"动词词组可以是动词，也可以是副词加动词"* 等等 ( 译注：事实上，这也是我们最初学习英语语法的主要方式 )。这种形式对于人类来说是非常容易理解的，但是对于计算机却太模糊的、难以理解的。所以在写程序时，我们需要对语法有一个更标准化的描述。

为了定义一门编程语言(例如我们将要编写的 `Lisp`)，我们首先需要能够正确解析用户按照语法规则写就的程序。为此需要编写一个*语法解析器*，用来判断用户的输入是否合法，并产生解析后的内部表示。内部表示是一种计算机更容易理解的表示形式，有了它，我们后面的解析、求值等工作会变得更加的简单可行。

但是这一部分往往是枯燥繁琐的体力活，我们显然不希望在这上面浪费时间。所以我们就采用了一个叫做 `mpc` 的库来帮助我们完成工作。

## 解析器组合子

`mpc` 是我(原作者)编写的一个解析器组合子(Parser Combinators)库。这意味着，你可以使用这个库为任何语言编写语法解析器。编写语法解析器的方法有很多，使用解析器组合子的好处就在于，它极大地简化了原本枯燥无聊的工作，而仅仅编写高层的抽象语法规则就可以了。

<!--

TODO:

Many Parser Combinator libraries actually work by letting you write normal code that looks a bit like a grammar, not by actually specifying a grammar directly. In many situations this is fine, but sometimes it can get clunky and complicated. Luckily for us mpc allows us to write normal code that just looks like a grammar, or we can use special notation to write a grammar directly!

-->

## 编写语法规则

下面我们来编写一个柴犬语( [Doge](http://knowyourmeme.com/memes/doge) )的语法解析器以便熟悉 `mpc` 的用法。


先来看一下 `Doge` 语言的语法描述：

- 形容词 (`Adjective`) 包括 `wow`、`many`、`so`、`such` 符号。
- 名词 (`Noun`) 包括 `lisp`、`language`、`c`、`book`、`build` 符号。
- 短语 (`Phrase`) 由形容词 (`Adjective`) 后接名词 (`Noun`) 组成。
- `Doge` 语言由 0 到多个 短语(`Phrase`) 组成。

现在我们尝试定义一下形容词和名词，为此我们创建两个解析器，类型是 `mpc_parser_t*`，然后将解析器存储在 `Adjective` 和 `Noun` 两个变量中。`mpc_or` 函数产生一个解析器，它可接受的语句必须是指定语句中的一个。而 `mpc_sym` 将字符串转化为一个语句。

下面的代码也正如我们上面的描述一样：

```c
/* Build a parser 'Adjective' to recognize descriptions */
mpc_parser_t* Adjective = mpc_or(4, 
  mpc_sym("wow"), mpc_sym("many"),
  mpc_sym("so"),  mpc_sym("such")
);

/* Build a parser 'Noun' to recognize things */
mpc_parser_t* Noun = mpc_or(5,
  mpc_sym("lisp"), mpc_sym("language"),
  mpc_sym("book"),mpc_sym("build"), 
  mpc_sym("c")
);
```

> #### 我怎样才能使用上面的这些 `mpc` 库提供的函数？

> 现在先不用担心编译和运行程序的事情，先确保理解背后的理论知识。在下一章中我们将使用使用`mpc` 实现一个更加接近我们的 Lisp 的语言。

接下来，我们使用已经定义好的解析器 `Adjective` 、 `Noun` 来定义短语(`Phrase`)解析器。`mpc_and` 函数返回的解析器可接受的语句必须是各个语句按照顺序出现。所以我们将先前定义的 `Adjective` 和 `Noun` 传递给它，表示形容词后面紧跟名词组成短语。`mpcf_strfold` 和 `free` 指定了各个语句的组织及删除方式，我们可以暂时忽略它们。

```c
mpc_parser_t* Phrase = mpc_and(2, mpcf_strfold, Adjective, Noun, free);
```

Doge 语言是由 0 到多个短语(Phrase) 组成的。`mpc_many` 函数表达的正是这种逻辑关系。同样的，我们可以暂时忽略 `mpcf_strfold` 参数。

```c
mpc_parser_t* Doge = mpc_many(mpcf_strfold, Phrase);
```

上述语句表明 Doge 可以接受任意多条语句。这也意味着 Doge 语言是无穷的。下面列出了一些符合 Doge 语法的例子：

```
"wow book such language many lisp"
"so c such build such language"
"many build wow c"
""
"wow lisp wow c many language"
"so c"
```

我们可以继续使用 `mpc` 提供的其他函数，一步一步地编写能解析更加复杂的语法的解析器。相应地，随着复杂度的增加，代码的可读性也会越来越差。所以，这种写法其实并不简单。`mpc` 还提供了一系列的帮助函数来帮助用户更加简单地完成常见的任务，具体的文档说明可以参见[项目主页](http://github.com/orangeduck/mpc)。使用这些函数能够更好更快地构建复杂语言的解析器，并能够提供更加精细地控制。

## 更加自然的语法规则

`mpc` 允许我们使用一种更加自然的方式来编写语法规则。我们可以将整个语言的语法规则写在一个长字符串中，而不是使用啰嗦难懂的 C 语句。我们也不再需要关心如何 使用 `mpcf_strfold` 或是 `free` 参数组织或删除各个语句。所有的这些工作都是都是自动完成的。

下面，我们使用这个方法重新编写了上面实现过的 Doge 语言：

```c
mpc_parser_t* Adjective = mpc_new("adjective");
mpc_parser_t* Noun      = mpc_new("noun");
mpc_parser_t* Phrase    = mpc_new("phrase");
mpc_parser_t* Doge      = mpc_new("doge");

mpca_lang(MPCA_LANG_DEFAULT,
  "                                           \
    adjective : \"wow\" | \"many\"            \
              |  \"so\" | \"such\";           \
    noun      : \"lisp\" | \"language\"       \
              | \"book\" | \"build\" | \"c\"; \
    phrase    : <adjective> <noun>;           \
    doge      : <phrase>*;                    \
  ",
  Adjective, Noun, Phrase, Doge);

/* Do some parsing here... */

mpc_cleanup(4, Adjective, Noun, Phrase, Doge);
```

即使你现在暂时不理解上面的长字符串的语法规则，也能明显地感觉到这个方法要比之前的清晰的多。下面就来具体的学习一下其中的某些特殊符号的意义及用法。

注意到，现在定义一个语法规则分为两个步骤：
1. 使用 `mpc_new` 函数定义语法规则的名字。
2. 使用 `mpca_lang` 函数具体定义这些语法规则。

`mpca_lang` 函数的第一个参数是操作标记，在这里我们使用默认选项。第二个参数是 C 语言的一个长字符串。这个字符串中定义了具体的语法。它包含一系列的递归规则。每个规则分为两部分，用冒号 `:` 隔开，冒号左边是规则的名字，右边是规则的定义，使用 `;` 表示规则结束。

定义语法规则的一些特殊符号的作用如下：

| 语法表示 | 作用 |
|------------|-------------------------|
| `"ab"` | 要求字符串 `ab` |
| `'a'` | 要求字符 `a` |
| `'a' 'b'` | 要求先有一个字符 `a`，后面紧跟一个字符 `b` |
| `'a'｜'b'` | 要求有字符 `a` 或字符 `b` |
| `'a'*` | 要求有 0 个或多个字符 `a` |
| `'a'+` | 要求有 1 个或多个字符 `a` |
| `<abba>` | 要求满足名为 `abba` 定义的语法规则 |

> #### 似曾相识的感觉？

> 上面的一些语法规则有没有似曾相识的感觉？你没有猜错，`mpca_lang` 函数就是用 `mpc_many`、`mpc_and` 、 `mpc_or` 这些函数来实现的，干净利落，不拖泥带水。

根据表中给出的规则尝试着理解一下上面的代码，看看是不是等价于之前我们前面用代码定义过的语法解析器？

在后面的章节中，我们会使用这个方法来定义我们的语法规则。刚开始可能并不是那么容易理解，但随着时间的推移，我们练习的越来越多，你也将会更加熟悉，并将知道如何创建和编辑自己的语法规则。

本章更加注重的是理论知识，所以在做彩蛋部分时，不要太在意正确性，思考实现的方式才是最重要的。

## 参考

`doge_code.c`

```c
#include "mpc.h"

int main(int argc, char** argv) {

  /* Build a parser 'Adjective' to recognize descriptions */
  mpc_parser_t* Adjective = mpc_or(4, 
    mpc_sym("wow"), mpc_sym("many"),
    mpc_sym("so"),  mpc_sym("such")
  );

  /* Build a parser 'Noun' to recognize things */
  mpc_parser_t* Noun = mpc_or(5,
    mpc_sym("lisp"), mpc_sym("language"),
    mpc_sym("book"), mpc_sym("build"), 
    mpc_sym("c")
  );
  
  mpc_parser_t* Phrase = mpc_and(2, mpcf_strfold, 
    Adjective, Noun, free);
  
  mpc_parser_t* Doge = mpc_many(mpcf_strfold, Phrase);

  /* Do some parsing here... */
  
  mpc_delete(Doge);
  
  return 0;
  
}
```

`doge_grammar.c`

```c
#include "mpc.h"

int main(int argc, char** argv) {

  mpc_parser_t* Adjective = mpc_new("adjective");
  mpc_parser_t* Noun      = mpc_new("noun");
  mpc_parser_t* Phrase    = mpc_new("phrase");
  mpc_parser_t* Doge      = mpc_new("doge");

  mpca_lang(MPCA_LANG_DEFAULT,
    "                                           \
      adjective : \"wow\" | \"many\"            \
                |  \"so\" | \"such\";           \
      noun      : \"lisp\" | \"language\"       \
                | \"book\" | \"build\" | \"c\"; \
      phrase    : <adjective> <noun>;           \
      doge      : <phrase>*;                    \
    ",
    Adjective, Noun, Phrase, Doge);

  /* Do some parsing here... */

  mpc_cleanup(4, Adjective, Noun, Phrase, Doge);
  
  return 0;
  
}
```

## 彩蛋

- 为 Doge 语言的形容词和名词添加更多的字符串。
- 为什么在定义语法规则的字符串中，`"` 前面要加 `\` 符号？
- 为什么在定义语法规则的字符串中的每行的结尾要加 `\` 符号？
- 描述诸如 `0.01`、`52.221` 这种小数的语法规则。
- 描述诸如`https://github.com/ksco` 这种 URL 的语法规则。
- 尝试描述一些简单英语句子的语法规则，例如：`the cat sat on the mat`。
- 使用更加正式的语言来描述前面三个问题。如 `|`、`*`、`+` 等符号语言。
- 如果你熟悉 JSON，请尝试描述一下它的语法。
