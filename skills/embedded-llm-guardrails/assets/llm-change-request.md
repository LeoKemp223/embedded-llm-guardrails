# LLM 单次修改请求模板

## 需求澄清

```text
需求理解：

最小可验证目标：

暂不实现的扩展功能：

需要用户确认的问题：

最小验收标准：
```

如果需求复杂或模糊，用户确认前不要生成业务代码。

## 修改边界

```text
是否已阅读：
- LLM_RULES.md: yes/no
- LLM_BOUNDARY.md: yes/no

Allowed Files:
- 

Forbidden Files / Areas:
- 

Protected Verified Code:
- 

Need User Confirmation Before Touching:
- 
```

未填写 `Allowed Files` 时，不允许修改代码。

## 最小验证

```text
TDD/验证优先：
- 先写什么测试、验证入口或上板验证步骤：

Build:
- 

Host Test:
- 

Board Test:
- 

Expected Result:
- 
```

## 特殊场景

```text
涉及外设驱动：yes/no
如果 yes，先输出开发流程、已知信息、缺失信息、待确认问题，用户确认后再生成代码。

涉及 main / 主 while：yes/no
如果 yes，只允许添加带注释的函数调用；不删除已有调用；临时屏蔽必须备注原因、日期和恢复条件。

是否需要临时调试代码：yes/no
如果 yes，放置位置：
```

## 输出要求

修改前说明：

```text
计划修改文件：
不会修改的关键区域：
最小实现范围：
验证方法：
风险点：
```

修改后说明：

```text
修改文件：
实现内容：
未实现内容：
验证方法：
是否只完成最小可验证功能：
下一步扩展前需要用户确认：
硬件风险：
```
