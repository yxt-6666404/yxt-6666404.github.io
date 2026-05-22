# 渐成画迹 - 开发任务列表

## Phase 1: 基础设施（第1周）

### Task 1: 项目初始化与架构搭建

- [x] 1.1 使用Vite创建Vue3项目
  - [x] 运行 `npm create vite@latest habit-tracker -- --template vue`
  - [x] 安装依赖：`vue`, `vue-router`, `pinia`
  - [x] 配置TypeScript支持
- [x] 1.2 配置Tailwind CSS
  - [x] 安装Tailwind CSS及相关依赖
  - [x] 创建tailwind.config.js，配置darkMode: 'class'
  - [x] 配置内容扫描路径
  - [x] 创建基础CSS变量（主题颜色）
- [x] 1.3 安装核心依赖
  - [x] 安装Dexie.js：`npm install dexie`
  - [x] 安装Capacitor核心：`npm install @capacitor/core @capacitor/cli`
  - [x] 安装本地通知插件：`npm install @capacitor/local-notifications`
  - [x] 安装图标库（推荐：lucide-vue-next或heroicons）
  - [x] 安装日期处理库：`npm install date-fns`
- [x] 1.4 建立项目目录结构
  ```
  src/
  ├── assets/           # 静态资源（图标、图片）
  ├── components/       # 可复用组件
  │   ├── common/       # 通用组件（Button、Modal等）
  │   ├── habits/       # 习惯相关组件
  │   ├── calendar/     # 日历组件
  │   └── stats/        # 统计图表组件
  ├── views/            # 页面视图
  │   ├── TodayView.vue
  │   ├── HabitsView.vue
  │   ├── TagsView.vue
  │   ├── StatsView.vue
  │   ├── MotivationView.vue
  │   └── SettingsView.vue
  ├── stores/           # Pinia状态管理
  │   ├── habits.ts
  │   ├── checkins.ts
  │   ├── oneOff.ts
  │   ├── tags.ts
  │   ├── settings.ts
  │   ├── snapshots.ts
  │   └── theme.ts
  ├── repositories/     # 数据访问层
  │   ├── base.ts       # Repository基类
  │   ├── habits.ts
  │   ├── checkins.ts
  │   ├── oneOff.ts
  │   ├── tags.ts
  │   ├── settings.ts
  │   └── snapshots.ts
  ├── utils/            # 工具函数
  │   ├── date.ts       # 日期处理
  │   ├── algorithms.ts # 核心算法
  │   └── constants.ts  # 常量定义
  ├── types/            # TypeScript类型定义
  │   └── index.ts
  ├── composables/      # Vue组合式函数
  │   ├── useHabits.ts
  │   ├── useCheckins.ts
  │   └── useTheme.ts
  ├── router/           # 路由配置
  │   └── index.ts
  └── App.vue
  ```
- [x] 1.5 实现响应式自适应布局
  > **PRD功能点**：FR-14（响应式自适应布局）
  > **原型参考**：HTML原型中的手机框架仅用于预览移动端显示效果，开发使用相同的响应式组件
  > **状态**：✅ 核心功能已完成，触摸目标优化待后续迭代
  - [x] 配置Tailwind响应式断点（sm:640px, md:768px, lg:1024px）
  - [x] 实现单栏垂直布局组件（同时适配移动端和桌面端）
    - 说明：当前通过 App.vue 全局壳 + 各 View 单栏结构实现（未单独抽出 AppShell 组件作为路由壳）
  - [x] 移动端（<768px）：内容宽度100%，底部Tab导航
    - 说明：BottomTabBar 由 App.vue 统一渲染；主 Tab 页面内容区需预留底部空间避免遮挡
  - [x] 桌面端（≥768px）：内容区最大宽度1200px居中，随屏幕宽度扩展
  - [x] 使用Tailwind CSS md:断点实现布局自适应
  - [x] 确保任意宽度下功能完全一致（拖拽/长按/弹窗定位/无横向溢出等）
  - [x] 滚动条隐藏实现（全局CSS规则，跨浏览器兼容）
  - [x] 文字换行处理（关键位置使用whitespace-nowrap）
  - [x] 弹窗和菜单响应式定位（fixed定位+动态计算位置）
  - [x] 拖拽功能跨断点正常工作
  - [x] 触摸目标大小优化（已完成主要页面优化）
    - [x] 拖拽手柄符合44px最小要求
    - [x] 打卡按钮已优化到44px（TodayView、TagsView）
    - [x] 顶部按钮已优化到44px（TodayView、TagsView、HabitsView）
    - [x] 产品说明按钮已优化到44px（StatsView）
    - [x] 新建按钮已优化到44px×44px（ExpandableAddButton）
    - [x] TagsView 所有按钮已优化到44px
    - [x] HabitsView 所有按钮已优化到44px
    - [x] CalendarView 月份切换按钮已优化到44px
    - [ ] 日历日期格子（32px，日历网格权衡，保持紧凑布局）
    - [ ] 其他设置页面按钮（后续迭代优化）

### Task 2: 数据库设计与实现

- [x] 2.1 定义数据类型接口
  - [x] 创建src/types/index.ts
  - [x] 定义Habit接口（包含所有字段）
  - [x] 定义Checkin接口
  - [x] 定义OneOffTask接口
  - [x] 定义Tag接口
  - [x] 定义Settings接口
  - [x] 定义Snapshot接口
- [x] 2.2 实现Dexie数据库
  - [x] 创建src/db/index.ts
  - [x] 定义HabitTrackerDB类继承Dexie
  - [x] 定义6个表结构
  - [x] 配置版本和升级策略
  - [x] 建立复合索引
- [x] 2.3 实现Repository基类
  - [x] 创建src/repositories/base.ts
  - [x] 实现泛型BaseRepository<T>
  - [x] 提供标准CRUD方法
  - [x] 添加错误处理
- [x] 2.4 实现各实体Repository
  - [x] HabitsRepository（扩展基础CRUD）
  - [x] CheckinsRepository（支持按habitId+date查询）
  - [x] OneOffRepository（支持按date查询）
  - [x] TagsRepository
  - [x] SettingsRepository（key-value存储）
  - [x] SnapshotsRepository

### Task 3: 状态管理Store实现

- [x] 3.1 创建useHabitsStore
  - [x] 定义state：habits列表、loading状态
  - [x] 实现getters：按状态筛选、按标签筛选
  - [x] 实现actions：loadHabits、createHabit、updateHabit、archiveHabit、deleteHabit
- [x] 3.2 创建useCheckinsStore
  - [x] 定义state：checkins映射（habitId -> date -> checkin）
  - [x] 实现getters：获取某日某习惯的打卡记录
  - [x] 实现actions：loadCheckins、checkin、updateCheckin
- [x] 3.3 创建useOneOffStore
  - [x] 定义state：oneOffTasks列表
  - [x] 实现actions：loadOneOffs、createOneOff、completeOneOff、deleteOneOff
- [x] 3.4 创建useTagsStore
  - [x] 定义state：tags列表
  - [x] 实现actions：loadTags、createTag、updateTag、deleteTag
- [x] 3.5 创建useSettingsStore
  - [x] 定义state：settings对象、viewSortOrders
  - [x] 实现actions：loadSettings、updateSettings、updateSortOrder
- [x] 3.6 创建useSnapshotsStore
  - [x] 定义state：snapshots列表、currentPicture信息
  - [x] 实现actions：loadSnapshots、saveSnapshot、deleteSnapshot
- [x] 3.7 创建useThemeStore
  - [x] 定义state：themePreference、themeColor
  - [x] 实现actions：setThemePreference、setThemeColor、initTheme

### Task 4: 核心算法实现

- [x] 4.1 实现日期工具函数
  - [x] 创建src/utils/date.ts
  - [x] 实现formatDate、parseDate、getToday、isSameDay
  - [x] 实现getWeekday、getDayOfMonth、getDayDiff
  - [x] 实现getWeekStart、getWeekEnd、getMonthStart、getMonthEnd
- [x] 4.2 实现应出现判定算法
  - [x] 创建src/utils/algorithms.ts
  - [x] 实现shouldAppear(habit, date): boolean
  - [x] 实现频率匹配逻辑（每日/每周/每月/自定义N天）
  - [x] 编写单元测试验证各种边界情况
- [x] 4.3 实现完成度计算
  - [x] 实现calculateDayCompletion(doneCount, target): number
  - [x] 确保返回值在0-1范围内
- [x] 4.4 实现连续性算法
  - [x] 实现calculateStreak(habit, checkins, endDate): {currentStreak, maxStreak}
  - [x] 处理paused期间跳过逻辑
  - [x] 处理非应出现日逻辑
- [x] 4.5 实现权重计算
  - [x] 实现calculateWeight(checkin, habit): boolean
  - [x] 实现权重冻结逻辑

## Phase 2: 核心业务逻辑（第1-2周）

### Task 5: 今日清单页面开发

> **PRD功能点**：FR-2（今日清单）、FR-3（打卡与补打卡）、FR-7（排序系统）、FR-14（新建功能）
> **原型参考**：`今日清单.html`、`今日清单-没有完成的.html`、`今日清单-排序.html`、`今日清单-新建.html`、`今日清单-长按习惯或事项.html`
> **data-prd标注说明**：
>
> - `data-prd="FR-8: 支持月/年视图切换与选日"` → 日历按钮
> - `data-prd="FR-7: 提供排序选项弹窗"` → 排序按钮
> - `data-prd="FR-3: 目标次数大于1且已开始打卡的习惯，点击后增加已完成次数"` → +1按钮
> - `data-prd="FR-3: 已完成当日目标的习惯，按钮变为灰色不可点击"` → 完成按钮
> - `data-prd="FR-3: 目标次数等于1的习惯，点击后标记习惯为已完成状态"` → 打卡按钮
> - `data-prd="FR-1: 已归档习惯，打卡按钮禁用"` → 归档习惯显示
> - `data-prd="FR-14: 新建功能，提供新建选项弹窗"` → 新建按钮

- [x] 5.1 创建页面布局和路由
  - [x] 创建src/views/TodayView\.vue
  - [x] 配置路由：/today
  - [x] 设置默认路由重定向
  - [x] 实现底部Tab导航组件 (BottomTabBar.vue)
- [x] 5.2 实现习惯卡片组件
  - [x] 在TodayView中直接实现习惯卡片（无需独立组件）
  - [x] 显示图标、名称、标签、进度
  - [x] 实现打卡按钮（根据状态显示不同样式）
  - [x] 实现+1按钮（多次目标，通过isCheckIcon函数判断）
  - [x] 实现长按菜单（编辑/归档/删除）
- [x] 5.3 实现临时事项卡片
  - [x] 在TodayView中与习惯卡片统一渲染
  - [x] 虚线边框样式区分 (border-2 border-dashed)
  - [x] 完成按钮逻辑
- [x] 5.4 实现列表渲染
  - [x] 习惯与临时事项混排 (buildTodayListItems函数)
  - [x] 按viewSortOrders排序
  - [x] 空状态引导
- [x] 5.5 实现打卡功能
  - [x] 点击打卡/+1更新状态 (onAction函数)
  - [x] 完成时触发权重获得 (canEarnWeight检查)
- [x] 5.6 实现顶部操作栏
  - [x] 日期显示
  - [x] 排序按钮（打开排序面板）
  - [x] 日历按钮

### Task 6: 习惯管理页面

> **PRD功能点**：FR-1（习惯管理）、FR-5（临时事项）
> **原型参考**：`习惯编辑页面.html`、`习惯创建编辑页.html`、`临时事项新建页.html`、`临时事项编辑页.html`
> **data-prd标注说明**：
>
> - `data-prd="FR-1: 支持创建/编辑习惯（名称、图标、标签、频率、每日目标次数等）"` → 保存按钮
> - `data-prd="FR-5: 创建新临时事项，默认插入当日列表末尾"` → 保存按钮
> - `data-prd="FR-1: 编辑选项，跳转到习惯/临时事项编辑页"` → 编辑按钮
> - `data-prd="FR-1: 归档操作，归档当日仍在今日清单显示（标记为已归档），次日起隐藏"` → 归档按钮
> - `data-prd="FR-1: 删除选项，删除后不在当前列表出现，删除即彻底删除"` → 删除按钮
> - `data-prd="FR-1: 取消编辑，返回上一页"` → 取消按钮

- [x] 6.1 创建习惯列表页面
  - [x] 创建src/views/HabitsView\.vue
  - [x] 显示所有习惯（按状态分组：进行中/已归档）
- [x] 6.2 实现创建习惯表单
  - [x] 创建src/views/HabitFormView\.vue（页面级组件）
  - [x] 名称输入（20字符限制，maxlength=20）
  - [x] 图标选择器（18个emoji图标）
  - [x] 标签多选（支持新建标签）
  - [x] 频率选择（每日/每周/每月/自定义）
  - [x] 每日目标次数（增减按钮）
  - [x] 开始日期选择（日历抽屉）
  - [x] 备注输入
  - [x] 提醒设置（最多3个时间点）
- [x] 6.3 实现编辑习惯
  - [x] 复用HabitFormView组件
  - [x] 预填充现有数据（onMounted加载）
  - [x] 通过路由区分创建/编辑模式
- [x] 6.4 实现临时事项创建
  - [x] 创建src/views/OneOffFormView\.vue
  - [x] 简化版表单（名称、目标次数、标签、备注）
  - [x] 支持编辑和删除

### Task 7: 排序功能实现

> **PRD功能点**：FR-7（排序系统）
> **原型参考**：`今日清单-排序.html`、`今日清单-没有完成的.html`
> **data-prd标注说明**：
>
> - `data-prd="FR-7: 手动排序，拖拽可用且在重启后保持稳定"` → 拖拽手柄
> - `data-prd="FR-7: 按名称排序规则，仅影响当前展示，不覆盖手动排序"` → 按名称排序
> - `data-prd="FR-7: 按创建时间排序规则，仅影响当前展示"` → 按创建时间排序
> - `data-prd="FR-7: 按完成度排序规则，仅影响当前展示"` → 按完成度排序

- [x] 7.1 实现排序面板组件
  - [x] 在TodayView中直接实现排序弹窗（无需独立组件）
  - [x] 单选选项：手动排序、名称、创建时间、完成进度
  - [x] 拖拽手柄提示
- [x] 7.2 实现拖拽排序
  - [x] 使用原生HTML5拖拽API
  - [x] 显示☰拖拽手柄 (DragHandleIcon)
  - [x] 拖拽结束持久化到settings (updateSortOrder)
- [x] 7.3 实现规则排序
  - [x] 按名称排序（A-Z）(buildTodayListItems中实现)
  - [x] 按创建时间排序
  - [x] 按完成进度排序

## Phase 3: 高风险功能（第2-3周）

### Task 8: 标签系统

> **PRD功能点**：FR-6（标签系统与标签页聚合）
> **原型参考**：`标签页.html`、`标签页-健康.html`、`标签页-空.html`、`标签页-搜索后.html`、`标签页-标签长按.html`

- [x] 8.1 实现标签页布局
  - [x] 创建src/views/TagsView\.vue（1207行完整实现）
  - [x] 顶部标签筛选栏（横向滚动 overflow-x-auto）
  - [x] 双区结构：今日待办置顶区 + 其余区（todayTodoItems + otherItems）
  - [x] 搜索功能（isSearchMode, searchQuery）
- [x] 8.2 实现标签管理
  - [x] 创建标签弹窗（名称输入，maxlength="10"限制）
  - [x] 编辑标签弹窗（预填充现有名称）
  - [x] 删除标签确认弹窗
  - [x] 长按标签编辑/删除菜单（500ms长按触发）
- [x] 8.3 实现标签筛选逻辑
  - [x] 按选中标签过滤习惯（filteredHabits/filteredOneOffs）
  - [x] 双区内容动态计算（今日待办/其他）
  - [x] 排序功能（手动/名称/创建时间/完成度）
  - [x] 拖拽排序持久化（使用viewSortOrders）

### Task 9: 拼豆激励系统

> **PRD功能点**：FR-10（拼豆激励系统）
> **原型参考**：`激励页面原型.html`、`阶段0.html`、`阶段1.html`、`阶段2.html`、`阶段3.html`、`阶段4.html`、`阶段5.html`

- [x] 9.1 实现激励页面布局
  - [x] 创建src/views/MotivationView\.vue
  - [x] 三标签页：当前图片、全部图案、照片箱子
- [x] 9.2 实现拼豆展示组件
  - [x] 创建src/components/motivation/PixelArt.vue
  - [x] Canvas渲染，阶段0-5渐进显示效果
  - [x] 根据revealRatio显示/隐藏像素
  - [x] 根据clarity调整清晰度
- [x] 9.3 实现阶段计算逻辑
  - [x] 监听权重获得事件
  - [x] 更新currentPictureWeight
  - [x] 计算当前阶段（0-5）
- [x] 9.4 实现图片解锁
  - [x] 全部图案网格展示
  - [x] 解锁状态判定
  - [x] 类型解锁逻辑
- [x] 9.5 实现快照功能
  - [x] Canvas截图生成（质量0.8）
  - [x] 阶段4/5自动提示保存
  - [x] 手动保存按钮
  - [x] 照片箱子列表展示
  - [x] 大图查看模式
  - [x] 存储限制（50张LRU）

### Task 10: 日历与补打卡

> **PRD功能点**：FR-8（日历交互）
> **原型参考**：`calendar.html`
> **data-prd标注说明**：
>
> - `data-prd="关闭日历，返回上一页"` → 关闭按钮
> - `data-prd="FR-8: 快速返回今天"` → 今天按钮
> - `data-prd="FR-8: 月视图切换"` → 左右切换按钮

- [x] 10.1 实现日历组件
  - [x] 创建src/views/CalendarView\.vue
  - [x] 月视图（7列网格）([CalendarGrid.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/components/calendar/CalendarGrid.vue))
  - [x] 月份选择器 ([MonthPicker.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/components/calendar/MonthPicker.vue))
  - [x] 年份选择器（只显示有数据的年份）([YearPicker.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/components/calendar/YearPicker.vue))
  - [x] 日期选择功能
  - [x] 圆点标记（最多6个，区分已完成/未完成）([CalendarDay.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/components/calendar/CalendarDay.vue))
- [x] 10.2 实现日历页面
  - [x] 创建src/views/CalendarView\.vue
  - [x] 日历下方显示选中日期清单 ([DateDetailPanel.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/components/calendar/DateDetailPanel.vue))
- [x] 10.3 实现补打卡逻辑
  - [x] 限制仅过去日期可补 (`isPastDate` 检查)
  - [x] 限制仅应出现日可补 (`shouldHabitAppearOnDate` 检查)
  - [x] 阻止未来日期打卡 (`isFutureDate` 检查返回空列表)
  - [x] 明确提示原因（未来日期显示"还未到该日，无法查看"）

## Phase 4: 系统能力（第3周）

### Task 11: 主题系统

> **PRD功能点**：FR-12（设置）、FR-12.1（外观设置：深色模式+主题颜色）
> **原型参考**：`设置页面.html`、`设置-主题颜色页面.html`
> **状态**：✅ 已完成

- [x] 11.1 实现深色模式
  - [x] 监听prefers-color-scheme ([useThemeStore.ts](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/stores/theme.ts))
  - [x] 动态添加/移除dark类 (applyTheme函数)
  - [x] 三种模式切换（跟随系统/浅色/深色）([DarkModeView.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/views/settings/DarkModeView.vue))
- [x] 11.2 实现主题颜色
  - [x] 10种预设颜色配置 ([useThemeColor.ts](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/composables/useThemeColor.ts))
  - [x] CSS变量动态注入 (--theme-primary)
  - [x] 实时预览组件 ([ThemeColorView.vue](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/habit-tracker/src/views/settings/ThemeColorView.vue))
- [x] 11.3 实现设置页面
  - [x] 创建src/views/SettingsView\.vue
  - [x] 外观设置区域（模式、主题颜色）
  - [x] 深色模式选择页面
  - [x] 主题颜色选择（带实时预览）

### Task 12: 本地通知

- [ ] 12.1 配置Capacitor
  - [x] 初始化Capacitor配置（配置文件+依赖）
  - [ ] 配置iOS/Android权限（需生成 ios/android 工程并 cap sync 后验收）
- [x] 12.2 实现提醒设置
  - [x] 每个习惯最多3个提醒时间
  - [x] 全局免打扰时段
- [x] 12.3 实现通知调度
  - [x] 使用LocalNotifications.schedule
  - [x] 豁免规则检查
  - [x] 免打扰顺延逻辑
- [ ] 12.4 实现点击行为
  - [ ] 点击通知打开应用（需真机/原生工程验证）
  - [x] 跳转到今日清单
- [x] 12.5 单测与构建验证
  - [x] notificationScheduler 单测通过（6/6）
  - [x] npm run build 通过（type-check + vite build）

### Task 13: 统计系统

> **PRD功能点**：FR-9（统计系统）
> **原型参考**：`统计页.html`
> **状态**：✅ 已完成

#### 13.1 统计页面基础架构

- [x] 13.1.1 创建统计页面主组件
  - [x] 创建 `src/views/StatsView.vue`（统计页主容器）
  - [x] 配置路由 `/stats`
  - [x] 实现页面布局（顶部标题栏 + 周期切换 + 内容区 + 底部导航）
  - [x] 添加产品说明入口按钮（跳转产品说明页面）
- [x] 13.1.2 创建周期切换组件
  - [x] 创建 `src/components/stats/PeriodSelector.vue`
  - [x] 实现日/周/月/年四个选项卡
  - [x] 当前选中项高亮显示（bg-primary）
  - [x] 切换时触发事件通知父组件
- [x] 13.1.3 创建日期范围选择器
  - [x] 创建 `src/components/stats/DateRangePicker.vue`
  - [x] 根据当前周期显示对应日期格式：
    - 日视图："3月17日 周日"
    - 周视图："3月11日 - 3月17日"
    - 月视图："2024年3月"
    - 年视图："2024年"
  - [x] 点击展开日期选择弹窗
  - [x] 实现日/周/月/年选择器逻辑

#### 13.2 日视图统计

- [x] 13.2.1 创建日统计组件
  - [x] 创建 `src/components/stats/DayView.vue`
  - [x] 实现统计卡片（今日统计标题 + 进度圆环 + 三个指标）
  - [x] 进度圆环使用 SVG 实现（stroke-dasharray 控制进度）
  - [x] 三个指标：完成总数、打卡总次数、未打卡事件
- [x] 13.2.2 实现今日打卡列表
  - [x] 显示当日所有习惯打卡记录
  - [x] 习惯卡片显示：图标、名称、状态、进度
  - [x] 已完成：显示绿色勾选 + 进度（如 8/8）
  - [x] 进行中：显示进度圆环 + 百分比
  - [x] 临时事项：虚线边框 + 特殊标识

#### 13.3 周视图统计

- [x] 13.3.1 创建周统计组件
  - [x] 创建 `src/components/stats/StatsSummaryCard.vue`
  - [x] 实现周统计指标卡片（打卡事件总数、打卡总次数、完成率）
- [x] 13.3.2 实现趋势图表组件
  - [x] 创建 `src/components/stats/TrendChart.vue`
  - [x] **技术方案**：使用纯 CSS 实现柱状图（不引入 Chart.js）
  - [x] 柱状图结构：flex布局 + 动态高度
  - [x] 数据计算：根据 checkins 按周聚合，计算每天完成率
  - [x] 动画效果：高度变化使用 transition-all duration-300
  - [x] 悬停提示框显示详细数据
- [x] 13.3.3 实现习惯榜单组件
  - [x] 创建 `src/components/stats/HabitRankList.vue`
  - [x] 实现排序切换按钮（次数/完成率/共打）
  - [x] 榜单项显示：图标、名称、完成次数、完成率、共打卡次数
  - [x] 排序逻辑：
    - 次数：periodDoneCount 降序
    - 完成率：periodCompletion 降序
    - 共打：currentStreak 降序

#### 13.4 月视图统计

- [x] 13.4.1 创建月统计组件
  - [x] 复用 `src/components/stats/StatsSummaryCard.vue`
  - [x] 实现月统计指标卡片（打卡事件总数、完成率、最长连续天）
- [x] 13.4.2 实现迷你月历热力图
  - [x] 在 `HabitRankList.vue` 中实现月历展示
  - [x] 7列网格布局（周一至周日）
  - [x] 每个日期格子显示：
    - 已完成：bg-primary + 白色文字
    - 未完成：根据完成率显示渐变颜色
    - 非应出现日：透明背景
  - [x] 根据 habits 和 checkins 计算每日完成状态
- [x] 13.4.3 实现习惯月历列表
  - [x] 每个习惯项下方显示该月的迷你日历
  - [x] 显示习惯名称、打卡天数、完成率
  - [x] 临时事项特殊标识

#### 13.5 年视图统计

- [x] 13.5.1 创建年统计组件
  - [x] 复用 `src/components/stats/StatsSummaryCard.vue`
  - [x] 实现年统计指标卡片（打卡事件总数、完成率、最长连续天）
- [x] 13.5.2 实现年历热力图
  - [x] 在 `HabitRankList.vue` 中实现年历展示
  - [x] 12个月份网格（6列 x 2行）
  - [x] 每个月份显示：
    - 有打卡数据：根据完成率显示渐变颜色
    - 无打卡数据：bg-gray-100
  - [x] 显示习惯名称、全年打卡天数

#### 13.6 统计计算 Composables

- [x] 13.6.1 创建统计计算 composable
  - [x] 创建 `src/composables/useStats.ts`
  - [x] 管理当前周期（day/week/month/year）
  - [x] 管理当前日期
  - [x] 计算周期范围（getPeriodRange）
  - [x] 计算周期完成率（averageCompletion）
  - [x] 计算习惯榜单（habitsStats）
  - [x] 计算趋势图数据（trendData）
  - [x] 使用 computed 缓存计算结果（statsCache Map）

***

### Task 14: 设置系统

> **PRD功能点**：FR-12（设置）、FR-12.1（外观设置）
> **原型参考**：`设置页面.html`、`设置-主题颜色页面.html`、`设置-免打扰时段页面.html`、`设置-产品说明页面.html`、`设置-导出数据页面.html`、`设置-导入数据页面.html`

#### 14.1 设置页面主架构

- [x] 14.1.1 创建设置页面主组件
  - [x] 创建 `src/views/SettingsView.vue`
  - [x] 配置路由 `/settings`
  - [x] 实现分组列表布局（核心功能、提醒与通知、数据管理、外观、关于）
- [x] 14.1.2 创建设置项组件
  - [x] 创建 `src/components/settings/SettingsGroup.vue`（分组容器）
  - [x] 创建 `src/components/settings/SettingsItem.vue`（设置项基础组件）
  - [x] 支持图标、标题、副标题、右侧操作区插槽
  - [x] 支持点击跳转或切换开关两种模式
- [x] 14.1.3 创建开关设置组件
  - [x] 创建 `src/components/settings/ToggleSetting.vue`
  - [x] 使用自定义 CSS 实现 toggle switch（不使用第三方库）
  - [x] 样式：44px x 24px，圆角 24px
  - [x] 开关动画：transform translateX(20px)

#### 14.2 核心功能设置

- [x] 14.2.1 产品说明页面
  - [x] 创建 `src/views/settings/ProductInfoView.vue`
  - [x] 配置路由 `/settings/product-info`
  - [x] 内容包含：
    - 统计与图表口径说明
    - 频率与应出现规则
    - 标签聚合规则
    - 激励与文件箱子
    - 排序机制
    - 名词解释
- [x] 14.2.2 归档习惯库页面
  - [x] 创建 `src/views/settings/ArchivedHabitsView.vue`
  - [x] 配置路由 `/settings/archived-habits`
  - [x] 显示所有已归档习惯列表
  - [x] 支持恢复归档操作

#### 14.3 提醒与通知设置

- [x] 14.3.1 全局提醒开关
  - [x] 在 SettingsView 中添加全局提醒开关项
  - [x] 绑定到 settingsStore.settings.notificationsEnabled
  - [x] 关闭时禁用所有提醒（取消已调度通知；调度器不再生成新通知）
- [x] 14.3.2 免打扰时段设置
  - [x] 创建 `src/views/settings/DoNotDisturbView.vue`
  - [x] 配置路由 `/settings/do-not-disturb`
  - [x] 复用时间选择器组件（TimePickerDrawer）
  - [x] 设置开始时间（默认 22:00）和结束时间（默认 08:00）
  - [x] 保存到 settingsStore.updateDoNotDisturb（持久化到 IndexedDB）

#### 14.4 数据管理设置

- [x] 14.4.1 数据导出功能
  - [x] 创建 `src/views/settings/ExportDataView.vue`
  - [x] 配置路由 `/settings/export`
  - [x] 创建 `src/composables/useDataExport.ts`
  - [x] 导出格式：JSON（包含 habits、checkins、tags、settings、snapshots）
  - [x] 文件名格式：`habit_backup_YYYYMMDD_HHmmss.json`
  - [x] 支持脱敏导出选项（移除 reminders、checkin notes）
  - [x] 导出元数据标记（export\_type: full/sanitized）
- [x] 14.4.2 数据导入功能
  - [x] 创建 `src/views/settings/ImportDataView.vue`
  - [x] 配置路由 `/settings/import`
  - [x] 创建 `src/composables/useDataImport.ts`
  - [x] 文件选择器（支持 .json 文件）
  - [x] 文件格式验证（schema\_version、export\_metadata）
  - [x] 脱敏文件警告提示
  - [x] 导入确认弹窗（显示影响的数据统计）
  - [x] 执行导入并刷新应用状态
- [x] 14.4.3 清空数据功能
  - [x] 创建 `src/views/settings/ClearDataView.vue`
  - [x] 配置路由 `/settings/clear-data`
  - [x] 二次确认弹窗（输入"清空数据"）
  - [x] 执行清空所有数据操作
  - [x] 清空后刷新页面

#### 14.5 外观设置

- [x] 14.5.1 深色模式实现
  - [x] 更新 `src/stores/useThemeStore.ts`
  - [x] 添加 themePreference 状态（'light' | 'dark' | 'system'）
  - [x] 实现 applyTheme() 方法（动态添加/移除 dark 类）
  - [x] 监听 prefers-color-scheme 媒体查询
  - [x] 在 SettingsView 中添加深色模式设置项
  - [x] 创建深色模式选择页面（跟随系统/浅色/深色）
- [x] 14.5.2 主题颜色实现
  - [x] 创建 `src/views/settings/ThemeColorView.vue`
  - [x] 配置路由 `/settings/theme-color`
  - [x] 创建 `src/composables/useThemeColor.ts`
  - [x] 定义 10 种预设颜色：
    ```typescript
    const themeColors = [
      { name: '温暖橙', color: '#D97757', gradient: '...' },
      { name: '天空蓝', color: '#6A9BCC', gradient: '...' },
      { name: '森林绿', color: '#788C5D', gradient: '...' },
      { name: '薰衣草紫', color: '#9B8AA3', gradient: '...' },
      { name: '玫瑰粉', color: '#C49BA8', gradient: '...' },
      { name: '阳光黄', color: '#C9B896', gradient: '...' },
      { name: '薄荷青', color: '#7DA3A8', gradient: '...' },
      { name: '活力橙', color: '#E67E22', gradient: '...' },
      { name: '深紫', color: '#8E44AD', gradient: '...' },
      { name: '深绿', color: '#27AE60', gradient: '...' }
    ]
    ```
  - [x] 使用 CSS 变量动态注入（--theme-primary）
  - [x] 实时预览效果
  - [x] 保存到 settingsStore.themeColor

#### 14.6 关于设置

- [x] 14.6.1 关于应用页面
  - [x] 创建 `src/views/settings/AboutView.vue`
  - [x] 配置路由 `/settings/about`
  - [x] 显示应用名称、版本号、Logo
  - [x] 显示功能特点
- [x] 14.6.2 反馈与支持页面
  - [x] 创建 `src/views/settings/FeedbackView.vue`
  - [x] 配置路由 `/settings/feedback`
  - [x] 反馈表单（问题描述、联系方式）
  - [x] 支持生产环境API提交和本地存储
- [x] 14.6.3 隐私政策页面
  - [x] 创建 `src/views/settings/PrivacyPolicyView.vue`
  - [x] 配置路由 `/settings/privacy`
  - [x] 显示隐私政策内容

#### 14.7 设置数据持久化

- [x] 14.7.1 更新 Settings Store
  - [x] 更新 `src/stores/settings.ts` - 使用 Pinia Composition API 实现
  - [x] 添加状态：themePreference、themeColor、doNotDisturbEnabled、doNotDisturbStart、doNotDisturbEnd、notificationsEnabled
  - [x] 实现 loadSettings() 从 IndexedDB 加载 - 通过 settingsRepository.getSettings()
  - [x] 实现 updateSettings() 保存到 IndexedDB - 自动更新 updatedAt 时间戳
  - [x] 应用启动时自动加载设置 - 在 AppShell.vue 中调用 settingsStore.init()
- [x] 14.7.2 应用设置初始化
  - [x] 在 `src/layouts/AppShell.vue` 中初始化主题
  - [x] 调用 themeStore.init() 传入持久化的 themePreference 和 themeColor
  - [x] 调用 settingsStore.init() 加载设置
  - [x] 监听 settingsStore.settings 变化，设置加载完成后自动应用主题

## Phase 5: 完善（第3-4周）

### Task 14: 空状态与异常处理

> **原型参考**：`标签页-空.html`（空状态参考）

#### 14.1 空状态组件实现

- [x] 14.1.1 创建EmptyState组件
  - [x] 创建 `src/components/common/EmptyState.vue`
  - [x] 支持6种预设类型：habits、tags、stats、snapshots、search、custom
  - [x] 可配置图标、标题、描述、按钮文本
  - [x] 支持紧凑模式（compact prop）
  - [x] 点击按钮触发action事件

#### 14.2 各页面空状态集成

- [x] 14.2.1 今日清单空状态
  - [x] 在 `TodayView.vue` 中集成EmptyState（type="habits"）
  - [x] 当items为空时显示引导创建第一个习惯
  - [x] 点击按钮跳转到新建习惯页面
- [x] 14.2.2 标签页空状态
  - [x] 在 `TagsView.vue` 中集成EmptyState（type="custom"）
  - [x] 当todayTodoItems和otherItems都为空时显示
  - [x] 显示"暂无相关习惯"引导信息
- [x] 14.2.3 照片箱子空状态
  - [x] 在 `MotivationView.vue` 中集成EmptyState（type="snapshots"）
  - [x] 当filteredSnapshots为空时显示
  - [x] 引导用户在"当前图片"标签页保存快照
- [x] 14.2.4 统计页空状态
  - [x] 在 `StatsView.vue` 中集成EmptyState（type="stats"）
  - [x] 当habitsStore.habits.length === 0时显示
  - [x] 引导用户去创建习惯

#### 14.3 错误边界实现

- [x] 14.3.1 创建ErrorBoundary组件
  - [x] 创建 `src/components/common/ErrorBoundary.vue`
  - [x] 使用 `onErrorCaptured` 捕获子组件错误
  - [x] 显示错误图标和友好的错误信息
  - [x] 支持显示详细错误堆栈（开发环境）
  - [x] 提供重试按钮，触发retry事件

#### 14.4 代码质量检查

- [x] 14.4.1 Vue最佳实践合规
  - [x] 使用Composition API + `<script setup lang="ts">`
  - [x] Props使用withDefaults定义默认值
  - [x] 使用computed计算显示内容
  - [x] 组件职责单一，专注空状态展示
- [x] 14.4.2 类型安全
  - [x] 完整的TypeScript类型定义
  - [x] Props接口明确定义
  - [x] Emits事件类型定义

#### 14.5 未实现项（后续迭代）

- [ ] PWA离线页面配置（P1/P2迭代方向）
  - PWA扩展预留：当前为纯Web应用架构，PWA功能作为后续迭代方向
  - 技术参考：vite-plugin-pwa插件、Service Worker缓存策略（Cache First/Network First/Stale While Revalidate）、manifest.json配置、离线指示器组件、PWA安装提示
- [ ] 数据库操作错误全局提示（通过ErrorBoundary组件和Toast通知实现）

### Task 15: 产品说明页面

- [x] 15.1 创建设置子页面
  - [x] 产品说明页面
  - [x] 统计口径说明
  - [x] 频率规则说明
  - [x] 激励规则说明

### Task 16: PWA配置（P1/P2迭代方向）

> 状态：P1/P2迭代方向
> 说明：当前阶段优先保证Web端核心体验，PWA功能作为增强功能在后续迭代中实现

- [ ] 16.1 配置PWA（P1/P2迭代方向）
  - 技术方案参考：
    - 插件：vite-plugin-pwa（简化PWA配置和Service Worker生成）
    - 核心配置：manifest.json（名称、图标、主题色、快捷方式、截图）
    - Service Worker：sw\.js实现多种缓存策略（Cache First/Network First/Stale While Revalidate）
    - 离线页面：index.html作为离线回退，noscript提示
  - 组件设计参考：
    - usePWA composable：Service Worker注册、更新管理、安装提示、在线状态检测
    - OfflineIndicator组件：离线/在线状态提示
    - PWAInstallPrompt组件：引导用户安装到主屏幕
  - 应用图标规格参考：8种尺寸（72x72至512x512），以及快捷方式图标
  - Meta标签参考：theme-color、apple-mobile-web-app-capable等

### Task - 17.1 单元测试

- [ ] 核心算法测试
- [ ] Repository测试
- [ ] 日期工具测试
- [ ] 通知调度器测试
- [ ] Store测试
- [ ] 组件测试
- [ ] 17.2 性能优化
  - [ ] 路由懒加载（除TodayView外所有页面使用动态导入）
  - [ ] 响应式布局（useResponsive composable实现断点检测）
  - [ ] 大列表虚拟滚动（待评估：当前习惯数量通常<50，暂不需要）
  - [ ] 图片懒加载（激励系统图片已按需加载）
- [ ] 17.3 真机测试
  - [ ] iOS测试（需生成Xcode工程后测试）
  - [ ] Android测试（需生成Android工程后测试）
  - [ ] 桌面浏览器测试（Chrome/Edge/Safari功能正常）

测试已基本完成，具体报告见 D:\Trae work\计划.trae\project\documents\develop-habit-tracker-app\test-report.md

#

## Task Dependencies

```
Phase 1:
  Task 1.1 → Task 1.2 → Task 1.3 → Task 1.4
  Task 2.1 → Task 2.2 → Task 2.3 → Task 2.4
  Task 2.4 → Task 3.x
  Task 3.x → Task 4.x

Phase 2:
  Task 4.x → Task 5.x
  Task 4.x → Task 6.x
  Task 5.x → Task 7.x

Phase 3:
  Task 7.x → Task 8.x
  Task 4.x → Task 9.x
  Task 9.x → Task 10.x

Phase 4:
  Task 3.7 → Task 11.x
  Task 2.x → Task 12.x
  Task 4.x → Task 13.x

Phase 5:
  All previous → Task 14.x
  All previous → Task 15.x
  All previous → Task 16.x
  All previous → Task 17.x
```

## 关键里程碑

| 里程碑 | 时间   | 交付物                                |
| --- | ---- | ---------------------------------- |
| M1  | 第1周末 | 可运行的基础项目，数据库和Store就绪               |
| M2  | 第2周末 | 今日清单、习惯管理、排序功能可用                   |
| M3  | 第3周末 | 标签、激励、日历、主题系统完成                    |
| M4  | 第4周末 | 通知、统计完成，Web应用可发布（PWA功能作为P1/P2迭代方向） |

