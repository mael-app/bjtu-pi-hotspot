# Contributing / 贡献指南

[English](#english) | [简体中文](#简体中文)

## English

Bug reports, ideas and pull requests are welcome.

### Report a problem

Open an issue with:

1. What you expected and what happened.
2. Your Pi model and OS version (`cat /etc/os-release`).
3. The relevant logs, e.g. `journalctl -u bjtu-login -n 50`. **Remove your username, password and IPs first.**

### Submit a change

1. Fork the repo and create a branch.
2. Make your change. Keep it small and focused on one thing.
3. Check it:
   ```
   ansible-playbook site.yml --syntax-check
   ansible-playbook site.yml --check --diff
   ```
   If you have a Pi: run the playbook twice. The second run must report `changed=0`.
   If you touched `roles/bjtu_login/files/bjtu-login`: run `shellcheck` on it.
4. Open a pull request that says what changed and how you tested it. CI runs the syntax check, `ansible-lint` and `shellcheck` on every pull request.

### Conventions

- Code, comments and commit messages in English.
- Settings belong in `group_vars/all.yml`, not hard-coded in roles.
- Use fully qualified module names (`ansible.builtin.copy`, not `copy`).
- Update **both** `README.md` and `README.zh-CN.md` when behaviour changes.
- Never commit `.env` or `authorized_keys` (both are gitignored).

## 简体中文

欢迎提交问题反馈、建议和 Pull Request。

### 反馈问题

提交 issue 时请包括：

1. 你预期的结果和实际发生的情况。
2. 树莓派型号和系统版本（`cat /etc/os-release`）。
3. 相关日志，例如 `journalctl -u bjtu-login -n 50`。**请先删除其中的账号、密码和 IP 地址。**

### 提交修改

1. Fork 本仓库并新建分支。
2. 进行修改。保持改动小而专注于一件事。
3. 检查：
   ```
   ansible-playbook site.yml --syntax-check
   ansible-playbook site.yml --check --diff
   ```
   如果你有树莓派：连续运行两次 playbook，第二次必须显示 `changed=0`。
   如果修改了 `roles/bjtu_login/files/bjtu-login`：请用 `shellcheck` 检查。
4. 提交 Pull Request，说明改了什么、如何测试的。CI 会对每个 Pull Request 运行语法检查、`ansible-lint` 和 `shellcheck`。

### 约定

- 代码、注释和提交信息使用英文。
- 配置项放在 `group_vars/all.yml`，不要在角色中写死。
- 使用完整模块名（`ansible.builtin.copy`，而不是 `copy`）。
- 行为有变化时，请**同时**更新 `README.md` 和 `README.zh-CN.md`。
- 切勿提交 `.env` 或 `authorized_keys`（两者都已被 git 忽略）。
