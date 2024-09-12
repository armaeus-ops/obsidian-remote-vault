## Linkliste

- Kostenloses Git-Buch: (http://gitbu.ch/ "http://gitbu.ch/")
- Offizielles Git-Buch (englisch): https://git-scm.com/book/ "https://git-scm.com/book/")
- Offizielles Git-Buch (deutsch):https://git-scm.com/book/de/ "https://git-scm.com/book/de/")
- Git Kommando-Assistent (englisch):https://gitexplorer.com/ "https://gitexplorer.com/")
- Git Cheat-Sheet: (https://www.git-tower.com/blog/git-cheat-sheet/ "https://www.git-tower.com/blog/git-cheat-sheet/")
- Git-Kommandos mit Katzen erklärt:https://girliemac.com/blog/2017/12/26/git-purr/ "https://girliemac.com/blog/2017/12/26/git-purr/")
- Git Advanced-Kurs:https://codehub.sva.de/Schulungen/git-advanced "https://codehub.sva.de/schulungen/git-advanced")
- Git spielerisch lernen:https://ohmygit.org/ "https://ohmygit.org/") und https://learngitbranching.js.org/ "https://learngitbranching.js.org/")
- Einige Tipps zum Beheben von Fehlern:https://ohshitgit.com/ "https://ohshitgit.com/")
  
  ___
## Git Repo von Null aufziehen und Pushen:
 
```
bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte
$ git init
Initialized empty Git repository in C:/Users/bwuske/Desktop/gitschosch/Powershell-Skripte/.git/

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git add .

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ ls
ServerSchedTask_Ownership.ps1  ServerServices_Ownership.ps1

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   ServerSchedTask_Ownership.ps1
        new file:   ServerServices_Ownership.ps1


bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git reset ServerServices_Ownership.ps1

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git commit -m "Skript fuer alle Scheduled Tasks, auszer MS, mit Owner"
[master (root-commit) 94f8d7f] Skript fuer alle Scheduled Tasks, auszer MS, mit Owner
 1 file changed, 20 insertions(+)
 create mode 100644 ServerSchedTask_Ownership.ps1

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git add ServerServices_Ownership.ps1

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git commit -m "Skript fuer alle Services mit Owner"
[master 26ce0eb] Skript fuer alle Services mit Owner
 1 file changed, 22 insertions(+)
 create mode 100644 ServerServices_Ownership.ps1

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ nano README.md

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git add README.md
warning: in the working copy of 'README.md', LF will be replaced by CRLF the next time Git touches it

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git commit -m "added README"
[master afb3380] added README
 1 file changed, 3 insertions(+)
 create mode 100644 README.md

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git push
fatal: No configured push destination.
Either specify the URL from the command-line or configure a remote repository using

    git remote add <name> <url>

and then push using the remote name

    git push <name>


bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git remote add origin git@codehub.sva.de:Benjamin.Wuske/TasksSkripte.git

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git add .

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git commit -m "init"
On branch master
nothing to commit, working tree clean

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$ git push --set-upstream origin master
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 8 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (9/9), 1.37 KiB | 1.37 MiB/s, done.
Total 9 (delta 3), reused 0 (delta 0), pack-reused 0
To codehub.sva.de:Benjamin.Wuske/TasksSkripte.git
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.

bwuske@TPBWUSKE-1 MINGW64 ~/Desktop/gitschosch/Powershell-Skripte (master)
$

```