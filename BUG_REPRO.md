# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

推理运行开始后自身状态已变为运行中，算力池也已占用，但关联数据快照仍停留在已预留状态。请修复跨实体状态传播，保证启动操作原子地推进所有关联对象。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-featuremesh-task-05
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-featuremesh-task-05.git
- parent SHA：1036abcae1d23334ccea5d9d6dc3ae2d79b0a2bf

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-featuremesh-task-05.git bug-repro
cd bug-repro
git checkout --detach 1036abcae1d23334ccea5d9d6dc3ae2d79b0a2bf
go test ./internal/service -run "^TestInferenceLifecycleMovesSnapshotsAndComputePool$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestInferenceLifecycleMovesSnapshotsAndComputePool$" -count=1
--- FAIL: TestInferenceLifecycleMovesSnapshotsAndComputePool (0.54s)
    service_test.go:189: in execution batch = {ID:snapshot_752af56bede2fea5e21a87e1 WorkspaceID:workspace_5a28f68c9298f294399d9e8f SourceZoneID:data_zone_c97b3530a576d2490cf244c0 SourceRevision:EXT-1 SchemaFamily:plasma PartitionCount:2 EstimatedRows:100 State:reserved ExpiresAt:2026-08-20 08:00:00 +0000 UTC InferenceRunID:run_2afc35164d0cc78e0b43c807 QuarantineNote: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:4}
FAIL
FAIL	github.com/zhanglei10281852-gif/ai-featuremesh-base/internal/service	0.553s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestInferenceLifecycleMovesSnapshotsAndComputePool$" -count=1
--- FAIL: TestInferenceLifecycleMovesSnapshotsAndComputePool (1.21s)
    service_test.go:189: in execution batch = {ID:snapshot_fb03f73b7286d0bb5e02fbb1 WorkspaceID:workspace_81df270beb4ce25bdb4062e1 SourceZoneID:data_zone_f970c55d91de9107042e4de1 SourceRevision:EXT-1 SchemaFamily:plasma PartitionCount:2 EstimatedRows:100 State:reserved ExpiresAt:2026-08-20 08:00:00 +0000 UTC InferenceRunID:run_319ba38711bad149880c312b QuarantineNote: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:4}
FAIL
FAIL	github.com/zhanglei10281852-gif/ai-featuremesh-base/internal/service	1.402s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令必须由修复前失败变为修复后通过；相关包、go test ./... -count=1、go vet ./... 和 linux/amd64 构建必须通过；回退 gold 关键修改后定向命令重新失败。
