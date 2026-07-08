# Notes（笔记）

一个参考三星笔记 App 重新设计的 Android 原生笔记应用，**新增图形输入能力**（手画图形可保存为模板并在任意笔记中复用），并支持把笔记内容同步到 **三星云 / 百度网盘 / 群晖 NAS / 通用 WebDAV / Google Drive**。

---

## ✨ 主要特性

| 类别 | 能力 |
|------|------|
| 编辑器 | 标题、文本、手写（钢笔/荧光笔/铅笔/毛笔/书法笔）、橡皮擦（笔画/区域）、选择、撤销、便利贴、表格、PDF 嵌入、文档扫描、相机、录音、音频文件 |
| 三星笔记复刻 | 顶部栏（返回 + 标题 + 图形库 + ＋ + 三点）、底部工具栏（表格/钢笔/荧光笔/橡皮擦/选择/图形/保存为模板/手指绘画）、笔设置面板（5 种笔、粗细、颜色行）、橡皮擦设置面板、背景颜色（7 种纸张色）、全屏、添加标签、另存为、关闭手指绘画、数学求解器开关 |
| **图形输入（新）** | 在画布上手绘一个图形（如箭头、流程图节点、自创符号），点工具栏「保存为模板」即可把它存入「图形模板库」，之后任意笔记中都能从图形库直接拖入复用 |
| 云同步 | 本地优先（Room），后台 WorkManager 同步；支持三星云、百度网盘、群晖 NAS、通用 WebDAV、Google Drive |
| 数据 | Room 数据库、kotlinx.serialization、DataStore、协程 + Flow |
| UI | Jetpack Compose + Material3，自适应深色模式 |

---

## 🏗 项目结构

```
NoteApp/
├── app/
│   ├── build.gradle.kts
│   ├── proguard-rules.pro
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/superz/notes/
│       │   ├── NotesApp.kt                 # Application
│       │   ├── MainActivity.kt
│       │   ├── data/
│       │   │   ├── db/                      # Room: entities, DAOs, database
│       │   │   ├── model/                   # 领域模型 + Mappers
│       │   │   └── repository/
│       │   │       ├── NoteRepository.kt
│       │   │       └── sync/                # ★ 同步框架
│       │   │           ├── SyncContract.kt
│       │   │           ├── SyncProvider.kt          # 抽象接口
│       │   │           ├── SyncManager.kt           # 调度器
│       │   │           ├── SyncForegroundService.kt # 通知栏同步
│       │   │           ├── WebDAVSyncProvider.kt    # WebDAV (✅ 可用)
│       │   │           ├── SynologyNASSyncProvider.kt# 群晖 NAS (✅ 走 WebDAV)
│       │   │           ├── BaiduNetdiskSyncProvider.kt# 百度网盘 (OAuth2 骨架)
│       │   │           ├── SamsungCloudSyncProvider.kt # 三星云 (SDK 占位)
│       │   │           └── GoogleDriveSyncProvider.kt  # Google Drive (SDK 占位)
│       │   ├── ui/
│       │   │   ├── theme/                   # 颜色 / 字体 / 主题
│       │   │   ├── navigation/              # NavHost
│       │   │   └── screens/
│       │   │       ├── notelist/            # 首页（笔记列表）
│       │   │       ├── editor/              # 编辑器、＋菜单、三点菜单、笔/橡皮擦面板
│       │   │       ├── drawing/             # DrawingCanvas、StrokeRenderer、SvgPathEncoder
│       │   │       ├── shapelibrary/        # 图形模板库
│       │   │       └── syncsettings/        # 同步设置
│       │   └── util/JsonCodec.kt
│       └── res/
│           ├── values/                      # strings/colors/themes
│           ├── values-night/
│           └── xml/                         # backup / file-provider paths
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

---

## 🛠 构建与运行

### 环境要求
- Android Studio Koala / Ladybug 或更高
- JDK 17
- Android SDK 34（最低运行 API 26 / Android 8.0）

### 步骤
1. 在 Android Studio 中 `Open` 选择 `NoteApp/` 目录；
2. 等待 Gradle 同步（首次需要下载依赖）；
3. 连接 Android 设备或模拟器，点 ▶ Run 'app'。

> **可选**：把 `applicationId` 改为你自己的包名（同步需要在每个云服务商处更新对应的 OAuth 回调包名）。

---

## ☁️ 接入云同步服务（要达到「笔记记录的内容存入某个网络的账号」需要做什么）

> 下面分服务介绍。**WebDAV / 群晖 NAS 已经在代码中跑通**，其它三个（三星云 / 百度网盘 / Google Drive）有完整骨架，按各服务要求补全 API 凭据后即可用。

### ───────────────────────────────────────────
### A. 通用 WebDAV（**推荐，零成本，开箱即用**）
### ───────────────────────────────────────────

**适用**：Nextcloud / ownCloud / 坚果云 / Box / 群晖 WebDAV Server / 任意 WebDAV 服务。

**你只需要**：
1. 在你的 WebDAV 服务器上准备一个账号（用户名 + 密码 / 应用专用密码）；
2. 在 App「同步设置 → 通用 WebDAV → 连接」中填入：
   - 服务器地址：例如 `https://dav.jianguoyun.com` 或 `https://cloud.example.com`
   - 用户名
   - 密码（坚果云请用「应用密码」而非登录密码）
   - 远端目录：例如 `/Notes`
3. 点「连接」，App 会执行 PROPFIND 校验凭据，通过即可同步。

**无需注册开发者账号、无需 API Key、无需审核。** 这是最快上线的方案。

### ───────────────────────────────────────────
### B. 群晖 NAS（Synology）—— 走 WebDAV 路线（**推荐**）
### ───────────────────────────────────────────

**你只需要**：
1. 登录 DSM → 套件中心 → 安装 **WebDAV Server** 套件；
2. 打开 WebDAV Server → 启用 **HTTPS**（默认端口 5006），建议关闭 HTTP；
3. 控制面板 → 用户 → 新建专用账号 `notes-sync`，授予 `home` 目录读写权限；
4. 控制面板 → 外部访问 → DDNS，配置一个域名（例如 `xxx.synology.me`），并在路由器上把 5006 端口转发到 NAS；
5. 在 App「同步设置 → 群晖 NAS → 连接」中填入：
   - 服务器地址：`https://xxx.synology.me:5006`
   - 用户名：`notes-sync`
   - 密码
   - 远端目录：`/Notes`（自动在用户的 home 下创建）
6. 点连接。

**优点**：完全自有数据，无月费，无第三方审核，HTTPS 加密传输。

**备选方案（高级）**：使用 Synology DSM REST API（`entry.cgi?api=SYNO.FileStation.*`），可享受 2FA、文件版本、共享链接，但开发量大，可后续按需切换。

### ───────────────────────────────────────────
### C. 百度网盘
### ───────────────────────────────────────────

**要达到可上线，需要：**

1. **开发者注册**：访问 https://pan.baidu.com/developers 注册开发者账号；
2. **创建应用**：选择「网盘应用」类型，获得：
   - `AppKey`（client_id）
   - `SecretKey`（client_secret）
   - `redirect_uri`：建议填 `oob` 或自定义 scheme（如 `com.superz.notes://oauth`）
3. **配置 AppKey 到工程**：
   - 在 `app/build.gradle.kts` 中添加：
     ```kotlin
     buildConfigField("String", "BAIDU_APP_KEY", "\"你的AppKey\"")
     buildConfigField("String", "BAIDU_SECRET_KEY", "\"你的SecretKey\"")
     buildConfigField("String", "BAIDU_REDIRECT_URI", "\"oob\"")
     ```
   - 在 `BaiduNetdiskSyncProvider.connect()` 中读取 `BuildConfig.BAIDU_APP_KEY` 等；
4. **OAuth 授权流程**（已在 `BaiduNetdiskSyncProvider.exchangeCodeForToken` 中实现）：
   - 用 `CustomTabsIntent` 打开
     `https://openapi.baidu.com/oauth/2.0/authorize?response_type=code&client_id=APP_KEY&redirect_uri=REDIRECT_URI&scope=basic,netdisk`
   - 用户同意后跳转回 `REDIRECT_URI?code=XXXX`，App 用 code 换 token；
   - 保存 access_token + refresh_token（30 天有效期，自动刷新）；
5. **API 调用**（已在 `uploadFile` / `downloadFile` 中实现骨架）：
   - 上传：`https://pan.baidu.com/rest/2.0/xpan/file?method=upload&access_token=...&path=/apps/Notes/{id}.json`
   - 下载：`https://pan.baidu.com/rest/2.0/xpan/multimedia?method=download&...`
6. **审核**：百度要求应用提交审核才能给最终用户使用，审核通过后会显示在百度网盘「我的应用」中。

**当前代码状态**：token 刷新 / 上传 / 下载的 HTTP 调用骨架已写好，填入 AppKey / SecretKey 并补全 OAuth UI 后即可上线。

### ───────────────────────────────────────────
### D. 三星云（Samsung Cloud）
### ───────────────────────────────────────────

**注意**：三星云不像百度网盘那样提供公开 REST API 给第三方写入，必须走 Samsung 官方 SDK。

**要达到可接入，需要：**

1. 申请「Samsung Cloud Partner」：
   - 访问 https://developer.samsung.com/samsung-cloud
   - 提交 partner application，描述应用场景、目标用户、数据类型；
   - Samsung 审核通过后会提供 SDK + 接入文档；
2. 集成 SDK：
   ```kotlin
   // build.gradle.kts
   implementation("com.samsung.android.scloud:samsung-cloud-sdk:x.x.x")
   ```
3. 在 `SamsungCloudSyncProvider` 中替换占位代码：
   - `connect()` → 调用 `SamsungCloud.getAccount(activity)` 拉起三星账号登录；
   - `push()` → `SamsungCloud.registerAppData(SYNC_TYPE_NOTES, payload)`
   - `pull()` → `SamsungCloud.requestSync(SYNC_TYPE_NOTES, listener)`
4. 在 `AndroidManifest.xml` 中添加 Samsung SDK 要求的 meta-data：
   ```xml
   <meta-data android:name="com.samsung.android.scloud.appid" android:value="你的AppId" />
   ```
5. **限制**：只能在搭载 One UI 的三星设备上使用，用户在「设置 → 账户 → Samsung Account → 同步」中能看到本应用。

**当前代码状态**：占位实现，框架已就绪，接入 SDK 后替换 `TODO` 即可。

**替代方案**：如果只是想让用户用「三星账号登录」而非真正同步到三星云，可以用 Samsung Account SDK + 自建后端（AWS S3 / 阿里云 OSS / Cloudflare R2）实现，详见 `SamsungCloudSyncProvider` 类注释中的「方案 B」。

### ───────────────────────────────────────────
### E. Google Drive
### ───────────────────────────────────────────

**要达到可接入，需要：**

1. 在 https://console.cloud.google.com/ 创建项目，启用 **Google Drive API**；
2. 创建 Android OAuth 客户端，配置：
   - 包名：`com.superz.notes`（或你的 applicationId）
   - SHA-1 指纹：`keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android | grep SHA1`
3. 添加依赖：
   ```kotlin
   implementation("com.google.android.gms:play-services-auth:21.2.0")
   implementation("com.google.api-client:google-api-client-android:2.6.0")
   implementation("com.google.apis:google-api-services-drive:v3-rev20240609-2.0.0")
   ```
4. 在 `GoogleDriveSyncProvider` 中：
   - `connect()` → `GoogleSignIn.requestPermissions(... SCOPE_DRIVE_APPFOLDER)`
   - `push()` → `Drive.files().create(...).setSpaces("appDataFolder").execute()`
   - `pull()` → `Drive.files().list().setSpaces("appDataFolder").execute()`
5. 在 `strings.xml` 中填入 OAuth 客户端 ID：
   ```xml
   <string name="google_server_client_id">your-client-id.apps.googleusercontent.com</string>
   ```

**优点**：使用 `appDataFolder`（应用私有空间）不占用用户 Google Drive 可见配额；不依赖设备品牌。

---

## 📋 数据同步设计要点

- **本地优先**：所有编辑先写 Room 数据库，再异步同步到云端；离线可用。
- **同步状态字段**：`NoteEntity.syncState ∈ {PENDING, SYNCED, CONFLICT}`。
- **软删除 + 墓碑**：删除笔记时设置 `is_deleted=1` 并上传墓碑文件，避免远端复活。
- **冲突解决策略**：默认 `NEWEST_WINS`，可在 `SyncManager` 中切换为 `LOCAL_WINS` / `REMOTE_WINS` / `KEEP_BOTH`。
- **后台同步**：基于 WorkManager（待接入），可选「仅 Wi-Fi」。
- **前台通知**：`SyncForegroundService` 在同步进行时显示进度通知。

---

## 🎨 图形输入（Shape Input）使用流程

1. 在笔记编辑器底部工具栏选择「钢笔」或「荧光笔」；
2. 在画布上手画一个图形（箭头、流程图节点、自创符号等）；
3. 点击工具栏 **「保存为模板」** 按钮；
4. 在弹窗中输入名称 + 分类（如「箭头」「流程图」「几何」）；
5. 点保存，图形进入「图形模板库」；
6. 之后在任意笔记中，点工具栏 **「图形」** 按钮 → 选择模板 → 自动插入到当前画布。

支持按分类过滤、查看使用次数、删除不再需要的模板。

---

## 🚧 后续可迭代项

- [ ] 接入 WorkManager 周期同步任务
- [ ] 实现笔记锁（生物识别 / PIN）
- [ ] 接入 CameraX 完成相机 / 文档扫描
- [ ] 接入 MediaRecorder 完成录音
- [ ] 笔触预设收藏
- [ ] 数学公式求解器（接 ML Kit 或第三方）
- [ ] 图形模板支持旋转 / 缩放
- [ ] 富文本编辑（粗体、列表、引用）
- [ ] PDF 嵌入预览
- [ ] 表格编辑器
- [ ] 全屏模式

---

## 📄 License

MIT License. 仅供学习参考。
