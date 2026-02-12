---
layout: post
title: Mastering Regular Expressions - tool that everyone should know
date: 2023-02-07 00:00:00
categories: [programming]
tags: [automation, intermediate, text processing]
last_modified_at: 2023-02-08
---

When I first started as a junior engineer on an MCAL (Microcontroller Abstraction Layer) development team, my responsibilities mainly involved testing and maintaining existing test scripts and environment structures. I vividly remember one specific instance where we updated a module's **Parameter Definition File (PDF)**. This change required us to update the entire test environment’s **Configuration Data Files (CDFs)**—the configured output `.arxml` files used in AUTOSAR MCAL module.

We had introduced a new **DEM (Diagnostic Event Manager)** error into the parameter definition of CAN module. Consequently, every CDF in the CAN module’s test environment needed an additional DEM reference added under the `DemEventParameterRefs` container as below:

{% highlight xml %}
<ECUC-REFERENCE-VALUE>
    <DEFINITION-REF DEST="ECUC-SYMBOLIC-NAME-REFERENCE-DEF">/VENDOR/EcucDefs_Can/Can/CanDemEventParameterRefs/CAN_E_TX_HISTORY_OVERFLOW</DEFINITION-REF>
    <VALUE-REF DEST="ECUC-CONTAINER-VALUE">/ActiveEcuC/Dem/DemConfigSet/CAN_E_TX_HISTORY_OVERFLOW</VALUE-REF>
</ECUC-REFERENCE-VALUE>
{% endhighlight %}

At the time, my workflow was to load the updated PDF into a configuration tool, manually import each test environment CDF, re-save them, and export them again so they would include the new reference parameters. 

My initial plan was to go through all of the CDFs—several hundred files—one by one. (Don’t ask me why I didn’t ask a senior engineer for guidance, my team culture back then was "figure it out yourself first, and only ask if you are truly stuck.") 

After agonizing through a dozen files, I realized this was not a sustainable way to work. I compared a new CDF against an old one and realized I could simply insert the text block into a specific position programmatically. 

I whipped up a Bash script after a few searches on Stack Overflow. It wasn't pretty, but it got the job done:

{% highlight bash %}
#!/usr/bin/bash

for filename in ./*.arxml; do
  echo "File name: $filename"
  # Find the line number of the anchor tag
  linef=`sed -n '/<SHORT-NAME>CanDemEventParameterRefs/=' $filename`
  # Offset by 3 lines to hit the correct insertion point
  linef=`expr $linef + 3`
  
  sed -i ''$linef' i\
                <ECUC-REFERENCE-VALUE>\
                  <DEFINITION-REF DEST="ECUC-SYMBOLIC-NAME-REFERENCE-DEF">/VENDOR/EcucDefs_Can/Can/CanDemEventParameterRefs/CAN_E_TX_HISTORY_OVERFLOW</DEFINITION-REF>\
                  <VALUE-REF DEST="ECUC-CONTAINER-VALUE">/ActiveEcuC/Dem/DemConfigSet/CAN_E_TX_HISTORY_OVERFLOW</VALUE-REF>\
                </ECUC-REFERENCE-VALUE>' $filename
done
{% endhighlight %}

Looking back, I realize how much easier this would have been if I had known **Regular Expressions (Regex)**. That entire scripting headache could have been resolved with a single "Search and Replace" in Notepad++ using a pattern like this:

![example_regex](/assets/img/blogs/2023_01_02/regex_example.png)

**Find:** 
{% highlight regex %}
(<SHORT-NAME>CanDemEventParameterRefs<\/SHORT-NAME>)([\s\S]*?)(^\s*)(<ECUC-REFERENCE-VALUE>)
{% endhighlight %}

**Replace:** 
{% highlight regex %}
$1$2$3<ECUC-REFERENCE-VALUE>\r\n$3  <DEFINITION-REF DEST="ECUC-SYMBOLIC-NAME-REFERENCE-DEF">/VENDOR/EcucDefs_Can/Can/CanDemEventParameterRefs/CAN_E_TX_HISTORY_OVERFLOW</DEFINITION-REF>\r\n$3  <VALUE-REF DEST="ECUC-CONTAINER-VALUE">/ActiveEcuC/Dem/DemConfigSet/CAN_E_TX_HISTORY_OVERFLOW</VALUE-REF>\r\n$3</ECUC-REFERENCE-VALUE>\r\n$3$4
{% endhighlight %}

After a few more encounters like this, I eventually stumbled upon **regular expression** and read some more advanced concept about it. Since then, Regex has become one of the most valuable skills I picked up in my early career.

To someone who doesn't know about regex yet, above Find and Replace regex looks like random gibberish. But once you understand the logic behind it, it becomes a superpower for manipulating text, cleaning data, and automating the mundane. In this blog, we will explore the building blocks of Regex—from basic syntax for daily tasks to advanced techniques for niche use cases.

## What is Regex?
At its core, a Regular Expression is a **`specialized string that describes a search pattern`**. Think of "Find and Replace" but on steroids. While a standard search looks for exact matches, regex looks for *`patterns`*—like "any sequence of 5 digits followed by 1 capital letter."

## Why Bother Learning It?
1. **Efficiency:** Tasks that take hours of manual editing and inspection can be done in seconds.
2. **Validation:** It’s the standard way to ensure user input (filepath, email, etc..) is formatted correctly.
3. **Data Scraping:** Extracting specific information from massive logs or text files becomes trivial.
4. **Universal:** Almost every language (Python, JavaScript, Java, C#) and text editor (VS Code, Vim, Sublime) have the library or function to support it.

---

## The Building Blocks: Basic Syntax
First, you need to know the building block that form the foundation of any regex:

### 1. Characters

Regular expressions consist of two types of characters: **Metacharacters**, which have special meanings (e.g. `.`, `\`), and **Literals** (normal text), which match themselves.

| Metachacters                   | Name              | Description                                                                                                                                                                                               |
| ------------------------ | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.`                      | Wildcard or Dot          | Matches any single character except a newline <br/> Example: `c.t` matches `cat`, `cut`, `c-t`, and `c9t`.                                                                                                          |
| `[`ca`]` `[`a`-`z`]` `[^`12`]`| Character classes | `[]`: matches any characters inside `[]` <br/>  Example: `gr[ea]y` match either `gray` or `grey` <br/><br/> `[-]`: matches any characters in range between char before dash `-` and after it <br/>  Example: `<H[1-3]>` matches `<H1>`, `<H2>` or `<H3>` <br/><br/> `[^]`: matches any character that isn't listed after caret `^` in `[]`(negated) <br/>  Example: `[^1-6]` matches any characters that not from 1 to 6  <br/><br/> NOTE: metacharacters inside `[]` is interpreted literally as it is(except for the dash `-` and `^` if they are used in position as above) .e.g `[(a.]` look for a single character `(`, `a` or `.` ; `[ab^-]` look for a single character `a`, `b`, `^` or `-` <br/><br/>`[0123456789abcdefABCDEF]` can be written as `[0-9a-fA-F]`|
| `\s` `\S` <br/>`\w` `\W` <br/>`\d` `\D` | Shorthand classes | `\s` / `\S`: matches whitespace (space, tab, newline) / Non-whitespace <br/> `\w` / `\W`: matches "word" characters (A-Z, 0-9, `_`) / Non-word <br/> `\d` / `\D`: matches any digit (0-9) / Non-digit |
| `\b` `\B`     | Boundary       | `\b`: matches the position between a word character and a non-word character(word boundary) <br/> `\B`: Non-word boundary |
| `\t` `\r` `\n` | Non printable | `\t`: Tab. <br/> `\r`: Carriage return. <br/> `\n`: Newline. |
| `a` `afc` `s` | Plain text character | Matches the literal string which is specified by character sequence|
| `()`| Capturing group | Groups characters together to act as a single unit and "remembers" the match for later use (back-referencing). <br/> Example: `(abc)` matches "abc" and stores it in group 1. |
| `\`                      | Escape            | If you want to match a meta characters like `.`, `$`, `[`, `]`, `(`, `)` or `\` as literal characters, you put a backslash before it. <br/>  Example: `\$` matches a literal `$`, and `\.` matches a literal `.`, and `\(\)` matches literal `()` <br/><br/> Another special usage is to back reference, as we can use `\`{index} to reference the matches in `()` group <br/>  Example: regex `(first|1st).*(<H[1-3]>)\2` then the `\2` is refer to 2nd matches unit `<H[1-3]>` in the current regex |
| `|` | Alternation | Combine multiple expressions into a single expression that matches any of the individual ones <br/>  Example: `(first|First|1st)` matches `first` or `First` or `1st` ; `gr(a|e)y` match either `gray` or `grey`|

### 2. Quantifier
Quantifiers specify how many times the **preceding** character, group, or character class should occur.

| Symbol       | Repetition | Description |
| ------------ | ---------- | ----------- |
| `+`            | 1 to n     | **At least once.** Matches one or more of the preceding element. <br/> Example: `\d+` matches `1`, `23`, `456` and so on |
| `*`            | 0 to n     | **Zero or more.** Matches the preceding element any number of times, or not at all. <br/> Example: `\t*` matches zero or many tabs |
| `{1}` `{2,}` `{0,3}` `{1,5}` | *min* to *max* | Matches between *n* and *m* repetitions. <br/> `{n}`: Exactly *n* times. <br/> `{n,}`: *n* or more times. <br/> Example: `[a-z]{1,4}` matches `a`, `ab`, `abc`, or `abcd` |
| `?`            | 0 or 1     | Matches the preceding element zero or one time. <br/> Example: `colou?r` matches `color` and `colour`|

NOTE: `?` is consider as lazy matching and other quantifier(`+` `*`) are greedy match. Refer to advanced section for it usage.

### 3. Anchor
Anchors do not match actual characters. Instead, they match **positions** within the text.

| Symbol | Name | Description |
| :--- | :--- | :--- |
| `^` | Caret | Matches the **start** of the line or string. <br/> Example: `^Start` matches "Start" only at the beginning. <br/><br/> NOTE:`^` consider as negated if placed at the beginning of `[]` |
| `$` | Dollar | Matches the **end** of the line or string. <br/> Example: `End$` matches "End" only at the very end. <br/><br/> NOTE:`$` act similar as `\` in case of referencing i.e. refer to unit group of the matched regex `$1`, `$2` |
| `\b` | Word Boundary | (As mentioned above) Matches the start or end of a word. <br/> Example: `\bcan\b` matches the word "can" but not "canvas" or "scan". |

### Now combine all of them  

The table above covers the core syntax, but the magic comedown on how you combine these pieces to solve specific problems. Let’s look at a possible scenario in actual project.  
Imagine you are analyzing a UART log from a Gateway ECU. The log spans several days of operation, and you need to find all security-related logs (`SEC:`) from **February 9, 2026**, specifically where a request was made to **ECU 0x1121**: 

{% highlight log %}
2026-02-09 01:35:08.883420 SYS:Started Security Auditing Service.
2026-02-09 01:35:08.931006 SYS:Started Measure the time from kernel boot to SGA-READY.
2026-02-09 01:35:08.995003 SYS:Started SOME/IP Gateway.
2026-02-09 01:35:09.011094 SEC:Started UDS firewall.
2026-02-09 01:35:10.044021 SYS:Started UDS message forwarding.
2026-02-09 01:35:12.000051 SEC:Request security access level 0x11 to ECU 0x1121.
2026-02-09 01:35:12.094231 SEC:Request security access level 0x17 to ECU 0x1100.
2026-02-09 01:35:15.052119 SEC:Request diagnostic protection role Production 0x31 to ECU 0x1121.
2026-02-09 01:35:30.954138 SYS:Started LIN Network Service.
2026-02-09 01:35:30.986575 SYS:Started LIN Heartbeat Service.
2026-02-09 01:35:31.086513 SYS:Started gPTP Bridge is ready.
....
2026-02-13 01:35:31.086513 SEC:Request security access level 0x11 to ECU 0x1121.
{% endhighlight %}

#### The Regex Solution:
To match this specific pattern, we can build the following expression:

{% highlight regex %}
^2026-02-09.*SEC:.*ECU 0x1121.*$
{% endhighlight %}

#### Breaking Down the Logic:
*   `^2026-02-09` : We use the **Caret (Anchor)** to ensure the line starts exactly with our target date.
*   `.*` : The **Wildcard and Greedy Quantifier** combo tells the engine to skip everything until find the follow-up pattern in the regex"
*   `SEC:` : A **Literal** match to filter only the security module entries.
*   `ECU 0x1121` : A literal match for our target logical address.
*   `$` : The **Dollar (Anchor)** ensures there is no extra text after our match, confirming we've reached the end of the line.

⟹ as we can see, there are plenty number of small building block characters, and from these we can arrive to infinity number of combined expression, which can help solve a single pattern matching problem.  


---

## Leveling Up: The Advanced Usage
Once you have mastered the basic building blocks, you will eventually run into complex problems—like parsing nested XML or extracting data from messy logs—where simple matching isn't enough. This is where the advanced logic of the regex engine comes into play.

#### 1. Groups and Capturing `()`
Parentheses serve **2 purposes**: they group parts of a pattern together and they **capture** the text that matches that group into a "memory slot", which can be referenced directly in current regex pattern or later(i.e. post matched/replaced operation)
*   **Grouping:** `(abc)+` matches "abc", "abcabc", "abcabcabc",... <- adjacent `abc` and repeated more than one
*   **Capturing:** If you have `(\d{2})-(\d{2})-(\d{4})` to match the dd-mm-yyyy, the regex engine remembers the day in Group 1, month in Group 2, and year in Group 3. You can then "recall" these in your code or replacement string using `$1` or `$2` or `$3` or backreference to them in the same searching regex as `\1`, `\2` or `\3`.

<figure>
  <img src="/assets/img/blogs/2023_01_02/regex_group_matching_example.png" alt="Group capturing example">
  <figcaption>Group capturing example</figcaption>
</figure>
Above image is a use case that I encounter daily and I utilize group capturing regex to do quick text modification where:  
- the input text is "TCVX-{index}" string on each line (few hundred lines)
- and I need to create a python array style string from this text, so I can put it into a simple python script  

search: `(TCVX-\d+)\r\n` -> replace `"$1", ` and I have the desired output.  

⟹ same operation can be done during programming, with group capturing you can get out data of interest(.i.e group in the regex) from regex matched result. 

<figure>
  <img src="/assets/img/blogs/2023_01_02/regex_backreference_example.png" alt="Backreference example">
  <figcaption>Backreference example</figcaption>
</figure>
Above image is use cases where I find all the  `<TEST-REQUESTS>` container in a xml file by using:  
`<(TEST-REQUESTS)>[\s\S]*?<\/\1>`: \1 is backreference to group 1 the literal string `TEST-REQUESTS`  

Another common use case is to check for duplicated word by `\b(\w+)\s+\1\b`.  

#### 2. Greedy vs. Lazy Matching
In above regex example you can see a weird part in the regex `[\s\S]*?`.  
- `[\s\S]`: just a quick way to match any character, including newlines/line breaks, because I want to get everything between xml TAG that spread multiple line
- `*?`: why optional quantifier `?` after a quantifier `*`. In this case, it called `lazy quantifier`

**Greedy quantifier**: `*`, `+`, `{num,num}` i.e. regex engine will try to extend the matching of regex unit before it as much as possible.  

**-> Lazy quantifiers**: `*?`, `+?`, `{num,num}?` i.e. regex engine stop at the **first** possible opportunity, so in above example regex engine will stop as soon as it hit the first `</TEST-REQUESTS>` tag, even though that literal string is matched by `[\s\S]*`

<figure>
  <img src="/assets/img/blogs/2023_01_02/regex_lazy.png" alt="Lazy quantifier example">
  <figcaption>Lazy quantifier example</figcaption>
</figure>
Example, in 2nd regex `<.*?>`, adding the lazy quantifier make the regex match a single xml tag instead of going for the whole line as in the 1st one.  

---

## Final Thoughts
You are not required to be an expert in regular expression and create most efficient regex for every issue. A basic understanding of how regex works and when to apply it is already enough to dramatically simplify your daily work.  
A very good site where you can learn and test your regex with detailed, real‑time explanations: [regex101.com](https://regex101.com/).  

And if you want to optimize your regex writing and explore more advanced concept, you can refer to this book **Mastering Regular Expressions** by Jeffrey E. F. Friedl.  

---