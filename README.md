## 03.10 [Lerna with Monorepo]

https://zhuanlan.zhihu.com/p/35237759

https://juejin.cn/post/6844903985900421127

https://juejin.cn/post/6844903911095025678

### MultiRepo vs MonoRepo
Multirepo(multiple repository)是比较传统的做法，即每一个 package 都单独用一个仓库来进行管理。例如：Rollup

Monorepo (monolithic repository)：把所有相关的 package 都放在一个仓库里进行管理，每个 package 独立发布。例如：React, Angular, Babel, Jest, Umijs, Vue ...

![image](https://user-gold-cdn.xitu.io/2019/8/7/16c6ba2a8a6ce740?imageslim)

已知管理 monorepo 的工具有 bolt、lerna、rush

### Lerna
Lerna 是一个管理多个 npm 模块的工具，是 Babel 自己用来维护自己的 Monorepo 并开源出的一个项目。优化维护多包的工作流，解决多个包互相依赖，且发布需要手动维护多个包的问题。

 [lerna 指令](http://www.febeacon.com/lerna-docs-zh-cn/routes/commands/exec.html)
- lerna publish: 在当前项目中发布包
- lerna version: 更改自上次发布以来的包版本号
- lerna bootstrap: 将本地包链接在一起并安装剩余的包依赖项
    1. npm install每个包所有的外部依赖。
    2. 将所有相互依赖的 Lerna package符号链接在一起。
    3. 在所有引导包中运行npm run prepublish(除非传入了--ignore-prepublish)。
    4. 在所有引导包中运行npm run prepare
- lerna list: 列出本地包
- lerna changed: 列出自上次标记发布以来发生变化的本地包
- lerna diff: 自上次发布以来的所有包或单个包的区别
- lerna exec: 在每个包中执行任意命令
- lerna run: 在包含该脚本中的每个包中运行npm脚本
- lerna init: 创建一个新的Lerna仓库或将现有的仓库升级到Lerna的当前版本
- lerna add: 向匹配的包添加依赖关系
- lerna clean: 从所有包中删除node_modules目录
- lerna import: 将一个包导入到带有提交历史记录的monorepo中
- lerna link: 将所有相互依赖的包符号链接在一起
- lerna create: 创建一个新的由lerna管理的包
- lerna info: 打印本地环境信息


#### 过滤器配置项
- --scope: 指定包，可使用通配符 `lerna run --scope toolbar-* test`
- --ignore: 排除指定包 `lerna run --ignore package-@(1|2) --ignore package-3 lint`

#### 指令
##### 初始化
```
npm i -g lerna
lerna init  // --independent 设置为独立模式
```
会创建如下文件目录
```
lerna-repo/
    ┣━ packages/
    ┣━ lerna.json
    ┗━ package.json
```
Lerna 提供两种[模式](http://www.febeacon.com/lerna-docs-zh-cn/routes/basic/how_it_works.html)来管理项目:
- Fixed(默认)：固定模式。固定模式下 Lerna 项目在单一版本线上运行。版本号保存在项目根目录下 lerna.json 文件中的 version 下。当你运行 lerna publish 时，如果一个模块自上次发布版本以后有更新，则它将更新到你将要发布的新版本。这意味着你在需要发布新版本时只需发布一个统一的版本即可。
- Independent：独立模式。允许维护人员独立地的迭代各个包版本。每次发布时，你都会收到每个发生更改的包的提示，同时来指定它是 patch，minor，major 还是自定义类型的迭代。

![image](https://note.youdao.com/yws/public/resource/45f838ecd7f89458203b1eec28a865cf/xmlnote/6DF2EF641BB649E2940CABD51049D57C/13681)

##### 安装依赖
```
lerna bootstrap
```
当执行完上面的命令后，会发生以下的行为：

- 在各个模块中执行 npm install 安装所有依赖
- 将所有相互依赖的 Lerna 模块 链接在一起
- 在安装好依赖的所有模块中执行 npm run prepublish
- 在安装好依赖的所有模块中执行 npm run prepare

为packages文件夹下的package安装依赖
```
// 创建两个 pakages
lerna create @mo-demo/cli
lerna create @mo-demo/cli-shared-utils

lerna add chalk                                           // 为所有 package 增加 chalk 模块
lerna add semver --scope --dev @mo-demo/cli-shared-utils        // 为 @mo-demo/cli-shared-utils 增加 semver 模块
lerna add @mo-demo/cli-shared-utils --scope @mo-demo/cli  // 增加内部模块之间的依赖
```

> lerna 中使用`软链接(symlink)`来实现本地多项目之间的依赖，该功能在执行 bootstrap 时会自动在根目录的 node_modules 下创建本地package对应的软链接

对于多个 package 都依赖的包，会被多个 package 安装多次，如babel 等，会导致项目臃肿。可以使用 `--hoist` 来把每个 package 下的依赖包都提升到工程根目录，来降低安装以及管理的成本
```
lerna bootstrap --hoist 
```

yarn 本身提供了较 lerna 更好的依赖分析与 hoisting 的功能. [对比](https://yrq110.me/post/devops/how-lerna-manage-package-dependencies/)

#### [yarn workspace](https://classic.yarnpkg.com/en/docs/workspaces/)


```
// package.json
{
    "private": true,
    "workspaces": [
        "packages/*"
    ],
}

// lerna
{
    "npmClient": "yarn",
    "useWorkspaces": true,
}

// 添加公共依赖
yarn add @babel/core @babel/preset-env --dev -W
// 添加/移除单包依赖
lerna add react --scope @mo-demo/cli-shared-utils
yarn workspace @mo-demo/cli-shared-utils remove react --save
```
开启后lerna bootstrap命令会代理给 yarn 执行

**卸载**
```
lerna exec -- yarn remove semver # 将所有包下的 semver 卸载
```

**清理依赖包**
```
lerna clean
```

**变更检测**
```
lerna updated
# 或
lerna diff
```

**import**

导入一个已存在的模块，同时保留之前的提交记录，方便将其他正在维护的项目合并到一起。基本命令格式如下
```
lerna import <dir>  // dir 是本项目外的包含 npm 包的 git 仓库路径（相对于本项目根路径的相对路径）
```
执行后会将该模块整体复制到指定的依赖包存放路径下，同时会把该模块之前所有提交记录合并到当前项目提交记录中

**运行脚本**

lerna run 运行 npm script，可以指定具体的 package

```
lerna run test # 运行所有包的 test 命令
lerna run --parallel watch # 观看所有包并在更改时发报，流式处理前缀输出
```

**执行命令**

lerna exec 执行命令，用 -- 分割要执行的命令
```
lerna exec --scope @mo-demo/cli -- babel src -d dist    
```

**版本迭代**
```
lerna version 1.0.1 # 按照指定版本进行迭代
lerna version patch # 根据 semver 迭代版本号最后一位
lerna version       # 进入交互流程选择迭代类型 
```
> 如果你的 lerna 项目中各个模块版本不是按照同一个版本号维护（即创建时选择 independent 模式），那么会分别对各个包进行版本迭代

当执行此命令时，会发生如下行为：

1. 标记每一个从上次打过 tag 发布后产生更新的包
2. 提示选择此次迭代的新版本号
3. 修改 package.json 中的 version 值来反映此次更新
4. 提交记录此次更新并打 tag
5. 推送到远端仓库

**发布**

```
lerna publish   # 发布多个模块
```
当执行此命令时，会发生如下行为：

1. 发布自上次发布以来更新的包(在底层执行了 lerna version，2.x 版本遗留的行为)
2. 发布当前提交中打了 tag 的包
3. 发布在之前的提交中更新的未经版本化的 “canary” 版本的软件包（及其依赖项）

>  Lerna 不会发布在 package.json 中将 private 属性设置为 true 的模块，如果要发布带域的包，你还需要在 'package.json' 中设置如下内容：
```
"publishConfig": {
    "access": "public"
}
```

如果之前已执行过 lerna version 命令，这里如果直接执行 lerna publish 会提示没有发现有更新的包需要更新，我们可以通过从远端的 git 仓库来发布

```
lerna publish from-git
```


#### husky
`
yarn add husky@4.2.3 --dev -W
`
> `git init`时创建的默认hook 都是以`.sample` 结尾的, 防止它们被默认执行。5 版本的`husky` 不会自动在 .git/hooks 中创建对应的 hook，可以回退到 4 版本或者手动将 `.sample` 删除。 https://github.com/typicode/husky/issues/866

#### [git hooks](https://git-scm.com/docs/githooks)
可以定制化的脚本程序，所以它实现的功能与相应的git动作相关
