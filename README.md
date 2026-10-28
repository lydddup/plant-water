# 养花浇水提醒

一个纯本地的浇水记录 + 看板工具。记录家里每盆花什么时候浇过水，打开就知道谁该浇了。

默认**没有服务器、没有账号**。数据只保存在浏览器里；开启云同步后，数据会存到你自己 GitHub 账号下的**私密 Gist**，实现手机与电脑自动同步。

## 功能

| | 功能 | 说明 |
|---|---|---|
| 1 | 添加 / 编辑 / 删除植物 | 名称、位置、浇水间隔、备注 |
| 2 | 记录浇水 | 点「浇了」记今天；展开后可「补记」任意日期 |
| 3 | 状态计算 | 自动算出下次浇水日、还剩几天 |
| 4 | 状态分级 | 🔴 逾期 / 🟠 今天 / 🟡 快该浇了 / 🟢 安全 |
| 5 | 紧急度排序 | 逾期最久的永远排最前 |
| 6 | 今日概览 | 共几盆 · 今天该浇 · 已逾期 |
| 7 | 本地持久化 | 数据存在浏览器 localStorage |
| 8 | 备份导出 / 导入 | JSON 文件，换设备或恢复用 |
| 9 | 位置筛选 | 顶部 chips 按阳台 / 客厅 / 书房筛选 |
| 10 | 季节系数 | 夏季间隔 ×0.7、冬季 ×1.3，可在设置里开关 |
| 11 | 浇水历史 | 点卡片展开，看最近 8 次浇水日期与实际间隔 |
| 12 | 撤销误操作 | 手滑点错可撤销（Toast 内 5 秒，或设置里点「撤销上一步」） |
| 13 | **云同步** | 通过 GitHub 私密 Gist 在多台设备间自动同步 |

## 本地直接用

双击 `index.html` 即可，无需安装任何东西。

## 开启云同步（手机 ↔ 电脑）

### 1. 生成 Token

打开 https://github.com/settings/tokens → **Generate new token (classic)**

- 勾选 **`gist`** 一项即可（不要勾别的）
- 过期时间建议选 `No expiration`
- 点 Generate，**复制生成的字符串**（关掉页面后就再也看不到了）

### 2. 在应用里开启

打开页面 → 设置与备份 → 云同步 → 把 Token 粘贴进输入框 → 点「开启同步」

应用会自动创建一个**私密 Gist**（只有你能访问），之后：

- 每次改动后约 1 秒自动同步
- 打开页面时自动拉取最新数据
- 从后台切回页面时也会同步一次

### 3. 让第二台设备接入同一个 Gist

第二台设备需要拿到同一个 Gist 的 ID：

- 浏览器打开 https://gist.github.com/ → 找到名为「养花浇水提醒 · 数据同步」的私密 Gist
- 地址栏最后的那一串字符就是 Gist ID

然后在该设备的浏览器控制台（F12 → Console）执行：

```js
localStorage.setItem('flower-water-reminder:sync', JSON.stringify({
  token: '你的Token',
  gistId: '你的GistID',
  lastSyncAt: ''
}))
```

刷新页面即可同步。

> ⚠️ 两台设备各点一次「开启同步」是**不行**的，那样会各建一个 Gist。必须让第二台设备接入已有的 Gist。

## 同步的合并规则

两台设备都能改数据，冲突时按以下规则处理：

| 情况 | 规则 |
|---|---|
| 两边都记了浇水 | **取并集**，不会丢 |
| 名称 / 间隔 / 备注等不同 | 以 `updatedAt` **较新的一方**为准 |
| 一边删了某盆花 | 用**墓碑机制**记录删除，不会从另一台设备「复活」 |

## 关于安全

- Token 只保存在你自己的浏览器 localStorage 里（键名 `flower-water-reminder:sync`）
- Token **不会**被写进导出的备份文件，可以放心把备份发给别人
- Gist 是 **secret** 类型，不出现在公开列表；但拿到 URL 的人可以访问，别外传
- Token 只勾选了 `gist` 权限，不能读写你的代码仓库

如果怀疑 Token 泄露：https://github.com/settings/tokens → 删掉即可，然后重新生成一个。

## 部署到 GitHub Pages（手机上使用）

想在安卓手机上「添加到主屏幕」当 App 用，需要一个 https 地址。

1. 在 GitHub 新建一个仓库（例如 `flower-water-reminder`）

   > ⚠️ **免费账户必须选 Public（公开）**，私有仓库在 GitHub Free 下无法开启 Pages。
   > 仓库里只有页面代码，不含任何你的浇水数据，公开没有隐私风险。
   > 如果你只在电脑上双击 `index.html` 使用，那就不需要 Pages，私有仓库也完全可以。

2. 把本目录下的 **4 个文件**全部上传到仓库根目录：
   - `index.html`
   - `manifest.json`
   - `icon.svg`
   - `README.md`
3. 打开仓库 `Settings → Pages`
4. `Build and deployment → Source` 选 **Deploy from a branch**
5. `Branch` 选 `main`，目录选 `/ (root)`，点 Save
6. 等 1~2 分钟，访问 `https://<你的用户名>.github.io/flower-water-reminder/`

**安卓添加到主屏幕**：用 Chrome 打开上面的网址 → 右上角菜单 → 「添加到主屏幕 / 安装应用」。

> 通过 `file://` 直接双击打开时，部分浏览器会拦截对 GitHub API 的跨域请求。云同步建议通过 https 地址使用。

## 数据结构

```json
{
  "version": 1,
  "updatedAt": "2026-10-28T08:30:00.000Z",
  "deletedIds": [],
  "settings": {
    "seasonalEnabled": true,
    "soonDays": 2
  },
  "plants": [
    {
      "id": "p_lz8x9k2",
      "name": "绿萝",
      "location": "阳台",
      "intervalDays": 7,
      "note": "见干见湿",
      "createdAt": "2026-10-01",
      "waterings": ["2026-10-05", "2026-10-12", "2026-10-20"]
    }
  ]
}
```

- `waterings` 是浇水日期数组，`YYYY-MM-DD` 格式，自动保持升序
- 上次浇水日 = 数组里最大的那个；数组为空则用 `createdAt`
- 下次浇水日 = 上次浇水日 + `intervalDays` × 季节系数
- `updatedAt` 用于同步时判断谁的数据更新
- `deletedIds` 是已删除植物的墓碑，防止同步后被「复活」

## 季节系数对照

| 月份 | 系数 | 含义 |
|---|---|---|
| 12、1、2 | ×1.3 | 冬季蒸发慢，拉长间隔 |
| 3、4、5 | ×1.0 | 正常 |
| 6、7、8 | ×0.7 | 夏季蒸发快，缩短间隔 |
| 9、10、11 | ×1.0 | 正常 |

不需要可在设置里关闭。
