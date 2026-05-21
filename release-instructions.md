# 发布流程

1. 在项目根目录的 `gradle.properties` 文件中，更新 `VERSION_NAME` 属性。通常只需要去掉 `-SNAPSHOT` 后缀即可。
1. 提交并推送变更。
1. 打标签：`git tag v[VERSION_NAME]`。注意标签前缀必须带 `v`，这会告诉 CI 这是一个发布标签。
1. 推送标签：`git push --tags`
1. 确认 `publish` GitHub Action 执行成功。
1. 在 GitHub 上创建一个新的 Release 并填写更新日志。
1. 准备下一版本的开发：向上调整 `VERSION_NAME` 并加回 `-SNAPSHOT` 后缀。
1. 同时更新 `README.md` 中的版本号。
