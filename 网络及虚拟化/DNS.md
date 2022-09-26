```sh
root@G:/mnt/d/cloudb# dig cloudb.pub +nostats +nocomments +nocmd
;cloudb.pub.                    IN      A
cloudb.pub.             30      IN      A       185.199.110.153
cloudb.pub.             30      IN      A       185.199.109.153
cloudb.pub.             30      IN      A       185.199.108.153
cloudb.pub.             30      IN      A       185.199.111.153

root@G:/mnt/d/cloudb# dig cloudb.pub +noall +answer -t A
cloudb.pub.             237     IN      A       185.199.109.153
cloudb.pub.             237     IN      A       185.199.110.153
cloudb.pub.             237     IN      A       185.199.111.153
cloudb.pub.             237     IN      A       185.199.108.153

root@G:/mnt/d/cloudb# dig cloudb.pub +noall +answer -t name
;; Warning, ignoring invalid type name
cloudb.pub.             213     IN      A       185.199.108.153
cloudb.pub.             213     IN      A       185.199.109.153
cloudb.pub.             213     IN      A       185.199.110.153
cloudb.pub.             213     IN      A       185.199.111.153

root@G:~# dig cloudb.pub +noall +answer -t A
cloudb.pub.             1792    IN      A       185.199.110.153
cloudb.pub.             1792    IN      A       185.199.111.153
cloudb.pub.             1792    IN      A       185.199.108.153
cloudb.pub.             1792    IN      A       185.199.109.153

root@G:~# dig cloudb.pub +noall +answer -t AAAA
cloudb.pub.             1796    IN      AAAA    2606:50c0:8002::153
cloudb.pub.             1796    IN      AAAA    2606:50c0:8001::153
cloudb.pub.             1796    IN      AAAA    2606:50c0:8000::153
cloudb.pub.             1796    IN      AAAA    2606:50c0:8003::153

```

解析一个域名

