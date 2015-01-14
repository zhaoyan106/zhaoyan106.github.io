---
layout: post
title: "git分支模型"
date: 2014-10-13 09:39:03 +0800
comments: true
categories: 
---
###主分支(main branches)
&#160; &#160;&#160; &#160;本质上，该开发模型是从现有的模型中获得灵感而产生的。中心仓库中维护着两个永生不灭的主分支：

*master

*develop


&#160; &#160;&#160; &#160; 每一个Git用户应该都熟悉origin中的master分支。与master平行，还存在另一个被称为develop的分支。
&#160; &#160;&#160; &#160;我们将origin/master考虑成这样的一个主分支，其源码的HEAD始终代表了产品就绪的状态。
&#160; &#160;&#160; &#160;我们将origin/develop考虑成这样的一个主分支，其源码的HEAD总是代表已经并入了最新的开发变更的状态，这些变更将用于下次的发布。有人喜欢将其称之为“集成分支(integration branch)”。这也是自动的nightly build的源码来源。
&#160; &#160;&#160; &#160;当develop分支中的源码达到一个稳定点且准备发布时，所有的变更应当被合并回master分支，然后将打上版本号标签。至于具体是怎么实现的，接下来将进一步讨论。

&#160; &#160;&#160; &#160;因此，按(我们的)定义，每当变更被合并回master，都将导致一个新产品发布。我们倾向于采用非常严格的策略，因此理论上，我们可以使用一个Git的脚本钩子，当master上有一个提交(commit)时，可以自动构建并将软件转到产品服务器。

###辅助分支(Supporting branches)
&#160; &#160;&#160; &#160;除了主分支master和develop，我们的开发模型使用了各种各样的辅助分支来——帮助团队成员间的并行开发，减轻特性的追踪的痛苦，准备产品的发布以及快速修复发布产品中的缺陷。不同于主分支，这些分支的生存周期总是有限的，因为它们最终会被移除。

我们可能用到的不同的分支类型：
		
&#160; &#160;&#160; &#160;特性分支(Feature branches)

&#160; &#160;&#160; &#160;发布分支(Release branches)

&#160; &#160;&#160; &#160;快速修复分支(Hotfix branches)

&#160; &#160;&#160; &#160;这些分支中的每一个都有着特定的目的，也必须遵守严格的规则：哪些它们所起源于的分支，也是它们必须要合并到的目标分支。我们将很快将介绍一遍它们。

&#160; &#160;&#160; &#160;从技术角度看，这些分支绝没有任何“特殊性”。分支类型是依据我们的使用方式划分的。毫无疑问，它们仍是普通的Git分支。

特性分支(Feature branches)

可能起源于分支：develop

必须合并回分支：develop

分支命名惯例：除 master、develop、release-* 或 hotfix-* 外的任何名字


&#160; &#160;&#160; &#160;Feature分支(有时被称为topic分支)，用于为即将到来的或远期的版本开发新功能。当开始开发一个新的功能时，此时包含该功能的目标可能尚未很好定义。Feature分支的本质是，在新功能开发期间始终存在；但是它最终要合并回develop分支(将添加到下次发布的版本中)或者废弃(在实验让人沮丧的情况下)。

典型情况下，feature分支只存在于开发者的仓库中，而不在origin中出现。

####创建feature分支

&#160; &#160;&#160; &#160;当开始工作于一个新功能时，从develop分支创建新的分支：

	$ git checkout -b myfeature developSwitched to a new branch "myfeature"
将完成的feature并入develop

完成的特性将会合并回develop分支，并终会被加入将到来的release中：

	$ git checkout developSwitched to branch 'develop'$ git merge --no-ff myfeatureUpdating ea1b82a..05e9557(Summary of changes)$ git branch -d myfeatureDeleted branch myfeature (was 05e9557).$ git push origin develop
标记--no-ff使得，即使在可以fast-forward的条件下，合并操作也总是生成一个commit对象。这可以避免feature分支中历史信息的丢失，并可使feature分支中的所有commit保持仍在一块。比较：


&#160; &#160;&#160; &#160;在后一种情况下，从Git的历史中是不可能看出，哪些commit对象共同实现了一个特性——你必须手动阅读所有的日志信息。而要移除整个特性(即，一组commit)，在后一种情况下真真切切地让人头痛；而若使用了--no-ff标记，则很容易做到。

&#160; &#160;&#160; &#160;当然，这会创建一些空的commit对象，但收益比这点浪费要大得多。

&#160; &#160;&#160; &#160;不幸的是，我尚未找到使得git merge中默认行为启用--no-ff的方法，但它真的很需要如此。

####发布分支(Release branches)
可能起源于分支：develop

必须合并回分支：develop 和 master

分支命名惯例：release-*

&#160; &#160;&#160; &#160;Release分支用于做新产品发布前的准备。它使得在最后一刻可以进行细枝末节的完善。更进一步讲，它允许小bug的修复以及发布所需元数据(版本号，构建日期等)的准备。通过在release分支做这些工作，develop分支可以干净地接收为下一个大发行版准备的功能。

&#160; &#160;&#160; &#160;从develop创建release分支的关键时机是，develop分支(几乎)可以反映新版本就绪状态的时刻。至少是将要建构的发行版所有的feature分支都已经被合并到了develop分支的时刻。所有的针对未来版本开发的feature可能不会被并入——它们必须等到该release分支创建以后。

&#160; &#160;&#160; &#160;正是在release分支开始时，即将发布的产品才被分配一个版本号——而不是先前。直到这一刻，develop分支才开始反映“下一版”的变化，但是在下一个release分支开始前，下一版最终将变成0.3还是1.0都是不明确的。这种决定是在release分支开始时，通过项目的版本号规则实施的。

####创建发布分支

&#160; &#160;&#160; &#160;Release分支是从develop分支中创建而来。例如，假定1.1.5是当前产品的版本，而我们即将有一个大的发行版。Develop分支对于“下一版”已经就绪，而且我们已经确定这将是版本1.2(而不是1.1.6或2.0)。因此，我们创建release分支并赋予它一个反映新版本号的名字：

	$ git checkout -b release-1.2 developSwitched to a new branch "release-1.2"$ ./bump-version.sh 1.2Files modified successfully, version bumped to 1.2.$ git commit -a -m "Bumped version number to 1.2"[release-1.2 74d9424] Bumped version number to 1.21 files changed, 1 insertions(+), 1 deletions(-)
&#160; &#160;&#160; &#160;创建一个新的分支并切换进来以后，我们修改版本号。这儿的 bump-version.sh 是一个虚构的脚本，被用来修改工作区中的一些文件以反映出新版本。(当然，这也可以是手动修改的)。而后，修改后的版本号被提交。

&#160; &#160;&#160; &#160;在发布最终完成之前，这个新的分支可能一直存在。在这期间，修复的bug会应用到该分支(而不是develop分支)。在这儿添加大的新特性是严格被禁止的。它们必须被合并到develop，并去等待下一个大的发行版本。

####完成发布分支

&#160; &#160;&#160; &#160;当release分支的状态已经符合正式发行的要求时，一些动作需要被实施。首先，release分支被合并进master(回想一下，按定义，master上的每一个提交都导致一个新的版本)。其次，master上的该次提交必须被打上标签，以方便以后引用这个历史版本。最后，release分支中的所有变更都需要合并回develop，使得未来版本也包含这些bug补丁。

在Git中，前两步操作：

	$ git checkout masterSwitched to branch 'master'$ git merge --no-ff release-1.2Merge made by recursive.(Summary of changes)$ git tag -a 1.2
release现在已经完成，且打上标签以备未来引用。

注: 你可能也想使用 -s 或 -u <key> 标记来私密签名。

&#160; &#160;&#160; &#160;要保持我们在release分支所做的变更，我们需要将其合并会develop。在Git中：

	$ git checkout developSwitched to branch 'develop'$ git merge --no-ff release-1.2Merge made by recursive.(Summary of changes)
这步操作可能引起合并冲突(很可能，因为我们已经改变了版本号)。如果是这样，修复并提交。

&#160; &#160;&#160; &#160;现在，我们真的完成了，且该release分支将被移除，因为我们不再需要它了：

	$ git branch -d release-1.2Deleted branch release-1.2 (was ff452fe).
快速修复分支(Hotfix branches)
可能起源于分支：master

必须合并回分支：develop 和 master

分支命名惯例：hotfix-*



&#160; &#160;&#160; &#160;Hotfix分支非常像release分支，因为它也是为新产品(虽然是计划外的)发布做准备的。它们起源于对当前已发布产品中不理想状态的立即响应的需求。当产品中出现一个紧急bug需要被立即解决时，一个hotfix分支可以从master分支中标记该产品版本的标签开始创建。

&#160; &#160;&#160; &#160;其实质是，当一个人在准备产品的快速修复时，团队其他成员在develop分支的工作能够继续进行。

###创建hotfix分支

&#160; &#160;&#160; &#160;Hotfix分支从master分支中创建。例如，假定1.2是当前已经发布的产品，却由于严重的bug导致了问题。但是develop分支中的变更尚不稳定。我们可能选择创建hotfix分支并开始修复这个问题：

	$ git checkout -b hotfix-1.2.1 masterSwitched to a new branch "hotfix-1.2.1"$ ./bump-version.sh 1.2.1Files modified successfully, version bumped to 1.2.1.$ git commit -a -m "Bumped version number to 1.2.1"[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.11 files changed, 1 insertions(+), 1 deletions(-)
创建分支以后不用忘记修改版本号！

然后，修复bug并将代码以一个或多个commit的形式提交。

	$ git commit -m "Fixed severe production problem"[hotfix-1.2.1 abbe5d6] Fixed severe production problem5 files changed, 32 insertions(+), 17 deletions(-)
结束hotfix分支

&#160; &#160;&#160; &#160;当完成时，对bug的修复需要合并进master分支，也需要合并回develop分支，以确保补丁代码也被包含在下一个版本中。这与release分支结束时的情况是完全类似的。

首先，更新master并对该次发布打上标签。

	$ git checkout masterSwitched to branch 'master'$ git merge --no-ff hotfix-1.2.1Merge made by recursive.(Summary of changes)$ git tag -a 1.2.1
注: 你可能也想使用 -s 或 -u <key> 标记来私密签名。

接着，使develop中也包含该修复代码：

	$ git checkout developSwitched to branch 'develop'$ git merge --no-ff hotfix-1.2.1Merge made by recursive.(Summary of changes)
&#160; &#160;&#160; &#160;此处规则的一个例外是，当release分支已经存在，hotfix的变更需要合并进这个release分支，而不是develop。合并进release分支的补丁代码最终也会在release分支完成时被合并进develop。(如果develop中的工作急需这个补丁且等不及release最终完成，你也可以安全地将补丁合并进develop。)

最后，移除这个临时分支：

	$ git branch -d hotfix-1.2.1Deleted branch hotfix-1.2.1 (was abbe5d6)

###git 常用指令
	git checkout -b develop master  基于master创建develop(如果当前在master可省略不写)
	# 切换到Master分支 
	git checkout master
	# 对Develop分支进行合并
	git merge --no-ff develop (使用--no-ff参数后，会执行正常合并，在Master分支上生成一个新节点。为了保证版本演进的清晰，我们希望采用这种做法)
	git checkout 查看本地分支
	git checkout -a 查看远程分支