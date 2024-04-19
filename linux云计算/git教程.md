# 使用git初始化项目

对于已经创建好的gitee仓库项目，向仓库中上传文件

```sh
git init # 第一步：初始化项目，然后会生成.git文件夹
git add .  # 第二步：将当前目录加入到git（.代表全部）
git commit -m "first （提交的描述信息也就是提交的说明）" # 第三步：git提交到本地版仓库，并注明提交的缘由
git remote add origin https://gitee.com/hui829/web-vue-project-exercise.git
# 第四步：将文件上传到gitee中的master分支,-u代表第一次上传
git push -u origin master
```

注意：一般一个完整项目中只有第一级目录下会生成.git文件夹，如果上传的第二级目录的文件夹中有.git文件夹，则只会上传二级目录文件夹，而不会上传二级目录文件夹下的内容，如果上传的二级目录的文件夹中没有.git文件，则会上传二级目录文件夹下的内容（多级目录同理）

# 使用git更新项目

2.对于已有的仓库，且已经进行过一次或多次上传文件后，还需要更新代码至gitee

```sh
git status  # 第一步：列出自己做出修改的文件，会列出你修改的文件以及新增的文件
git add . # 第二步：将当前目录加入到git, .代表全部更新, 指定文件更新则点替换为指定文件名
git add test.txt # 第二步-2：上传单个文件
git commit -m "更新说明" # 第三步： 添加更新说明
git push origin master  # 第四步： 执行更新操作
```



git status

```go
注:新增的文件和修改过后的文件都是红色
```

