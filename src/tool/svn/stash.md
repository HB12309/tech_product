svn实现类似git stash及git stash pop的功能
git下，有git stash这个命令可以方便地保存当前的修改，并还原代码到未修改的状态。然后处理完之后又可以使用git stash pop将之前的修改内容合并到当前代码。

svn下，缺乏这样的命令。不过可以用svn diff和svn patch来实现基本类似的功能。如下内容保存为svnstash.bat，并放到任意path环境变量目录（如C:\window\）下即可。

命令：

svnstash：暂存。类似git stash，可多次执行。

svnstash pop：恢复之前暂存的内容，可多次执行，以此弹出。
使用的时候，在svn工程目录下，


bat文件
```
@echo off
if not exist %CD%\.svn (
   echo %CD% 不是svn目录
   goto out
)

set sdir=%CD%\.svn\stashed
if '%1'=='pop' (
   goto pop
) else (
   goto stash
)
goto out

:stash
   if not exist %sdir% mkdir %sdir%
   set dt=%Date%
   set tm=%Time%
   set stime=%dt:~0,4%%dt:~5,2%%dt:~8,2%-%tm:~0,2%%tm:~3,2%%tm:~6,2%

   set tfn=%sdir%\svnstash-%stime%.diff

   svn diff >> "%tfn%"
   if %ERRORLEVEL% EQU 0   svn revert -R .
   
   echo 使用 svnstash pop 恢复上一次保存的内容
   goto out

:pop
  FOR /F "delims==" %%f IN ('dir %sdir% /a/b /o-d') DO (
   echo %%f
   svn patch %sdir%\%%f . --ignore-whitespace
   del %sdir%\%%f
   echo poped %%f
   goto out
  )
  echo 没有暂存的内容
  goto out

:out
```