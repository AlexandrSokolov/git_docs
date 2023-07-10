* undo 'git add' before commit: `git reset <file>`
* remove the file only from the Git repository and not remove it from the filesystem:
```bash
git rm --cached file1.txt
git commit -m "remove file1.txt"
```
or folder:
```bash
git rm --cached -r folder
git commit -m "remove folder"
```

* remove the file both from the Git repository and the filesystem:
```bash
git rm file1.txt
git commit -m "remove file1.txt"
```
or folder:
```bash
git rm -r folder
git commit -m "remove folder"
```
* 