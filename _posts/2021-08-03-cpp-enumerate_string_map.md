---
layout: post
title: C++ enumerate
date: 2021-08-03 12:00:00
categories: [programming]
tags: [C++, beginner]
last_modified_at: 2021-12-16
---

## Mapping enumerate value to a string
  In C, normally when you want to map a enumerate value to a string, to print it,
we can implement like this.
{% highlight C %}
typedef enum {
  none = 0,
  pass,
  fail
} enVerdictValueType;
const char *VERDICT_STR[] = {
  [none] = "NONE",
  [pass] = "PASS",
  [fail] = "FAIL"
}
void func(enVerdictValueType val)
{
  // because of enum type, some time we could push int value to variable "val"
  // incase of val receive from other source, it could cause out of array access if we don't check it carefully
  printf("VERDICT: %s\r\n", VERDICT_STR[val]);
}
{% endhighlight %}

  Let try to improve above print enum implementation in C++.

{% highlight C++ %}
#include <vector>
// create a general class that will do the enumerate mapping with string and do the checking
class EnumMap
{
private:
    // vector to store the const string for enum map
    std::vector<const char*> table;
    // string to print when input enum value is out of index range
    const char* ERROR_STRING = "OUT_OF_RANGE";
public:
    // && is rvalue reference, specify rvalue because we initialize
    // the string vector table with rvalue
    EnumMap(std::vector<const char*>&& input_table)
      : table(std::move(input_table)) {}
    ~EnumMap() {}
    // override the operator []
    const char* operator[](ssize_t idx) const
    {
        // index checking
        if ((idx < 0) || ((size_t)idx > table.size()))
        {
            // return ERROR string incase of input value is not exist in enum type
            return ERROR_STRING;
        }
        else
        {
            // return corresponding string
            return table[idx];
        }
    }
};

// here is the part we using
// create enum
typedef enum {
  none = 0,
  pass,
  fail
} enVerdictValueType;
// create corresponding enum string map with easier syntax
const EnumMap VERDICT_STR({"NONE", "PASS", "FAIL"});
// use it
>> std::cout << VERDICT_STR[pass] << std::endl;
{% endhighlight %}

How about when you have a enumerate with its value does not span continuously,
We could modify the EnumMap to use std::unordered_map<int, std::string> instead of std::vector<string>
{% highlight C++ %}
#include <unordered_map>

typedef enum {
  none = 0,
  pass = 22,  // << weird, but could happen
  fail = 12
} enVerdictValueType;
const EnumMap VERDICT_STR({
  {none, "NONE"}, // << have to provide the index corresponding with string
  {pass, "PASS"},
  {fail, "FAIL"}});

class EnumMap
{
private:
    // unordered_map to store the const string for enum map
    std::unordered_map<int, const char*> table;
    // string to print when input enum value is not existed
    const char* ERROR_STRING = "OUT_OF_RANGE";
public:
    // && is rvalue reference, specify rvalue because we initialize
    // the string vector table with rvalue
    EnumMap(std::unordered_map<int, const char*>&& input_table)
      : table(std::move(input_table)) {}
    ~EnumMap() {}
    // override the operator []
    const char* operator[](ssize_t idx) const
    {
        // index checking
        if (0 == table.count(idx))
        {
            // return ERROR string incase of input value is not exist in enum type
            return ERROR_STRING;
        }
        else
        {
            // return corresponding string
            return table.at(idx);
        }
    }
};
{% endhighlight %}

Above class implementation help us initialize enumerate string map easier and less error-prone.
