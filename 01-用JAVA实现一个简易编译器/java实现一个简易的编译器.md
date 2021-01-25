# 用JAVA实现简易的编译器

吃水不忘挖井人

原文链接：https://blog.csdn.net/tyler_download/article/details/50668983

又重新排版

课程链接：https://study.163.com/course/courseLearn.htm?courseId=1002830012#/learn/video?lessonId=1003210315&courseId=1002830012

喜欢的可以支持一下这位老师



代码的仓库 附带学习笔记 ：https://github.com/wdkang123/MyLanguageCompiler



# 1.简介

**编译原理由两部分组成：**

词法分析、语义分析



**词法分析：**

将一个语句分割成若干有意义的字符串的组合 然后给分割的字符串打标签

**例如：**

`1 + 2 * 3 + 4;`

将被分割成：`1+, 2*, 3+, 4`

但这些字符串并没有意义 有意义的分割是

`1, +, 2, * , 3, +, 4, ;`

接着就是给这些标签打上标签

`1234`打上NUM_OR_ID

`+` 打上 PLUS

`*` 打上TIMES

`；`的标签是SEMI



下面有三个版本 

每个版本都是这是在上一版本上做的改进

（看不懂没关系 先把代码跑起来）

后边有好几十节课 刚开始代码先有个大概的认识和感知



# 2.初级版 课时1

## 2.1 代码

先运行起来再说 Lexer.java

```java
package com.mycompiler;

import java.util.Scanner;

public class Lexer {
  public static final int EOI = 0;
  public static final int SEMI = 1;
  public static final int PLUS = 2;
  public static final int TIMES = 3;
  public static final int LP = 4;
  public static final int RP = 5;
  public static final int NUM_OR_ID = 6;

  private int lookAhead = -1;

  public String yytext = "";
  public int yyleng = 0;
  public int yylineno = 0;

  private String input_buffer = "";
  private String current = "";

  private boolean isAlnum(char c) {
    if (Character.isAlphabetic(c) == true || Character.isDigit(c) == true) {
      return true;
    }
    return false;
  }

  private int lex() {
    while (true) {
      while (current == "") {
        Scanner s = new Scanner(System.in);
        while (true) {
          String line = s.nextLine();
          if (line.equals("end")) {
            break;
          }
          input_buffer += line;
        }
        s.close();
        
        if (input_buffer.length() == 0) {
          current = "";
          return EOI;
        }
        current = input_buffer;
        ++yylineno;
        current.trim();
      } // while (current != "")

      for (int i = 0; i < current.length(); i++) {

        yyleng = 0;
        yytext = current.substring(0, 1);
        switch (current.charAt(i)) {
        case ';':
          current = current.substring(1);
          return SEMI;
        case '+':
          current = current.substring(1);
          return PLUS;
        case '*':
          current = current.substring(1);
          return TIMES;
        case '(':
          current = current.substring(1);
          return LP;
        case ')':
          current = current.substring(1);
          return RP;

        case '\n':
        case '\t':
        case ' ':
          current = current.substring(1);
          break;

        default:
          if (isAlnum(current.charAt(i)) == false) {
            System.out.println("Ignoring illegal input: " + current.charAt(i));
          } else {

            while (isAlnum(current.charAt(i))) {
              i++;
              yyleng++;
            } // while (isAlnum(current.charAt(i)))

            yytext = current.substring(0, yyleng);
            current = current.substring(yyleng);
            return NUM_OR_ID;
          }

          break;

        } // switch (current.charAt(i))
      } // for (int i = 0; i < current.length(); i++)

    } // while (true)
  }// lex()

  public boolean match(int token) {
    if (lookAhead == -1) {
      lookAhead = lex();
    }

    return token == lookAhead;
  }

  public void advance() {
    lookAhead = lex();
  }

  public void runLexer() {
    while (!match(EOI)) {
      System.out.println("Token: " + token() + " ,Symbol: " + yytext);
      advance();
    }
  }

  private String token() {
    String token = "";
    switch (lookAhead) {
    case EOI:
      token = "EOI";
      break;
    case PLUS:
      token = "PLUS";
      break;
    case TIMES:
      token = "TIMES";
      break;
    case NUM_OR_ID:
      token = "NUM_OR_ID";
      break;
    case SEMI:
      token = "SEMI";
      break;
    case LP:
      token = "LP";
      break;
    case RP:
      token = "RP";
      break;
    }

    return token;
  }
}
```

下面加个主类 就可以跑了

```java
package com.mycompiler;

public class Compiler {
  public static void main(String[] args) {
    Lexer lexer = new Lexer();
    lexer.runLexer();
  }
}
```

直接运行代码 此时卡住了 在控制台输入（这是两次输入的 别搞错了）

```java
1+2*3+4;
end
```

**运行结果：**

```java
Token: NUM_OR_ID ,Symbol: 1
Token: PLUS ,Symbol: +
Token: NUM_OR_ID ,Symbol: 2
Token: TIMES ,Symbol: *
Token: NUM_OR_ID ,Symbol: 3
Token: PLUS ,Symbol: +
Token: NUM_OR_ID ,Symbol: 4
Token: SEMI ,Symbol: ;
```



## 2.2 解析

```java
  public static final int EOI = 0;
  public static final int SEMI = 1;
  public static final int PLUS = 2;
  public static final int TIMES = 3;
  public static final int LP = 4;
  public static final int RP = 5;
  public static final int NUM_OR_ID = 6;
  
  private int lookAhead = -1;

  public String yytext = "";
  public int yyleng = 0;
  public int yylineno = 0;

  private String input_buffer = "";
  private String current = "";
```

这些是对标签的定义

LP代表左括号 RP代表右括号

EOI代表语句结尾

lookAead用于表明当前在分割的字符串的指向的标签值

yytext用于存储当前正在分析的字符串

yyleng是当前分析字符串的长度

yylineno是当前分析的字符串所在的行号

input_buffer用于存储要分析的语句

其他的代码 就是一个逻辑 大家打个断点自己看看 就都明白了



# 3.升级版 课时2

```java
package com.mycompiler;

public class BasicParser {
  private Lexer lexer;
  private boolean isLegalStatement = true;

  public BasicParser(Lexer lexer) {
    this.lexer = lexer;
  }

  public void statements() {
    /*
     * statements -> expression ; | expression ; statements
     */

    expression();

    if (lexer.match(Lexer.SEMI)) {
      /*
       * look ahead 读取下一个字符，如果下一个字符不是 EOI 那就采用右边解析规则
       */
      lexer.advance();
    } else {
      /*
       *  如果算术表达式不以分号结束，那就是语法错误
       */
      isLegalStatement = false;
      System.out.println("line: " + lexer.yylineno + " Missing semicolon");
      return;
    }

    if (lexer.match(Lexer.EOI) != true) {
      /*
       * 分号后还有字符，继续解析
       */
      statements();
    }

    if (isLegalStatement) {
      System.out.println("The statement is legal");
    }
  }

  private void expression() {
    /*
     * expression -> term expression'
     */
    term();
    expr_prime(); // expression'
  }

  private void expr_prime() {
    /*
     * expression' -> PLUS term expression' | '空'
     */
    if (lexer.match(Lexer.PLUS)) {
      lexer.advance();
      term();
      expr_prime();
    } else if (lexer.match(Lexer.UNKNOWN_SYMBOL)) {
      isLegalStatement = false;
      System.out.println("unknow symbol: " + lexer.yytext);
      return;
    } else {
      /*
       * "空" 就是不再解析，直接返回
       */
      return;
    }
  }

  private void term() {
    /*
     * term -> factor term'
     */
    factor();
    term_prime(); // term'
  }

  private void term_prime() {
    /*
     * term' -> * factor term' | '空'
     */
    if (lexer.match(Lexer.TIMES)) {
      lexer.advance();
      factor();
      term_prime();
    } else {
      /*
       * 如果不是以 * 开头， 那么执行 '空' 也就是不再做进一步解析，直接返回
       */
      return;
    }
  }

  private void factor() {
    /*
     * factor -> NUM_OR_ID | LP expression RP
     */

    if (lexer.match(Lexer.NUM_OR_ID)) {
      lexer.advance();
    } else if (lexer.match(Lexer.LP)) {
      lexer.advance();
      expression();
      if (lexer.match(Lexer.RP)) {
        lexer.advance();
      } else {
        /*
         * 有左括号但没有右括号，错误
         */
        isLegalStatement = false;
        System.out.println("line: " + lexer.yylineno + " Missing )");
        return;
      }
    } else {
      /*
       * 这里不是数字，解析出错
       */
      isLegalStatement = false;
      System.out.println("illegal statements");
      return;
    }
  }
}
```



```java
package com.mycompiler;

public class Compiler {
  public static void main(String[] args) {
    Lexer lexer = new Lexer();
    BasicParser basic_parser = new BasicParser(lexer);
    basic_parser.statements();
  }
}
```



```java
package com.mycompiler;

import java.util.Scanner;

public class Lexer {
    public static final int  EOI = 0;
    public static final int  SEMI = 1;
    public static final int  PLUS = 2;
    public static final int  TIMES = 3;
    public static final int  LP = 4;
    public static final int  RP = 5;
    public static final int  NUM_OR_ID = 6;
    public static final int  UNKNOWN_SYMBOL = 7;
    
    private int lookAhead = -1;
    
    public String yytext = "";
    public int yyleng = 0;
    public int yylineno = 0;
    
    private String input_buffer = "";
    private String current = "";
    
    private boolean isAlnum(char c) {
      if (Character.isAlphabetic(c) == true ||
            Character.isDigit(c) == true) {
        return true;
      }
      
      return false;
    }
    
    private int lex() {
    
      while (true) {
        
        while (current == "") {
            Scanner s = new Scanner(System.in);
            while (true) {
              String line = s.nextLine();
              if (line.equals("end")) {
                break;
              }
              input_buffer += line;
            }
            s.close();
            
            if (input_buffer.length() == 0) {
              current = "";
              return EOI;
            }
            
            current = input_buffer;
            ++yylineno;
            current.trim();
        }//while (current == "")
        
        if (current.isEmpty()) {
          return EOI;
        }
            
            for (int i = 0; i < current.length(); i++) {
              
              yyleng = 0;
              yytext = current.substring(0, 1);
              switch (current.charAt(i)) {
              case ';': current = current.substring(1); return SEMI;
              case '+': current = current.substring(1); return PLUS;
              case '*': current = current.substring(1);return TIMES;
              case '(': current = current.substring(1);return LP;
              case ')': current = current.substring(1);return RP;
              
              case '\n':
              case '\t':
              case ' ': current = current.substring(1); break;
              
              default:
                if (isAlnum(current.charAt(i)) == false) {
                  return UNKNOWN_SYMBOL;
                }
                else {
                  
                  while (i < current.length() && isAlnum(current.charAt(i))) {
                    i++;
                    yyleng++;
                  } // while (isAlnum(current.charAt(i)))
                  
                  yytext = current.substring(0, yyleng);
                  current = current.substring(yyleng); 
                  return NUM_OR_ID;
                }
                
              } //switch (current.charAt(i))
            }//  for (int i = 0; i < current.length(); i++) 
        
      }//while (true) 
    }//lex()
    
    public boolean match(int token) {
      if (lookAhead == -1) {
        lookAhead = lex();
      }
      
      return token == lookAhead;
    }
    
    public void advance() {
      lookAhead = lex();
    }
    
    public void runLexer() {
      while (!match(EOI)) {
        System.out.println("Token: " + token() + " ,Symbol: " + yytext );
        advance();
      }
    }
    
    private String token() {
      String token = "";
      switch (lookAhead) {
      case EOI:
        token = "EOI";
        break;
      case PLUS:
        token = "PLUS";
        break;
      case TIMES:
        token = "TIMES";
        break;
      case NUM_OR_ID:
        token = "NUM_OR_ID";
        break;
      case SEMI:
        token = "SEMI";
        break;
      case LP:
        token = "LP";
        break;
      case RP:
        token = "RP";
        break;
      }
      
      return token;
    }
}

```



```java
package com.mycompiler;

public class Parser {
  private Lexer lexer;

  String[] names = { "t0", "t1", "t2", "t3", "t4", "t5", "t6", "t7" };
  private int nameP = 0;

  private String newName() {
    if (nameP >= names.length) {
      System.out.println("Expression too complex: " + lexer.yylineno);
      System.exit(1);
    }

    String reg = names[nameP];
    nameP++;

    return reg;
  }

  private void freeNames(String s) {
    if (nameP > 0) {
      names[nameP] = s;
      nameP--;
    } else {
      System.out.println("(Internal error) Name stack underflow: " + lexer.yylineno);
      ;
    }
  }

  public Parser(Lexer lexer) {
    this.lexer = lexer;
  }

  public void statements() {
    String tempvar = newName();
    expression(tempvar);

    while (lexer.match(Lexer.EOI) != false) {
      expression(tempvar);
      freeNames(tempvar);

      if (lexer.match(Lexer.SEMI)) {
        lexer.advance();
      } else {
        System.out.println("Inserting missing semicolon: " + lexer.yylineno);
      }
    }
  }

  private void expression(String tempVar) {
    String tempVar2;
    term(tempVar);
    while (lexer.match(Lexer.PLUS)) {
      lexer.advance();
      tempVar2 = newName();
      term(tempVar2);
      System.out.println(tempVar + " += " + tempVar2);
      freeNames(tempVar2);
    }
  }

  private void term(String tempVar) {
    String tempVar2;

    factor(tempVar);
    while (lexer.match(Lexer.TIMES)) {
      lexer.advance();
      tempVar2 = newName();
      factor(tempVar2);
      System.out.println(tempVar + " *= " + tempVar2);
      freeNames(tempVar2);
    }

  }

  private void factor(String tempVar) {
    if (lexer.match(Lexer.NUM_OR_ID)) {
      System.out.println(tempVar + " = " + lexer.yytext);
      lexer.advance();
    } else if (lexer.match(Lexer.LP)) {
      lexer.advance();
      expression(tempVar);
      if (lexer.match(Lexer.RP)) {
        lexer.advance();
      } else {
        System.out.println("Missmatched parenthesis: " + lexer.yylineno);
      }
    } else {
      System.out.println("Number or identifier expected: " + lexer.yylineno);
    }
  }
}

```



# 4.最终版 课时3

```java
package com.mycompiler.third;

public class BasicParser {
  private Lexer lexer;
  private boolean isLegalStatement = true;

  public BasicParser(Lexer lexer) {
    this.lexer = lexer;
  }

  public void statements() {
    /*
     * statements -> expression ; | expression ; statements
     */

    expression();

    if (lexer.match(Lexer.SEMI)) {
      /*
       * look ahead 读取下一个字符，如果下一个字符不是 EOI 那就采用右边解析规则
       */
      lexer.advance();
    } else {
      /*
       *  如果算术表达式不以分号结束，那就是语法错误
       */
      isLegalStatement = false;
      System.out.println("line: " + lexer.yylineno + " Missing semicolon");
      return;
    }

    if (lexer.match(Lexer.EOI) != true) {
      /*
       * 分号后还有字符，继续解析
       */
      statements();
    }

    if (isLegalStatement) {
      System.out.println("The statement is legal");
    }
  }

  private void expression() {
    /*
     * expression -> term expression'
     */
    term();
    expr_prime(); // expression'
  }

  private void expr_prime() {
    /*
     * expression' -> PLUS term expression' | '空'
     */
    if (lexer.match(Lexer.PLUS)) {
      lexer.advance();
      term();
      expr_prime();
    } else if (lexer.match(Lexer.UNKNOWN_SYMBOL)) {
      isLegalStatement = false;
      System.out.println("unknow symbol: " + lexer.yytext);
      return;
    } else {
      /*
       * "空" 就是不再解析，直接返回
       */
      return;
    }
  }

  private void term() {
    /*
     * term -> factor term'
     */
    factor();
    term_prime(); // term'
  }

  private void term_prime() {
    /*
     * term' -> * factor term' | '空'
     */
    if (lexer.match(Lexer.TIMES)) {
      lexer.advance();
      factor();
      term_prime();
    } else {
      /*
       * 如果不是以 * 开头， 那么执行 '空' 也就是不再做进一步解析，直接返回
       */
      return;
    }
  }

  private void factor() {
    /*
     * factor -> NUM_OR_ID | LP expression RP
     */

    if (lexer.match(Lexer.NUM_OR_ID)) {
      lexer.advance();
    } else if (lexer.match(Lexer.LP)) {
      lexer.advance();
      expression();
      if (lexer.match(Lexer.RP)) {
        lexer.advance();
      } else {
        /*
         * 有左括号但没有右括号，错误
         */
        isLegalStatement = false;
        System.out.println("line: " + lexer.yylineno + " Missing )");
        return;
      }
    } else {
      /*
       * 这里不是数字，解析出错
       */
      isLegalStatement = false;
      System.out.println("illegal statements");
      return;
    }
  }
}

```



```java
package com.mycompiler.third;


public class Compiler {

  public static void main(String[] args) {
    Lexer lexer = new Lexer();
    //ImprovedParser improvedParser = new ImprovedParser(lexer);
    //improvedParser.statements();
    
    Parser parser = new Parser(lexer);
    parser.statements();
  }
}

```



```java
package com.mycompiler.third;

public class ImprovedParser {

  private Lexer lexer;
  private boolean isLegalStatement = true;

  public ImprovedParser(Lexer lexer) {
    this.lexer = lexer;
  }

  public void statements() {
    /*
     * statements -> expression ; | expression ; statements
     */

    while (lexer.match(Lexer.EOI) != true) {
      expression();

      if (lexer.match(Lexer.SEMI)) {
        lexer.advance();
      } else {
        isLegalStatement = false;
        System.out.println("line: " + lexer.yylineno + " Missing semicolon");
      }
    }

    if (isLegalStatement) {
      System.out.println("The statement is legal");
    }
  }

  private void expression() {
    /*
     * expression -> term expression' expression -> PLUS term expression' | 空
     */
    term();
    while (lexer.match(Lexer.PLUS)) {
      lexer.advance();
      term();
    }

    if (lexer.match(Lexer.UNKNOWN_SYMBOL)) {
      isLegalStatement = false;
      System.out.println("unknow symbol: " + lexer.yytext);
      return;
    } else {
      /*
       * "空" 就是不再解析，直接返回
       */
      return;
    }
  }

  /*
   * expr_prime 的递归调用可以整合到expression 里 由于当 lexer.match(Lexer.PLUS)
   * 判断成立时，expr_prime() 有递归调用，因此其等价于 while (lexer.match(Lexer.PLUS)) {
   * lexer.advance(); term(); }
   * 
   * private void expr_prime() {
   * 
   * if (lexer.match(Lexer.PLUS)) { lexer.advance(); term(); expr_prime(); } else
   * if (lexer.match(Lexer.UNKNOWN_SYMBOL)) { isLegalStatement = false;
   * System.out.println("unknow symbol: " + lexer.yytext); return; } else {
   * 
   * return; } }
   */

  private void term() {
    factor();

    /*
     * 将term_prime 的递归调用改成循环方式
     * 
     */

    while (lexer.match(Lexer.TIMES)) {
      lexer.advance();
      factor();
    }
  }

  /*
   * term_prime 递归调用改循环的道理与expression 的改进同理
   * 
   * private void term_prime() {
   * 
   * if (lexer.match(Lexer.TIMES)) { lexer.advance(); factor(); term_prime(); }
   * else {
   * 
   * return; } }
   */

  private void factor() {
    /*
     * factor -> NUM_OR_ID | LP expression RP
     */

    if (lexer.match(Lexer.NUM_OR_ID)) {
      lexer.advance();
    } else if (lexer.match(Lexer.LP)) {
      lexer.advance();
      expression();
      if (lexer.match(Lexer.RP)) {
        lexer.advance();
      } else {
        /*
         * 有左括号但没有右括号，错误
         */
        isLegalStatement = false;
        System.out.println("line: " + lexer.yylineno + " Missing )");
        return;
      }
    } else {
      /*
       * 这里不是数字，解析出错
       */
      isLegalStatement = false;
      System.out.println("illegal statements");
      return;
    }
  }

}

```



```java
package com.mycompiler.third;

import java.util.Scanner;

public class Lexer {
  public static final int EOI = 0;
  public static final int SEMI = 1;
  public static final int PLUS = 2;
  public static final int TIMES = 3;
  public static final int LP = 4;
  public static final int RP = 5;
  public static final int NUM_OR_ID = 6;
  public static final int UNKNOWN_SYMBOL = 7;

  private int lookAhead = -1;

  public String yytext = "";
  public int yyleng = 0;
  public int yylineno = 0;

  private String input_buffer = "";
  private String current = "";

  private boolean isAlnum(char c) {
    if (Character.isAlphabetic(c) == true || Character.isDigit(c) == true) {
      return true;
    }

    return false;
  }

  private int lex() {

    while (true) {

      while (current == "") {
        Scanner s = new Scanner(System.in);
        while (true) {
          String line = s.nextLine();
          if (line.equals("end")) {
            break;
          }
          input_buffer += line;
        }
        s.close();

        if (input_buffer.length() == 0) {
          current = "";
          return EOI;
        }

        current = input_buffer;
        ++yylineno;
        current.trim();
      } // while (current == "")

      if (current.isEmpty()) {
        return EOI;
      }

      for (int i = 0; i < current.length(); i++) {

        yyleng = 0;
        yytext = current.substring(0, 1);
        switch (current.charAt(i)) {
        case ';':
          current = current.substring(1);
          return SEMI;
        case '+':
          current = current.substring(1);
          return PLUS;
        case '*':
          current = current.substring(1);
          return TIMES;
        case '(':
          current = current.substring(1);
          return LP;
        case ')':
          current = current.substring(1);
          return RP;

        case '\n':
        case '\t':
        case ' ':
          current = current.substring(1);
          break;

        default:
          if (isAlnum(current.charAt(i)) == false) {
            return UNKNOWN_SYMBOL;
          } else {

            while (i < current.length() && isAlnum(current.charAt(i))) {
              i++;
              yyleng++;
            } // while (isAlnum(current.charAt(i)))

            yytext = current.substring(0, yyleng);
            current = current.substring(yyleng);
            return NUM_OR_ID;
          }

        } // switch (current.charAt(i))
      } // for (int i = 0; i < current.length(); i++)

    } // while (true)
  }// lex()

  public boolean match(int token) {
    if (lookAhead == -1) {
      lookAhead = lex();
    }

    return token == lookAhead;
  }

  public void advance() {
    lookAhead = lex();
  }

  public void runLexer() {
    while (!match(EOI)) {
      System.out.println("Token: " + token() + " ,Symbol: " + yytext);
      advance();
    }
  }

  private String token() {
    String token = "";
    switch (lookAhead) {
    case EOI:
      token = "EOI";
      break;
    case PLUS:
      token = "PLUS";
      break;
    case TIMES:
      token = "TIMES";
      break;
    case NUM_OR_ID:
      token = "NUM_OR_ID";
      break;
    case SEMI:
      token = "SEMI";
      break;
    case LP:
      token = "LP";
      break;
    case RP:
      token = "RP";
      break;
    }

    return token;
  }
}

```



```java
package com.mycompiler.third;

public class Parser {
  private Lexer lexer;

  String[] names = { "t0", "t1", "t2", "t3", "t4", "t5", "t6", "t7" };
  private int nameP = 0;

  private String newName() {
    if (nameP >= names.length) {
      System.out.println("Expression too complex: " + lexer.yylineno);
      System.exit(1);
    }

    String reg = names[nameP];
    nameP++;

    return reg;
  }

  private void freeNames(String s) {
    if (nameP > 0) {
      names[nameP] = s;
      nameP--;
    } else {
      System.out.println("(Internal error) Name stack underflow: " + lexer.yylineno);
      ;
    }
  }

  public Parser(Lexer lexer) {
    this.lexer = lexer;
  }

  public void statements() {
    String tempvar = newName();
    expression(tempvar);

    while (lexer.match(Lexer.EOI) != false) {
      expression(tempvar);
      freeNames(tempvar);

      if (lexer.match(Lexer.SEMI)) {
        lexer.advance();
      } else {
        System.out.println("Inserting missing semicolon: " + lexer.yylineno);
      }
    }
  }

  private void expression(String tempVar) {
    String tempVar2;
    term(tempVar);
    while (lexer.match(Lexer.PLUS)) {
      lexer.advance();
      tempVar2 = newName();
      term(tempVar2);
      System.out.println(tempVar + " += " + tempVar2);
      freeNames(tempVar2);
    }
  }

  private void term(String tempVar) {
    String tempVar2;

    factor(tempVar);
    while (lexer.match(Lexer.TIMES)) {
      lexer.advance();
      tempVar2 = newName();
      factor(tempVar2);
      System.out.println(tempVar + " *= " + tempVar2);
      freeNames(tempVar2);
    }

  }

  private void factor(String tempVar) {
    if (lexer.match(Lexer.NUM_OR_ID)) {
      System.out.println(tempVar + " = " + lexer.yytext);
      lexer.advance();
    } else if (lexer.match(Lexer.LP)) {
      lexer.advance();
      expression(tempVar);
      if (lexer.match(Lexer.RP)) {
        lexer.advance();
      } else {
        System.out.println("Missmatched parenthesis: " + lexer.yylineno);
      }
    } else {
      System.out.println("Number or identifier expected: " + lexer.yylineno);
    }
  }
}

```



直接先把代码跑起来 结合老师的视频 看看效果 

后边课程再慢慢悟吧！