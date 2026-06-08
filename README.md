# office-adobe-user-provision

统一开通 / 管理 **Microsoft 365(世纪互联版)** 与 **Adobe Creative Cloud** 用户的自包含 **Agent Skill**:创建、授权、重置密码、删除、查询、批量、自检。CLI / HTTP API 共用同一套 provider 实现。

> 本仓库是一个 **Claude Code / Agent Skill**。任何智能体只要把它 clone 到自己的 skills 目录并填好 `.env`,即可学会这套开户能力。
> **本仓库不含任何真实凭据**——只有 `.env.example` 模板。`.env` 已被 `.gitignore` 永久排除。

---

## 一行安装(智能体 / 人都适用)

把仓库直接 clone 进 Claude Code 的 skills 目录,目录名即 skill 名:

```bash
git clone https://github.com/eggyrooch-blip/office-adobe-user-provision.git \
  ~/.claude/skills/office-adobe-user-provision
```

> 项目级安装把目标换成 `<repo>/.claude/skills/office-adobe-user-provision` 即可。
> 装好后**重启 Claude Code 会话**,skill 会在启动时被扫描加载,触发词见文末。

## 配置(填凭据)

```bash
cd ~/.claude/skills/office-adobe-user-provision
cp .env.example .env          # 然后按注释填入你自己的 Entra App / Adobe UMAPI / SMTP 凭据
pip install -r requirements.txt   # 需 Python 3.9+
```

所需凭据(自备,见 `.env.example` 模板):
- **Office 365**:Entra App `CLIENT_ID / TENANT_ID / CLIENT_SECRET`(需管理员同意 `User.ReadWrite.All` 等),`DEFAULT_DOMAIN`
- **Adobe**:UMAPI `ADOBE_CLIENT_ID / ADOBE_CLIENT_SECRET / ADOBE_ORG_ID`
- **可选**:SMTP 通知邮件配置

## 用法

```bash
./oup office365 init           # 验证 O365 凭据 + 拉产品
./oup adobe init               # 验证 Adobe 凭据 + 拉产品
./oup office365 create <ldap> --display-name "<姓名>"
./oup adobe create <ldap>@<domain> --product cc      # cc=全家桶 / ps / acrobat
./oup <provider> inspect <id> --json                 # 查询
```

`oup` 是自带的 CLI wrapper(自动定位目录、加载 `.env`,可从任意路径调用)。等价于 `python3 main.py ...`。
**别用裸 `python`**:很多机器上它指向 Python 2,会 SyntaxError。

详细步骤、安全红线、Red Flags 见 [`SKILL.md`](./SKILL.md)。

## 安全

- `.env` **永不入库**(`.gitignore` 已排除)。clone 后填的凭据只留在本地。
- 提交前请确认 `git status` 不含 `.env`。
- 凭据轮换只改 `.env`,代码不动(`config.py` 是唯一读取处)。

## 触发词(智能体自动识别)

`给 XX 开账号` · `新建用户` · `入职开账号` · `开 Office` · `开 Adobe` · `全家桶 / All Apps` · `重置密码` · `删除用户` · `批量开户` · `查用户`

## 架构

入口(`main.py` CLI / `app/api/server.py` Flask)→ `app/services/user_service.py`(统一业务门面)→ `app/providers/{office365,adobe}/`(对接外部 API)。扩展功能先加到 `user_service.py` 再让 CLI/API 暴露。
