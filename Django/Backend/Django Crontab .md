# Django Crontab 

## Ubuntu instal and start cron
```shell
apt-get update && apt-get upgrade
apt-get install cron

pgrep cron  # check if cron running
service cron start
crontab -e
```

```
run task every hour
0,10,20,30,40,50 * * * * python /home/proj/cronjob.py
```

```
# m h  dom mon dow  command

m 分钟 0-59

h 小时 0-23

dow 天1-31

mon 月 1-12

dow  星期 1-6  0表示星期天

command 就是要执行的命令



实例1：每1分钟执行一次command
命令：
* * * * * command

实例2：每小时的第3和第15分钟执行
命令：
3,15 * * * * command

实例3：在上午8点到11点的第3和第15分钟执行
命令：
3,15 8-11 * * * command

实例4：每隔两天的上午8点到11点的第3和第15分钟执行
命令：
3,15 8-11 */2 * * command

实例5：每个星期一的上午8点到11点的第3和第15分钟执行
命令：
3,15 8-11 * * 1 command

实例6：每晚的21:30重启smb 
命令：
30 21 * * * /etc/init.d/smb restart


实例7：每月1、10、22日的4 : 45重启smb 
命令：
45 4 1,10,22 * * /etc/init.d/smb restart


实例8：每周六、周日的1 : 10重启smb
命令：
10 1 * * 6,0 /etc/init.d/smb restart


实例9：每天18 : 00至23 : 00之间每隔30分钟重启smb 
命令：
0,30 18-23 * * * /etc/init.d/smb restart


实例10：每星期六的晚上11 : 00 pm重启smb 
命令：
0 23 * * 6 /etc/init.d/smb restart


实例11：每一小时重启smb 
命令：
* */1 * * * /etc/init.d/smb restart


实例12：晚上11点到早上7点之间，每隔一小时重启smb 
命令：
* 23-7/1 * * * /etc/init.d/smb restart

实例13：每月的4号与每周一到周三的11点重启smb 
命令：
0 11 4 * mon-wed /etc/init.d/smb restart

实例14：一月一号的4点重启smb 
命令：
0 4 1 jan * /etc/init.d/smb restart


实例15：每小时执行/etc/cron.hourly目录内的脚本
命令：
01 * * * * root run-parts /etc/cron.hourly
说明：
run-parts这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是目录名了

 

新创建的cron job，不会马上执行，至少要过2分钟才执行。如果重启cron则马上执行。
```

## Python script to query and update Django model

```python
import sys
import os
import django

manage_dir = py_dir = os.path.dirname(os.path.abspath(__file__))
# manage_dir = os.path.join(py_dir, '../../')
# print(manage_dir)

# sys.path.append('/Users/CK/Git/qif/proj')
sys.path.append(manage_dir)
os.environ['DJANGO_SETTINGS_MODULE'] = 'proj.settings.prod'
django.setup()
```