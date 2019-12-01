# ubuntu
- 开放端口
sudo ufw allow 19999

- 关闭端口
sudo ufw deny 19999

- 查看端口
sudo netstat -lnp|grep 19999

- 关闭端口监听
kill -9 

- wget basic authentication required
wget --user=admin --ask-password https://www.yourwebsite.com/file.zip

- 行号
wc -l Company_1c76cbfe21c6f44c1d1e59d54f3e4420.data.json -- 148681983

- 切割
```
split -l 10000 Company_1c76cbfe21c6f44c1d1e59d54f3e4420.data.json -d -a 5 company/company_
ls|grep company_|xargs -n1 -i{} mv {} {}.json


split -b 30m CUST_INFO.dat -d -a 2 file_&&ls|grep file_|xargs -n1 -i{} mv {} {}.txt

命令解释：

split -b 30m CUST_INFO.dat -d -a 2 file_&&ls 将文件以30M大小进行分割，并且前缀为file_;

xargs -n1 -i{} mv {} {}.txt 将生成的文件重命名为扩展名为txt的
```
