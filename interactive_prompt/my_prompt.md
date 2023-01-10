# Editing input
if you're working on Linux or Mac you'll notice some weird behaviour when you use the arrow keys to attempt to edit you input
```bash
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> hel^[[D^[[C
``` 
使用箭头创建这些奇怪的字符，而不是在输入中移动光标。我们真正想要的是能够在线上移动，删除和编辑输入，以防我们犯错退出

在Windows上，此行为是默认行为，在Linux和Mac上，它是由一个editline，我们需要这个库提供的函数fputs调用替换我们的调用

# Using Editline
该库editline提供了我们将要使用的两个函数，称为readline和add_history。第一个函数，readline用于从一些提示中读取输入，同时允许对该输入进行编辑。第二个函数add_history让我们记录输入的历史，以便可以使用向上和向下箭头检索它们。
我们用对这些函数的调用替换fputs和fgets以获得以下内容。
```c
#include <stdio.h>
#include <stdlib.h>

#include <editline/readline.h>
#include <editline/history.h>

int main(int argc, char** argv) {

  /* Print Version and Exit Information */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  /* In a never ending loop */
  while (1) {

    /* Output our prompt and get input */
    char* input = readline("lispy> ");

    /* Add input to history */
    add_history(input);

    /* Echo input back to user */
    printf("No you're a %s\n", input);

    /* Free retrieved input */
    free(input);

  }

  return 0;
}
```
# The C ProProcessor

C的移植性问题，C提供了一些解决问题，称为预处理器。
我们一直在使用它来包含头文件，使我们能够访问标准库和其他库中的函数

预处理其的另一个用途是检测代码正在那个操作系统上编译，并使用它来发出不同的代码。

这正是我们要使用它的方式。如果我们运行的是Windows，我门将让预处理器发出带有我准备的一些假的函数readline和函数的代码，否则我们将包含头文件并使用它们。

  add_history editline

要声明编译器应该发出什么代码，我们可以将其包装在`ifdef`、`#else`、`#endif`预处理语句中。

```c

```