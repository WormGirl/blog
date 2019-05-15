## 回滚已提交到远程分枝的代码
1.本地代码回退
```bash
git reset --hard HEAD^        回退到上个版本
git reset --hard commit_id    退到/进到 指定commit_id
```
2.推送到远端
```bash
git push origin HEAD --force
```