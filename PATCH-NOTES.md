# 公司兼容补丁 1.2.5-patch1

基于 aliyun/credentials-php 官方 1.2.5，提交 47f180664b1f603e4141d4d7df3777041c031dfa。
包名仍为 alibabacloud/credentials。

## 修改内容

- DEBUG 未设置或值为 null 时，先转成字符串再调用 strtolower，避免 PHP 8.4 的弃用提示被 Hyperf 转成启动异常。
- MockTrait 中两个可空参数保留默认值和 PHPDoc 类型，去掉隐式可空类型声明；继续保持官方声明的 PHP 5.6 语法兼容。实际构造 Guzzle RequestException 时仍由 Guzzle 校验参数类型。
- 补充 12 组 DEBUG 环境值回归测试，不修改凭证来源、签名和超时设置。

## 本地验证

- PHP 8.4.27：请求配置 13 项、ECS RAM role 模拟测试 9 项、MockTrait 测试 1 项，全部通过。
- PHP 7.3.33：请求配置 13 项通过；未验证旧 PHP 的完整依赖组合。
- 同一组新增测试在官方 1.2.5 上有 3 项报错，均为 strtolower(null)。
- ECS 测试使用模拟响应，没有访问真实云元数据服务或 Nacos。部署环境仍需验证真实凭证获取及 Nacos 配置加载。

## 业务项目接入

先将修复分支和 1.2.5-patch1 标签发布到公司 GitHub fork，再在业务项目根 composer.json 的 repositories 中添加下列条目（保留原有其他仓库配置）：

```json
{
  "type": "vcs",
  "url": "https://github.com/shangminet-dev/credentials-php"
}
```

将根项目 require 中 alibabacloud/credentials 约束设为精确版本 1.2.5-patch1。使用目标 PHP 8.4.27 环境执行：

```sh
composer update alibabacloud/credentials --with-dependencies
composer check-platform-reqs --no-dev
composer show alibabacloud/credentials
```

检查 composer.lock 中该包的 source.url、source.reference 和版本，确认来自公司 fork 的补丁提交，再提交 composer.json 和 composer.lock。构建继续使用 composer install，不添加 --ignore-platform-reqs。

补丁版本满足 x7sy/rpc-bundle 的 ^1.2 约束。先在 x7market 开发环境验证 worker 启动、真实 ECS 凭证获取和 Nacos 配置加载，再安排其他使用方更新。此补丁不要求修改 Satis 中的 x7sy/* 包版本。
