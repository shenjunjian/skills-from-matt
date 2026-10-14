---
name: implement
description: "依据 spec 或一组工单实现某项工作。"
disable-model-invocation: true
---

实现用户在 spec 或工单里描述的工作。

如果用户给出了工单引用，从工单系统把它取来，开始前先说明它的标题。引用含义不明确就问。

在可行的地方，在事先约好的 seam 上调用 Skill 工具，用 "tdd"。

勤跑类型检查，勤跑单个测试文件，最后在完整测试套件上跑一次。

完成后，调用 Skill 工具，用 "code-review" 评审这份工作。

把成果提交到当前分支。
