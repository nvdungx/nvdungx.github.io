---
layout: post
title: Embedded Coding Qualities
date: 2021-09-12 00:00:00
categories: [software]
tags: [development, code quality]
last_modified_at: 2021-09-12
---

<hr>

>  Creating source code (code implementation) is an inevitable task for developing embedded software. Success or failure of this task greatly affects the quality of the resulting software.  
>  C language, the most commonly used programming language for embedded software development, is said to give the programmers a relatively extensive writing flexibility.  
>  The quality of programs written in C thus tends to reflect quite clearly the difference in coding skill level between seasoned and less experienced programmers. It is undesirable to have source code varying largely in quality, depending on the programmers’ individual coding skills and experience.  
>  To prevent this risk from leading into serious quality issues, forward-thinking companies are working proactively toward standardization of their source codes by establishing coding standards or conventions to be followed organization-wide or group-wide

# Embedded Coding Quality Concepts

**1. Reliability:**
Practices to improve the reliability of software that has been developed fall under this
category. Main points taken into consideration include:
- Minimizing problems arising while using the software.
- Increasing tolerability against bugs and interface violation.

**2. Maintainability:**
Practices to create source code that is easy to modify and maintain fall under this category.
Main points taken into consideration include:
- Making the code easy to understand and modify.
- Minimizing the impact of modifications on the entire code.
- Making the modified code easy to check.

**3. Portability:**
Practices to port the software program that has been created on the assumption of being
used to operate under a certain environment to another environment as efficiently as
possible without error.

**4. Efficiency:**
Practices to effectively utilize the performance and resources of the software that has been
developed fall under this category. Main points taken into consideration include:
- Coding that is processing time-conscious.
- Coding that takes account of memory size.

![qualities](/assets/img/blogs/2021_09_12/qualities.png)
<hr>

# Detail Rules
> Term "areas": used to specify general variables(local, global), arguments, arrays, ptr,...  
> Side-effect: processing that cause changes to a state of execution environment. The following processings appliy: reference and change to volatile data, change to data, change to files, and function-calls that perform these operations.

### I. Reliability
> A large number of embedded software is incorporated into products and used to support our daily lives in various situations. Consequently, the level of reliability demanded to quite a number of embedded software is extremely high.  
> Software reliability requires the software to be capable of not behaving wrongly (not causing failure), not affecting the functionality of the entire software and system in case of malfunction, and promptly restoring its normal behavior after a malfunction occurs.  
> At the source code level, the point to be noted in regard to software reliability is the need of contriving methods to avoid coding that may cause such malfunctions as much as possible

#### 1. *Initialize areas and use them by taking their sizes into consideration*
  * Automatic variables shall be initialized at the time of declaration, or the initial values shall be assigned just before using them
  * Integer addition to or subtraction from (including ++ and --) pointers shall not be made; Array format with [] shall be used for references and assignments to the allocated area.
  * Integer addition to or subtraction from (including ++ and --) pointers shall be made only when the pointer points to the array and the result must be pointing within the range of the array.

#### 2. *Use data by taking their ranges, sizes and internal representations into consideration.*
  * *Memcmp* shall not be used to compare structures and unions (structures and unions may contain unused areas. Since the values in the areas are unknown, memcmp should not be used).
  * Floating-point type, values written in the source code do not exactly match with those actually implemented => don't use it for equal comparison or counter.
  * In C language, true is represented by any non-zero value, not necessarily 1 => don't use logical compare with TRUE (equal 1)
  * Use same type in comparison or arithmetic operations, use explicit cast(cast the involved operant to target operant's type) and beware of overflow.
  * Use explicit cast to target type when operation involve one’s complement (~) or left shift (&lt;&lt;) (since these operation could cause signed bit turn on)
  * Data used as bit sequences shall be defined with unsigned type.
  * Pointer type shall not be converted to other pointer type or integer type, and vice versa, with the exception of mutual conversion between “pointer to data” type and “pointer to void*” type.
  * A cast shall not be performed that removes any const or volatile qualification from the type addressed by a pointer.

#### 3. *Write in a way that ensures intended behavior.*
  * Write in a way that is conscious of area size (i.e. no function with variable number of arguments, size of array shall always be specified).
  * Prevent operations that may cause runtime error from falling into error cases (i.e. always check for zero for division operation and nullptr with dereference operation).
  * Check the interface restrictions when a function is called (i.e. always check function input parameters are in constrain range, check return value when called function could return error information).
  * Do not perform recursive calls (functions should not call themself directly or indirectly because recursive's stack size can not be predicted at runtime).
  * All branch condition should be write explicitly(i.e. if-else if-else, switch-case-default), in case of else, default branch that is not reached normally then a comment shall be added to explain.
  * Equality operators (==) or inequality operators (!=) shall not be used for comparisons of loop counters. (<=, >=, <, or > shall be used.) (if loop counter change is not 1 then ==, != could cause infinite loop).
  * Pay attention to the order of evaluation, be explicit, split the operation that could cause possible side-effect (ex: f (x, x++); < this could cause side-effect since the compiler does not guarantee the order of execution from left or right of arguments).

### II. Maintainability
> Many embedded software developments require maintenance tasks, including the modification of the software that has already been developed. There are various reasons for maintenance. For example, maintenance becomes necessary:  
  * When a bug is found in one part of the released software and must be modified
  * When a new function is added to existing software in reWhen any kind of additional work is carried out on the already developed software as in the above examples, it is important to perform such work as accurately and efficiently as possible to maintain the quality of the software.

#### 1. *Keep in mind that others will read the program.*
  * Do not leave unused descriptions (remove unused things: vars, args, code, comment).
  * Do not mix declared variables with init value and without init value.
  * Use suffix in upper case (L, U) for constant number.
  * Use ( ) to clearly specify the operator precedence.
  * Always use preceding & operator to get function address.
  * One variable used for one purpose.
  * Do not reuse name in different scope.
  * The right-hand operand of a logical && or &#124;&#124; operator shall not contain side effects (The right-hand side of && or &#124;&#124; operators may not be executed, depending on the result of the condition of their left-hand side)
  * Do not embed magic numbers, meaningful constant shall be defined as macro.
  * Read-only areas shall be declared as const type.
  * Areas that may be updated by other execution units shall be declared as volatile.

#### 2. *Write in a style that can prevent modification errors.*
  * If arrays and structures are initialized with values other than 0, their structural form shall be indicated by using braces ‘{ }’. Data shall be described without any omission, except when all values are 0.
  * The body of if, else if, else, while, do, for, and switch statements shall be enclosed into blocks.
  * Variables used only in one function shall be declared within the function.
  * Variables accessed by several functions defined in the same file shall be declared with static in the file scope.
  * Functions that are called only by functions defined in the same file shall be static.
  * enum shall be used rather than #define when defining related constants.

#### 3. *Write programs simply.*
  * For any iteration statement, there shall be at most one break statement used for loop termination.
  * The goto statement shall not be used.
  * A function shall end with one return statement (except for the case of recovery from abnormality).
  * Limit the number of side effects per statement to one.
  * Multiple assignments shall not be written in one statement, except when the same value is assigned to multiple variables.
  * Write expressions that differ in purpose separately.
  * Numeric variables being used within a for loop for iteration counting shall not be modified in the body of the loop.
  * Do not use complicated pointer operations.

#### 4. *Write in a unified style.*
  * Unify the coding styles. (brace position {}, space, tab usage)
  * Unify the style of writing comments. (comment format)
  * Unify the naming conventions (variable, function, file, function should be verb to describe operation, variable should be noun to describe data content).
  * Unify the contents to be described in a file and the order of describing them, example order content in header file:
    1. File header comment
    2. Inclusion of system headers
    3. Inclusion of user defined headers
    4. #define macros
    5. #define function macros
    6. typedef definitions (type definitions for basic types such as int or char)
    7. enum tag definitions (together with typedef )
    8. struct/union tag definitions (together with typedef)
    9. extern variable declarations
    10. Function prototype declarations
    11. Inline function
  * Only declarations or type definitions should be described in a header file.
  * Header files shall has header guard macro to prevent redundant inclusions.
  * In a function prototype declaration, all the parameters shall be named. (C allow to omit parameter name in prototype declaration but should not be used).
  * Unify the style of writing declarations, example order content in source file:
    1. File header comment
    2. Inclusion of system headers // not to include unnecessary items
    3. Inclusion of user-defined headers // not to include unnecessary items
    4. #define macros used only in this file  // should be avoid if possible
    5. #define function macros used only in this file
    6. typedef definitions used only in this file
    7. enum tag definitions used only in this file
    8. struct/union tag definitions used only in this file  // should be avoid if possible
    9. static variable declarations shared in this file
    10. static function declarations
    11. Variable definitions
    12. Function definitions
  * Unify the style of writing null pointers.
  * NULL shall be used for the null pointer. NULL shall not be used for anything other than the null pointer
  * Unify the style of writing preprocessor directives. 
  * The body and parameters of a macro that includes operators shall be enclosed with parentheses ( )
  * #if defined(macro_name) or #if defined macro_name shall be used to check whether the macro name has already been defined by #if or #elif
  * #else , #elif or #endif that correspond to #ifdef, #ifndef or #if shall be described in the same file, and《their correspondence relationship shall be clearly stated with a comment defined in the project》.
  * #undef shall not be used.
  * Controlling expression of #if or #elif preprocessing directive shall be evaluated as 0 or 1.
#### 5. *Write in a style that makes testing easy.*
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
  * Dynamic memory shall not be used.

### III. Portability
> One of the distinctive aspects of embedded software is that there are diverse options in the platform used for software operation. This also means that there are many possible combinations of MPU options and OS options to select the hardware and software platforms from. As the number of functionalities realized by the embedded software increases, opportunities to port the existing software to other platforms by modifying or remodeling it to make it compatible with multiple platforms are also on the rise.  
Due to this trend, software portability is becoming an extremely important element also at the source code level. In particular, writing in a style that is implementation-dependent is one of the most common mistakes made on a regular basis.

#### 1. *Write in a style that is not dependent on the compiler.*
  * Do not use functionalities that are advanced features(not specified in the language standard) or implementation-defined.
  * Use only the characters and escape sequences defined in the language standard.
  * Confirm and document data type representations, behavioral specifications of advanced functionalities and implementation-dependent parts.
  * For source file inclusion, confirm the implementation dependent parts and write in a style that is not implementation-dependent.
  * Write in a style that does not depend on the environment used for compiling. (ex: no absolute path)

#### 2. *Localize the code that has a problem with portability.*
  * When assembly language programs are called from C language, expressing them as functions or inline functions of C language that contain only inline assembly language code or describing them using macros.
  * The basic types (char, int, long, long long, float , double and long double) shall not be used. Instead, the types defined by typedef shall be used (i.e. int8_t int16_t int32_t int64_t uint8_t uint16_t uint32_t uint64_t)

### IV. Efficiency
> Embedded software is characteristic for being embedded in a product and operating together with hardware to serve its purposes in the real world. The increasing demand for further product cost reduction has imposed various restrictions, not only on, such as, MPU or memory, but also on software.  
In addition, requirements, such as, on real-time property have placed stricter time constraints that need to be met. Embedded software must therefore be coded with particular attention on resource efficiency like efficient use of memory and time efficiency that takes account of time performance.

#### *Write in a style that takes account of resource and time efficiencies.*
  * Macro functions shall be used only in parts related to speed performance. (Function is safer than macro function. So, use function as much as possible, inline function can be one way of preventing the processsing speed from slowing down. But since inlining is implementation-dependent, use macro function instead)
  * Instead of structures, pointers to structures shall be used as function parameters.
  * The policy of selecting either switch or if statement shall be determined and defined by taking readability and efficiency into consideration.
    * switch statements often provide higher readability than if statements
    * recent compilers tend to output optimized code using, such as, table jump or binary search when they process switch statements

<hr>

# Typical Coding Errors in Embedded Software
1. Meaningless expressions and statements
  * Writing statements that are not executed
  * Writing statements whose execution result is not used
  * Writing expressions whose execution result is not used (ex: return cnt++;)
2. Wrong expressions and statements
  * Incorrect range specification (ex: if (0 < x < 10))
  * Comparing outside the range (ex: unsigned char uc; if (uc == 256))
  * String comparisons cannot be performed with == operation (this is embedded C not C++)
  * Inconsistency between a function type and return statement of the function

3. Wrong memory usage
  * Reference and update outside the array bounds (ex: char var1[N]; var1[-1] = 0; var1[N] = 0; /* error */)
  * Passing the address of an automatic variable to the caller mistakenly
  * Referencing memory after being freed as dynamic memory
  * Writing into string literals mistakenly
  * Specifying copy sizes mistakenly

4. Errors due to misunderstanding in logical expressions
  * Using a logical product mistakenly instead of a logical sum or other way around (&& , &#124;&#124; )
  * Using a bitwise operation mistakenly instead of a logical operation

5. Mistakes due to typos
  * Writing = operator instead of == operator (ex: if (x = 0) to avoid this case always let constant to  be 1st operant if (0 = x) then compiler shall detect this error)

6. Wrong descriptions that do not cause errors in some compilers
  * Macro with the same name that has multiple definitions
  * Writing into the const area mistakenly

> Ref: MISRA C guidelines