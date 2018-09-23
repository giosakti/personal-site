---
title: "Java"
authors: ["gio"]
---

#### Debugging JVM Crash

Whenever fatal error occurs, the system attempts to create the file in the working directory of the process. 

In the event that the file cannot be created in the working directory, the file is created in the temporary directory for the operating system. 

To specify where the log file will be created, use the product flag -XX:ErrorFile=file, where file represents the full path for the log file location. The substring %% in the file variable is converted to %, and the substring %p is converted to the PID of the process.

In the following example, the error log file will be written to the directory /var/log/java and will be named java_errorpid.log:

`java -XX:ErrorFile=/var/log/java/java_error%p.log`

See:
- https://www.oracle.com/technetwork/java/javase/felog-138657.html#gbwcy
- https://www.oracle.com/technetwork/java/javase/index-137495.html
