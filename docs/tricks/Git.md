# Git

## Git 使用

## Git 规范

常见 Git 规范是 AngularJS 标准 的 Git 规范。

### 公式

```公式
<type>(scope): <subject>
```

#### type

1. feat：新功能(feature)
2. fix：修补bug
3. docs：文档（documentation）
4. format：格式（不影响代码运行的变动）
5. refactor：重构（即不是新增功能，也不是修改bug）
6. test：增加测试
7. chore：构建过程或辅助工具的变动

```my format
:tada: init: initial commit

:sparkles: feature: add feature
:bug: fix: fix bug
:memo: docs: update docs
:art: format: format code
:recycle: refactor: refactor code
:white_check_mark: test: update tests
:elephant: dep: update deps
```

#### scope

用于说明commit影响的范围，比如数据层、控制层、视图层等等。

#### subject

commit目的的简短描述，不超过50个字，以动词开头。

## Git Commit 工具

### [Commitizen](https://github.com/commitizen/cz-cli)

Commitizen 是一个自动生成格式化 commit messsage 的工具，安装后只需要运行 git cz 就能够自动根据你的选择帮你生成整洁美观的 commit messsage。

Commitizen 通常还可以配合 [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog) 来根据格式化的 commit messsage 自动生成 CHANGELOG.md 文件。

安装后可使用以下命令自动生成 CHANGELOG.md 文件：

```shell
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
```

-p 用来指定使用的 commit message 标准
-i 用来指定写入 changelog 的文件
-s 表示读写 changelog 为同一文件
-r 表示生成 changelog 所需要使用的 release 版本数量，默认为1（基于上次 tag 版本之后的变更），全部则是0（生成之前所有 commit 变更）

### git message emoji

### [cz-emoji](https://github.com/ngryman/cz-emoji)

cz-emoji 可直接生成 emoji 格式的 commit message

### [nielsgl/conventional-changelog-emoji](https://github.com/nielsgl/conventional-changelog-emoji)

cz-emoji 生成的 emoji 格式的 commit message 用emoji取代了 \<type\>，无法用 conventional-changelog-cli 生成 changelog , 因此需要使用 cz-customizable 来进行自定义。

可使用以下命令进行自定义：

```shell
npm install -g commitizen conventional-changelog conventional-changelog-cli cz-customizable
echo '{ "path": "cz-customizable" }' > ~/.czrc
wget https://raw.githubusercontent.com/nielsgl/conventional-changelog-emoji/master/.cz-config.js -O ~/.cz-config.js
```
