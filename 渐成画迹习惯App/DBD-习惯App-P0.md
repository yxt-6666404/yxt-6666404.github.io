# 习惯 App（P0）数据库/存储设计（DBD）

源材料（保留不改）：[define-habit-app-p0-improved/spec.md](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/.trae/specs/define-habit-app-p0-improved/spec.md)  
对应 TDD：[TDD-习惯App-P0.md](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/.trae/project/TDD-%E4%B9%A0%E6%83%AFApp-P0.md)

## 1. 存储目标与约束

- P0 默认本地存储，离线可用，不依赖账号与云端。
- 需要支持：按日期读取今日清单、按习惯+日期读取 checkin、按标签聚合、按周期统计聚合、保存与管理快照、排序持久化。

## 2. 本地数据库（IndexedDB）

### 2.1 数据库标识
- 数据库名：HabitTrackerDB
- schemaVersion：从 1 起，后续仅增不减

### 2.2 Stores（表）与索引

#### 2.2.1 habits
- 主键：id（UUID string）
- 常用索引（建议）：archived、enabled、createdAt
- **完整索引定义**：
  ```
  createIndex('archived', 'archived', { unique: false })
  createIndex('enabled', 'enabled', { unique: false })
  createIndex('createdAt', 'createdAt', { unique: false })
  ```

#### 2.2.2 checkins
- 主键：id（UUID string）
- 关键索引：
  - [habitId+date] 复合索引：查询某习惯某日记录
  - [date]：按日期批量查询（统计/回顾/日历摘要）
  - [isHabitDeleted]：筛选/排除已删除习惯的历史
- **完整索引定义**：
  ```
  createIndex('habitId+date', ['habitId', 'date'], { unique: true })
  createIndex('date', 'date', { unique: false })
  createIndex('isHabitDeleted', 'isHabitDeleted', { unique: false })
  createIndex('habitId', 'habitId', { unique: false })
  ```

#### 2.2.3 oneOffTasks
- 主键：id（UUID string）
- 关键索引：
  - [date]：按日期读取当日临时事项
- **完整索引定义**：
  ```
  createIndex('date', 'date', { unique: false })
  ```

#### 2.2.4 tags
- 主键：id（UUID string）
- 关键索引：
  - [type]：领域/场景 vs 状态/目标
- **完整索引定义**：
  ```
  createIndex('type', 'type', { unique: false })
  ```

#### 2.2.5 settings
- 主键：key（string）
- 说明：可采用单条记录存放所有设置对象，或以多个 key 拆分；P0 以单条为主简化读取。
- **完整索引定义**：
  ```
  // P0 使用单条记录，key 固定为 'app_settings'
  createIndex('key', 'key', { unique: true })
  ```

#### 2.2.6 snapshots
- 主键：id（UUID string）
- 关键索引：
  - [pictureId]：按图片筛选快照
  - [savedAt]：按时间排序
  - [stage]：按阶段筛选（阶段4或阶段5）
- **完整索引定义**：
  ```
  createIndex('pictureId', 'pictureId', { unique: false })
  createIndex('savedAt', 'savedAt', { unique: false })
  createIndex('stage', 'stage', { unique: false })
  createIndex('pictureId+savedAt', ['pictureId', 'savedAt'], { unique: false })
  ```

### 2.3 索引设计原则
- **复合索引顺序**：遵循查询条件从左到右的最左前缀原则
- **唯一性约束**：habitId+date 组合确保每日每习惯仅一条打卡记录
- **冗余字段**：isHabitDeleted 用于避免关联查询 habits 表，提升性能
- **日期字符串格式**：统一使用 YYYY-MM-DD 字符串格式，便于索引排序

## 3. 领域模型（字段定义）

### 3.1 Habit
- id: string（UUID）
- name: string
- icon: string | null
- color: string | null
- habitTagIds: string[]
- frequency: object（每日/每周/每月/自定义，含配置）
- unit: string | null
- dailyTargetCount: number（>=1）
- expectedDailyMaxCount: number | null
- startDate: string（YYYY-MM-DD）
- targetDays: number（>0）
- enabled: boolean
- paused: boolean
- archived: boolean
- archivedAt: number | null（timestamp，记录归档操作时间，用于判断归档是否在当日完成）
- completed: boolean
- completedAt: string | null（YYYY-MM-DD）
- note: string | null
- createdAt: number（timestamp）
- updatedAt: number（timestamp）

#### 频率配置 frequency 详细结构
```typescript
type FrequencyConfig = 
  | { type: 'daily' }
  | { type: 'weekly'; weekdays: number[] }  // 1-7（周一至周日）
  | { type: 'monthly'; days: number[] }      // 1-31
  | { type: 'custom'; interval: number }      // N天（interval >= 2）
```

#### 提醒配置 reminders（可选字段）
```typescript
type Reminder = {
  id: string;
  time: string;  // HH:mm 格式，24小时制
  enabled: boolean;
};
// 数组长度 <= 3
```

### 3.2 Checkin
- id: string（UUID）
- habitId: string（UUID，必填；习惯删除后仍保持不变）
- habitNameSnapshot: string（必填；用于删除后可读性）
- date: string（YYYY-MM-DD）
- doneCount: number（>=0）
- note: string | null
- eventTimestamps: number[] | null
- updatedAt: number
- isHabitDeleted: boolean（默认 false；删除 habit 时批量更新为 true）
- weightEarned: boolean（默认 false；一旦 true 永不回退）

### 3.3 OneOffTask
- id: string（UUID）
- date: string（YYYY-MM-DD）
- name: string
- icon: string | null
- color: string | null
- tagIds: string[]
- oneOffTargetCount: number（>=1）
- doneCount: number（>=0）
- note: string | null
- eventTimestamps: number[] | null
- createdAt: number
- updatedAt: number

### 3.4 Tag
- id: string（UUID）
- name: string
- type: "domain" | "status"（或等价枚举）
- color: string（低饱和色板内的值）
- createdAt: number
- updatedAt: number

### 3.5 Snapshot
- id: string（UUID）
- pictureId: string（UUID，关联激励图片，如"orange_elephant_01"）
- pictureNameSnapshot: string（图片名称快照，如"橘子象-1号图"，用于显示）
- stage: 0 | 1 | 2 | 3 | 4 | 5（阶段标记：0-5对应权重0-15的各个阶段）
- weightAtSave: number（保存时的权重值：12或15）
- savedAt: number（timestamp）
- imageData: string（Base64 PNG，阶段截图）
- pixelData: unknown | null（可选，保留像素数据用于重绘）
- note: string | null（用户可选添加的备注）

### 3.6 Settings（关键结构）

#### 3.6.1 viewSortOrders（排序持久化）
- viewSortOrders: Record<string, Array<{ id: string, type: "habit" | "oneOff", sortOrder: number }>>
- viewKey 约定：
  - today_YYYY-MM-DD
  - tag_<tagId>_todo
  - tag_<tagId>_others

#### 3.6.2 激励全局状态（示例字段）
- currentPictureId: string
- currentPictureWeight: number（0..15）
- globalTotalWeight: number（只增不减）
- unlockedTypes: string[]
- unlockedPictures: string[]
- completedPictures: string[]
- currentType: string | null
- currentPictureNumber: number | null
- completedCountInType: number | null
- pictureTypeMap: Record<string, string>（图片与类型的映射，如 {"1": "type1", "1_1": "type1", "2": "type1", "2_1": "type1", "3": "type2", "3_1": "type2"}）
- typeUnlockRequirements: Record<string, number>（类型解锁权重要求，如 {"type1": 0, "type2": 15, "type3": 30}，每类递增15）

#### 3.6.3 提醒与免打扰设置
- doNotDisturbEnabled: boolean
- doNotDisturbStart: string（HH:mm 格式）
- doNotDisturbEnd: string（HH:mm 格式）

#### 3.6.4 外观设置（P0）
- themePreference: "light" | "dark" | "system"  // 深色模式偏好，默认"system"
- themeColor: string  // 主题颜色，默认"#D97757"
- 存储键：app_settings（与其他设置统一存储）

外观设置数据结构：
```typescript
interface AppearanceSettings {
  themePreference: "light" | "dark" | "system";  // 深色模式偏好
  themeColor: string;  // 主题颜色Hex值
}
```

主题颜色可选值（与PRD一致）：
- 温暖橙: "#D97757"（默认）
- 蓝色: "#6A9BCC"
- 绿色: "#788C5D"
- 紫色: "#9B8AA3"
- 粉色: "#C49BA8"
- 黄色: "#C9B896"
- 青色: "#7DA3A8"
- 橙色: "#E67E22"
- 深紫: "#8E44AD"
- 深绿: "#27AE60"

#### 3.6.5 导出元数据
- lastExportDate: number | null
- exportMetadata: object | null

## 4. 删除策略与历史可读性

- habits：硬删（移除记录本体）。
- checkins/snapshots：不删除；保留 habitId；依赖 habitNameSnapshot 保可读性。
- isHabitDeleted：作为冗余字段加速查询；删除 habits 时对关联记录批量更新。

## 5. schemaVersion 与升级策略

- 版本升级只允许向前：新增字段、补默认值、增加索引等。
- 升级时的迁移原则：
  - 旧记录缺失字段：补默认值，不改变既有业务含义
  - 新索引：尽量在升级阶段建立，避免运行期全表扫描
- 降级保护：检测到数据版本高于当前应用支持时，提示用户升级应用版本以继续使用。

## 6. 数据容量与清理策略（建议）

- 快照上限：50（业务限制，避免存储爆炸）。
- 大量历史 checkins 下的性能基线：100+ 习惯/一年数据仍可用。
- 可选的 P1 清理能力：压缩历史时间戳、归档旧数据（不属于 P0）。

### 6.1 存储空间预估
- 100个习惯 + 1年数据 ≈ 5MB（含文件箱子图片）
- 容量预警：IndexedDB 使用超过 80% 时提示用户

### 6.2 版本升级与数据迁移
- 使用 Dexie.js 的 upgrade 机制
- 字段新增时补默认值，不改变既有业务含义
- 降级保护：检测到数据版本高于当前应用支持时，提示用户升级应用版本
