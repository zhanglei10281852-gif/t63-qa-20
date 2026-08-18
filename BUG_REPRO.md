# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

处理器 panic 后响应状态是 200，客户端把错误当成成功结果。请修复 panic 恢复路径，返回统一的 500 JSON 错误。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/t63-qa-20
- 仓库地址：https://github.com/zhanglei10281852-gif/t63-qa-20.git
- parent SHA：949b8eeba86e021b2712f26a3799a7679588ef42

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/t63-qa-20.git bug-repro
cd bug-repro
git checkout --detach 949b8eeba86e021b2712f26a3799a7679588ef42
go test ./internal/middleware -run TestRecoverConvertsPanicToServerError -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/middleware -run TestRecoverConvertsPanicToServerError -count=1
--- FAIL: TestRecoverConvertsPanicToServerError (0.00s)
    middleware_test.go:101: panic status=200
FAIL
FAIL	sanitation-operations/internal/middleware	0.006s
FAIL

```

stderr：

```text
warning: internal/middleware/middleware_test.go has type 100755, expected 100644
warning: internal/middleware/middleware_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/middleware -run TestRecoverConvertsPanicToServerError -count=1
--- FAIL: TestRecoverConvertsPanicToServerError (0.03s)
    middleware_test.go:101: panic status=200
FAIL
FAIL	sanitation-operations/internal/middleware	0.233s
FAIL

```

stderr：

```text
warning: internal/middleware/middleware_test.go has type 100755, expected 100644
warning: internal/middleware/middleware_test.go has type 100755, expected 100644

```

## 通过条件

在触发条件下，定向测试 TestRecoverConvertsPanicToServerError 应通过，相关包、全量测试、竞态测试和构建检查均通过；回退 gold 唯一修复后定向测试重新失败。
