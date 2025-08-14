1. 当众承诺，荣誉锁定

2. 基本用法

   > <img src="https://mmbiz.qpic.cn/mmbiz_png/v1JN0W4OpXjuEjicbH8PQibIRkiaNHr9ibqusjLdmEbIVyxtkjfdtVF9qLMkEkVD69ZwzCuOHiaEFczrkAtib8ic7JznA/640?wx_fmt=png&randomid=gfppso45&tp=webp&wxfrom=5&wx_lazy=1" alt="img" style="zoom: 67%;" />
   >
   > ```shell
   > # 本地仓库、暂存区、工作区
   > git commit # 将暂存区生成快照并提交
   > git add    # 将当前文件放入暂存区
   > git reset -- file #  撤销最后一次git add后的暂存区文件
   > git checkout -- file # 将文件从暂存区复制到工作区，丢弃本地修改
   > ```
   >
   > <img src="https://mmbiz.qpic.cn/mmbiz_png/v1JN0W4OpXjuEjicbH8PQibIRkiaNHr9ibqud6LC2aicjNKDWT21Hia4rNsykKdUKvNLicCGEb17M3RSz3ica06iaGsJ26Q/640?wx_fmt=png&randomid=0x0f5u5k&tp=webp&wxfrom=5&wx_lazy=1" alt="img" style="zoom:67%;" />
   >
   > ```shell
   > # 跳过暂存区
   > git commit -a # 运行git add把所有当前目录下的文件加入暂存区，然后再运行git commit
   > git commit files # 只提交指定的文件（需要先被git add，或者用-a提交），不提交其他内容
   > git checkout HEAD --files # 放弃工作区某个文件的修改，恢复成HEAD（最近一次提交）时的状态
   > ```

3. diff

   > <img src="https://mmbiz.qpic.cn/mmbiz_png/v1JN0W4OpXjuEjicbH8PQibIRkiaNHr9ibqulXh7n4XFXW9Su6bIuaYrO1QVSG0DUx3XQV39r47DtTibzpKMWybdCaA/640?wx_fmt=png&randomid=jdugu9cq&tp=webp&wxfrom=5&wx_lazy=1" alt="img" style="zoom:67%;" />
   >
   > ```shell
   > # 你刚刚编辑了文件，但还没 git add，你想看看改了什么。
   > git diff # 工作区-暂存区
   > # 你已经 git add 了一些文件，但还没 git commit，你想看看这次提交会提交什么内容。
   > git diff --cached # 暂存区-最近一次提交（head）
   > git diff --staged # 暂存区-最近一次提交（head）
   > # 你想整体看下当前修改（无论是否 add）和最后一次提交之间的区别。
   > git diff HEAD # 工作区和暂存区的联合状态-最近一次提交（HEAD）
   > git diff && git diff --cached # 等价写法
   > # 比如你当前在 feature1 分支开发新功能，你想看一下你和 stable（通常是主分支之一）之间有多大差别。
   > git diff stable # 当前工作分支-stable分支的最新提交
   > # 你想知道某一段提交历史中，具体都改了哪些内容。
   > git diff abc1 abc3 # 两个指定提交之间
   > ```

4. commit

   > <img src="https://mmbiz.qpic.cn/mmbiz_png/v1JN0W4OpXjuEjicbH8PQibIRkiaNHr9ibquicW9XtH5x8uZYXgxTibAzI6NMHyYQicTQpx088lCjODPjyntB6fxcqEwA/640?wx_fmt=png&randomid=yy1x2gcs&tp=webp&wxfrom=5&wx_lazy=1" alt="图片" style="zoom:67%;" />
   >
   > ```sh
   > git commit 
   > # 用暂存区的文件，创建一个新的提交，把此时节点设为父节点，把当前分支指向新提交的节点
   > # 当前分支是main，指向ed489，提交时，main指向新节点f0cec，并以ed489作为父节点
   > ```

5. 总结：

   > | 区域/操作                        | 常用命令                                                   | 含义与注意点                                                 |
   > | -------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
   > | **初始化与配置**                 | `git init / clone` & `git config`                          | 建 `.git` 仓库；全局 vs 局部配置                             |
   > | **工作区变更 → 暂存区**          | `git add <file>`                                           | 新文件、修改文件写入索引                                     |
   > | **查看当前状态**                 | `git status`                                               | 查看哪些变更已 add，哪些还没写入缓存                         |
   > | **查看差异**                     | `git diff`                                                 | 默认显示 *未 add 的变更* 与暂存区对比                        |
   > | **查看已缓存与上次提交差异**     | `git diff --cached` 或 `--staged`                          | 比较暂存区内容与最新 commit 差异                             |
   > | **查看暂存 + 未暂存 两部分差异** | `git diff HEAD`                                            | 等同同时比较 `--cached` 与 `工作区` 差异                     |
   > | **提交变更**                     | `git commit [-m / -a]`                                     | `-a` 跳过 `git add`；默认 commit 暂存区所有内容              |
   > | **放弃缓存或回滚变更**           | `git reset [--mixed/soft/hard]`                            | `--soft`只修改HEAD，`--mixed` 修改索引但保留工作区，`--hard` 工作区+索引一起回退 |
   > | **切换分支或状态**               | `git checkout branch / tag / SHA`                          | 如果是分支，切换文件快照；如果以 SHA/tag 形式 checkout，则进入 detached HEAD 状态。在这个状态下提交不会更新任一分支，除非你用 `git checkout -b new` 保留它。否则容易丢失改动。 \| |
   > | **分支管理 & 合并**              | `git branch`, `git checkout -b`, `git merge`, `git rebase` | Merge 保留两条并行分支历史；Rebase 会“线性化”历史，相当于 sequential cherry‑pick |
   > | **选取特定提交到当前分支**       | `git cherry-pick <commit>`                                 | 将某次提交的内容复制到当前分支，生成新的提交                 |
   > | **跨项目版本合入**               | `git tag`, `git push`, `git pull`, `git fetch`             | `pull = fetch + merge`；fetch 只拿远端数据不自动合并         |
   > | **深入理解 Git 流程**            | `git log`, `reflog`, `.git/objects` 探索                   | 理解 Git 是如何通过 tree/blob/commit 建立内容寻址结构，以及 reflog/gc 是如何管理“遗忘的提交” |

6. 小林coding

   > 1. git的工作原理
   >
   >    > Git：分布式版本控制系统，开发者本地拥有完整仓库，无需中央服务器，支持离线提交和灵活分支管理
   >    >
   >    > ![image-20250320161914470](https://cdn.xiaolincoding.com//picgo/image-20250320161914470.png)
   >    >
   >    > - WorkSpace：工作区：实际编辑的地方
   >    > - Index/Stage：暂存区：执行力`git add`并准备`git commit`的快照
   >    > - Repository：本地仓库：最终提交的版本历史，HEAD 是指向当前分支的游标
   >    > - Remote：远程仓库
   >
   > 2. 工作流程
   >
   >    > - 首先`git pull`或`git clone`或`git init`获取远程仓库（Github、Gitlab）的最新代码，或者从零开始创建新项目（需要绑定远程仓库` git remote add origin <url>`）
   >    > - 创建新分支准备开发功能`git checkout -b feature/something`
   >    > - 完成相应功能开发后，`git add .`将工作区代码添加到暂存区
   >    > - 然后`git commit -m "something"`，将暂存区代码提交到本地仓库并添加注释
   >    > - 再`git push`，推送到远程仓库做同步
   >    > - 如果要做合并分支：切换到主分支`git checkout main`，获取最新代码`git pull origin main`，做合并`git merge`；发生冲突，手动解决后再保存`git add`并`git commit`，再推送到主分支`git push origin main`
   >
   > 3. 撤回本地的提交
   >
   >    > ![img](https://i-blog.csdnimg.cn/blog_migrate/6ba2cf0c133939b1d54d3243f584f59c.png#pic_center)
   >    >
   >    > ```sh
   >    > # 移动当前分支指针HEAD位置，删除之后所有的提交
   >    > git reset --<option> <commit-id>
   >    > # 移动HEAD，只需要commit就可以回到reset之前的状态
   >    > git reset --soft HEAD^
   >    > # 移动HEAD，重置了暂存区（和HEAD内容一样），工作区保留
   >    > git reset --mixed HEAD^
   >    > # 移动HEAD，重置了暂存区和工作区，所有修改完全丢失，可以使用git reflog后悔
   >    > git reset --hard HEAD^
   >    > ```
   >
   > 4. 撤回远程仓库的推送
   >
   >    > - `git reset` + 强制推送
   >    >
   >    >   > ```sh
   >    >   > # 将本地仓库回退到当前版本的上一个版本，再将本地仓库当前版本强制推送到远程仓库
   >    >   > git reset --hard HEAD^
   >    >   > git push origin main --force
   >    >   > ```
   >    >
   >    > - 使用`revert`撤销某次提交
   >    >
   >    >   > ```sh
   >    >   > # 撤销最近一次提交，生成一个新的提交，内容是某个指定提交的反操作
   >    >   > git revert HEAD
   >    >   > git push origin main
   >    >   > ```
   >
   > 5. git merge和rebase的区别
   >
   >    > - `merge`
   >    >
   >    >   > 将两个分支的历史合并在一起，保留各自的提交记录，适合多人协作
   >    >   >
   >    >   > Git会创建一个**新的合并提交**，将两个分支的提交历史链接在一起
   >    >   >
   >    >   > ```sh
   >    >   > git checkout main
   >    >   > git merge feature
   >    >   > ```
   >    >   >
   >    >   > 
   >    >
   >    > - `rebase`
   >    >
   >    >   > 将一个分支的提交挪到另外一个分支的后面，实现线性提交历史，适合整理提交记录
   >    >   >
   >    >   > Git会将目标分支与源分支的共同祖先以来的所有提交挪到目标分支的最新位置
   >    >   >
   >    >   > ```sh
   >    >   > # 将feature的提交移到main后面
   >    >   > git checkout feature
   >    >   > git rebase main
   >    >   > ```
   >
   > 6. 合并两个分支，在冲突发生时（CONFLICT）如何解决
   >
   >    > - 查看并定位冲突文件`git status`
   >    >
   >    > - 解决冲突
   >    >
   >    >   > 打开包含冲突的文件，会有冲突标记，进行手动编辑
   >    >   >
   >    >   > ```cpp
   >    >   > <<<<<<<< HEAD
   >    >   > // 代码来自目标分支
   >    >   > void login(pair<int, string> NumANDPwd);
   >    >   > =========
   >    >   > // 代码来自要合并的分支
   >    >   > void login(int Num, string Pwd);
   >    >   > >>>>>>>>> branchName
   >    >   > ```
   >    >
   >    > - 标记并提交



