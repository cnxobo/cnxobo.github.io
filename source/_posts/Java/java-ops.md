查看JAVA阻塞方法
```Shell
jstack -l 1 | awk -v RS='' '/BLOCKED/' | sort | uniq -c | awk '$1 >= 5' | sort -nr
```