# 什么是编程语言？

编程语言与真实语言非常相似。它背后有一个结构，一些规则规定什么是有效的，什么不是。我们在阅读和书写自然语言时，都在不自觉地学习这些规则，编程语言也是如此。我们可以利用这些规则来理解他人，并生成我们自己的语音或代码。

作为这方面的一个例子，我们可以检查这个短语。

    The cat walked on the carpet.

使用英语的规则，名词cat可以被两个用 隔开的名词代替and。

    The cat and dog walked on the carpet.

这些新名词中的每一个都可以依次被再次替换。我们可以使用与以前相同的规则，并替换cat为两个与 连接的新名词and。或者我们可以使用不同的规则，将每个名词替换为形容词和名词，以添加对它们的描述。

    The cat and mouse and dog walked on the carpet.

    The white cat and black dog walked on the carpet.

这只是两个例子，但英语对于如何更改、操纵和替换单词类型有许多不同的规则。

我们在编程语言中也注意到了这种确切的行为。在 C 中，语句的主体if包含一系列新语句。这些新陈述中的每一个，本身都可以是另一个if陈述。这些重复的结构和替换反映在语言的所有部分。这些有时被称为重写规则，因为它们告诉您如何将一件事重写为另一件事。

    if (x > 5) { return x; }

    if (x > 5) { if (x > 10) { return x; } }

乔姆斯基的这一观察结果很重要。这意味着尽管可以用一种特定的语言说出或写下无数种不同的事物，但仍然可以使用有限数量的重写规则来处理和理解所有这些事物。赋予一组重写规则的名称是语法。

我们可以用多种方式描述重写规则。一种方式是文本。我们可以这样说，“一个句子必须是一个动词短语”，或者“一个动词短语可以是一个动词，也可以是一个副词和一个动词”。这种方法对人类来说很好，但对计算机来说太不精确了。在编程时，我们需要写下更正式的语法描述。

要编写像我们的 Lisp 这样的编程语言，我们需要理解语法。为了读取用户输入，我们需要编写描述它的语法。然后我们可以将它与我们的用户输入一起使用，以确定输入是否有效。我们还可以使用它来构建结构化的内部表示，这将使理解它、评估它、执行其中编码的计算的工作变得更加容易。

这就是一个名为 library 的mpc用武之地（解析器和组合器）。



# 解析器组合器
`mpc`是一个 `Parser Combinator`库.这意味着它是一个库，允许您构建理解和处理特定语言的程序。这些被称为解析器。构建解析器的方法有很多种，但是使用Parser Combinator库最酷的地方在于它可以让您轻松构建解析器，只需指定语法......有点。

许多 `Parser Combinator` 库实际上是通过让您编写看起来有点像语法的普通代码来工作的，而不是通过实际直接指定语法来工作。在许多情况下这很好，但有时它会变得笨拙和复杂。幸运的是，我们`mpc`可以编写看起来像语法的普通代码，或者我们可以使用特殊符号直接编写语法！


# 编码语法

那么看起来像语法的代码……看起来像什么？让我们通过尝试为识别`Shiba Inu` 语言的`mpc`语法编写代码来了解一下。更通俗地称为总督。我们将按如下方式定义这种语言。
    形容词可以是"wow"、"many"、"so"或"such"。

    名词可以是"lisp"、"language"、"c"、"book"或"build"。

    短语是形容词后跟名词。

    总督是零个或多个短语。

我们可以从尝试定义Adjective和Noun开始。为此，我们创建了两个新的解析器，由 `type` 表示`mpc_parser_t*`，并将它们存储在变量`Adjective`和`Noun`中。我们使用该函数`mpc_or`创建一个解析器，其中应使用多个选项之一，并使用该函数`mpc_sym`来包装我们的初始字符串。

如果你眯着眼睛，你可以尝试阅读代码，就好像它是我们上面指定的规则一样。
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
要定义`Phrase`我们可以引用我们现有的解析器。我们需要使用函数`mpc_and`，它指定需要一件事然后再做另一件事。作为输入，我们传递它`Adjective`和`Noun`，我们之前定义的解析器。此函数还采用参数`mpcf_strfold`和`free`，说明如何加入或删除这些解析器的结果。暂时忽略这些论点。

```c
mpc_parser_t* Phrase = mpc_and(2, mpcf_strfold,
  Adjective, Noun, free);
```
要定义Doge，我们必须指定需要零个或多个解析器。为此，我们需要使用函数`mpc_many`. 和以前一样，这个函数需要特殊变量`mpcf_strfold`来说明结果是如何连接在一起的，我们可以忽略它。
```c
mpc_parser_t* Doge = mpc_many(mpcf_strfold, Phrase);
```
通过创建一个解析器来查找零次或多次出现的另一个解析器，一件有趣的事情发生了。我们的Doge解析器接受任何长度的输入。这意味着它的语言是无限的。这里只是一些Doge可能接受的字符串的例子。正如我们在本章第一节中发现的那样，我们使用了有限数量的重写规则来创建无限的语言。

```c
"wow book such language many lisp"
"so c such build such language"
"many build wow c"
""
"wow lisp wow c many language"
"so c"
```

如果我们使用更多的mpc功能，我们可以慢慢建立解析器来解析越来越复杂的语言。我们使用的代码读起来有点像语法，但随着复杂性的增加变得更加混乱。因此，采用这种方法并不总是一件容易的事。mpc 存储库中记录了一整套建立在简单结构上以简化频繁任务的辅助函数。对于复杂的语言，这是一种很好的方法，因为它允许进行细粒度的控制，但我们的需求并不需要。

# 自然语法
`mpc`让我们也以更自然的形式编写语法。我们可以在一个长字符串中指定整个内容，而不是使用看起来不太像语法的 C 函数。使用此方法时，我们不必担心如何使用`mpcf_strfold`, 或等函数加入或丢弃输入`free`。所有这些都是自动为我们完成的。
以下是我们如何使用此方法重新创建前面的示例。

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
无需对该长字符串的语法有确切的理解，这种格式的语法清晰度应该是显而易见的。如果我们了解所有特殊符号的含义，我们几乎不需要眯着眼睛。

另一件需要注意的事情是，该过程现在分为两个步骤。首先，我们使用创建和命名几个规则mpc_new，然后我们使用定义它们mpca_lang。

第一个参数mpca_lang是选项标志。为此，我们只使用默认值。二是C中的多行长字符串，这是语法规范。它由许多重写规则组成。每条规则的左侧为规则名称，一个冒号:，右侧为其定义以分号结尾;。

用于定义右侧规则的特殊符号的工作方式如下。

    "ab"	字符串ab是必需的。
    'a'	字符a是必需的。
    'a' 'b'	首先'a'是必需的，然后'b'是必需的。
    'a' | 'b'	要么'a'是必需的，要么'b'是必需的。
    'a'*	需要零个或多个'a'。
    'a'+	需要一个或多个'a'。
    <abba>	调用的规则abba是必需的。

使用上面描述的表格验证我上面写的内容是否等于我们在代码中指定的内容。

这种指定语法的方法是我们将在以下章节中使用的方法。乍一看似乎势不可挡。语法可能很难理解。但是随着我们的继续，您将更加熟悉如何创建和编辑它们。

本章是关于理论的，所以如果你打算尝试一些奖励任务，不要太担心正确性。以正确的心态思考更为重要。随意为某些概念发明符号和符号，使它们更容易写下来。一些奖励任务也可能需要循环或递归语法结构，所以如果您必须使用这些，请不要担心！

包含的mpc解析器
https://github.com/orangeduck/mpc

# Parsing

## 波兰语

为了尝试，mpc我们将实现一个类似于 Lisp 的数学子集的简单语法。它称为波兰表示法，是一种算术表示法，其中运算符位于操作数之前。

例如:
  1 + 2 + 6	是	+ 1 2 6
  6 + (2 * 9)	是	+ 6 (* 2 9)
  (10 * 2) / (4 + 2)	是	/ (* 10 2) (+ 4 2)


我们需要制定一个描述这种符号的语法。我们可以从文字描述开始，然后再将我们的想法形式化。

首先，我们观察到，在波兰语中，运算符总是在表达式中排在第一位，后面是括号中的数字或其他表达式。这意味着我们可以说“一个程序是一个运算符后跟一个或多个表达式”，其中“一个表达式是一个数字，或者在括号中是一个运算符后跟一个或多个表达式”。

more formally

  Program input 的开始，一个`Operator`，一个或多个`Expression`，以及input 的结束。
  Expression 一个`Number` 或 `'('`，一个`Operator`，一个或多个`Expression`，和一个`')'`
  Operator `'+'`、`'-'`、`'*'`或`'/'`。
  Number 可选的, 和之间的-一个或多个字符`0` `9`


# Regular Expressions
我们应该能够使用已知的东西对上述大部分规则进行编码，但Number和Program可能会带来一些麻烦。它们包含一些我们还没有学会如何表达的结构。我们不知道如何表达输入的开始或结束、可选字符或字符范围。

这些可以表达，但它们需要称为正则表达式的东西。正则表达式是一种为小部分文本（例如单词或数字）编写语法的方法。使用正则表达式编写的语法不能由多个规则组成，但它们确实对匹配和不匹配的内容提供了精确和简洁的控​​制。下面是编写正则表达式的一些基本规则。

  `.`	需要任何字符。  

  `a`	字符a是必需的。 

  `[abcdef]`	集合中的任何字符abcdef都是必需的。  

  `[a-f]`	a范围内的任何字符f都是必需的。  

  `a?`	该字符a是可选的。 

  `a*`	需要零个或多个字符a。 

  `a+`	需要一个或多个字符a。 

  `^`	输入的开始是必需的。  

  `$`	输入结束是必需的。  、


这些就是我们现在需要的所有正则表达式规则。都是关于学习正则表达式的。对于好奇的更多信息，可以在网上或从这些来源找到。我们将在后面的章节中使用它们，因此需要一些基本知识，但您暂时不需要掌握它们。

在mpc语法中，我们通过将正则表达式放在正斜杠之间来编写正则表达式/。使用上面的指南，我们的数字规则可以表示为使用字符串的正则表达式/-?[0-9]+/。

# Installing mpc

在我们开始编写这个语法之前，我们首先需要包含头`mpc`文件，然后链接到`mpc`库，就像我们`editline`在 Linux 和 Mac 上所做的那样。从第 4 章的代码开始，您可以将文件重命名为`mpc repo`并从中`parsing.c`下载。将它们放在与源文件相同的目录中。`mpc.h` `mpc.c`

包括  `mpc` 放置在  #include `"mpc.h"`文件的顶部。要链接直接`mpc`放入`mpc.c`编译命令。在 Linux 上，您还必须通过添加标志来链接到数学库-lm。  

在Linux 和Mac上 

`cc -std=c99 -Wall parsing.c mpc.c -ledit -lm -o parsing`

在Windows上
`cc -std=c99 -Wall parsing.c mpc.c -o parsing`

# Polish Notation Grammar

进一步形式化上述规则，并使用一些正则表达式，我们可以为波兰符号语言编写如下最终语法。阅读下面的代码并验证它是否符合我们所写的文本以及我们 的波兰符号的想法。 

```c
/* Create Some Parsers */
mpc_parser_t* Number   = mpc_new("number");
mpc_parser_t* Operator = mpc_new("operator");
mpc_parser_t* Expr     = mpc_new("expr");
mpc_parser_t* Lispy    = mpc_new("lispy");

/* Define them with the following Language */
mpca_lang(MPCA_LANG_DEFAULT,
  "                                                     \
    number   : /-?[0-9]+/ ;                             \
    operator : '+' | '-' | '*' | '/' ;                  \
    expr     : <number> | '(' <operator> <expr>+ ')' ;  \
    lispy    : /^/ <operator> <expr>+ /$/ ;             \
  ",
  Number, Operator, Expr, Lispy);
```

我们需要将此添加到我们在第 4 章开始的交互式提示中。在打印版本和退出信息之前将此代码放在main函数的开头。在我们的程序结束时，我们还需要在完成解析器后删除它们。在返回之前，我们应该放置以下清理代码。main 

```c
/* Undefine and Delete our Parsers */
mpc_cleanup(4, Number, Operator, Expr, Lispy);
```

# Parsing User Input

我们的新代码为我们的波兰表示法mpc语言创建了一个解析器，但我们仍然需要在每次从提示中提供的用户输入中实际使用它。我们需要编辑我们的循环，以便它不只是回显用户输入，而是实际尝试使用我们的解析器解析输入。我们可以通过使用以下代码替换对的调用来完成此操作，该代码使用了我们的程序解析器。 

`while` `printf` `mpc` `Lispy`  

```c
/* Attempt to Parse the user Input */
mpc_result_t r;
if (mpc_parse("<stdin>", input, Lispy, &r)) {
  /* On Success Print the AST */
  mpc_ast_print(r.output);
  mpc_ast_delete(r.output);
} else {
  /* Otherwise Print the Error */
  mpc_err_print(r.error);
  mpc_err_delete(r.error);
}
```
`mpc_parse`此代码使用我们的解析器`Lispy`和输入字符串调用该函数`input`。它将解析的结果复制到r并1在成功和0失败时返回。当我们将它传递给函数时，我们使用 operator `&` on的地址。`r`该运算符将在后面的章节中进行更详细的解释。

成功时，内部结构被复制到`r`, in the field 中`output`。我们可以使用 打印出这个结构`mpc_ast_print`并使用 删除它`mpc_ast_delete`。

否则出现错误，将其复制到r字段中error。我们可以使用 打印出来并使用`mpc_err_print`删除它`mpc_err_delete`。

编译这些更新，并试一试这个程序。尝试不同的输入，看看系统如何反应。正确的行为应如下所示。

```c
Lispy Version 0.0.0.0.2
Press Ctrl+c to Exit

lispy> + 5 (* 2 2)
>
  regex
  operator|char:1:1 '+'
  expr|number|regex:1:3 '5'
  expr|>
    char:1:5 '('
    operator|char:1:6 '*'
    expr|number|regex:1:8 '2'
    expr|number|regex:1:10 '2'
    char:1:11 ')'
  regex
lispy> hello
<stdin>:1:1: error: expected whitespace, '+', '-', '*' or '/' at 'h'
lispy> / 1dog
<stdin>:1:4: error: expected one of '0123456789', whitespace, '-', one or more of one of '0123456789', '(' or end of input at 'd'
lispy>
```








