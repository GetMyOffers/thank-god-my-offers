### 如果本地还没有项目：
1. 打开终端执行
```shell 
git clone https://github.com/GetMyOffers/thank-god-my-offers.git
```

### 如果本地已经有项目：
2. 本地 cd 到项目当中执行
```shell
git checkout develop
```
永远不要切回到 master 分支，以免误操作

### 添加主仓库（只需要刚 clone 项目的时候操作一次）
```shell
git remote add upstream https://github.com/GetMyOffers/thank-god-my-offers.git
git checkout develop
git pull upstream develop
```

3. 开始本地修改
4. 每次项目提交前先执行
```shell
git pull upstream develop
```
5. 开始提交，依次执行
```shell
git add .
git commit -m 'type your comments here'
git push origin develop
```





