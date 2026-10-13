OS Command Injection — via Stock Check Parameter (PortSwigger Web Security Academy)

Lab: OS command injection, simple case
Category: OS Command Injection
Difficulty: Apprentice
Status: Solved
Tools used: Burp Suite Community Edition (Proxy, Repeater)

Objective
Exploit an OS command injection vulnerability in the stock-check feature to execute an arbitrary command on the server.

Hypothesis
Having just learned the concept of OS command injection, it seemed worth testing whether the `storeId` parameter — likely used server-side to look up stock for a specific store — was passed directly into a system command without sanitization. If so, injecting shell metacharacters (like a pipe, `|`) alongside the expected value might cause the server to execute an additional, attacker-supplied command.

 Steps

 1. Identify the target parameter
Intercepted a "Check stock" request and found the `storeId` parameter, normally holding a plain numeric value (e.g. `storeId=1`).

2. Inject a command using a pipe character
Modified the parameter to:
```
storeId=1| whoami
```

![Screenshot: Burp view showing the modified storeId parameter and the response]()

3. Observe the result
The server's response included the output of the `whoami` command — the current user the server process was running as (e.g. `peter-13x7yv`) — in addition to (or instead of) the normal stock response. This confirmed the injected command had executed on the server's operating system.

4. Confirm
Lab marked as solved.

![Screenshot: lab marked Solved]()

 Root Cause
The `storeId` parameter was intended only to identify which store to check stock for — a plain numeric value. Instead of validating or sanitizing this input, the server passed it directly into a system shell command used to perform the stock lookup. Because the pipe character (`|`) was not stripped or escaped, the shell interpreted the input as two separate, sequential commands: the intended stock lookup, and the injected `whoami` command. The shell executed both, and the output of the injected command was returned in the response.

This is the same underlying pattern as SQL Injection, applied to the operating system instead of a database: untrusted input is concatenated directly into a command that gets executed, without any separation between "data" and "instructions."

 Fix Recommendations
- Never pass user-supplied input directly into a system shell command.
- Where shell commands are unavoidable, use parameterized/argument-array APIs (which pass arguments as discrete values rather than a single concatenated string) instead of building a command string via concatenation.
- Strictly validate input against an expected format (e.g. `storeId` should match a numeric pattern only) and reject anything that doesn't conform.
- Avoid running shell commands with more privilege than necessary, limiting the impact of any injection that does occur.
