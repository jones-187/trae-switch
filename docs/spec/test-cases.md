# 局域网监听功能测试用例

## 1. 测试目标

验证局域网监听功能的正确性、稳定性和安全性，确保功能符合需求规范。

## 2. 测试环境

| 环境 | 描述 |
|------|------|
| 操作系统 | Windows 10/11, macOS, Linux |
| 网络环境 | 局域网环境（至少2台设备） |
| 浏览器 | Chrome, Firefox, Safari |
| 权限 | 管理员权限 |

## 3. 测试用例

### 3.1 功能测试

| 测试用例ID | 测试用例名称 | 测试步骤 | 预期结果 |
|-----------|-------------|---------|----------|
| F001 | 默认状态测试 | 1. 启动应用<br>2. 检查"监听局域网"开关状态 | 开关默认处于关闭状态 |
| F002 | 开启局域网监听 | 1. 点击"监听局域网"开关<br>2. 检查开关状态<br>3. 检查代理服务器监听地址 | 1. 开关变为开启状态<br>2. 代理服务器监听地址变为0.0.0.0 |
| F003 | 关闭局域网监听 | 1. 确保"监听局域网"开关处于开启状态<br>2. 点击开关<br>3. 检查开关状态<br>4. 检查代理服务器监听地址 | 1. 开关变为关闭状态<br>2. 代理服务器监听地址变为127.0.0.1 |
| F004 | 代理未运行时设置 | 1. 确保代理未运行<br>2. 点击"监听局域网"开关<br>3. 启动代理 | 1. 开关状态正常切换<br>2. 代理启动后使用正确的监听地址 |
| F005 | 代理已运行时设置 | 1. 确保代理已运行<br>2. 点击"监听局域网"开关<br>3. 检查代理状态 | 1. 代理自动重启<br>2. 代理使用正确的监听地址 |
| F006 | 局域网访问测试 | 1. 开启局域网监听<br>2. 从局域网内其他设备访问代理服务 | 其他设备能够成功访问代理服务 |

### 3.2 异常测试

| 测试用例ID | 测试用例名称 | 测试步骤 | 预期结果 |
|-----------|-------------|---------|----------|
| E001 | 权限不足测试 | 1. 以普通用户权限运行应用<br>2. 开启局域网监听 | 1. 显示权限不足提示<br>2. 功能无法使用 |
| E002 | 端口冲突测试 | 1. 确保443端口被占用<br>2. 开启局域网监听 | 1. 显示端口冲突提示<br>2. 功能无法使用 |
| E003 | 网络异常测试 | 1. 开启局域网监听<br>2. 断开网络连接<br>3. 检查应用状态 | 1. 应用能够正常运行<br>2. 显示网络异常提示 |

### 3.3 边界测试

| 测试用例ID | 测试用例名称 | 测试步骤 | 预期结果 |
|-----------|-------------|---------|----------|
| B001 | 快速切换测试 | 1. 快速多次点击"监听局域网"开关 | 1. 应用能够正常响应<br>2. 最终状态正确 |
| B002 | 多设备并发访问测试 | 1. 开启局域网监听<br>2. 多台设备同时访问代理服务 | 1. 代理服务能够正常处理并发请求<br>2. 响应时间在合理范围内 |
| B003 | 长时间运行测试 | 1. 开启局域网监听<br>2. 持续运行24小时 | 1. 应用稳定运行<br>2. 无内存泄漏 |

### 3.4 回归测试

| 测试用例ID | 测试用例名称 | 测试步骤 | 预期结果 |
|-----------|-------------|---------|----------|
| R001 | 现有功能不受影响 | 1. 开启/关闭局域网监听<br>2. 测试其他功能（如服务商切换、代理启动/停止） | 其他功能正常工作，不受局域网监听功能影响 |
| R002 | 配置持久化测试 | 1. 开启局域网监听<br>2. 重启应用<br>3. 检查开关状态 | 开关状态保持为开启 |

## 4. 测试脚本

### 4.1 后端测试脚本

```go
// 测试 SetLanMode 方法
func TestSetLanMode(t *testing.T) {
    app := NewApp()
    app.startup(context.Background())
    
    // 测试默认状态
    if app.IsLanMode() != false {
        t.Errorf("默认 lanMode 应为 false，实际为 %v", app.IsLanMode())
    }
    
    // 测试开启
    err := app.SetLanMode(true)
    if err != nil {
        t.Errorf("开启 lanMode 失败: %v", err)
    }
    if app.IsLanMode() != true {
        t.Errorf("开启后 lanMode 应为 true，实际为 %v", app.IsLanMode())
    }
    
    // 测试关闭
    err = app.SetLanMode(false)
    if err != nil {
        t.Errorf("关闭 lanMode 失败: %v", err)
    }
    if app.IsLanMode() != false {
        t.Errorf("关闭后 lanMode 应为 false，实际为 %v", app.IsLanMode())
    }
}

// 测试 GetStatus 方法
func TestGetStatus(t *testing.T) {
    app := NewApp()
    app.startup(context.Background())
    
    status := app.GetStatus()
    if status["lanMode"] == nil {
        t.Errorf("GetStatus 返回值应包含 lanMode 字段")
    }
    if status["lanMode"] != false {
        t.Errorf("默认 lanMode 应为 false，实际为 %v", status["lanMode"])
    }
    
    // 测试开启后状态
    app.SetLanMode(true)
    status = app.GetStatus()
    if status["lanMode"] != true {
        t.Errorf("开启后 lanMode 应为 true，实际为 %v", status["lanMode"])
    }
}
```

### 4.2 前端测试脚本

```javascript
// 测试开关状态
async function testLanModeToggle() {
    // 打开应用
    await openApp();
    
    // 检查默认状态
    const switchElement = document.querySelector('.lan-mode-switch');
    if (switchElement.checked) {
        throw new Error('默认状态应为关闭');
    }
    
    // 点击开启
    switchElement.click();
    await sleep(1000);
    if (!switchElement.checked) {
        throw new Error('开启后开关应为打开状态');
    }
    
    // 点击关闭
    switchElement.click();
    await sleep(1000);
    if (switchElement.checked) {
        throw new Error('关闭后开关应为关闭状态');
    }
}

// 测试局域网访问
async function testLanAccess() {
    // 开启局域网监听
    const switchElement = document.querySelector('.lan-mode-switch');
    if (!switchElement.checked) {
        switchElement.click();
        await sleep(1000);
    }
    
    // 获取本地IP地址
    const localIP = await getLocalIP();
    
    // 从另一设备访问
    const response = await fetch(`https://${localIP}:443/v1/models`);
    if (!response.ok) {
        throw new Error('局域网访问失败');
    }
    
    const data = await response.json();
    if (!data.object || data.object !== 'list') {
        throw new Error('响应格式不正确');
    }
}
```

## 5. 测试工具

| 工具名称 | 用途 |
|---------|------|
| Go Test | 后端单元测试 |
| Selenium | 前端UI测试 |
| curl | HTTP请求测试 |
| Wireshark | 网络流量分析 |

## 6. 测试验收标准

| 标准 | 描述 |
|------|------|
| 功能完整性 | 所有功能测试用例通过 |
| 异常处理 | 所有异常测试用例通过 |
| 边界情况 | 所有边界测试用例通过 |
| 回归测试 | 所有回归测试用例通过 |
| 性能指标 | 响应时间 < 100ms，内存使用稳定 |
| 安全性 | 无安全漏洞 |

## 7. 测试报告模板

### 测试执行报告

| 测试用例ID | 测试用例名称 | 执行结果 | 执行时间 | 错误信息 |
|-----------|-------------|----------|----------|----------|
| F001 | 默认状态测试 | PASS | 2026-04-09 10:00 | - |
| F002 | 开启局域网监听 | PASS | 2026-04-09 10:05 | - |
| F003 | 关闭局域网监听 | PASS | 2026-04-09 10:10 | - |
| F004 | 代理未运行时设置 | PASS | 2026-04-09 10:15 | - |
| F005 | 代理已运行时设置 | PASS | 2026-04-09 10:20 | - |
| F006 | 局域网访问测试 | PASS | 2026-04-09 10:25 | - |
| E001 | 权限不足测试 | PASS | 2026-04-09 10:30 | - |
| E002 | 端口冲突测试 | PASS | 2026-04-09 10:35 | - |
| E003 | 网络异常测试 | PASS | 2026-04-09 10:40 | - |
| B001 | 快速切换测试 | PASS | 2026-04-09 10:45 | - |
| B002 | 多设备并发访问测试 | PASS | 2026-04-09 10:50 | - |
| B003 | 长时间运行测试 | PASS | 2026-04-10 10:50 | - |
| R001 | 现有功能不受影响 | PASS | 2026-04-09 11:00 | - |
| R002 | 配置持久化测试 | PASS | 2026-04-09 11:05 | - |

### 测试总结

- 测试用例总数：14
- 通过数：14
- 失败数：0
- 成功率：100%

**结论**：局域网监听功能测试通过，符合需求规范。