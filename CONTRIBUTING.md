# 贡献指南

欢迎你提交任何修改的 PR！在提交 PR 之前，请参考以下注意事项和步骤。

* 请参阅[开发环境](https://github.com/livekit/client-sdk-android?tab=readme-ov-file#dev-environment) 说明来搭建本地开发环境。否则项目可能无法正常编译。

* 需要添加一个 changeset 文件来描述 PR 中包含的变更。

  在项目根目录下，执行以下命令：
  ```
  pnpm install
  pnpm changeset
  ```

  按照屏幕提示完成 changeset 文件的创建。

* 提交代码前请格式化：`./gradlew spotlessApply`。

* 在你首次提交 PR 时，CLA Assistant 机器人会提供一个链接来签署本项目的贡献者许可协议（CLA），这是将代码加入本仓库的必要条件。未签署 CLA 的贡献者所提交的 PR 无法被接受。
