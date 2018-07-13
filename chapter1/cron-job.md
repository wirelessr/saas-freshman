# APScheduler

要用python起一個timer task或者cron job最常見的做法就是使用celery搭配redis，甚至使用MQ做broker，但對於一些小型專案來說，這是殺雞用牛刀。雖然可以用docker快速把redis建起來，但是celery的相關設定很複雜(字很多)。
以測試用的小專案來說，重要的是快速上線、快速修改；celery在這點上稍嫌笨重，甚至若是使用heroku免費方案又不登錄信用卡的前提，還必須想辦法生出一個redis。因此，這邊介紹一個套件：APScheduler。
APScheduler不需要額外的server role來支援timer和cron，而且只要套上decorator就可以快速進行設定，非常適合小型專案的開發。

```python
from apscheduler.schedulers.blocking import BlockingScheduler

sched = BlockingScheduler()

@sched.scheduled_job('interval', minutes=3)
def timed_job():
    print('This job is run every three minutes.')

@sched.scheduled_job('cron', day_of_week='mon-fri', hour=17)
def scheduled_job():
    print('This job is run every weekday at 5pm.')

sched.start()
```

按照上面的範例就快速產生一個timer和cron job。若是要推上heroku，只需要再加上兩個檔案：Procfile和requirements.txt即可。

Procfile
> clock: python clock.py

requirements.txt
```
APScheduler==3.5.1
funcsigs==1.0.2
futures==3.2.0
pytz==2018.5
six==1.11.0
tzlocal==1.5.1
```

在heroku把clock這個instance帶起來，就正式上線了。若是想看執行結果：
> heroku logs -a xxx

就能夠看到print出來的結果。

[github](https://github.com/wirelessr/sched_tasks)