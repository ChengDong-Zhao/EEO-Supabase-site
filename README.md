# 法律案件线索收集系统 - 部署指南

## 一、获取 Supabase 参数

登录 [Supabase Dashboard](https://supabase.com/dashboard)，进入你的项目，然后：

| 参数 | 获取路径 | 示例 |
|------|----------|------|
| **SUPABASE_URL** | Settings → API → Project URL | `https://xxxxx.supabase.co` |
| **SUPABASE_ANON_KEY** | Settings → API → Project API keys → anon public | `eyJhbGci...` |
| **SUPABASE_SERVICE_KEY** | Settings → API → Project API keys → service_role → Reveal | `eyJhbGci...` |

---

## 二、初始化数据库

1. 打开 Supabase Dashboard，进入你的项目
2. 左侧菜单点击 **SQL Editor**
3. 将 `setup.sql` 的全部内容粘贴进去
4. 点击 **Run** 执行

### 配置 Storage（需手动操作）

SQL 无法直接创建 Storage Bucket，请在 Dashboard 中手动配置：

1. 左侧菜单点击 **Storage**
2. 点击 **New Bucket**，创建名为 `evidence-files` 的存储桶
3. **Public bucket** 设为 **关闭**
4. 点击创建

### 配置 Storage 策略

在 `evidence-files` bucket 中，点击 **Policies** 标签页，依次添加以下 3 个策略：

**策略 1：允许任何人上传文件**
- 点击 **New Policy** → **For full customization**
- Policy name: `允许上传证据文件`
- Allowed operation: `INSERT`
- Target roles: `anon`
- Policy definition: `(bucket_id = 'evidence-files')`

**策略 2：仅管理员可读取文件**
- 点击 **New Policy** → **For full customization**
- Policy name: `仅管理员可读取文件`
- Allowed operation: `SELECT`
- Target roles: `authenticated`
- Policy definition: `(bucket_id = 'evidence-files')`

**策略 3：仅管理员可删除文件**
- 点击 **New Policy** → **For full customization**
- Policy name: `仅管理员可删除文件`
- Allowed operation: `DELETE`
- Target roles: `authenticated`
- Policy definition: `(bucket_id = 'evidence-files')`

---

## 三、配置前端页面

打开 `index.html`，找到顶部配置区域，替换两个参数：

```javascript
const SUPABASE_URL = '你的_Supabase_URL';
const SUPABASE_ANON_KEY = '你的_anon_key';
```

> **注意**：`index.html` 只使用 anon key，绝不包含 service_role key。

`admin.html` 无需预填参数，登录时在页面中输入即可。

---

## 四、本地预览

直接双击 `index.html` 即可在浏览器中打开。或使用本地服务器：

```bash
# Python 3
python -m http.server 8080

# Node.js
npx serve .
```

然后访问 `http://localhost:8080`

---

## 五、部署上线

将 `index.html` 和 `admin.html` 上传到任意静态网站托管服务即可：

| 平台 | 免费额度 | 说明 |
|------|----------|------|
| [Vercel](https://vercel.com) | 免费 unlimited | 拖拽上传文件夹即可 |
| [Netlify](https://netlify.com) | 100GB/月 | 同上 |
| [GitHub Pages](https://pages.github.com) | 免费 | 需要仓库 |
| Supabase Hosting | 免费 | 直接在 Supabase 中托管 |

**安全提醒**：建议将 `admin.html` 部署在不公开的 URL 路径下，或单独部署，不要公开分享管理端链接。

---

## 六、自检清单

### Supabase 参数获取

- [ ] 已获取 Project URL
- [ ] 已获取 anon public key
- [ ] 已获取 service_role key

### 数据库配置

- [ ] 已执行 `setup.sql`（表、RLS 策略已创建）
- [ ] 已创建 `evidence-files` Storage bucket
- [ ] 已添加 3 个 Storage 策略（INSERT/SELECT/DELETE）

### 前端配置

- [ ] `index.html` 中已替换 SUPABASE_URL
- [ ] `index.html` 中已替换 SUPABASE_ANON_KEY
- [ ] 确认 `index.html` 中 **没有** service_role key

### 功能测试

- [ ] 填写表单并提交，能成功提交线索
- [ ] 上传不同格式文件（图片、PDF、音频、视频），均成功
- [ ] 上传超过 10MB 文件，被前端拦截
- [ ] 上传不允许的格式，被前端拦截
- [ ] 管理端使用 service_role key 能登录
- [ ] 管理端能看到所有提交记录
- [ ] 管理端能查看、下载已上传的文件
- [ ] 管理端能导出 CSV
- [ ] 管理端能删除记录

---

## 七、Anon Key 说明与安全限制

### 什么是 Anon Key？

Anon Key 是 Supabase 的**匿名公共密钥**，设计上就是可以安全地暴露在前端代码中的。它不是密钥，而是一个**身份标识符**。

### Anon Key 的安全限制

1. **受 RLS 策略保护**：所有数据库操作都必须通过 RLS 策略审核。在本系统中，anon key 只被授予 INSERT 权限，意味着用户：
   - 可以提交线索数据
   - **不能**读取他人提交的数据
   - **不能**修改或删除任何数据

2. **Storage 操作也受策略保护**：anon key 只能上传文件到 Storage，不能读取或删除文件。

3. **无管理员权限**：anon key 无法绕过任何安全限制。即使有人拿到了 anon key，也只能向你的系统提交数据。

### Service Role Key 的安全

- Service Role Key 拥有**完全的数据库访问权限**，可以绕过所有 RLS 策略
- 因此它 **绝不能** 出现在前端代码中
- 本系统中，它仅在管理端（`admin.html`）由管理员手动输入
- 管理端使用 `localStorage` 保存，关闭浏览器后仍需手动清除（退出登录时会清除）
- **不要将 `admin.html` 的 URL 分享给非管理员**
