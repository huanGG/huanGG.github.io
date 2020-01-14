---
title: python调用c/c++
translate_title: python-call-c
tags:
  - python
  - c++
date: 2018-12-02 19:29:40
---

现在，很多时候用 python 来写业务逻辑，用 c/c++ 来写计算密集型的算法。这就涉及到pytho 对 c/c++ 程序的调用，在此做一个总结。

<!--more-->
## 1. Python 调用 C 动态库
Python 调用 c 动态库比较简单，分为以下三步：
* 编写 c 程序代码，如 test.c
{% codeblock  lang:c %}
#include <stdio.h>  
#include <stdlib.h>  
int add(int a, int b)  
{  
  printf("input: %d and %d\n", a, b);  
  return a+b;  
}
{% endcodeblock %}
* 编译成动态库文件
 {% codeblock  lang:shell %}
gcc -o test.so -shared -fPIC test.c
{% endcodeblock %} 
* 在python 中通过 ctypes 引入动态库文件，然后调用。如 add.py
{% codeblock  lang:python %}
import ctypes  
ll = ctypes.cdll.LoadLibrary   
lib = ll("./test.so")    
result = lib.add(1, 3)  
print 'result = {}'.format(result) 
{% endcodeblock %}
运行结果： result = 4

## 2. Python 调用 C++ 动态库
与调用 c 动态库不同的是，c++ 动态库中被python 调用的函数必须由 extern "C" 修饰,其他的函数可以不加。步骤同样分为三步：
* 编写 c++ 程序。如 test.cpp
{% codeblock  lang:c++ %}
#include <string>

extern "C"{
    //注意这里传的参数必须是 ctypes 支持的类型，传递 string 会报错。ctypes 支持的类型见参考链接 1 中的 Fundamental data types 一节。
    void print_hello(char* str1, char * str2){ 
        printf("%s %s", str1, str2);
    }
}
{% endcodeblock %}
* 编译成动态库文件
{% codeblock  lang:shell %}
g++ -o test.so -shared -fPIC test.cpp
{% endcodeblock %} 
* python 中调用动态库文件。如 test.py中
{% codeblock  lang:python %}
import ctypes

ll = ctypes.cdll.LoadLibrary
lib = ll("./test.o")
result = lib.print_hello("hello", "world")
{% endcodeblock %}
运行结果： hello world

## 3. Python 调用可执行程序
python 调用可执行程序的方式有三种，分别是os.system、os.popen、subprocess.popen，官方推荐使用最后一种。下面展示一个使用subprocess.popen创建子程序的方法：
{% codeblock  lang:python %}
def create_process(cmdline_list, timeout_second):
    """
    根据命令行创建进程
    :param cmdline_list: 命令清单
    :param timeout_second: 超时时间
    :return:
    """
    cmdline_list = [c.strip('"') for c in cmdline_list]
    process = Popen(cmdline_list, stdout=PIPE, stderr=PIPE)
    # 同时加入超时机制
    kill_timer = Timer(timeout_second, kill_process, [process])
    kill_timer.start()
    return_code = process.wait()
    kill_timer.cancel()
    return return_code


def kill_process(process):
    """
    杀死进程
    :param process:
    :return:
    """
    process.kill()
{% endcodeblock %}

## 4. C++为Python编写扩展模块
python 中并不提供直接调用 c/c++ 代码。但可以用 c/c++ 为 python 编写扩展模块。很多常用的 python 库就是通过这种方式生成的。
这块内容相对比较多，这里只提供一个简单的 demo。感兴趣的朋友可以参考 《Python高级编程》第 7 章。
* 首先创建c 程序文件，需要按照一些约定进行包装。
{% codeblock  lang:c %}
// fibonacci.c

// 注意将这个头文件指定到你使用的python 版本的 include 路径下
#include <Python.h>

long long fibonacci(unsigned int n){
    if (n < 2){
        return 1;
    }
    else {
        return fibonacci(n - 2) + fibonacci(n - 1);
    }
}

// 提供给 python的接口函数
static PyObject* fibonacci_py(PyObject* self, PyObject* args){
    PyObject* result = NULL;
    long n;
    if (!PyArg_ParseTuple(args, "i", &n))
        return NULL;
    return (PyObject*)Py_BuildValue("i", fibonacci(n));
}

// 接口说明
static char fibonacci_docs[] = "return nth Fibonacci number";

// python 模块中的函数定义
static PyMethodDef fibonacci_module_methods [] = {
        // 第一个参数是在python程序中使用的函数的名称，第二个参数是c程序中的接口函数名
        {
            "Fibonacci", (PyCFunction)fibonacci_py, METH_VARARGS, fibonacci_docs
        },
        {
            NULL, NULL, 0, NULL
        }
};

static struct PyModuleDef fibonacci_module_definition = {
        PyModuleDef_HEAD_INIT,
        "fibonacci",
        "Extension module that provide fibonacci function",
        -1,
        fibonacci_module_methods
};

PyMODINIT_FUNC PyInit_fibonacci(void){
    Py_Initialize();
    return PyModule_Create(&fibonacci_module_definition);
}
{% endcodeblock %}

* 同一目录下创建 setup.py 文件
{% codeblock  lang:python %}
from setuptools import setup, Extension


setup(
    name="fibonacci",
    ext_modules=[
        Extension("fibonacci", ["fibonacci.c"]),
    ]
)
{% endcodeblock %}

* 然后执行如下命令，安装扩展包
{% codeblock  lang:shell %}
python setup.py install
{% endcodeblock %}

* 最后在项目中import即可使用。
{% codeblock  lang:python %}
import fibonacci

print (fibonacci.Fibonacci(5))
{% endcodeblock %}

运行结果： 8
