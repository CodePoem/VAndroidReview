# Git

## Git使用

## Git规范

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
