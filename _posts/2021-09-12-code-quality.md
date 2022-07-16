---
layout: post
title: Embedded Coding Qualities
date: 2021-09-12 00:00:00
categories: [software]
tags: [beginner, code quality]
last_modified_at: 2023-02-15
---

> Creating source code (code implementation) is an inevitable task for developing embedded software. Success or failure of this task greatly affects the quality of the resulting software.  
>C language, the most commonly used programming language for embedded software development, is said to give the programmers a relatively extensive writing flexibility.  
>The quality of programs written in C thus tends to reflect quite clearly the difference in coding skill level between seasoned and less experienced programmers. It is undesirable to have source code varying largely in quality, depending on the programmers’ individual coding skills and experience.  
>To prevent this risk from leading into serious quality issues, standardization of source codes by establishing coding standards or conventions to be followed organization-wide or group-wide is necessary.

![sw_product](/assets/img/blogs/2021_09_12/coding_quality_product.jpg)

 ISO/IEC 25010 defines the quality of software product by breaking it down into eight characteristics (quality characteristics):
 **“reliability”, “maintainability”, “portability”, “efficiency”, “security”, “functionality”, “usability” and “compatibility”**.  
 Among them, **“functionality”, “usability” and “compatibility”** are considered to be the three quality characteristics that
 **should be addressed at an early stage**, preferably before moving on to the **design phases** in the upstream process.  
Whereas, **“reliability”, “maintainability”, “portability”, and “efficiency”** are considered to be the quality characteristics
 that **have close relevance with the development of high-quality source code** and should therefore be examined in depth during the coding phase.  
 **“Security”**, which has been defined as the quality **sub-characteristic of “functionality”** in the
previous standard, ISO/IEC 9126-1, is considered basically as a quality characteristic that **is relevant
in the design phase**, **but coding such as for avoiding stack overflow can also affect security**.  
For more information on coding practices related to security, please refer to “CERT C Secure Coding Standard”.

![quality_chracteristics](/assets/img/blogs/2021_09_12/coding_quality_characteristics.jpg)

# Embedded Coding Quality Concepts
 This blog shall address 4 coding quality characteristics as below and corresponding rules should be apply to achieve them.

<b style="color:#dc6300;font-size:1.25rem"><u>1. Reliability:</u></b>  
Degree to software performs specified functions under specified conditions for a specified period of time
  - <b style="color:#dc6300">Maturity</b>: low occurrence of bugs through continued use  
  - <span><strike>Availability</strike></span>: N/A (system level characteristic: operational and accessible when required for use)
  - <b style="color:#dc6300">Fault Tolerance</b>: software tolerate against bugs and interface violations, etc
  - <span><strike>Recoverability</strike></span>: N/A (system level characteristic: in the event of an interruption or a failure, system can recover the data directly affected and re-establish the desired state of the system)

<b style="color:#009ddc;font-size:1.25rem"><u>2. Maintainability:</u></b>  
Degree of effectiveness and efficiency with which software can be modified by the intended maintainers
  - <b style="color:#009ddc">Modularity</b>: components are composed such that a change to one component of the code has minimal impact(loose coupling) on other components.
  - <b style="color:#009ddc">Reusability</b>: degree to which a code can be used in other programs.
  - <b style="color:#009ddc">Analysability</b>: easiness of understanding the code.
  - <b style="color:#009ddc">Modifiability</b>: easiness of modifying the code, and lowness of impact from modifications.
  - <b style="color:#009ddc">Testability</b>: easiness of testing and debugging the modified code.

<b style="color:#00dc54;font-size:1.25rem"><u>3. Portability:</u></b>  
Degree of effectiveness and efficiency with which software can be transferred from one hardware, other operational or usage environment to another.
  - <b style="color:#00dc54">Adaptability</b>: easiness of adapting to different environments.
  - <span><strike>Installability</strike></span>: N/A (system level characteristic: degree of effectiveness and efficiency with which system can be successfully installed and/or uninstalled in a specified environment).
  - <span><strike>Replaceability</strike></span>: N/A (system level characteristic: degree to which a product can be replaced by another specified software product for the same purpose in the same environment).

<b style="color:#b568d8;font-size:1.25rem"><u>4. Efficiency:</u></b>  
Performance relative to the amount of resources used under stated conditions 
  - <b style="color:#b568d8">Time Behaviour</b>: efficiency with regard to processing time.
  - <b style="color:#b568d8">Resource Utilization</b>: efficiency with regard to resources.
  - <span><strike>Capacity</strike></span>: N/A (system level characteristic: degree to which the maximum limits of a product or system parameter meet requirements)

<b style="color:red;font-size:1.25rem"><u>5. Security:</u></b>  
Degree to which software protects information and data so that other software have the degree of data access appropriate to their types and levels of authorization.
  - <b style="color:red">Confidentiality</b>: degree of certainty that data are accessible only to those authorized to have access.
  - <b style="color:red">Integrity</b>: degree of prevention of unauthorized access to, or modification of, computer programs or data.
  - <span><strike>Nonrepudiation</strike></span>: N/A (system level characteristic: degree to which actions or events can be proven to have taken place, so that the events or actions cannot be repudiated later)
  - <span><strike>Accountability</strike></span>: N/A (system level characteristic: degree to which the actions of an entity can be traced uniquely to the entity)
  - <span><strike>Authenticity</strike></span>: N/A (system level characteristic: degree to which the identity of a subject or resource can be proved to be the one claimed)

![quality_concept](/assets/img/blogs/2021_09_12/qualities.png)

# Detail Rules
> Term "areas": used to specify the usage of memory in general such as variables(local, global), arguments, arrays, ptr,...  
> 
> Side-effect: processing that cause changes to a state of execution environment. The following process apply: reference and change to volatile data, change to data, change to files, and function-calls that perform these operations.
> 

## I. Reliability
> A large number of embedded software is incorporated into products and used to support our daily lives in various situations. Consequently, the level of reliability demanded to quite a number of embedded software is extremely high.  
> Software reliability requires the software to be capable of not behaving wrongly (not causing failure), not affecting the functionality of the entire software and system in case of malfunction, and promptly restoring its normal behavior after a malfunction occurs.  
> At the source code level, the point to be noted in regard to software reliability is the need of contriving methods to avoid coding that may cause such malfunctions as much as possible

Practices to **improve the reliability of software** that has been developed fall under this
category. Main points taken into consideration include:
- Minimizing problems arising while using the software.
- Increasing tolerability against bugs and interface violation.

### 1. *Initialize areas and use them by taking their sizes into consideration*
  * Variables shall be **initialized at the time of declaration**, or the initial values shall **be assigned just before using them**.
{% highlight c %}
/* DO */
int var1;
int var2 = 0; /* Initialize at the time of declaration */
var1 = 0;
func(var1, var2);

/* DON'T */
int var1;
func(&var1); /* Use without initialize */
{% endhighlight %}
  * Arrays with specified number of elements shall be **initialized with values that match the number of the elements**.
{% highlight c %}
/* DO */
char var[] = "abc"; - or - char var[4] = "abc"; /* size 4 = 3 char and NULL char for string termination */

/* DON'T */
char var[3] = "abc";  /* No string termination */
{% endhighlight %}
  * Integer addition to or subtraction from (including `++` and `--`) pointers **shall be made only when the pointer points to the array and the result must be pointing within the range of the array**. Otherwise, array format with `[]` shall be used for references and assignments to the allocated area.
{% highlight c %}
int data[10];
int *p = data;
/* DO */
data[0] = 10;
*(p + 9) = 10;

/* DON'T */
*(p + 20) = 10;
{% endhighlight %}
  * **Comparison or subtraction between pointers** shall be **used only when the two pointers are both pointing at** either the **elements in the same array** or the **members of the same structure**.
{% highlight c %}
int var1[10], var2[10];
int *p1, *p2, *p3;
ptrdiff_t offset;
p1 = &var1[5];
p2 = &var1[2];
p3 = &var2[3]
/* DO */
offset = p1 - p2;

/* DON'T */
offset = p1 - p3;
{% endhighlight %}

### 2. *Use data by taking their ranges, sizes and internal representations into consideration.*
  * Floating-point type, values written in the source code do not exactly match with those actually represented by the hardware ⟹ don't use it for equal comparison or counter.
{% highlight c %}
float var1 = 1.13, var2 = 1.13;
/* DON'T */
if (var1 == var2) { ... }
for (var1 = 0.0; var1 < 1.0; var1 += 0.1) { ... }
{% endhighlight %}
  * *Memcmp* should not be used to compare structures and unions (structures and unions may contain unused areas because of default memory padding for memory alignment. Since the values in the areas are unknown, memcmp should not be used). However, memcmp could still be used when struct packing is explicitly specified (e.g. in GCC compiler by adding `__attribute__((__packed__))` to struct keyword).
{% highlight c %}
struct TAG { char c; long l; };
struct TAG var1, var2;
/* DO */
if (var1.c == var2.c && var1.l == var2.l) { ... }

/* DON'T */
if (memcmp(&var1, &var2, sizeof(var1)) == 0) { ... }
{% endhighlight %}
  * In C language, true is represented by any non-zero value, not necessarily 1 ⟹ don't use logical compare with TRUE(not 0), use comparison with FALSE(0) instead.
{% highlight c %}
#define TRUE 1
#define FALSE 0
/* func1 may return a value other than 0 and 1 */
/* DO */
if (func1() != FALSE) { ... } - or - if (func1()) { ... }
/* DON'T */
if (func1() == TRUE) { ... }
{% endhighlight %}
  * Use same type in comparison or arithmetic operations, use explicit cast(cast the involved operant to target operant's type) and beware of overflow.
{% highlight c %}
void func(unsigned int arg) {
  int i, var1, var2;
  long result1;
  float result2;
  /* DON'T */
  for (i = 0; i < arg; i++) { ... } /* << infinite loop incase of arg larger than MAX value of signed int */
  result2 = var1 / var2; /* integer division, rounded result, losing data */
  result1 = var1 << var2; /* risk of losing data */
}
{% endhighlight %}
  * Use explicit cast to target type when operation involve one’s complement (`~`) or left shift (`<<`) (since these operation could cause signed bit turn on)
{% highlight c %}
uc = 0x0f;
/* DO */
if((unsigned char)(~uc) >= 0x0f) { ... }

/* DON'T */
if((~uc) >= 0x0f) { ... } /* It is not true */
{% endhighlight %}
  * The right-hand side of a shift operator shall be zero or more, and less than the bit width of the left-hand side.
{% highlight c %}
unsigned char a; /* 8 bits */
unsigned short b; /* 16 bits */
/* DO */
b = (unsigned short)a << 12; /* Clearly indicated that the operation is 16 bits */

/* DON'T */
b = a << 12; /* There may be an error in the shift count */
{% endhighlight %}
  * Pointer type shall not be converted to other pointer type or integer type with less data width than that of the pointer type, with the exception of mutual conversion between “pointer to data” type and “pointer to other data” type, and between “pointer to data” type and “pointer to void*” type.
{% highlight c %}
int *ip;
int (*fp)(void) ;
char *cp;
int i;
void *vp;

/* OK */
ip = (int*)vp;
i = (int)ip;
i = (int)fp;
cp = (char*)ip;

/* DON'T */
ip = (int*)cp;
c =(char) ip;
ip =(int*) fp;
{% endhighlight %}
  * A cast shall not be performed that removes any const or volatile qualification from the type addressed by a pointer.
{% highlight c %}
void func(char *);
const char *str;
void func2() {
  /* DON'T */
  func((char*)str);
}
{% endhighlight %}

### 3. *Write in a way that ensures intended behavior.*
  * Write in a way that is conscious of area size (i.e. no function with variable number of arguments, size of array shall always be specified, iteration conditions for a loop to sequentially access array elements shall include the decision to whether the access is within the range of the array or not.).
{% highlight c %}
/* DO */
char var1[MAX];
for (i = 0; i < MAX && var1[i] != 0; i++) { ... }

/* DON'T */
int func(int a, char b, ... ); /* << a variable number of arguments in the processing system, their use may cause stack overflow or other unexpected results */
{% endhighlight %}
  * Prevent operations that may cause runtime error from falling into error cases (e.g. always check for zero for division operation and `nullptr` with dereference operation).
{% highlight c %}
/* DO */
if (y != 0) ans = x/y;
if (p != NULL) *p = 1;
{% endhighlight %}
  * Check the interface restrictions when a function is called (e.g. always check function input parameters are in constrain range, check return value when called function could return error information).
{% highlight c %}
int func(int para) {
/* DO */
  if (!((MIN <= para) && (para <= MAX)))
  {
    return range_error;
  }
  /* Normal processing */
  ...
}
{% endhighlight %}
  * Do not perform recursive calls (functions should not call them self directly or indirectly because recursive's stack size can not be predicted at runtime).
{% highlight c %}
/* SHOULD NOT */
unsigned int calc(unsigned int n)
{
  if (n <= 1) {
    return 1;
  }
  return n * calc(n-1);
}
{% endhighlight %}
  * All branch condition should be write explicitly(i.e. `if-else` `if-else if`, `switch-case-default`), in case of else, switch-default branch that is not reached normally then a comment shall be added to explain.
{% highlight c %}
/* DO */
  if (var1 == 0) {
    …
  }
  else if (0 < var1) {
    …
  }
  else {
    /* Write an exception handling process */
    …
  }
  if (var1 == 0) {
    …
  }
  else {
    /* DO NOTHING */
  }
{% endhighlight %}
  * Equality operators (`==`) or inequality operators (`!=`) shall not be used for comparisons of loop counters. (`<=`, `>=`, `<`, or `>` shall be used.) (if the amount of change for the loop counter is **not 1** then `==`, `!=` could cause infinite loop).
{% highlight c %}
/* DO */
for (uint8_t i = 0; i < 9; i += 2) { ... }

/* DON'T */
for (uint8_t i = 0; i != 9; i += 2) { ... }
{% endhighlight %}
  * Pay attention to the order of evaluation, be explicit, split the operation that could cause possible side-effect (ex: `f (x, x++);` << this could cause side-effect since the compiler does not guarantee the order of execution from left or right of arguments).
{% highlight c %}
/* DO */
f (x, x);
x++;
/* or */
f (x + 1, x);
x++;

/* DON'T */
f (x, x++);
{% endhighlight %}

## II. Maintainability
> Many embedded software developments require maintenance tasks, including the modification of the software that has already been developed. There are various reasons for maintenance. For example, maintenance becomes necessary:  
  * When a bug is found in one part of the released software and must be modified.  
  * When a new function is added to existing software.  
>
> When any kind of additional work is carried out on the already developed software as in the above examples, it is important to perform such work as accurately and efficiently as possible to maintain the quality of the software.

Practices to create source code that is **easy to modify and maintain** fall under this category.
Main points taken into consideration include:
- Making the code easy to understand and modify (keep in mind that others will read and make modify to the program).
- Minimizing the impact of modifications on the entire code (keep it simple and modular).
- Making the modified code easy to check.

### 1. *Keep in mind that others will read the program.*
  * Do not leave unused descriptions (remove unused things: vars, args, typedefs, code, comment).
{% highlight c %}
/* Don't comment out section of code, if you want to keep the code for future reuse or reference */
/* DO */
#if 0
a++;
#endif
{% endhighlight %}
  * Do not mix declared variables with init value and without init value.
{% highlight c %}
/* DO */
int i, j;
int k = 0;

/* DON'T */
int i, j, k = 0;
{% endhighlight %}
  * Use suffix in upper case (`L`, `U`) for constant number.
{% highlight c %}
/* DO */
f = f + 1.0F;
if (ui < 0x8000U) { ... }

/* DON'T */
if (ui < 1l) { ... }
{% endhighlight %}
  * Use `( )` to clearly specify the operator precedence.
{% highlight c %}
/* DO */
if ((x > 0) && (x < 10))
if ((x != 1) && (x != 4) && (x != 10))
a = (b << 1) + c;
/* - or - */
a = b << (1 + c);

/* DON'T */
if (x > 0 && x < 10)
if (x != 1 && x != 4 && x != 10)
a = b << 1 + c;
{% endhighlight %}
  * Always use preceding & operator to get function address.
{% highlight c %}
/* DO */
void func(void);
void (*fp)(void) = &func;

/* DON'T */
void (*fp)(void) = func;
{% endhighlight %}
  * One variable used for one purpose (e.g. if a variable is declared to be used as counter in a loop, don't use it for other purpose in different part of the program).
{% highlight c %}
{% endhighlight %}
  * Do not reuse name in different scope (C language will not prevent you to have the same identifier in different namespace/scope but you should not do it).
{% highlight c %}
/* DON'T */
int var1;
void func(int arg1) {
  int var1;
  var1 = arg1;
  {
    int var1; /* The same name of a variable in the outer scope is used */
    ...
    var1 = 0; /* Intention of which var1 is assigned is unclear */
  }
}
{% endhighlight %}
  * The right-hand operand of a logical `&&` or `||` operator shall not contain side effects (The right-hand side of `&&` or `||` operators may not be executed, depending on the result of the condition of their left-hand side)
{% highlight c %}
/* DON'T */
#define STATUS_REG (*(volatile int *(0xF002001)))
/* read STATUS_REG operation might not be performed depend on result of precedence operation, same for function call */
if ((x != 0) && (STATUS_REG != 0)) {
  ...
}

/* DO */
if ((result != 0) && (x != 0)) {
  ...
}
{% endhighlight %}
  * Do not embed magic numbers, meaningful constant shall be defined as macro.
{% highlight c %}
/* DO */
#define MAXCNT 8
if (cnt == MAXCNT) { ... }

/* DON'T */
if (cnt == 8) { ... }
{% endhighlight %}
  * Read-only areas shall be declared as const type (a variable is only referenced and not modified, declaring it as const-qualified variable makes it clear that it is not modified).
{% highlight c %}
{% endhighlight %}
  * Areas that may be updated by other execution units shall be declared as volatile (volatile prohibit the compiler from optimizing them).
{% highlight c %}
volatile int x = 1;
while (x == 0) { /* << this operation shall not be optimized by compiler, compare operation shall always be performed. */
  /* x is not modified within the loop and is modified by other execution units */
}
{% endhighlight %}

### 2. *Write in a style that can prevent modification errors.*
  * If arrays and structures are initialized with values other than 0, their structural form shall be indicated by using braces `{ }`. Data shall be described without any omission, except when all values are 0.
{% highlight c %}
/* DO */
int arr1[2][3] = {  {0, 1, 2},
                {3, 4, 5} };
int arr2[3] = {1, 1, 0};

/* DON'T */
int arr1[2][3] = {0, 1, 2, 3, 4, 5};
int arr2[3] = {1, 1};
{% endhighlight %}
  * The body of `if`, `else if`, `else`, `while`, `do`, `for`, and `switch` statements shall be enclosed into blocks.
{% highlight c %}
/* DO */
if (x == 1) {
  func();
}

/* DON'T */
if (x == 1)
  func();
{% endhighlight %}
  * Variables used only in one function shall be declared within the function.
{% highlight c %}
{% endhighlight %}
  * Variables accessed by several functions defined in the same file shall be declared with static in the file scope (the fewer the global variables, the higher the readability of the entire program becomes).
{% highlight c %}
{% endhighlight %}
  * Functions that are called only by functions defined in the same file shall be static.
{% highlight c %}
{% endhighlight %}
  * `Enum` shall be used rather than `#define` when defining related constants. By defining related constants as `enum` type, and using this type, mistakes caused by the use of incorrect values can be prevented. While macro names defined by `#define` are expanded at the preprocessing stage and the compiler does not process those names, `enum` constants defined by `enum` declaration will be the names processed by the compiler. The names processed by the compiler are easier to debug, because they can be referenced.
during symbolic debugging.
{% highlight c %}
{% endhighlight %}

### 3. *Write programs simply.*
  * For any iteration statement, there shall be at most one break statement used for loop termination (keep only one return point for code block so it easier to trace)
{% highlight c %}
/* DON'T */
for (i=0; loop iteration condition; i++) {
  Iterated processing 1;
  if (termination condition1) {
    break;
  }
  if (termination condition1) {
    break;
  }
  Iterated processing 2;
}
{% endhighlight %}
  * The goto statement shall not be used (to prevent the program logic from becoming complex).
{% highlight c %}
{% endhighlight %}
  * A function shall end with one return statement (except for the case of recovery from abnormality).
{% highlight c %}
{% endhighlight %}
  * Multiple assignments shall not be written in one statement, except when the same value is assigned to multiple variables.
{% highlight c %}
/* OK */
x = y = 0;

/* DON'T */
y = (x += 1) + 2;
y = (a++) + (b++);
{% endhighlight %}
  * Write expressions that differ in purpose separately.
{% highlight c %}
/* DO */
for (i = 0; i < MAX; i++) {
  ...
  j++;
}

/* DON'T */
for (i = 0; i < MAX; i++, j++) { ... }
{% endhighlight %}
  * Numeric variables being used within a for loop for iteration counting shall not be modified in the body of the loop.
{% highlight c %}
/* DON'T */
for (i = 0; i < MAX; ) {
  ...
  i++;
}
{% endhighlight %}
  * Do not use complicated pointer operations.
{% highlight c %}
/* DON'T */
int ***p;
typedef char **strptr_t;
strptr_t *q;
{% endhighlight %}

### 4. *Write in a unified style.*
  * Unify the coding styles. (brace position `{}`, space, tab usage)
{% highlight c %}
{% endhighlight %}
  * Unify the style of writing comments. (comment format)
{% highlight c %}
{% endhighlight %}
  * Unify the naming conventions (variable, function, file, function should be verb to describe operation, variable should be noun to describe data content).
{% highlight c %}
{% endhighlight %}
  * Unify the contents to be described in a file and the order of describing them, example **order content in header file** as below:
{% highlight c %}
    1. File header comment
    2. Inclusion of system headers
    3. Inclusion of user defined headers
    4. #define macros
    5. #define function macros
    6. typedef definitions (type definitions for basic types such as int or char)
    7. enum tag definitions (together with typedef)
    8. struct/union tag definitions (together with typedef)
    9. extern` variable declarations
    10. Function prototype declarations
    11. Inline function
{% endhighlight %}
  * Only declarations or type definitions should be described in a header file.
{% highlight c %}
{% endhighlight %}
  * Header files shall has header guard macro to prevent redundant inclusions.
{% highlight c %}
#ifndef MYHEADER_H
#define MYHEADER_H
  Contents of the header file
#endif /* MYHEADER_H */
{% endhighlight %}
  * In a function prototype declaration, all the parameters shall be named. (C allow to omit parameter name in prototype declaration but should not be used).
{% highlight c %}
{% endhighlight %}
  * Unify the style of writing declarations, example **order content in source file** as below:
{% highlight c %}
    1. File header comment
    2. Inclusion of system headers /* not to include unnecessary items */
    3. Inclusion of user-defined headers /* not to include unnecessary items */
    4. #define macros used only in this file /* should be avoid if possible */
    5. #define function macros used only in this file
    6. typedef definitions used only in this file
    7. enum tag definitions used only in this file
    8. struct/union tag definitions used only in this file /* should be avoid if possible */
    9. static variable declarations shared in this file
    10. static function declarations
    11. Variable definitions
    12. Function definitions
{% endhighlight %}
  * Unify the style of writing null pointers. `NULL` shall be used for the null pointer. `NULL` shall not be used for anything other than the null pointer.
{% highlight c %}
{% endhighlight %}
  * Unify the style of writing preprocessor directives. The body and parameters of a macro that includes operators shall be enclosed with parentheses `( )`.
{% highlight c %}
{% endhighlight %}
  * `#if defined(macro_name)` or `#if defined macro_name` shall be used to check whether the macro name has already been defined by `#if` or `#elif`
{% highlight c %}
{% endhighlight %}
  * `#else` , `#elif` or `#endif` that correspond to `#ifdef`, `#ifndef` or `#if` shall be described in the same file, and《their correspondence relationship shall be clearly stated with a comment defined in the project》.
{% highlight c %}
{% endhighlight %}
  * Controlling expression of `#if` or `#elif` preprocessing directive shall be evaluated as 0 or 1.
{% highlight c %}
/* DON'T */
#define ABC 2
#if ABC
{% endhighlight %}

### 5. *Write in a style that makes testing easy.*
  * Write in a style that makes it easy to investigate the causes of problems when they occur.
    1. by isolating the debug descriptions using macro definitions
    2. by using assert macros for debugging purpose.
      {% highlight C %}
        #ifdef DEBUG
        #define DEBUG_PRINT(str) fputs(str, stderr)
        #else
        #define DEBUG_PRINT(str) ((void) 0) /* no action */
        #endif /* DEBUG */

        #ifdef NDEBUG
        #define assert(exp) ((void) 0)
        #else
        #define assert(exp) (void) ((exp)) || (_assert(#exp, __FILE__, __LINE__)))
        #endif
        void _assert(char *mes, char *fname, unsigned int lno) {
          fprintf(stderr, "Assert:%s:%s(%d)¥n", mes, fname, lno);
          fflush(stderr);
          abort();
        }
      {% endhighlight %}

  * Outputting logs after release
    1. When: Logs should be output not only when an abnormal condition is detected, but also at the timing of, such as, data communication with an external system (when key events occur)
    2. What: data values processed at that time, and information for tracing memory usage
    3. Localize the log information output as a macro or a function
{% highlight c %}
{% endhighlight %}

  * Dynamic memory shall not be used. Issues with dynamic memory:
    1. Buffer overflow: result of writing past the end of the buffer. If this overwrites adjacent data or executable code, this may result in erratic program behavior, including memory access errors, incorrect results, and crashes.
    2. Forgetting to initialize: acquired memory fill with trash value. 
    3. Memory leak: cause memory depletion and system malfunction.
    4. Use after return: reference to memory that has been deleted.

## III. Portability
> One of the distinctive aspects of embedded software is that there are diverse options in the platform used for software operation. This also means that there are many possible combinations of MCU options and OS options to select the hardware and software platforms from. As the number of functionalities realized by the embedded software increases, opportunities to port the existing software to other platforms by modifying or remodeling it to make it compatible with multiple platforms are also on the rise.  
Due to this trend, software portability is becoming an extremely important element also at the source code level. In particular, writing in a style that is implementation-dependent is one of the most common mistakes made on a regular basis.

Practices to **port the software program** that has been created on the assumption of being
used to operate under a certain environment **to another environment as efficiently as possible without error**.
- **Abstract** the source code implementation with layer of independent(e.g. independent in term of HW, compiler, operating system,...).
- **Loose coupling** between components in source code.

### 1. *Write in a style that is not dependent on the compiler.*
  * Do not use functionalities that are advanced features(not specified in the language standard) or implementation-defined (e.g. when specific implementation-defined features which behavior varies depending on the compiler is used, they should be clearly documented).
{% highlight c %}
{% endhighlight %}
  * Use only the characters and escape sequences defined in the language standard.
{% highlight c %}
/* DO */
char c = '\t'; /* horizontal tab character */

/* DON'T */
char c = '\e'; /* escape is sequence not defined in the language standard. It is not portable */
{% endhighlight %}
  * Confirm and document data type representations, behavioral specifications of advanced functionalities and implementation-dependent parts.
{% highlight c %}
{% endhighlight %}
  * For source file inclusion, confirm the implementation dependent parts and write in a style that is not implementation-dependent.
{% highlight c %}
{% endhighlight %}
  * Write in a style that does not depend on the environment used for compiling. (ex: no absolute path)
{% highlight c %}
{% endhighlight %}

### 2. *Localize the code that has a problem with portability.*
  * When assembly language programs are called from C language or keywords extended by the compiler expressing them as functions or inline functions of C language that contain only inline assembly language code or describing them using macros.
{% highlight c %}
/* DO */
#ifdef _HW_VARIANT_A_
#define SET_PORT1 asm(" st.b 1, port1")
#elif _HW_VARIANT_B_
#define SET_PORT1 <platformB specific asm instruction>
#endif
void f() {
  ...
  SET_PORT1;
  ...
}
{% endhighlight %}
  * The basic types (`char`, `int`, `long`, `long long`, `float` , `double` and `long double`) shall not be used. Instead, the types defined by typedef shall be used (i.e. `int8_t int16_t int32_t int64_t uint8_t uint16_t uint32_t uint64_t`)
{% highlight c %}
{% endhighlight %}

## IV. Efficiency
> Embedded software is characteristic for being embedded in a product and operating together with hardware to serve its purposes in the real world. The increasing demand for further product cost reduction has imposed various restrictions, not only on, such as, MCU or memory, but also on software.  
In addition, requirements, such as, on real-time property have placed stricter **time constraints that need to be met**. Embedded software must therefore be coded with particular attention on resource efficiency like **efficient use of memory and time efficiency** that takes account of time performance.

Practices to **effectively utilize the performance and resources** of the software that has been
developed fall under this category. Main points taken into consideration include:
- Coding that is processing time-conscious.
- Coding that takes account of memory size.

### *Write in a style that takes account of resource and time efficiencies.*
  * Macro functions shall be used only in parts related to speed performance. (Function is safer than macro function. So, use function as much as possible, inline function can be one way of preventing the processing speed from slowing down. But since inlining is implementation-dependent, use macro function instead)
{% highlight c %}
{% endhighlight %}
  * Operations that remain unchanged shall not be performed within an iterated process.
{% highlight c %}
/* DO */
var1 = func(); /* Function func returns the same result */
for (i = 0; (i + var1) < MAX; i++) { ... }

/* DON'T */
for (i = 0; (i + func()) < MAX; i++) { ... }
{% endhighlight %}
  * Instead of structures, pointers to structures shall be used as function parameters (if a structure is passed as a function argument, all the structure data are copied into the area for storing arguments when the function is called. If the size of the structure is large, it will become the cause of speed performance degradation).
{% highlight c %}
typedef struct stag {
  int mem1;
  int mem2;
} STAG;
/* DO */
int func (const STAG *p) {
  return p->mem1 + p->mem2;
}

/* DON'T */
int func (STAG x) {
  return x.mem1 + x.mem2;
}
{% endhighlight %}
  * The policy of selecting either switch or if statement shall be determined and defined by taking readability and efficiency into consideration.
    * switch statements often provide higher readability than if statements
    * recent compilers tend to output optimized code using, such as, table jump or binary search when they process switch statements
{% highlight c %}
{% endhighlight %}

<hr>

# Typical Coding Errors in Embedded Software
### 1. Meaningless expressions and statements  
Leaving statements or expressions that are not executed in the source code is likely to create misunderstanding that often leads to problems as a result. It is said that confusion tends to be caused especially when the source code is modified by engineers who are not the originator of that particular code.
  * Writing statements that are not executed.
  * Writing statements whose execution result is not used
  * Writing expressions whose execution result is not used (e.g. `return cnt++;`)
  * Values passed as arguments are not used

### 2. Wrong expressions and statements
To write proper source code, it must be written according to the grammar of the programming
language being used. But even programmers who are familiar with the programming language being used can make careless mistakes. Presented below are some examples of wrong expressions and statements that are often seen.
  * Incorrect range specification (e.g. `if (0 < x < 10)`)
  * Comparing outside the range (e.g. `unsigned char uc; if (uc == 256)`)
  * String comparisons cannot be performed with `==` operation (no operator overloading in C)
  * Inconsistency between a function type and return statement of the function
  {% highlight c %}
  int func1(int in) {
    if (in < 0) return; /* DON'T */
    return in ;
  }
  {% endhighlight %}

### 3. Wrong memory usage
One of the characteristics of C language is that memory can be handled directly. While this is a
very useful feature when creating embedded software, it also often causes incorrect operations and
must therefore be used carefully.
  * Reference and update outside the array bounds (e.g. `char var1[N]; var1[-1] = 0; var1[N] = 0; /* error */`)
  * Passing the address of an automatic variable to the caller mistakenly
  {% highlight c %}
    int *func(tag *p) {
      int x;
      p->mem = &x; /* The automatic variable memory area is referenced after the function return (risky) */
      return &x; /* The automatic variable memory area is referenced after the function return (risky) */
    }
    ...
    tag y;
    int *p;
    p = func(&y);
    *p = 10; /* Destroying invalid memory area */
    *y.mem = 20; /* Destroying invalid memory area */
    /* Areas for automatic variables or parameters are freed to the system when the function ends, and
     * may be reused for other purposes. If the address of an automatic variable is specified as a function
     * return value or set in an area that can be referenced by the caller, as shown in the above example,
     * unexpected faults may occur when the area that has been returned to the system is referenced or updated.
     */
  {% endhighlight %}
  * Referencing memory after being freed as dynamic memory
  * Writing into string literals mistakenly
  {% highlight c %}
    char *s;
    s = "abc"; /* The string literal may be in ROM area */
    s[0] = 'A'; /* Cannot be written */
  {% endhighlight %}
  * Specifying copy sizes mistakenly
  {% highlight c %}
    #define A 10
    #define B 20
    char a[A];
    char b[B];
    ...
    memcpy(a, b, sizeof(b));
    /* When one array is copied to another, it will corrupt the memory area if the copy
    is made in the size of the source that is larger than the size of the destination. */
  {% endhighlight %}

### 4. Errors due to misunderstanding in logical expressions
The use of logical operators is relatively error-prone. In situations where they are used,
 close attention must be given especially to the operation results, since in many cases, they lead to different subsequent processes.
  * Using a logical product mistakenly instead of a logical sum or other way around ( `&&` , `||` )
  {% highlight c %}
    int i, data[10], end = 0;
      for (i = 0; i < 10 || !end; i++) {
      data[i] = Value_assigned; /* risk of corrupting outside the area */
      if (termination_condition) {
        end = 1;
      }
    }
  {% endhighlight %}
  * Using a bitwise operation mistakenly instead of a logical operation
  {% highlight c %}
    if (logicalResult1 & logicalResult2) 
    /* above example showing that bitwise AND operator (&) has been written mistakenly instead
        of a logical product operator (&&). */
  {% endhighlight %}

### 5. Mistakes due to typos
Some operators in C language like = and == have completely different meaning even though they
do not differ that much. When writing these operators, sufficient attention must be given to prevent
careless mistakes or typos.
  * Writing `=` operator instead of `==` operator (e.g. `if (x = 0)` to avoid this case always let constant to  be 1st operant `if (0 = x)` then compiler shall detect this error)

### 6. Wrong descriptions that do not cause errors in some compilers
Each compiler has various characteristics of its own. Note that some compilers do not cause compile errors during compilation even if the program contains inappropriate descriptions.
  * Macro with the same name that has multiple definitions
  {% highlight c %}
    /* Depending on where AAA is referenced, what is expanded varies */
    #define AAA 100
    a = AAA; /* 100 is assigned */
    #define AAA 10
    b = AAA; /* 10 is assigned */
    /* Macro name defined by #define will not become a compile error in some compilers even when it is redefined without applying #undef beforehand. */
  {% endhighlight %}
  * Writing into the const area mistakenly
  {% highlight c %}
    void func(const int *p) {
    *p = 0; /* Writing into the const area (error) */
    /* Some compilers do not cause a compile error even if the const area is rewritten.
        Programmers should be careful not to rewrite the const area. */
  {% endhighlight %}
> Ref:  
> - MISRA C guidelines  
> - Embedded System development Coding Reference guide
