# MoviePilot-Plugins（V3 插件仓库 · po9873）

MoviePilot **V3** 第三方插件仓库，**仅适配 V3**（`system_version >= 3.0.0`），不再维护 V2 版本。

## 仓库内的插件

| 插件名 | 插件 ID | 目录 | 当前版本 | 作者 |
| --- | --- | --- | --- | --- |
| DC助手V3 | `DockerCopilotHelper` | `plugins.v3/dockercopilothelper` | 3.0.5 | po9873 |
| CMS通知V3 | `CMSNotify` | `plugins.v3/cmsnotify` | 3.0.0 | po9873 |

> 两个插件名称都带「V3」后缀，作者署名均为 po9873，图标统一放在本仓库 `icons/` 下。

## 安装方法（通过 MoviePilot 插件市场）

1. 打开 MoviePilot → **设定 → 插件 → 插件市场**
2. 点「插件市场设置」，在输入框填入本仓库地址并添加：
   ```
   https://github.com/po9873/MoviePilot-Plugins
   ```
   （地址必须是 `https://github.com/<owner>/<repo>` 形式，不要带 `/tree/main` 等后缀，否则会被静默丢弃）
3. 保存后回到插件市场，刷新列表，搜索 **DC助手V3** 或 **CMS通知V3** → 安装
4. 以后有新版本，在插件页点「更新」即可自动升级（版本号以 `package.v3.json` 为准）

> 要求：MoviePilot **V3**（`system_version >= 3.0.0`）。V2 版 MoviePilot 无法安装本仓库插件。
> 若市场里搜不到：先在插件市场点「同步 Wiki」再刷新（本仓库已提交收录申请，合并后即可出现在官方清单里）。

## 本地安装（离线 zip）

插件 → 本地安装/上传，导入 zip 即可，zip 根目录必须是插件目录：

- `DC助手V3-手动安装.zip` → 根目录 `dockercopilothelper/`
- `CMS通知V3-手动安装.zip` → 根目录 `cmsnotify/`

也可以自行打包：进入 `plugins.v3/` 目录，把对应插件目录打包为 zip（`zip -r xxx.zip dockercopilothelper`）。

---

## 功能

配合 [DockerCopilot](https://github.com/xcz1997/dockerCopilot) 使用，**一个插件实例联动局域网全部 DC 实例**：

- **多设备**：每台 DC 填一组「服务器地址 + secretKey」，保存后自动出现下一行，想加几台加几台
- **按设备独立勾选容器**：更新通知 / 自动更新 / 各自独立的容器清单
- **更新通知**：按周期检查所选容器更新并发送 MP 通知，标题标注来源设备
- **自动更新**：按周期对所选容器执行 DC 一键更新，可跟踪进度
- **自动备份**：按周期对所有设备执行容器配置备份
- **TAG 异常自愈**：容器 `usingImage` 为空或为 `sha256:` 时，先删镜像按 ID 重拉，再用创建镜像名更新
- **数据页**：彩色统计 + 每设备「待更新」明细 + 原生展开查看全部容器 + 更新分布饼图 + 更新记录（**只显示当天**，标题带累计与历史跨度）
- **连不上要说人话**：某台 DC 连不上时，配置页下拉会显示「⚠ 读不到容器：端口拒绝连接…」而不是空白，
  数据页对应卡片也会写出具体原因；**此时不会清空你已勾选的通知容器**（修掉了以前一打开配置页就把勾选洗掉的 bug）。
  日后新增的 DC 设备 / 新建的容器不需要任何操作，配置页和数据页刷新后自动出现
- **配置导出 / 导入**：浏览器直接下载 / 上传 JSON 文件

## 配置导出格式（可移植，与独立 Docker 面板互通）

导出的 JSON 使用固定 schema，插件版与独立 Docker 面板共用同一份结构——插件「导出配置」的文件可直接在面板「数据迁移 → 导入设备」使用，面板「导出设备」的文件也能在本插件导入，两边自由搬家：

```json
{
  "schema": "dc-helper",
  "schema_version": 1,
  "exported_at": "2026-10-23 16:50:00",
  "source": "moviepilot-plugin/dockercopilothelper",
  "enabled": true,
  "devices": [
    {
      "name": "my-nas",
      "host": "http://dc-host.example:12712",
      "secret_key": "你的secretKey",
      "updatable": ["moviepilot", "qbittorrent"],
      "auto_update": ["moviepilot"]
    }
  ],
  "notify": {
    "update_cron": "0 9 * * *",
    "update_notify": true,
    "auto_update_cron": "",
    "auto_update_notify": true,
    "schedule_report": false,
    "delete_images": false,
    "backup_cron": "",
    "backup_notify": true,
    "dc_update_cron": "",
    "dc_update_notify": true,
    "interval": 10,
    "interval_limit": 6
  }
}
```

导入同时兼容两种格式：

1. 上面的 `dc-helper` 格式（推荐，跨版本/跨形态通用；dc-panel 导出的文件即此格式）
2. 旧版扁平格式（`host1` / `secretKey1` / `name1` / `updatablelist1` …）

## 接口一览（插件注册的动态 API）

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | `/api/v1/plugin/DockerCopilotHelper/container_update` | 数据页触发容器更新（JSON body: env/cid/cname/image） |
| GET | `/api/v1/plugin/DockerCopilotHelper/update_now` | 原生链接触发更新，结果显示在独立页面 |
| GET | `/api/v1/plugin/DockerCopilotHelper/download_config?apikey=` | 导出配置（附件下载） |
| POST | `/api/v1/plugin/DockerCopilotHelper/upload_config?apikey=` | 导入配置（multipart 上传） |
| POST | `/api/v1/plugin/DockerCopilotHelper/toggle_expand` | 展开/收起设备容器（保留接口） |

## 安装（DC助手V3）

本地安装：插件管理 → 上传 `DC助手V3-手动安装.zip`（zip 根目录为 `dockercopilothelper/`）。

仓库安装：在 `PLUGIN_MARKET` 添加本仓库地址，要求 main 分支且目录结构为 `plugins.v3/` + `package.v3.json`。

## 来源（DC助手V3）

参考自 gxterry 的 V1 插件 `dockercopilothelper`，按 MoviePilot V3 规范重写并升级为多设备支持，V3 版由 po9873 维护。DC API 逻辑：本地 HS256 签发 JWT，调用 `/api/containers`、`/api/container/{id}/update`、`/api/progress/{taskID}`、`/api/container/backup`、`/api/images`、`/api/image/{sha}`、`/api/version`、`/api/update`。

---

# CMS通知V3（CMSNotify）

整理完成（115 网盘等）媒体后，通知 CMS 进行增量同步（生成 strm）。V2 原版作者 imaliang，V3 版由 **po9873** 适配维护，版本号从 **3.0.0** 重新计数。

## 功能与配置

- 监听 MoviePilot 的 **整理完成**（`TransferComplete`）与 **刮削完成**（`MetadataScrape`）事件，命中目录后通知 CMS 增量同步
- **跨机器访问**：CMS 地址可填任意可达地址（局域网 IP、容器网关地址等），不限制本机。
  - MoviePilot 在 Docker 里、CMS 在宿主机：填 `http://172.17.0.1:9527`
  - 两台不同机器：填 CMS 所在机器的实际地址，例如 `http://192.168.6.244:9527`
- 配置项：`cms_domain`（CMS 地址）、`cms_token`（同步 token）、`monitor_dirs`（监控目录，多个换行分隔）、`scrape_metadata`（刮削后也通知）、`only_once`（同路径只通知一次）、`enabled`

## V3 适配要点（相对 V2 原版）

- 导入路径迁移到 `app.sdk.*`（V3 已移除 `app.core` / `app.utils` / `app.log`）
- 事件注册改为显式 `EventType.TransferComplete` 与 `EventType.MetadataScrape`（V3 不再支持空参数全量捕获）
- 事件负载改为直接读取 V3 契约对象属性（`TransferInfo` / `FileURI`）
- 修复 token 错误时仍误报「通知成功」：改为解析响应体 `code` 字段（0 / 200 才算成功）
- 修复刮削分支取错对象（`fileitem` 而非 `storage`）导致永不命中的 bug

## 安装（CMS通知V3）

本地安装：插件管理 → 上传 `CMS通知V3-手动安装.zip`（zip 根目录为 `cmsnotify/`）。

仓库安装：同上，在市场里搜索 **CMS通知V3**。
