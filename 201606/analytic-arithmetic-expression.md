# 利用栈解析算术表达式

在编写编译器时经常需要实现对算术表达式的解析，然而对于计算机的算法来说如果直接求解算术表达式的值，还是相当困难的。因此解析算术表达式经常分步实现：
1.将中缀的算术表达式转换为后缀表达式
2.计算后缀表达式的值
<!--more-->
在正式介绍算法的实现之前，先介绍一点有关表达式的基础知识

## 基础知识
### 1. 后缀表达式

日常算术表达式是将操作符(operator)(+,-,*,/)放在两个操作数(operands)(数字，或者代表数字的字母)中间的，由于操作符写在操作数中间，所以把这种写法称为中缀表达式.对人类而言，中缀表达式便于理解和阅读.

后缀表达式，又称为波兰逆序表达式(Reverse Polish Notation)，它是将操作符放在操作数后面的一种表达式的记录方法，比如`A+B`变成`AB+`，这是一种便于计算机计算的表达式.

|    中缀表达式   |   后缀表达式  |
|  :---------:    |   :---------: |
|A+B-C            |AB+C-        |
|A*B/C            |AB*C/   |
|A+B*C            |ABC*+|
|A+B*(C-D/(E+F))  | ABCDEF+/-*+|

### 2. 栈

栈(Stack)是计算机中应用的非常多的一种数据结构，其遵循先进后出的规律，可以用数组或者链表来实现栈，本文中就是使用数组实现一个简单的栈.

-----
## 中缀表达式转换为后缀表达式
将中缀表达式转换为后缀表达式是解析算术表达式中最重要的一步，本文中通过观察几个示例然后总结出转换规律.

### 1. 分析

将中缀表达式转换为后缀表达式的规则和计算中缀表达式值的规则相似，不需要做计算，只是把操作数和操作符重新排列为另一种形式：后缀表示法.

从左到右的扫描中缀表达式，遇到操作数直接输出，遇到操作符则按照转换规则入栈或者输出.以将A*(B+C)的转换为例：

|读取的字符|分解中缀表达式|求后缀表达式|栈中内容|
|:--------:|:--------:    |:--------:  |:--------:|
|A  |A  |A  |   |
|+  |A+ |B  |+  |
|B  |A+B    |AB |+  |
|*  |A+B*   |AB    |+*|
|(  |A+B*(| AB  |+*(   |
|C  |A+B*(C |   ABC |   +*(     |
|-  |A+B*(C- |  ABC | +*(-  |
|D  |A+B*(C-D   |ABCD   |+*(-   |
|)  |A+B*(C-D)  |ABCD-  |+*(  |
|   |A+B*(C-D)  |ABCD-  |+*(    |
|   |A+B*(C-D)  |ABCD-  |+*     |
|   |A+B*(C-D   |ABCD-* |+  |
|   |A+B*(C-D)  |ABCD-*+    |   |

从中可以看出，操作符的初始顺序在中缀表达式中是`+*-`，但是在后缀表达式中变成了`-*+`，这是因为`*`比`+`的优先级别高，而`-`在括号中所以优先级比`*`高.

### 2. 规律总结

通过观察上面的转换例子，可以一般地总结出中缀表达式转换为后缀表达式的转换规则.

|   从输入中读取的字符  |   动作          |
|   :----------------:  |                 |
|操作数             |   写至输出(postfix)|
|左括号 (           |   入栈            |
|右括号 )           |   栈非空时，重复以下步骤：弹出一项，若项不为（，则写至输出，项为(，则退出循环|
|Operator(opThis)|  若栈空，                                            推opThis；否则，栈非空时，重复：弹出一项，若项为（，推其入栈；或项为operator(opTop)，且，若`opTop<opThis`，推入opTop，或`opTop>=opThis`，输出opTop，若`opTop<opThis`，则退出循环，或项为(，推入opThis|
|没有更多项         |当栈非空时，弹出项目，将其输出|

### 3. 代码实现

利用java实现以上转换过程，关键如下：

``` java
/**
 * 中缀表达式转换为后缀表达式
 *
 * @return 转换结果
 */
public String doTrans() {
    for (int j = 0; j < input.length(); j++) {
        char ch = input.charAt(j);
        theStack.displayStack("For " + ch + " ");
        switch (ch) {
            case '+':
            case '-':
                gotOper(ch, 1);
                break;
            case '*':
            case '/':
                gotOper(ch, 2);
                break;
            case '(':
                theStack.push(ch);
                break;
            case ')':
                gotParen(ch);
                break;
            default:
                output += ch;
                break;
        }
    }
    while (!theStack.isEmpty()) {
        theStack.displayStack("While ");
        output += theStack.pop();
    }
    theStack.displayStack("End ");
    return output;
}

public void gotOper(char onThis, int prec1) {
    while (!theStack.isEmpty()) {
        char onTop = theStack.pop();
        if (onTop == '(') {
            theStack.push(onTop);
            break;
        } else {
            int prec2;
            if (onTop == '+' || onTop == '-') {
                prec2 = 1;
            } else {
                prec2 = 2;
            }

            if (prec2 < prec1) {
                theStack.push(onTop);
                break;
            } else {
                output += onTop;
            }
        }
    }
    theStack.push(onThis);
}

public void gotParen(char ch) {
    while (!theStack.isEmpty()) {
        char chx = theStack.pop();
        if (chx == '(') {
            break;
        } else {
            output += chx;
        }
    }
}
```
-----

## 计算后缀表达式

相对于转换而言，计算后缀表达式是比较容易的.从左到右扫描输入(后缀表达式)，碰到操作数将之入栈，每碰到一个操作符，从栈中提出两个操作数，用操作符将其执行运算，将计算结果入栈.

### 1. 代码实现

```
/**
 * 对后缀表达式求值
 *
 * @return 求值结果
 */
public int doParse() {
    int num1, num2, interAns = 0;
    char ch;
    theStack = new StackY(20);
    for (int j = 0; j < input.length(); j++) {
        ch = input.charAt(j);
        theStack.displayStack("" + ch + " ");
        if (ch >= '0' && ch <= '9') {
            theStack.push((ch - '0'));
        } else {
            num2 = theStack.pop();
            num1 = theStack.pop();
            switch (ch) {
                case '+':
                    interAns = num1 + num2;
                    break;
                case '-':
                    interAns = num1 - num2;
                    break;
                case '*':
                    interAns = num1 * num2;
                    break;
                case '/':
                    interAns = num1 / num2;
                    break;
                default:
                    interAns = 0;
            }
            theStack.push(interAns);
        }

    }
    interAns = theStack.pop();
    return interAns;
}
```

-----
## 括号匹配

在用户输入表达式的过程中，有时会出现左右括号不匹配的情况，对于这种情况，一个优秀的算术解析器应该能够识别出来并报错.这个实际上就是括号匹配的问题，也是利用栈来解决的，实现思路如下.

算法从左到右不断扫描输入的字符串，如果为左分隔符((、[、{)，直接入栈；如果为右分隔符，弹出栈顶的左分隔符，并且查看它是否和右分隔符匹配。如果它们不匹配则报错.如果栈中没有左分隔符和右分隔符匹配，或者一直存在着没有被匹配的分隔符(栈非空)，程序也报错.

### 1. 代码实现

```
public boolean check() {
    int size = input.length();
    StackY<Character> theStack = new StackY<>(size);
    char ch, chx;
    for (int j = 0; j < size; j++) {
        ch = input.charAt(j);
        switch (ch) {
            case '(':
                theStack.push(ch);
                break;
            case ')':
                if (!theStack.isEmpty()) {
                    chx = theStack.pop();
                    if (chx != '(') return false;
                } else
                    return false;
            default:
                break;
        }
    }
    if (!theStack.isEmpty())
        return false;
    return true;
}
```

-----
## 附录
### 1. 说明

* 为了实现方便，本算法只针对一位数的计算有效，即`12*2`不能解析，`3*9`可以解析
* 参考资料：Java数据结构和算法(第二版)

### 2. 完整代码

```
package com.lxl.stack;

import jdk.internal.org.objectweb.asm.signature.SignatureWriter;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * 利用栈解析算术表达式
 *
 * @author lixiaolin
 * @createDate 2016-05-31 16:59
 */
public class InfixApp {
    public static String getString() throws IOException {
        InputStreamReader isr = new InputStreamReader(System.in);
        BufferedReader br = new BufferedReader(isr);
        return br.readLine();
    }

    public static void main(String[] args) throws IOException {
        String input, output;
        int result;
        while (true) {
            System.out.println("Enter infix: ");
            System.out.flush();
            input = getString();
            if (input == "") {
                break;
            }
            Bracket bracket =new Bracket(input);
            if (!bracket.check()){
                System.out.println("The input is invalid...");
                break;
            }
            InToPost theTrans = new InToPost(input);
            output = theTrans.doTrans();
            System.out.println("Postfix is " + output);
            ParsePost parsePost = new ParsePost(output);
            result = parsePost.doParse();
            System.out.println("Final result is : " + result);
        }
    }
}

/**
 * 辅助栈
 *
 * @param <T>
 */
class StackY<T> {
    private int maxSize;
    private int top;
    private Object[] stackArray;

    public StackY(int maxSize) {
        this.maxSize = maxSize;
        this.top = -1;
        stackArray = new Object[maxSize];
    }

    public void push(T value) {
        stackArray[++top] = value;
    }

    public T peek() {
        return (T) stackArray[top];
    }

    public T pop() {
        return (T) stackArray[top--];
    }

    public boolean isEmpty() {
        return top == -1;
    }

    public int size() {
        return top + 1;
    }

    public T peekN(int n) {
        return (T) stackArray[n];
    }

    public void displayStack(String s) {
        System.out.print(s);
        System.out.print("Stack (botton->top): ");
        for (int i = 0; i < size(); i++) {
            System.out.print(peekN(i));
            System.out.print(" ");
        }
        System.out.println("");
    }
}

class InToPost {
    private String input;
    private StackY<Character> theStack;
    private String output = "";

    public InToPost(String input) {
        this.input = input;
        theStack = new StackY(input.length());
    }

    /**
     * 中缀表达式转换为后缀表达式
     *
     * @return 转换结果
     */
    public String doTrans() {
        for (int j = 0; j < input.length(); j++) {
            char ch = input.charAt(j);
            theStack.displayStack("For " + ch + " ");
            switch (ch) {
                case '+':
                case '-':
                    gotOper(ch, 1);
                    break;
                case '*':
                case '/':
                    gotOper(ch, 2);
                    break;
                case '(':
                    theStack.push(ch);
                    break;
                case ')':
                    gotParen(ch);
                    break;
                default:
                    output += ch;
                    break;
            }
        }
        while (!theStack.isEmpty()) {
            theStack.displayStack("While ");
            output += theStack.pop();
        }
        theStack.displayStack("End ");
        return output;
    }

    public void gotOper(char onThis, int prec1) {
        while (!theStack.isEmpty()) {
            char onTop = theStack.pop();
            if (onTop == '(') {
                theStack.push(onTop);
                break;
            } else {
                int prec2;
                if (onTop == '+' || onTop == '-') {
                    prec2 = 1;
                } else {
                    prec2 = 2;
                }

                if (prec2 < prec1) {
                    theStack.push(onTop);
                    break;
                } else {
                    output += onTop;
                }
            }
        }
        theStack.push(onThis);
    }

    public void gotParen(char ch) {
        while (!theStack.isEmpty()) {
            char chx = theStack.pop();
            if (chx == '(') {
                break;
            } else {
                output += chx;
            }
        }
    }
}

class ParsePost {
    private String input;
    private StackY<Integer> theStack;

    public ParsePost(String input) {
        this.input = input;
    }

    /**
     * 对后缀表达式求值
     *
     * @return 求值结果
     */
    public int doParse() {
        int num1, num2, interAns = 0;
        char ch;
        theStack = new StackY(20);
        for (int j = 0; j < input.length(); j++) {
            ch = input.charAt(j);
            theStack.displayStack("" + ch + " ");
            if (ch >= '0' && ch <= '9') {
                theStack.push((ch - '0'));
            } else {
                num2 = theStack.pop();
                num1 = theStack.pop();
                switch (ch) {
                    case '+':
                        interAns = num1 + num2;
                        break;
                    case '-':
                        interAns = num1 - num2;
                        break;
                    case '*':
                        interAns = num1 * num2;
                        break;
                    case '/':
                        interAns = num1 / num2;
                        break;
                    default:
                        interAns = 0;
                }
                theStack.push(interAns);
            }

        }
        interAns = theStack.pop();
        return interAns;
    }
}

class Bracket {
    private String input;

    public Bracket(String input) {
        this.input = input;
    }

    public boolean check() {
        int size = input.length();
        StackY<Character> theStack = new StackY<>(size);
        char ch, chx;
        for (int j = 0; j < size; j++) {
            ch = input.charAt(j);
            switch (ch) {
                case '(':
                    theStack.push(ch);
                    break;
                case ')':
                    if (!theStack.isEmpty()) {
                        chx = theStack.pop();
                        if (chx != '(') return false;
                    } else
                        return false;
                default:
                    break;
            }
        }
        if (!theStack.isEmpty())
            return false;
        return true;
    }
}
```




