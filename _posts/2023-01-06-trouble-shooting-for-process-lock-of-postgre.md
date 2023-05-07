---
layout: post
title: trouble-shooting-for-process-lock-of-postgre
date: 2023-01-06
category: trouble-shooting
---

# what happened

command

```
brew service start postgresql
```

err message

```
Bootstrap failed: 5: Input/output error
Error: Failure while executing; `/bin/launchctl bootstrap gui/501 /Users/<myUserName>/Library/LaunchAgents/homebrew.mxcl.postgresql@14.plist` exited with 5.
```

err discription  
postgresql를 실행하니 input output err 발생 exit signal은 5로 나왔다.

# reason

commnad

```
cat /opt/homebrew/var/log/postgresql@14.log
```

err message

```
2023-01-06 09:33:15.018 KST [9621] HINT:  Is another postmaster (PID 1086) running in data directory "/opt/homebrew/var/postgresql@14"?
2023-01-06 09:33:25.056 KST [9655] FATAL:  lock file "postmaster.pid" already exists
```

err discription  
postmaster process가 실행되고 있고 lock file 사용하고 있어서 postgresql process가 실행되지 않는다.

# solution

solution discription  
pid가 1086인 postmaster process를 종료시켜서 lock file를 종료시킨다.

command

```
kill 1086
```

# reference

[https://superuser.com/questions/553045/fatal-lock-file-postmaster-pid-already-exists-](https://superuser.com/questions/553045/fatal-lock-file-postmaster-pid-already-exists-)