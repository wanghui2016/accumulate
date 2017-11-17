####  1 tcpdump 过滤数据包长度
```
  tcpdump -i eth6 "tcp and (host 10.1.106.40 or host 10.1.106.41) and (host 10.1.106.91) and len > 60"
```
