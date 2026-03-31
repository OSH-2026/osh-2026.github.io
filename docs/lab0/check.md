# Lab 0 检查情况

## Git

### 避免上传可执行文件

可执行文件在 Git 仓库中用处不大，可以使用 `.gitignore` 去掉。但是 Git 无法检测哪些文件具有执行权限，因此，必须在文件名上作出区分。例如，`.exe` 或者 `.elf`。Linux 下，可执行文件可以是任意的扩展名。或者，最好地，使用一个专门的 `build/` 目录存储构建产物，这样直接让 `.gitignore` 忽略这个目录就行。

## Makefile 相关

### Debug 下建议增加 `-g` 方便调试。

例如：

```makefile
debug:
	$(Compiler) $(Source) -o $(Target) $(Flags) -std=$(Standard) -Ddebug # 建议加上 -g
```

### 如何引入额外的 flag

请看下面的 Makefile 内容：

```makefile
.PHONY: all clean debug

all: $(TARGET)

$(TARGET): $(SRCS)
	$(CXX) $(CXXFLAGS) $(SRCS) -o $(TARGET)

debug: CXXFLAGS += $(DEBUGFLAGS)
debug: clean $(TARGET)
```

这里如果使用 `make debug`，会做几件事情：

- 首先把 `-g` 加到 CXXFLAGS 上。
- 执行 `clean`，因为它是 `debug` 的第一个依赖
- 执行 `$(TARGET)`，使用新的 CXXFLAGS 构建得到可执行文件

### Makefile 中如果涉及多文件编译，建议先编译为 `.o` 然后再链接

例如下面的写法，如果需要增加 `.cpp` 源代码、加入其他链接库时也比较麻烦，也（最重要的）无法充分利用 GNU Make 的增量编译特性。

```makefile
Source = bubblesort.cpp main.cpp
release:
	$(Compiler) $(Source) -o $(Target)
```

- 如果使用了先编译成 `.o` 的写法，记得 `clean` 的时候要删除掉中间的构建产物 `*.o`。
- `rm -f` 可以不检查文件是否存在（即如果文件不存在，也不会错误退出）
- 这里可能需要使用 Makefile 的文件名“改写”机制，例如自动地把 `main.c` 这样的名称改写为 `main.o` 的名称。

### 名称改写

例如：

```makefile
CC = g++
CXXFLAGS = -std=c++11 -O2

SRCS = main.cpp bubblesort.cpp

# 改写：把 .cpp 换成 .o
OBJS = $(SRCS:.cpp=.o)

.PHONY: all clean

all: bubble_sort

bubble_sort: $(OBJS)
    # $@ 是目标名称 bubble_sort，$^ 是所有依赖，即 $(OBJS)
	$(CC) $(CXXFLAGS) -o $@ $^

%.o: %.cpp
    # $@ 是目标名称，$< 是一个依赖（例如 main.cpp）
	$(CC) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f bubble_sort $(OBJS)
```

### 使用 `.phony` 标记目标

例如：

```makefile
.PHONY: all
all: $(TARGET)
```

可以告诉 Make，`all` 不是一个文件目标，而只是一个简单的目标名称，这样避免了目录下出现了名字为 `all` 的文件时可能带来的冲突。

## C++

### 在与 `.h` 文件对应的 `.cpp` 文件中要 `include` 这个 `.h` 文件

这样可以引入 `.h` 中的函数原型等，避免出现写在前面的函数需要调用后面的函数时出现找不到符号的问题。

### `.h` 文件中，需要防止重复引用，避免编译报错

```c++
#ifndef _EXAMPLE_H
#define _EXAMPLE_H

// 你的内容
# endif
```

这样，如果这个头文件被 `include` 了两次，则第二次因为 `_EXAMPLE_H` 已经被定义，则头文件的内容会被跳过，避免了重复定义。

### `const auto&` 的使用

在只读的循环中，建议使用 `const auto& i`，这样在**语义上更清晰**，也可以**避免在循环中意外地修改对象的值**。

```c++
void bubbleSort(std::vector<int>& arr) {
    // ...
    for(auto&& i:arr) // 这里用 const auto& i 更好
    {
        std::cout<<i<<" ";
    }
    std::cout<<"\n";
    // ...
}
```

此外，如果需要传递只读的复合对象（例如 `std::string`）给函数，建议使用 `const Type&`，这样消除了拷贝负担，提高性能，同时避免了函数意外地对参数进行修改。例如，`split` 函数（自行练习）中，两个参数都应该是只读的，因此，可以使用这种写法。

### `emplace_back` 和 `push_back`

`emplace_back` 是可以直接把参数挪动过来，避免了再次创建一个重复的对象（可以在构造函数中打印信息来确认这一点）。在这里，它还有一个用途——可以省去 `substr`：

```c++
parts.push_back(s.substr(start, pos - start));
```

可以写成

```c++
parts.emplace_back(s, start, pos - start);
```

这样性能会有小的提升（当然 `push_back` 也很好）。
