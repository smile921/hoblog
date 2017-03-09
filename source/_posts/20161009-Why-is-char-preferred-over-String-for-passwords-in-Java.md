---
title: 'Why is char[] preferred over String for passwords in Java?'
date: 2016-10-09 09:05:18
tags:
   java
---

 ## Why is char[] preferred over String for passwords in Java?
 [来自 stackoverflow 的一个热门讨论](http://stackoverflow.com/questions/8881291/why-is-char-preferred-over-string-for-passwords-in-java?rq=1)
 In Swing, the password field has a getPassword() (returns char[]) method instead of the usual getText() (returns String) method. Similarly, I have come across a suggestion not to use String to handle passwords.

Why does String pose a threat to security when it comes to passwords? It feels inconvenient to use char[]

### ans1
* Strings are immutable. That means once you've created the string, if another process can dump memory, there's no way (aside from reflection) you can get rid of the data before garbage collection kicks in.

With an array, you can explicitly wipe the data after you're done with it. You can overwrite the array with anything you like, and the password won't be present anywhere in the system, even before garbage collection.

So yes, this is a security concern - but even using char[] only reduces the window of opportunity for an attacker, and it's only for this specific type of attack.

As noted in comments, it's possible that arrays being moved by the garbage collector will leave stray copies of the data in memory. I believe this is implementation-specific - the garbage collector may clear all memory as it goes, to avoid this sort of thing. Even if it does, there's still the time during which the char[] contains the actual characters as an attack window.

### ans2
While other suggestions here seem valid, there is one other good reason. With plain String you have much higher chances of accidentally printing the password to logs, monitors or some other insecure place. char[] is less vulnerable.

Consider this:

> public static void main(String[] args) {
>    Object pw = "Password";
>    System.out.println("String: " + pw);
>
>    pw = "Password".toCharArray();
>    System.out.println("Array: " + pw);
> }
> Prints:
>
> String: Password
> Array: [C@5829428e

### ans3

To quote an official document, the Java Cryptography Architecture guide says this about char[] vs. String passwords (about password-based encryption, but this is more generally about passwords of course):

> It would seem logical to collect and store the password in an object of type java.lang.String. However, here's the caveat: Objects of type String are immutable, i.e., there are no methods defined that allow you to change (overwrite) or zero out the contents of a String after usage. This feature makes String objects unsuitable for storing security sensitive information such as user passwords. You should always collect and store security sensitive information in a  char array instead.
> Guideline 2-2 of the Secure Coding Guidelines for the Java Programming Language, Version 4.0 also says something similar (although it is originally in the context of logging):

Guideline 2-2: Do not log highly sensitive information

> Some information, such as Social Security numbers (SSNs) and passwords, is highly sensitive. This information should not be kept for longer than necessary nor where it may be seen, even by administrators. For instance, it should not be sent to log files and its presence should not be detectable through searches. Some transient data may be kept in mutable data structures, such as char arrays, and cleared immediately after use. Clearing data structures has reduced effectiveness on typical Java runtime systems as objects are moved in memory transparently to the programmer.
> 
This guideline also has implications for implementation and use of lower-level libraries that do not have semantic knowledge of the data they are dealing with. As an example, a low-level string parsing library may log the text it works on. An application may parse an SSN with the library. This creates a situation where the SSNs are available to administrators with access to the log files.

### ans4 
Character arrays (char[]) can be cleared after use by setting each character to zero and Strings not. If someone can somehow see the memory image, they can see a password in plain text if Strings are used, but if char[] is used, after purging data with 0's, the password is secure.

