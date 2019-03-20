---
title: "Introduction"
teaching: 10
exercises: 0
questions:
- "What is secure coding?"
- "What are some types of attacks that exploit code vulnerabilities?"
- "What are the broader impacts of insecure code?"
objectives:
- "Describe the importance of secure and bug-free code"
keypoints:
- "Insecure or buggy code can have far-reaching consequences!"
---

## Secure Coding

Several cybersecurity flaws can ultimately be traced to a previously undetected vulnerability in
computer software. Secure coding is the practice of ensuring that software that works with privileged
information does not reveal such information directly, or make it possible to reverse engineer the information.
More broadly, it involves avoiding common bugs in software code that can have unintended cybersecurity
vulnerabilities. Such bugs can include failure to perform input validation, check for out of bounds
errors in array access etc. When such buggy code is used to manage privileged information such as
database records, user authentication information on web servers etc., the bugs can be exploited to
unearth the privileged information. A malicious user with knowledge of these bugs and the underlying
software library can extract privileged information, even when performing activities like any other
regular user of the software.

With the burgeoning use of computer software and libraries in every conceivable domain, such
vulnerabilities could be introduced from an imported library, without a software developer’s knowledge.
Thus, it is vital for both the current and future generation of developers (and not just
cybersecurity engineers) to be aware of such vulnerabilities and their solutions. In following lessons,
you will learn about two attacks that exploit vulnerable code; **SQL Injection** attacks and the
**Heartbleed** bug. 

{% include links.md %}
