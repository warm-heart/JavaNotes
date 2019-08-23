#### 一、初始化版本库

```
git init
```

#### 二、创建SSH密钥

第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash）

创建SSH Key：

```
ssh-keygen -t rsa -C "youremail@example.com"
```

第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：

 然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容：

#### 三、与远程仓库关联

```
git remote add origin git@github.com:warm-heart/springboot-master.git
```

命令中“origin”表示远程仓库的别名，也可以写成其他的。

#### 四、把文件添加到版本库中并提交到远程仓库

1. ```git
   git add .
   ```

   表示添加所有文件

1. ```
   git add readme.md  添加特定文件
   ```

2. ```
    git commit -m"描述语句"
   ```

3. ```
   git push -u origin master
   ```

   

#### 五、创建与合并分支

​      创建dev分支并切换

1. ``` 
   git checkout -b dev
   ```

   查看当前分支

2. ``` 
   git branch
   ```

   切换到master分支

3. ```
   git checkout master
   ```

   合并dev分支到master上

4. ```
   git merge dev
   ```

   删除分支

5. ```
   git branch -d <name>
   ```

      

