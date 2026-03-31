# cc

该仓库维护 Clash 规则文本，并通过 GitHub Actions 自动构建 `geosite.dat`。

## 使用方式

- 工作流：`.github/workflows/build-geosite.yml`
- 触发条件：`.txt` 规则文件变更或手动触发
- 生成产物：`geosite.dat`

在 Clash / Clash.Meta 规则中可直接使用：

```yaml
- GEOSITE,cc,Proxy
```

或在支持 geosite 表达式的场景使用：

```text
geosite:cc
```
