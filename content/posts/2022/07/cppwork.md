+++
title = "C++ 期末作业报告"
date = 2022-07-02T01:54:00+08:00
lastmod = 2022-07-02T14:44:34+08:00
tags = ["C++", "编辑器"]
categories = ["C++"]
draft = false
toc = true
+++

## 项目介绍 {#项目介绍}

这次作死来试试手写编辑器，作为一次 **Project Based Learning** ，我选择了[这个教程](https://viewsourcecode.org/snaptoken/kilo/index.html) <br/>
这个教程教你在单文件中用 1000 行C代码来实现一个迷你的编辑器，带有代码高亮，无第三方依赖 <br/>
这个教程一共有7步，每一步教你实现 <br/>

1.  `Setup` 环境设置 <br/>
2.  `Entering raw mode` 进入终端原始模式 <br/>
3.  `Raw input and output` 原始的输入和输出 <br/>
4.  `A text viewer` 文本浏览 <br/>
5.  `A text editor` 文本编辑 <br/>
6.  `Search` 文字搜索 <br/>
7.  `Syntax highlighting` 语法高亮 <br/>

这里由于能力有限，只实现到 文本浏览这一块，不过这个项目挺有意思，我打算有时间接着实现 <br/>

{{< figure src="/ox-hugo/editor.png" >}} <br/>


## 设计思路 {#设计思路}

> 在 Linux 中，可以使用 ANSI 转义码（ANSI escape codes）设置终端的字符显示颜色、移动光标位置、清除字符显示等。 <br/>
> ANSI 转义码是由终端自身支持，独立于编程语言之外，可以在 C 语言、Java、Python、或者 Shell 中使用。 <br/>

需要补充的是，电脑每秒都在按一定的频率刷新屏幕，在程序里的具体体现为，渲染好当前页面后，刷新，清楚页面，然后接着 <br/>
把数据渲染上去 <br/>

基于此，程序功能大致分为这么几块， <br/>

1.  `data` 全局数据相关 <br/>
2.  `terminal` 终端操作的 <br/>
3.  `init` 程序初始化操作 <br/>
4.  `input` 处理输入字符的 <br/>
5.  `output` 处理输出的，比如刷新屏幕 <br/>

由于此前用C代码超了一遍源程序，对代码有了一些理解，这里打算用 `C++` 来实现一版 <br/>
另外发现在这个程序中使用类的继承是不合适的，这里用到 `C++` 只是用来代替 `C` ，做一些抽象，少些点代码而已 <br/>


### 作为编辑器的程序 {#作为编辑器的程序}

这时候程序只是作为一个编辑器，需要响应的就只是光标的移动， 这时程序一共分为三大步 <br/>


#### 初始化环境 {#初始化环境}

-   设置终端为原始模式 <br/>
-   获取终端窗口大小 <br/>


#### 循环处理按键 {#循环处理按键}

1.  刷新屏幕 <br/>
2.  读取按键 <br/>
    -   是 组合键 <br/>
    -   是 方向键 或 `PageUp` 或 `Home` 等按键 <br/>
    -   是 普通的字符 <br/>
3.  渲染屏幕 <br/>


#### 退出 {#退出}

还原终端为规范模式 <br/>

注意 <br/>

1.  终端有三种工作模式：规范模式、非规范模式、原始模式 <br/>
2.  在termios结构的c_lflag中设置ICANNON标志来定义终端以何种模式工作，默认为规范模式。 <br/>
3.  **规范模式** 所有输入基于行进行处理。在用户输入一个行结束符（回车符、EOF等）之前， <br/>
    系统调用 `read` 函数读不到用户输入的任何字符 <br/>
    其次，除了EOF之外的行结束符与普通字符一样会被 `read` 函数读取到缓冲区中。一次调用 `read` 只能读取一行数据。 <br/>
4.  **非规范模式** 所有输入时即时有效的，不需要用户另外输入行结束符。 <br/>
5.  **原始模式** 是一种特殊的非规范模式，所有的输入数据以字节为单位被处理。即有一个字节输入时，触发输入有效。 <br/>

这里由于要处理方向键，组合键这类按键，所以选择 **原始模式** <br/>


### 作为文本浏览的程序 {#作为文本浏览的程序}

文本浏览不仅要在初始化环境时进入原始模式，还要读入文件 <br/>
光标的移动由于文件的引入，需要实现 **垂直滚动** 和 **水平滚动** ，需要全局状态来记录光标的位置 `cursorx` 与 `cursory` ， <br/>
还要记录所在的行数 `colsOffset` 和列数 `rowsOffset` ，并与终端窗口大小比较，判断是否需要滚动 <br/>

除此之外还要处理文本中的制表符 '\t' ，需要为制表符指定渲染的长度，设置光标移动遇到制表符时，直接调到下一个字符， <br/>
这里定义一个新的 `renderx` 来替代 `cursorx` 的部分功能，并提供 `cursorx` 到 `renderx` 的转换方法 `cxtorx` <br/>


## 代码实现 {#代码实现}


### 主函数 {#主函数}

```c++
int main(int argc, char * argv[]) {
  Editor editor;

  if(argc >= 2) {
    editor.open(argv[1]);
  }

  editor.setStatus("HELP: Ctrl-Q = quit");

  while (true) {
    editor.refreshScreen();
    int key = editor.readkey();
    editor.processkey(key);
  }

  return 0;
}
```

参考我们的设计思路，定义一个 `Editor` 类，在其构造时就将终端进入原始模式，并获取终端窗口大小 <br/>


### 数据对象 Editor {#数据对象-editor}

`Editor` 表示编辑器，通过其存储的变量来实现光标移动和文件读取等功能 <br/>

```c++
class Editor {
private:			// for basic
  int screenrows;
  int screencols;
  int cursorx;
  int cursory;
  int renderx;

private:			// for read file
  vector<string> rows;
  int rowoffset;
  int coloffset;

private:			// for status
  string filename;
  string status;
  time_t status_time;

private:
  int windowsize();
  void enableraw();
  int cxtorx(string & chars);

public:
  Editor(); 			/* this is for initialize */
  ~Editor();

};

```

他还有一系列函数来处理终端输入，输出 <br/>

```c++
public:
  int readkey();
  void processkey(int key);
  void open(const char * filename);
```

```c++
public:
  void refreshScreen();
  void drawrows(string & appendbuf);
  void drawStatusBar(string & appendbuf);
  void drawMessageBar(string & appendbuf);
  void setStatus(const char * fmt, ...);
  void scroll();
  string updaterow(string & chars);
```

这里我顺带着把光标的移动给抽象成函数了 <br/>

```c++
private:
  void moveleft();
  void moveright();
  void moveup();
  void movedown();
  void homekey();
  void endkey();
  void pageup();
  void pagedown();
```


### 初始化环境 {#初始化环境}

编辑器中还添加了一个功能， `status bar`  <br/>
在获取了 `screenrows` 后需要减掉两行来放 `status bar` 和 `status message` <br/>

在构造函数中 <br/>

```c++
Editor::Editor() {
  cursorx = cursory = renderx = 0;
  screencols = screenrows = 0;
  rowoffset = coloffset = 0;
  filename = "";
  status = "";
  status_time = 0;

  enableraw();
  windowsize();
  screenrows -= 2;
}
```

其中的一些私有函数定义为 <br/>

```c++
int Editor::windowsize() {
  struct winsize ws;

  if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    return -1;
  } else {
    screencols = ws.ws_col;
    screenrows = ws.ws_row;
    return 0;
  }
}

void Editor::enableraw() {
  struct termios raw;
  tcgetattr(STDIN_FILENO, &originTermios);

  raw = originTermios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}

```

其中 `originTermios` 是全局变量 <br/>


### 事件循环处理 {#事件循环处理}

在主函数中，编辑器打开了一个文本，然后处理键盘输入事件 <br/>

```c++
if(argc >= 2) {
  editor.open(argv[1]);
 }

editor.setStatus("HELP: Ctrl-Q = quit");

while (true) {
  editor.refreshScreen();
  int key = editor.readkey();
  editor.processkey(key);
 }


```


#### 1. 如何打开文件 {#1-dot-如何打开文件}

`Editor` 用一个 `string` 动态数组 `rows` 保存每一行的文本内容 <br/>

```c++
void Editor::open(const char * path) {
  filename = path;
  ifstream is(filename, ios::in);

  string buffer;
  std::getline(is, buffer);

  while(is) {
    rows.push_back(buffer);
    std::getline(is, buffer);
  }

  is.close();
}

```


#### 2. 如何刷新屏幕 {#2-dot-如何刷新屏幕}

```c++
void Editor::refreshScreen() {
  scroll();

  string appendbuf = "";
  appendbuf += "\e[?25l";
  appendbuf += "\e[H";
  drawrows(appendbuf);
  drawStatusBar(appendbuf);
  drawMessageBar(appendbuf);

  stringstream ss;
  // ss << "\e[" << cursory + 1 << ";" << cursorx + 1 << "H";
  ss << "\e["
     << cursory - rowoffset + 1
     << ";"
     << renderx - coloffset + 1
     << "H";
  appendbuf += ss.str();
  appendbuf += "\e[?25h";
  cout << appendbuf << std::flush;
}
```

1.  首先调整光标位置，处理一些越界的情况，调用函数 `scroll` <br/>
2.  添加 ANSI 转义字符 `\e[?25l` **隐藏光标** <br/>
3.  添加 ANSI 转义字符 `\e[H` **定位光标到左上角** <br/>
4.  渲染此时的文本内容 `drawrow` <br/>
5.  渲染 `status bar` <br/>
6.  渲染 `status message` <br/>
7.  添加 ANSI 转义字符 `\e[x;yH` **定位光标** 到 `(x, y)` <br/>
8.  添加 ANSI 转义字符 `\e[?25h` **显示光标** <br/>

注意， `cout` 后一定要加 `std::flush` ，马上打印出字符串，不然字符串就会留在缓存区中，看到的情况就是 **没有光标** <br/>


#### 3. 如何读取按键 {#3-dot-如何读取按键}

这里用的完全是作者的代码，其中全都是 `C` 的一些技巧，正考虑如何写一个 `C++` 风格的，更加精简的 `readKey` 函数 <br/>
这段代码的主要功能是，判断输入的按键是字符还是一些方向键，不管怎么样，他都扔一个 **整数** 的 `key` 出去 <br/>
切忌，一定要用 `int` 来接受这个 `key` ，不然按什么键都不会有反映 <br/>


#### 4. 如何处理按键 {#4-dot-如何处理按键}

还好这个程序没有没有实现文本编辑功能，只需要根据按键移动光标即可，这里用 `processKey` 来处理 <br/>

```c++
void Editor::processkey(int key) {
  switch (key) {
  case HOME_KEY:
    homekey();
    break;
  case END_KEY:
    endkey();
    break;
  case PAGE_UP:
    pageup();
    break;
  case PAGE_DOWN:
    pagedown();
    break;

  case ARROW_UP:
    moveup();
    break;
  case ARROW_DOWN:
    movedown();
    break;
  case ARROW_LEFT:
    moveleft();
    break;
  case ARROW_RIGHT:
    moveright();
    break;
  }

  string row = (cursory >= rows.size()) ? "" : rows[cursory];
  int rowlen = row.length();
  if(cursorx > rowlen) {
    cursorx = rowlen;
  }
}
```

最后几行的代码是在每次光标移动后，如果光标的 `x` 坐标超过这行文字的长度，将光标对其到行尾 <br/>


#### 5. 什么时候退出 {#5-dot-什么时候退出}

我们定义按下 `Ctrl + Q` 组合键退出， <br/>
首先要辨认 `Ctrl` 系列组合键，这里使用文本宏 `CTRL_KEY` <br/>

```c++
#define CTRL_KEY(k) ((k) & 0x1f)
```

在 `processKey` 中，添加一个条件开关即可 <br/>

```c++
case CTRL_KEY('q'):
    cout << "\e[2J" << std::flush;
    cout << "\e[H" << std::flush;

    exit(0);
    break;
```

转义符 `\e[2J` 表示 清除屏幕显示的内容，不过在 Ubuntu 上测试，光标位置会保持不变 <br/>
转义符 `\e[H` 表示将光标移动到左上角，不过在本机测试时好像不会 <br/>


#### 6. 如何将终端还原为规范模式 {#6-dot-如何将终端还原为规范模式}

由于我们是手动退出的，程序跳过了 `Editor` 的析构函数并回收资源，我们需要定义一个函数 `disableraw` 来注册到 `atexit` 系统调用中 <br/>
这样程序退出的时候终端就会还原 <br/>
其中 `disableraw` 定义为 `Editor` 的静态函数(这里是我设计失误) <br/>

```c++
void Editor::disableraw() {
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &originTermios);
}
```

然后在构造函数中添加即可 <br/>

```c++
atexit(Editor::disableraw);
```