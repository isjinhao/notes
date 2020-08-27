## GIT基础

**GIT组成**

GIT中存在工作区、本地GIT仓库和远程GIT仓库三个概念，我们日常开发都是在工作区进行，工作区对文件完成增删查改操作后可以添加至本地GIT仓库，但是在真正保存至本地仓库之前需要先加入暂存区。一个本地仓库的内容可以推送给多个远程仓库。 

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/02cc4093-53a9-4433-96d7-607254045f85" /></div>
**git init**

在某个目录（设为A）下执行`git init`指令会在此目录中生成一个`.git`目录，表明有一个和A目录对应的版本库被创建，以后对A目录下的工作文件都可以追踪在`.git`目录下。开发者不能修改`.git`目录下的文件，否则会破坏版本库的内容甚至结构。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/21f224eb-c6ac-4200-b456-d01ebf800f0d" /></div>
**git status**

在A目录下新建一个文件`1test`，使用`git status`命令可以查看到此时的文件是未追踪状态。未追踪指的是此文件还没有被GIT管理。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/3025f015-96b4-4828-9aa3-e1394ca40d71" /></div>
**git add**

此命令会将未被GIT追踪的文件添加至Stage。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/bd5047ee-a73e-4324-8b3d-2abb08121694" /></div>
**git rm --cached**

将文件从Stage区移除，可以使用`git rm --cached`命令

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/973cb2fd-29cd-494f-93b0-0a0a4413146e" /></div>
**git commit**

先使用`git add`将文件加入Stage区，然后使用`git commit`提交至Local Repository，GIT要求每一次的提交都必须有一个commit message，所以会自动弹出一个界面让我们输入提交信息。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/b18ab8fd-0768-4726-b516-dc4197513e08" /></div>
提交完成之后我们就会看到此时工作空间是干净的，即所有内容都添加到了Local Repository中。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/15a434e5-d8a6-4723-b8fd-8f809c9d214f" /></div>
*others*

- 如果message不长，我们可以使用`git commit -m "commit message"`来代替上面的操作。

- 如果提交时不小心将message写错了，使用`git commit -amend`来修改本次提交信息。
- `git commit -am`可以将`git add`和`git commit`两步操作合在一起。

**git log**

此命令可以显示提交的历史，每次提交都会产生一个commit id，这是一个SHA-1算法下的消息摘要。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/2311a447-c227-4bf6-aa02-215f1f8ccce2" /></div>
`git log`会显示全部的提交消息。

- 如果只想查看最近n条的提交消息，可以使用`git log -n`
- 以每次提交显示一行的状态显示log：`git log --pretty=oneline`
- 以自定义格式显示log，如：`git log --pretty=format:"%h - %an, %ar : %s"`

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/2a6c0a57-a931-4970-a1ee-4da1895d86f6" /></div>
*others*

如果想查看某个命令的详细信息，可以去`https://git-scm.com/`上寻找。但实际上在命令后面加上`--help`参数就可以在本地打开命令的详细解释。如`git log --help`就可以查找到`git log`命令的详细解。

**git config**

我们上面的图中可以看到Author是由name和email两个部分组成的。在GIT中，有三个范围的Author：

- 机器全局（基本不用）
- 本用户
- 本仓库

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/24ba8358-d867-4d1b-bef0-245b4dd36548" /></div>
本地仓库的配置在`.git`目录下的`config`文件里：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/0d602258-eb69-4b49-92ce-aa877bee0404" /></div>
Windows系统中用户级别的配置在`C:\Users\$USER`目录下的`.gitconfig`文件里。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/086b211c-d543-48e6-933e-996bf502ca2f" /></div>
如果我们想修改Author信息，可以使用`git config`配置：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/58784d0f-10ad-46e7-8382-e6aefc533f70" /></div>
*others*

- `git config --local -unset xxx.yyy`可以撤销配置的信息。

**git restore**

当我们在`test.txt`文件中增加一行welcome后，再使用`git status`就会发现被提示文件被修改。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/6f7739d0-5d40-4bc2-ad6a-efc58c262b39" /></div>
如果我们想将暂存区的内容恢复至工作区，使用`git restore`命令。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/0c32ba6b-c8aa-4039-b3a9-25b549107d47" /></div>
如果我们想将上一次commit的内容恢复到暂存区可以使用`git restore --staged`。在演示此命令之前先做如下操作：

- 在文件中添加`1line`并提交至本地仓库。
- 在文件中添加一行`2line`并添加至暂存区但不提交至本地仓库。

此时工作区状态如下：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/9172a16e-8a52-4349-83f2-b810f8c60af0" /></div>
此时再做如下操作：

- 应用`git restore --staged`至`1test`
- 应用`git restore`至`1test`

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/5bb4d45a-90c0-4c8e-9462-818d3b5c5fb1" /></div>
*others*

- 恢复最新的提交至工作区：`git restore --source=HEAD --worktree`
- 恢复最新的提交同时至工作区和暂存区：`git restore --source=HEAD --stage --worktree`

**文件4种状态**

对GIT来说，文件有多种状态，比如新创建一个文件后使用`git status`会看到新的文件是`new`状态，删除一个文件后使用`git status`会看到删除的文件是`delete`状态。除了这两种状态，还有四种常见的文件状态。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/ded189ca-5e2d-45d1-b245-9ff8b3310923" /></div>
- Untracked：此文件在工作区中，但并没有加入到GIT库，不参与版本控制。通过`git add`状态变为`Staged`。
- Unmodified：文件加入暂存区后没有再进行过修改，即版本库中的文件快照内容与文件夹中完全一致。
  - 如果它被修改，而变为`Modified`。
  - 如果使用`git rm --cached`移出暂存区，则成为`Untracked`文件。
- Modified：文件已修改，仅仅是修改，并没有进行其他的操作。这个文件有两个去处。
  - 通过`git add`可进入暂存状态
  - 使用`git restore`则丢弃修改过，返回到`Unmodify`状态。
- Staged：暂存状态。
  - 执行`git commit`则将修改同步到库中，这时库中的文件和本地文件又变为一致，文件为`Unmodify`状态。
  - 执行`git restore --staged`取消暂存，文件状态为`Modified`。

**git rm**

此命令会将工作区和暂存区的文件同时删除。

- 如果一个文件没有添加至暂存区就使用此命令会报错，因为GIT无法在暂存区找到这个文件。
- 如果一个文件只被添加到暂存区没有提交就使用此命令需要加上`-f`命令，因为一旦删除，就无法回退。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/9f2c9113-bf35-40bc-89d5-17ee4acb9476" /></div>
我们先将此文件提交至Repository，再使用`git rm`命令。此时等同于使用如下命令：

- `rm 2test`
- `git add 2test`

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/faa91252-9e51-40e3-b830-75e17c602ba2" /></div>
如果此时我们想恢复这个文件，只能使用`git restore --staged`丢弃从上次提交至现在的操作。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/59518d06-0b05-49bc-931f-3ae33cde3584" /></div>
**.gitignore**

如果目录中存在一些不想被GIT管理的文件，如秘钥文件，散列函数的盐文件等。我们可以在`.gitignore`文件中配置这些文件，这样GIT就不会去理会这些文件。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/a28d563e-066c-4bf1-b99d-d13edddde33e" /></div>
`.gitignore`可以支持正则表达式：

1. `*.b`：忽略所有以`.a`结尾的文件
2. `!a.b`：配合第一行使用，忽略所有以`.a`结尾的文件但是除了`a.b`文件
3. `dir_name/`：忽略一个目录
4. `dir_name/*.txt`：忽略目录下的以`.txt`结尾的文件
5. `dir_name/*/*.txt`：忽略目录及其一层子目录中以`.txt`结尾的文件
6. `dir_name/**/*.txt`：忽略目录及所有子目录中以`.txt`结尾的文件

**全量变化维护**

GIT使用全量变化维护。也就是说如果一个文件被修改，GIT会直接保存新的文件，而不是保存新文件与旧文件不一致的部分。如下图中`Version1`到`Version2`的过程中，`A`变成了`A1`，GIT直接在`Version2`中保存`A1`，而不是保存`A1`和`A`不一致的部分。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/91bacd9b-d0c1-4a89-a300-6c771142ff6f" /></div>
**commit的关联**

GIT的commit之间是使用一个链表连接起来的。除了第一次提交，其他的都至少有一个指向父提交的指针。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/748fddf0-1625-4846-83b0-8581172f579a" /></div>
**基础小结**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/ab05fad4-0994-4791-b478-c852a49b448d" /></div>
## 分支

**git branch**

- `git branch`：查看仓库有哪些分支及当前属于哪个分支
- `git branch xxx`：新建一个分支
- `git checkout xxx`：切换到某个分支
- `git branch -d xxx`：删除某分支

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/05a3056e-fea9-447c-9e7c-4e694357959f" /></div>
**git checkout细节**

在切换分支后，GIT会帮我们更新工作区和暂存区。 

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/3c966b05-0c98-490b-851f-4667ef41bf7a" /></div>
分支是相对于提交而言的，我们设计一个流程来验证一下：

- 在`1new_branch`上创建一个文件并纳入暂存区
- 切换到`master`分支提交
- 查看处在`1new_branch`和`master`分支时工作区的情况

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e21a1ba7-597f-469a-b9f6-eebcfe8e96bf" /></div>
从结果可以看出虽然我们是在`1new_branch`上创建并纳入到缓存区的，但是在`master`分支上提交之后`1new_branch`上并没有新创建的文件。

**git stash**

我们有时会遇到这样的情况，正在`dev`分支开发新功能，做到一半时有人过来反馈一个bug，让马上解决，但是新功能做到了一半你又不想提交，这时就可以使用`git stash`命令先把当前进度保存起来，然后切换到另一个分支去修改bug，修改完提交后，再切回`dev`分支，使用`git stash pop`来恢复之前的进度继续开发新功能。

在官方文档的`Description`中描述会stash工作区和暂存区：

> Use `git stash` when you want to record the current state of the working directory and the index, but want to go back to a clean working directory. The command saves your local modifications away and reverts the working directory to match the `HEAD` commit.

但是在pop命令的解释中又说将stashed state应用到工作区当中，并没有提到会恢复index。

> Remove a single stashed state from the stash list and apply it on top of the current working tree state, i.e., do the inverse operation of `git stash push`. The working directory must match the index.

所以我们做如下验证：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/78d3d14d-7812-4ec6-b473-b39d10e3fb03" /></div>
从结果中可以看到最终是将stash前的工作区的内容恢复至了现在的工作区。

**HEAD**

`HEAD`是当前活跃分支的标志。如图表示当前活跃分支是`master`，下一次提交会在master分支上进行，并且其父提交是当前HEAD间接指向的提交。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/9a58c2bb-664b-4504-8af7-5cace76c6ac0" /></div>
**git reset**

修改当前活跃分支的指向。下面的参数是分支指向改变后暂存区和工作区的处理策略。

- `--soft`：Does not touch the index file or the working tree at all (but resets the head to `<commit>`, just like all modes do). This leaves all your changed files "Changes to be committed", as `git status` would put it.
- `--mixed`：Resets the index but not the working tree (i.e., the changed files are preserved but not marked for commit) and reports what has not been updated. This is the default action. 
- `--hard`：Resets the index and working tree. Any changes to tracked files in the working tree since `<commit>` are discarded.
- `--merge`：Resets the index and updates the files in the working tree that are different between `<commit>` and `HEAD`, but keeps those which are different between the index and working tree (i.e. which have changes which have not been added). If a file that is different between `commit` and the index has unstaged changes, reset is aborted.
- `--keep`：Resets index entries and updates files in the working tree that are different between `<commit>` and `HEAD`. If a file that is different between `<commit>` and `HEAD` has local changes, reset is aborted.

上图中如果我们将master从指向提交4改为指向提交3，此时如果我们没有事先保存提交4的commit id，想要返回上一次的提交点需要使用`git reflog`。为了方便，使用时可以在当前提交点打一个临时分支用于保存进度。

下面演示了四个过程：

- 在dev分支上打一个暂时的分支，用于保存进度

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/c87932d8-974e-49cc-bbf6-db5b27c84920" /></div>
- 回退dev分支至前一个节点

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/b959ab7b-def9-4af3-a7c4-59884174cca4" /></div>
- 在回退后的提交点上新建一个提交

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/318b1c44-2e88-4172-a1c3-c8963c3d6622" /></div>
- 切换至dev分支

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/fee82b9b-df08-4163-9077-b02dcc8a6c69" /></div>
**merge**

*fast-forward*

现在的`stash_dev`和`master`分支都在同一个提交点上。我们再`stash_dev`上新建一个提交，然后切换到master分支让他去合并`stash_dev`分支。这种合并就是`fast-forward`模式。也就是说发起合并分支相对于被合并分支是落后的状态，合并时只需要发起合并分支的指针向前移动就行。此时不会产生新的提交。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/cd8343c7-ebae-4298-bd27-872a30d5b87a" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/756bd147-0009-4e55-8638-eb0fddae943f" /></div>
*no-conflict*

我们在`master`分支上对`1test`文件追加一行`test merge no-conflict 1test`，在`stash_dev`分支上对2test分支追加一行`test merge no-conflict 2test`。然后在`master`上合并`stash_dev`。此时会产生新的提交。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/ce8e75ea-929e-44eb-9507-541d867f26d8" /></div>
为了更清晰的看到效果，我们使用`git log --graph`来查看提交历史。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/ea7ec93f-ba03-4965-bc68-261c15bd5f9c" /></div>
从图中可以看到在`4eb6`分支后`master`分支新产生一个`b114`提交，`stash_dev`分支新产生了一个`dbf5`提交，`master`分支合并`stash_dev`分支后产生一个新的提交`cf40`。此时如果我们再让`stash_dev`分支合并`master`就是`fast-forward`模式，因为上图已经很明显，`stash_dev`是直接落后`master`的。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/5f613bb2-8ed8-434e-a381-5e6b8d3badbc" /></div>
*handle-conflict-by-hand*

我们先查看两个文件的内容：`2test`和`3test`

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/50a485f4-8f6d-479d-bb4e-fe45f4b0934f" /></div>
在`stash_dev`分支上做如下操作：

- 对`2test`文件增加一行`test handle-conflict-by-hand stash_dev`
- 新建文件`4new_flie`并增加一行`new_file_4 stash_dev`
- 新建文件`5new_file`并增加一行`new_file_5 stash_dev`
- 新建文件`6new_file`并增加一行`new_file_6 stash_dev` 

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/36c674f7-0753-4cde-8ee6-b7db196ee7ca" /></div>
在`master`分支上做如下操作：

- 对`2test`文件增加一行`test handle-conflict-by-hand master`
- 6新建文件`4new_flie`并增加一行`new_file_4 master`
- 新建文件`5new_file`但不做其他操作

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e8cf3069-9988-488b-b705-fdae1020b417" /></div>
合并两个分支：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/0b7d02dd-a3f7-433d-8514-1660ffc9e014" /></div>
提示`5new_file`、`4new_file`和`2test`出现冲突，需要手动解决。

在`5new_file`中由于只有`stash_dev`分支对其进行操作，所以只显示了该分支的操作结果。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/d7f3c1ae-777c-4cb0-8a1f-754f5cfcbd7f" /></div>
在`2test`文件中，可以看到不同分支的操作被区分开来。此时需要我们手动解决冲突，如保留master的操作。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/3c250222-3cfe-4ba0-80ff-86c5111208f7" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/8ac53d44-53c9-49bb-bcde-e89372e08953" /></div>
在`4new_file`中，同样可以看到不同分支的操作被区分开来。此时保留master的操作。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/aa2d2309-477b-4333-ab16-69020ae309fa" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/8e86e2b0-a33b-4289-bf03-da9ae93611bb" /></div>
使用`git add`和`git commit`完成此次提交。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e1db3771-1457-4460-9d28-c50c9115d876" /></div>
然后我们在`stash_dev`分支上合并`master`，直接是`fast-forward`模式

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/689b14bd-635c-429a-8bc4-f73b1cd2dc2d" /></div>
我们接下来演示一下删除文件发生冲突时解决效果。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/4a7bcc32-5204-4f2a-a4be-fc7978e50d6c" /></div>
从结果中可以看到虽然我们在`master`分支上删除了`2test`，但是在发生合并冲突时可以选择是否保留文件。

**git revert**

此命令也是还原的意思，但是其作用是撤销某一次提交的结果。其也可能发生冲突，比如在如下的提交线中，如果提交2、3、4修改了同一个文件，在撤销提交3的时候就会发生冲突。

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/ef6527dd-24d1-42e9-8bfb-ed9cd8fd9ee2" /></div>
发生冲突后的解决方式和合并时的类似。下面的演示的命令效果：

- 新建一个文件`9test`并提交
- 追加一行`1th line`并提交
- 追加一行`2th line`并提交
- 追加一行`3th line`并提交

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/d55e5472-050a-42e4-9da0-2ecc4e4a5c0a" /></div>
提示我们可以`git rm 9test`或者修改后`git add 9test`（需要注意一点，解决冲突后的文件内容不要和revert之前的文件内容相同，否则无法构成一次提交，就会一直处于REVERTING阶段）。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/ced22a40-b38a-43e2-a233-bdfa92f778bb" /></div>
和`git reset`不同，`git reset`会扔掉提交，而此命令是新建一个提交，但是此提交会撤销指定提交的效果。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/b89817a3-38b5-49d8-82cd-08fc108bf10c" /></div>
**git restore进阶**

*git restore [--source=\<tree\>] [--staged] [--worktree] \<pathspec\>…*

在之前我们也使用过`git restore`，但是并没有解释过这个命令。我们先看看下面的官方解释：

> Restore specified paths in the working tree with some contents from a restore source. If a path is tracked but does not exist in the restore source, it will be removed to match the source.
>
> The command can also be used to restore the content in the index with `--staged`, or restore both the working tree and the index with `--staged --worktree`.
>
> By default, the restore sources for working tree and the index are the index and `HEAD` respectively. `--source` could be used to specify a commit as the restore source.

官方的解释很清晰，此命令是用于将指定source的文件恢复至暂存区或者工作区。我们下面演示的便是将指定提交的`9test`文件恢复至工作区。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/922f9146-d1cb-48d2-9eec-27920f70693f" /></div>
**cherry-pick**

应用其他分支上的提交至本分支。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/2d26a400-24a7-4142-accf-9100f7559097" /></div>
如果此时在`master`分支上我们对`9test`文件作出修改，再`cherry-pick`就会产生冲突。需要手动合并。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/4ca2fb6e-c4cf-4623-a8a3-20f5f5afaa72" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/a99597cd-eb1d-45a6-ae4a-9c54aba2582b" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/cfdb91cc-a712-438b-a02b-ff8848f152d8" /></div>
**git rebase**

此命令产生的最终结果和`merge`一致，但是所用的方式却不同。假如我们有如下提交：

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/ca060ff5-988b-48fb-a1a8-02653a69a83e" /></div>
如果我们再`mywork`分支上合并`origin`，会产生如下结果：

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/124eceaa-7f2f-4b96-a609-dcbf748e7249" /></div>
但是使用`git rebase`后就会是如下结果：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/51083682-d4ca-4eab-8f9c-a44f90478fe6" /></div>
也就是说`rebase`是将本分支上分离出来的提交依次应用到指定分支上。

创建一个文件`8test`并提交，然后新建一个分支`rebase_branch`：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/aa28c5ba-78b7-4a58-94cb-8f407f448f53" /></div>
在`master`分支上新建两个提交，分别向`8test`文件中追加一句话：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/c54c2779-a301-4005-8887-389edd173d6e" /></div>
此时我们可以看到提交历史如下：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e4e83e82-a1e3-4d78-9c94-0403f8ccb0e5" /></div>
此时切换到`rebase_branch`分支上新建一个文件`10test`并提交，再向`8test`文件中追加一行并提交：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/bc9b6f64-878d-4017-8235-9a3908ba1ce6" /></div>
此时我们在`rebase_branch`分支上应用`git rebase`变基到`master`分支上。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e5fa0335-f013-454a-84bb-4fe53262e4a2" /></div>
在输出的信息中可以看到`Applying: add file 10test`是成功的。而`append one line on rebase_branch`时提示合并冲突。并且给出了三种解决方案：

1. 手工解决冲突，然后使用`git rebase --continue`继续rebase操作
2. 使用`git rebase --skip`跳过此次提交
3. 使用`git rebase --abort`丢弃此次rebase，此命令会将两个分支都恢复到rebase操作之前。

同时我们可以从`(rebase_branch | REBASE 2/2)`中看到一共有两个提交，此时是处于合并第二个提交的过程中。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/0120e74a-4185-45b6-bfc4-dc7f0e96d465" /></div>
选择手动修改并继续rebase之后查看提交记录。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/8e989564-00e8-48eb-9887-de9a1d260ddf" /></div>
此时`master`分支落后`rebase_branch`分支两个提交，符合预期。



## GitHub基础

**添加ssh秘钥**

本地仓库和GitHub之间通信的时候都是使用加密协议进行通信的，常见的加密协议有两种https和ssh，这两种都可以用于GitHub和本地仓库通信，但是https协议在使用时经常需要输入账户名和密码进行登录，所以我们一般都使用ssh协议。

使用ssh协议需要先在本地生成公钥和私钥（按照提示一步步的进行就可以了）：

```
ssh-keygen -t rsa -b 2048 -C "你自己的邮箱地址"
```

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e214dbc6-875f-4696-a7cf-b79d2ccc6bbf" /></div>
生成完秘钥后将公钥复制后加入GitHub的SSH秘钥页里。

<div align="center"><img width="1000%" src="http://blogfileqiniu.isjinhao.site/5e6350f5-5593-4fd5-9461-fb28bf02ad52" /></div>
**git clone**

在GitHub上新建一个repository，命名为`git-remote`：

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/06e800a8-7dff-4aef-9e6e-6f871bc34298" /></div>
获得他的ssh链接后使用此命令可以将远程仓库的信息克隆下来：

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/56531c23-8a43-47cd-98cd-093797622148" /></div>
**git remote**

为了便于管理，Git要求每个远程主机都必须指定一个主机名。`git remote`命令就用于管理主机名。`git remote`命令列出所有远程主机。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/a847b351-c8d8-4788-8c34-5ae207c87af3" /></div>
上面命令表示，当前只有一台远程主机，叫做origin，以及它的网址。

克隆版本库的时候，所使用的远程主机自动被Git命名为`origin`。如果想用其他的主机名，需要用`git clone`命令的`-o`选项指定。如：`$ git clone -o jQuery https://github.com/jquery/jquery.git`

*git remote show*

此命令加上主机名，可以查看该主机的详细信息。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/b566a8a5-70b3-4821-a13f-e50902767e62" /></div>
*git remote rm*

此命令用于删除远程主机。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/d841d7f4-965e-417f-9b02-38fed37008aa" /></div>
*git remote add*

此命令用于添加远程主机

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/4010fa6b-06a0-4683-b890-30c5b70b0149" /></div>
**git push**

此命令用于将本地分支的更新，推送到远程主机。

```
$ git push <远程主机名> <本地分支名>:<远程分支名>
```

我们现在本地的`master`分支上新建一个提交

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/b5912fba-b681-4b86-9fc3-ea42f494943a" /></div>
然后我们将本地`master`分支推送至远程（此时不需要本地分支也处于`master`上）。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/5baa8f6c-6a65-4a33-9a6d-e0632c26d1d3" /></div>
<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/3ebe2c06-e2c7-4555-b23f-584c68bcf0cf" /></div>
然后我们新建一个分支，再在新的分支上建立一个提交，对`1test`文件追加一行

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/72c58039-98a9-484a-8a4d-f96f430505e0" /></div>
此时本地有分支`1new_branch`，远程没有此分支，此时推送`1new_branch`会自动在远程创建同名分支。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/fa36d81e-4e1d-4742-808e-ce136c772261" /></div>
<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/14bf8298-9db6-4c00-9eec-516f44a4ac5d" /></div>
在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在`git clone`的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的`master`分支自动"追踪"`origin/master`分支。Git也允许手动建立追踪关系。

```
git branch --set-upstream <本地分支> <远程分支>
```

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。比如下面命令表示将当前分支推送到`origin`主机的对应分支，可以直接使用如下命令

```shell
git push origin
```

如果当前分支只有一个追踪分支，那么主机名都可以省略

```
git push
```

如果当前分支与多个远程主机存在追踪关系，则可以在推送时使用`-u`选项指定一个默认主机，这样后面就可以直接使用`git push`。如下面的命令将本地的`master`分支推送到`origin`主机，同时指定`origin`为默认主机。

```
git push -u origin master
```

不带任何参数的`git push`，默认只推送当前分支，这叫做`simple`方式。此外，还有一种`matching`方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用`matching`方法，现在改为默认采用`simple`方式。如果要修改这个设置，可以采用`git config`命令。

```shell
git config --global push.default matching
# 或者
git config --global push.default simple
```

如果我们想将本地的所有分支都推送到远程主机，需要使用`–all`选项。如下面的命令表示将所有本地分支都推送到`origin`主机。

```
git push --all origin
```

**git fetch**

此命令用于将修改从远程主机的分支上拉取到本地的远程分支上。如`git fetch origin master`会取回`origin`主机的`master`分支，而所取回的更新，在本地主机上要用”远程主机名/分支名”的形式读取。比如`origin`主机的`master`分支，就可以用`origin/master`读取。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/46c8a3cf-2ce2-4d2d-848e-83f756fb6903" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e7142853-0324-420b-a4d2-4582ba68a9f6" /></div>
虽然我们可以看见本地主机的远程分支但是不要在上面做提交，应该将其合并到普通的本地分支上再在普通本地分支上做提交。

`git branch`命令的`-r`选项，可以用来查看远程分支，`-a`选项查看所有分支。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/6da55dce-83a8-4bb5-ad00-ad79f32c6d9c" /></div>
**git pull**

`git pull`可以看出`git fetch`和`git merge FETCH_HEAD`的缩写。作用是取回远程主机某个分支的更新，再与本地的指定分支合并。

```
$ git pull <远程主机名> <远程分支名>:<本地分支名>
```

 比如，取回`origin`主机的`next`分支，与本地的`master`分支合并，需要写成下面这样。

```
$ git pull origin next:master
```

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

```
$ git pull origin next
```

比如我们现在在GitHub上手动进行了一次提交

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/1f2d0b13-02c3-482e-9d0e-9193c68b4eee" /></div>
<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/e856d411-2d35-4e42-aae3-891777c53615" /></div>
如果当前分支只有一个追踪分支，可以省略远程主机名。下面命令表示，当前分支自动与追踪分支进行合并。

```
$ git pull
```

如果合并需要采用rebase模式，可以使用`--rebase`选项。

```
$ git pull --rebase <远程主机名> <远程分支名>:<本地分支名>
```

如果远程主机删除了某个分支，默认情况下，`git pull` 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致`git pull`不知不觉删除了本地分支。

但是，你可以改变这个行为，加上参数 `-p` 就会在本地删除远程已经删除的分支。

```
$ git pull -p
# 等同于下面的命令
$ git fetch -p
```

> Incorporates changes from a remote repository into the current branch. In its default mode, git pull is shorthand for git fetch followed by git merge FETCH_HEAD.
>
> More precisely, git pull runs git fetch with the given parameters and calls git merge to merge the retrieved branch heads into the current branch. With --rebase, it runs git rebase instead of git merge.

从解释中可以看到此命令是将拉取到的分支合并到当前分支上，所以在应用此命令之前需要注意切换分支。

```
git pull <远程主机名> <远程分支名>:<本地分支名>
```





**git tag**

Git 使用的标签有两种类型：轻量级的（lightweight）和含附注的（annotated）。轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。而含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用 GPG 来签署或验证。一般我们都建议使用含附注型的标签，以便保留相关信息。当然，如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签也没问题。

创建一个含附注类型的标签非常简单，用 `-a` (译注：取 annotated 的首字母)指定标签名字即可：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/2772ca5a-13ce-45a6-88bb-c0ef775a23d6" /></div>
轻量级标签语法：

```
git tag <标签>
```

显示所有的标签：

```
git tag
```

默认情况下，`git push` 并不会把标签传送到远端服务器上，只有通过显式命令才能分享标签到远端仓库。其命令格式如同推送分支，运行 `git push origin [tagname]` 即可：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/92735eb0-b452-4427-98ad-40b8457a5e9e" /></div>
<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/c41761e3-3864-4c49-8147-e27481d01353" /></div>
## GitHub协作开发

**fork**

再申请一个GitHub账户，访问我们当前的项目。此时就可以进行fork，fork就是将别人的项目复制一份到自己的账户下。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/fab23a57-1055-472e-a399-eb0317eb6a73" /></div>
<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/df77b911-56f8-4a5f-822f-6a38d823efe4" /></div>
**拉到本地进行修改**

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/5bc6f26e-1265-4a53-937c-6acb7b43c584" /></div>
但此时我们没有push的权限，需要新账户将旧账户设置为此项目的协作者。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/0f555c2a-d61e-4c9b-ad60-53a72a3f1de0" /></div>
此时在本地就可以进行推送操作了

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/b5000987-95c9-4c30-892f-bb23f23f3328" /></div>
推送完成之后进入新账户进行pull request

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/9cfc1836-cc8a-4cbc-90c2-03f7dd3e251f" /></div>
<hr/>
<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/c99bed9b-057b-49b6-bb56-14213f208ae5" /></div>
进入原账户查看pull requests

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/99de1605-d1f5-45a7-9ae4-777b0a84367e" /></div>
点击`merge pull request`就可以进行合并

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/9f29a46d-e741-4f24-b9be-9af865a32107" /></div>
我们再新建一个pull request，让新项目的master分支合并到1new_branch上

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/6f09899a-c8a5-416c-9067-fff00dfada3a" /></div>
此时我们就会发现现在有冲突

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/c7e6bc5a-771b-4d2c-9b37-141321b66b25" /></div>
点击resolve conflicts按钮来解决冲突

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/6751c166-0c40-47bb-a808-8417fe51f04a" /></div>
修改完成后就可以标记，全部标记后就可以合并

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/059f1f5e-ca5e-4395-97d4-7775098e9b45" /></div>
现在再查看就是合并之后的内容了

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/776692d3-4953-4078-b911-293856850814" /></div>