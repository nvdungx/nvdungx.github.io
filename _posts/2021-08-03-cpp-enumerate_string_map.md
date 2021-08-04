---
layout: post
title: C++ enumerate
date: 2021-08-03 12:00:00
categories: [Programming]
tags: [C++, beginner]
last_modified_at: 2021-08-03
---

<p class="message">
  Print enumerate data in C++
</p>
{% highlight C++ %}
#include <vector>
class EnumMap
{
private:
    std::vector<const char*> table;
    const char* ERROR_STRING = "OUT_OF_RANGE";
public:
    EnumMap(std::vector<const char*>&& input_table)
      : table(std::move(input_table)) {}
    ~EnumMap() {}
    const std::string& operator[](ssize_t idx) const
    {
        if ((idx < 0) || ((size_t)idx > table.size()))
        {
            return ERROR_STRING;
        }
        else
        {
            return table[idx];
        }
    }
};
typedef enum {
  none = 0,
  pass,
  fail
} enVerdictValueType;
const EnumMap VERDICT_STR({"NONE", "PASS", "FAIL"});
>> std::cout << VERDICT_VAL[pass] << std::endl;
{% endhighlight %}
