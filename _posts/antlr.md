Domain-specific languages have been around since ages but have seen rapid resurgence lately owing to their specificity and ease of use across domains.
 
Applications speaking native FIX protocols can realize significant value by using a DSL - for applying dynamic logical validations, filtering, determining routing rules and so on.
We explore one such possibility below.
 
Problem statement:
 
1) Defining rules for validating FIX messages.
2) Implementing acceptance/ rejection filters
3) Defining workflow decision rules
 
This implicitly requires using some implementation of a rule engine - either custom-built or a generic off-the-shelf rule engine.
This article explores the option of using a DSL (Domain-Specific Language) to solve above problems.
 
A logical rule might read something like this:
If an order is a GTD, ExpiryTime must be specified  
 
or like this:
ExpiryTime must be in future  
TransactTime must be in past  
 
while technical rule might be expressed as below:
 
SendingTime must match the format "[0-9]{8}-[0-9]{2}:[0-9]{2}:[0-9]{2}.[0-9]{3}"  
 
Must haves:
 
Quick on-boarding without needing a complete compile-build-deploy cycle
Logical specification without requiring programming expertise
Extensible to add new operators/ conditions
Simple to understand and use
Good to have:
Content-assist and code completion
 
Approaches
Programming language
        The most robust approach would be to implement the rules in a programming language.
        Pros: Robust testing suite, content assist, well-known
        Cons: Higher time-to-market due to compile-build-deploy cycle and confined to developers
 
     2. Scripting
        The logical choice to achieve flexibility and dynamism, trade-off on execution speed.
        Pros: Lower time-to-market, robust, testing suite
        Cons: Confined to developers
 
     3. XML
       Most open-source projects follow this approach for its inherent flexibility and extensibility.
       (example: http://www.quickfixj.org/quickfixj/usermanual/1.5.3/usage/validation.html)
        Pros: Lower time-to-market
        Cons: Needs some technical expertise
                  Too generic and difficult to integrate content-assist with FIX data dictionary
 
     4. DSL
       Pros: Specific to domain
                Easy to understand
                Ease of integrating content-assist with FIX data dictionary
       Cons: Comparatively longer time to develop
 
We explore the process of building a domain-specific language (approach 4), which in our context has the following advantages:
 
NO/ minimal learning curve, reads "just like prose“
Integration with CitiFIX/ data dictionary to facilitate meta-data, tags, permissible values and allowed enums
Extensible and fluent API that allows defining of new operators/ conditions
Rules can act as a live document
Web editor with smart content-assist, integrated with FIX data dictionary
Simple textual comparison
Reusable and embedded solution
 
Building a DSL
 
While DSLs are easy to use, defining and implementing one is non-trivial.
 
ANTLR
Building DSLs using ANTLR is a easier done than said, since it provides a clean and well-defined process to implement a fully functional DSL.
 
What ANTLR requires:
A grammar file
 
What ANTLR provides:
1) Generated Lexer/ Parser
2) API to walk the AST (Abstract Syntax Tree) generated from the given grammar
 
Building a grammar definition:
We start with the most generic definition of a rule, which is devoid of all technical jargon.
 
If an order is a GTD, ExpiryTime must be present  
 
To have a consistent syntax, we re-phrase the above logical rule to take the below form.
 
ExpiryTime must be present, if an order is GTD  
 
Replacing field names with a generic syntax for tags,
 
tag(126) must be present, if tag(59) is 1  
 
And then further
tag(126) must be present if tag(59) = 1;  
 
(While semi-colons/ end-of-rule indicators look unnecessary here, they simplify defining ANTLR grammar files and hence used)
 
In short, most rules can now be defined using the below template:
 
tag(N) <condition> <operator> <operand1, operand2,...>  
(if tag(M) <condition> <operator> <operand1, operand2, ...> )?;  
 
 
Defining FIX validation rules
A recurrent problem encountered by most applications speaking native/ custom FIX protocol is to define rules to validate tags in FIX messages.
These are usually logical and domain-specific, hence they are not integrated with the FIX parser/generator libraries.
 
Examples:
 
tag(59) must be equal to 1;  
tag(8) must be in "FIX.4.2","FIX.4.3","FIX.4.4" ;  
tag(9) must be present;  
tag(35) = "RIO";  
tag(56) must not be equal to "CORE8";  
tag(9) must be equal to 954;  
tag(9) = 954;  
tag(9) != 841;  
tag(8) must be in "FIX.4.3", "FIX.4.5";  
tag(8) = "FIX.4.4";  
tag(55) is present;  
tag(1) = "EDJO005E";  
tag(11) must be equal to tag(41);  
tag(37) = tag(31);  
tag(59) must not be equal to 6;  
tag(60) must be in past;  
tag(52) must be in future;  
tag(126) must be present if tag(59) = 1;  
tag(126) must be present if tag(59) = 1 and tag(9) is numeric and tag(11) is present;  
 
 
Quick working demo:
 
http://vm-b26d-0e7a.nam.nsroot.net:8999/
 
1.png
2.png
3.png
4.png
 
What works:
Rule definitions and executions
 
Still in progress:
Error highlighting on the UI editor
