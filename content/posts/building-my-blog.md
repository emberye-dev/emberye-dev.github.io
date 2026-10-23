---
title: "从一次命令行报错开始：我用 Hugo 搭起了第一个博客"
date: 2026-10-23
draft: false
tags: ["Hugo", "博客搭建", "GitHub Pages", "踩坑记录"]
categories: ["思考与成长"]
summary: "记录从零搭建这个站点的完整过程：一次失败的 choco 安装、两次被重置的 clone、一个改名的决定，以及最后看到网站在线的那个瞬间。"
---

## 为什么先搭博客，而不是先写文章

其实在很早之前，我手里已经有一份写得很好的规划文档：定位、栏目设计、第一篇该写什么，全都列好了。唯一没定的是一个很小的东西——**博客用什么平台**。

就卡在这一步，卡了很久。

原因说出来有点可笑：我怕选错。我去询问了ai的意见，结果越来越纠结：掘金太浅、自建服务器太重、Hexo 要装 Node、WordPress 要续费。我在"选哪个"上消耗的精力，远远超过真正动手需要的成本。

后来我意识到，这跟技术无关，是另一种形式的拖延：**用一个看起来很重要的决策，掩盖自己不敢开始的事实**。于是，在拖延了快一个礼拜后，我给自己定了个标准——维护成本最低、内容就是纯 Markdown、仓库本身能当作品。ai很快就给了我答案：Hugo + GitHub Pages。

这篇文章就是那次动手的完整记录。中间踩的坑比预想的多，但每一个坑都值得写下来。

## 第一步：装 Hugo，第一条命令就失败了

按照常见教程，第一个命令是：

```powershell
choco install hugo-extended
```

结果：

```text
choco : 无法将“choco”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。
```

我下意识想去装 Chocolatey，但停下来想了一下：我只是要一个可执行文件，为什么先要装一个包管理器？

Windows 10 以后自带 `winget`，查一下确认可用：

```powershell
winget --version   # v1.29.380
```

于是直接装：

```powershell
winget install --id Hugo.Hugo.Extended -e
```

装完提示"已修改路径环境变量，重启 shell 以使用新值"。这里有个细节值得记下来：**新装的命令在当前已经打开的终端里是找不到的，必须重开一个窗口**。我第一次没注意，差点以为装失败了。

## 第二步：建站，顺利得不像真的

```powershell
cd D:\mine\Hugo
hugo new site blog --format yaml
cd blog
git init
```

三行命令，一个站点骨架就出来了。`--format yaml` 是让配置生成成 `hugo.yaml` 而不是默认的 `toml`，纯粹是我个人更喜欢 YAML 的可读性。

到这一步为止，一切都很顺。然后我撞上了第一个真正的墙。

## 第三步：装主题，连接被重置

PaperMod 是社区里最常用的 Hugo 主题之一，官方安装方式是 git submodule：

```powershell
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

结果：

```text
fatal: unable to access 'https://github.com/...': Recv failure: Connection was reset
```

换用 `gh-proxy` 镜像重试，**同样失败**。这时候我犯了一个错误：连续两次尝试同一条路径，只是在换参数。这属于无效重试。

真正有效的做法是换思路——**我要的是主题文件，不是 git 的 submodule 机制**。于是改成直接下载压缩包：

```powershell
Invoke-WebRequest -Uri "https://gh-proxy.com/https://github.com/adityatelange/hugo-PaperMod/archive/refs/heads/master.zip" -OutFile papermod.zip
```

288KB，下载成功。

解压后还遇到一个小问题：`Rename-Item` 的 `-NewName` 参数**不能带路径**，我第一次写成 `Rename-Item themes\hugo-PaperMod-master themes\PaperMod` 直接报错。正确写法是把目录和名字拆开传。

这带来一个后果：主题不是 submodule，而是普通文件，会被一起提交进仓库。对个人博客来说这反而是好事——**部署时不需要额外拉取子模块，少一个失败点**。

## 第四步：配置和第一篇

`hugo.yaml` 里主要改四类东西：`baseURL`、站点标题作者、导航菜单、首页自我介绍。

这里我留了个坑：`languageCode` 这个键在 Hugo 0.158 之后被废弃了，改叫 `locale`。第一次构建时出现了一行 WARN，虽然不影响运行，但既然看到了就顺手改掉——**警告不等于错误，但攒着迟早会变成错误**。

然后写了第一篇文章，起了预览服务验证：

```powershell
hugo server -D
```

首页返回 200，文章页返回 200，正文渲染正常。到这一步，本地已经是一个能用的博客了。

## 第五步：改名，以及一个差点埋下的坑

`Patrick-shock` 这个 ID 是我大一开始用的，跟现在的我没什么关系了。决定换名字的时候，我没有随便选一个英文单词，而是把三条线索摆在一起：

- 本名 **陈烨**，烨是火盛、光明
- 网名 **绒烬**，绒是质地，烬是余烬
- 最喜欢的角色 **Eyjafjalla**，名字直接来自冰岛的一座火山

火、火山、余烬。这三样本来就是一条线。于是有了 **Emberye**：Ember（余烬）+ ye（烨）。

改名本身在 GitHub 网页上点几下就完成了，但有个坑差点埋下：**GitHub Pages 的仓库必须叫 `<用户名>.github.io`，而仓库名不会跟着账号名一起改**。如果只在设置里改了账号名，旧仓库还叫 `Patrick-shock.github.io`，Pages 会直接挂掉。

所以我干脆把旧仓库删了，重新建一个空的。删之前确认过两点：一是删除不可撤销，但本地代码不受影响；二是旧站点的失效是预期行为。

建新仓库时也有讲究：**README、.gitignore、license 全部不要勾**。因为本地已经有一个完整的仓库，如果远端先初始化一次提交，push 时就会撞上 `non-fast-forward`，还得额外处理。空仓库才能一次推干净。

## 第六步：两个失败，才推上去

第一次 push，失败：

```text
Recv failure: Connection was reset
```

但奇怪的是，浏览器能正常打开 GitHub。这说明**网络本身是通的，只是 git 没走代理**。查了一下系统设置，代理确实开着（`127.0.0.1:7890`），而 git 默认不读 Windows 的系统代理。加上配置后网络就通了：

```powershell
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

第二次 push，还是失败，但错误变了：

```text
remote: Invalid username or token. Password authentication is not supported for Git operations.
```

这次不是网络问题，是**凭据问题**。改过用户名之后，本机凭据管理器里缓存的还是旧账号 `Patrick-shock` 的令牌，GitHub 自然不认。麻烦的是这类凭据用 `cmdkey /delete` 删不掉（会提示参数不正确），最后是用 Windows 的 `CredDelete` API 直接删的，再重新走一次浏览器授权。

第三次 push：

```text
* [new branch]      main -> main
```

成功。

## 结果

GitHub Actions 自动跑完构建，站点上线：

- `https://emberye-dev.github.io/` → 200
- 首页自我介绍渲染正常
- 文章页渲染正常

Actions 里还留了一条红色的失败记录，是 GitHub 默认的"从分支部署"工作流，它试图用 Jekyll 构建 Hugo 的源码目录，当然会失败。**它不影响站点**，真正生效的是我配的 Hugo 工作流。这个红叉我暂时留着，当作一个提醒：默认配置不等于正确配置。

## 回头看，学到了什么

技术上的收获是具体的，但我觉得更重要的是另外三件事。

**第一，无效重试和有效排查的区别。** clone 失败两次，我第二次只是换了个镜像，本质还是同一条路。真正解决问题的是重新问一遍"我到底需要什么"——需要的是文件，不是 submodule。这个习惯比记住任何一条命令都值钱。

**第二，报错信息要读完。** 网络错误和认证错误的提示完全不同，但都很容易被一句"push 失败"糊弄过去。分清"通不通"和"认不认"，排查范围立刻缩小一半。

**第三，开始的门槛比想象中低，但拖延的伪装比想象中深。** 整个搭建过程加起来不过几十条命令，真正花时间的部分是那几次卡住和想放弃的时刻。我从决定开始写博客，一直到写这篇记录，其实已经过了快一个礼拜。我才发现：那些我以为的"技术障碍"，一半是环境问题，一半是心理问题。

博客现在还很空，只有这一篇和一篇自我介绍。但是至少我真的已经开始了——这件事本身就比内容更重要。

希望我接下来可以养成每天写博客的习惯。

---

附：以后发新文章的流程（还是加一个吧，防止刚开始不熟练老是忘记）

```powershell
cd D:\mine\Hugo\blog
hugo new content posts/文章文件名.md
# 编辑 content\posts\文章文件名.md，把 draft 改成 false
git add .
git commit -m "post: 文章标题"
git push
```
