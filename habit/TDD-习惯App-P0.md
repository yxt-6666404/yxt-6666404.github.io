# 习惯 App（P0）技术设计文档（TDD）

源材料（保留不改）：[define-habit-app-p0-improved/spec.md](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/.trae/specs/define-habit-app-p0-improved/spec.md)  
对应 PRD：[PRD-渐成画迹-P0.md](file:///d:/Trae%20work/%E8%AE%A1%E5%88%92/.trae/project/PRD-%E6%B8%90%E6%88%90%E7%94%BB%E8%BF%B9-P0.md)

## 1. 范围与目标

### 1.1 本文负责
- 说明 P0 的整体架构、模块边界、关键规则算法、边界情况与降级策略。
- 说明与存储实现相关但不属于 PRD 的设计决策（Repository 抽象、排序持久化、提醒调度策略等）。

### 1.2 本文不负责
- UI 视觉稿与动效细节（归 PRD 的 UI/UX 原型与设计系统）。
- 可执行的细粒度测试用例（归 TestPlan）。
- 数据库表/索引的完整定义（归 DBD）。

## 2. 总体架构

### 2.1 分层与模块
- UI 层：页面与组件（今日清单、习惯、标签、统计、激励、设置）。
- 状态层：按域拆分的 Store（Habits/Checkins/OneOff/Tags/Settings/Snapshots/Theme）。
- 领域层：纯函数规则与用例（shouldAppear、统计口径、连续性、权重冻结、排序合并等）。
- 数据访问层：Repository（屏蔽 IndexedDB/Dexie 细节，为未来 Supabase/Postgres 迁移预留接口）。
- 平台能力层：提醒（Capacitor 本地通知），Web 环境适配策略，主题管理。
  - **PWA扩展预留**：当前为纯Web应用架构，PWA功能（Service Worker、离线缓存、manifest等）可作为后续迭代方向
- 交互层：长按操作处理、拖拽排序、搜索功能等用户交互逻辑。

### 2.2 关键数据流（高层）
- 今日清单：SelectedDate → 计算应出现集合 + 读取当日 OneOff → 合并排序视图 → 渲染 → 操作写入（checkin/oneOff）→ 触发派生计算（完成度/统计/激励）。
- 标签页：TagId + LocalToday → 计算“今日待办置顶区”与“其余区” → 双区各自合并排序 → 渲染与操作。
- 统计页：PeriodRange → 读取相关 checkins/习惯配置 → 规则重算（频率变更/暂停/旧规则打卡）→ 聚合指标与榜单。
- 激励：监听“权重获得事件” → 更新全局/图片权重 → 生成可视化阶段与显现比例 → 快照写入。

## 3. 核心规则与算法

### 3.1 日期口径与“本地日历天”
- 所有“今天/昨日/某日”的业务口径以设备本地日历天为准，日切为 00:00。
- 存储层统一使用 YYYY-MM-DD（本地日历）作为 date 字段表达，避免跨时区/跨 DST 的 UTC 归日偏移。

### 3.2 shouldAppear（应出现）判定
- 判定输入：habit 配置（enabled/paused/archived/completed/startDate/frequency）+ date（YYYY-MM-DD）。
- 优先级：paused 为最高优先级；paused=true 直接不应出现。
- 必要条件：enabled=true 且 archived=false 且 paused=false 且 completed=false 且 date >= startDate。
- 频率匹配：
  - 每日：恒为 true
  - 每周：date 的 weekday 在选中集合中
  - 每月：date 的 day-of-month 在选中集合中；当月不存在该日期则不出现
  - 自定义每 N 天：按 startDate 起算，dayDiff % N == 0 且 dayDiff >= 0（以本地 00:00 归一化后的天数差计算）

### 3.3 频率变更后的历史口径重算
- 原则：统计/分数等“口径型指标”按最新规则重算；历史记录本身保留。
- 需要被重算影响的点：
  - 统计分母（应出现日集合）
  - 连续性（按应出现日序列）
  - Top 榜的 periodDoneCount/periodCompletion
- 旧规则打卡（频率变更后变为“非应出现日打卡”）的处理：
  - 数据保留，可在历史回顾中展示
  - 不计入：统计、连续性、Top 榜
  - 可在视图层打上 legacy 标识用于灰显

### 3.4 打卡、多次目标、完成度与权重冻结
- doneCount：当日完成次数，允许超额。
- dayCompletion：用于展示与统计的完成度，封顶 1。
  - 公式：dayCompletion = min(1, doneCount / dailyTargetCount)
- 权重（weightEarned）：每项目每天最多一次权重；一旦获得永久不回退。
- 目标变更后的处理：
  - dayCompletion 可以按最新 dailyTargetCount 重算，以保持展示一致
  - weightEarned 不回溯修改，防止降低目标后倒灌权重

#### 权重冻结策略实现
- Checkin 记录包含 weightEarned 布尔字段
- 判定时点：基于用户打卡当日（而非当前）的 dailyTargetCount 配置
- 一旦 weightEarned=true，永不回退，即使后续目标变更

### 3.5 连续性（streak）算法
- 连续性以"应出现日"为序列基准计算。
- paused 期间跳过且不造成断裂；允许暂停前后拼接。
- 非应出现日不计入序列，不造成断裂（等价"缺席"）。

#### 连续性算法伪代码
```javascript
function calculateStreak(checkins, habit, startDate, endDate) {
  let currentStreak = 0;
  let maxStreak = 0;
  let date = startDate;
  
  while (date <= endDate) {
    if (habit.paused) {
      date = nextDay(date);
      continue;
    }
    
    if (shouldAppear(habit, date)) {
      const checkin = checkins.find(c => c.date === date && c.doneCount > 0);
      if (checkin) {
        currentStreak++;
        maxStreak = Math.max(maxStreak, currentStreak);
      } else {
        currentStreak = 0;
      }
    }
    date = nextDay(date);
  }
  
  return { currentStreak, maxStreak };
}
```

### 3.6 排序系统（手动 + 规则，共存且隔离）

#### 3.6.1 视图隔离
- 今日清单：按日期隔离（viewKey: today_YYYY-MM-DD）。
- 标签页：按标签与分区隔离（viewKey: tag_<tagId>_todo / tag_<tagId>_others）。
- 其他列表（习惯管理、统计榜单）各自独立，不共享顺序。

### 3.7 搜索功能实现
- 搜索范围：习惯名称、标签名称、备注内容
- 搜索算法：使用模糊匹配，对搜索关键词进行小写处理后与目标文本比较
- 搜索结果：按匹配度排序，显示匹配的习惯和临时事项
- 搜索结果统计：显示匹配结果数量，帮助用户快速了解搜索范围
- 性能优化：对搜索结果进行缓存，避免重复计算

#### 3.6.2 手动排序持久化模型
- 存储位置：Settings 内 viewSortOrders（结构见 DBD）。
- 单视图排序条目：{ id, type, sortOrder }。
- sortOrder 采用整数并使用步长 10 的默认分配策略，便于插入而不全量重排。

#### 3.6.3 列表合并策略（关键边界）
- 新项首次进入某视图的"应出现集合"时：默认追加到末尾。
- 项离开视图集合（暂停/归档/删除/频率不匹配/日期变化）时：从该 viewKey 的排序数组中物理移除。
- 临时事项过期（非当日）时：从对应日期 viewKey 中移除。
- 临时事项与固定习惯的排序权重相同，均可参与手动拖拽排序，排序状态与固定习惯一样持久化存储。

#### 3.6.4 规则排序与手动排序共存
- 默认展示使用手动排序顺序。
- 开启规则排序时，仅改变当前会话展示顺序，不覆盖持久化顺序数据。
- 退出规则排序应立即恢复手动顺序；若无手动顺序记录，则按默认规则初始化（例如创建时间）。

#### 3.6.5 写入与容错
- 拖拽结束即写入（可做短防抖）。
- 写入失败时，允许静默重试并在必要时提示用户"顺序未保存"。
- 保证不会因为单次失败导致顺序数据损坏（写入前后校验结构、避免空对象覆盖）。

### 3.7 拼豆激励系统

#### 3.7.1 权重获取规则
- 来源：任意习惯或临时事项在某日完成当日目标（dayCompletion=1）
- 每天每个项目最多获得1权重（超额完成不额外增加）
- 权重基于打卡当日目标判定，一旦获得永不回退

#### 3.7.2 阶段计算
- 阶段0：权重0
- 阶段1：权重1-3
- 阶段2：权重4-6
- 阶段3：权重7-9
- 阶段4：权重10-12
- 阶段5：权重13-15
- 显现比例：revealRatio = min(1, currentWeight / 15)
- 清晰度：clarity = 0.4 + 0.6 * min(1, currentWeight / 15)

#### 3.7.3 图片与类型解锁
- 图片解锁：
  - 用户在激励页面选择要解锁的图片
  - 同类型内，前一张图片达到阶段5后自动解锁下一张
  - 类型定义：
    - 类型按图片编号分组，如第一类、第二类等
    - 每个类型包含多个基础图片（如1、2、3等）及其对应的高清版本（如1_1、2_1、3_1等）
- 类型解锁阈值：
  - 第一类：默认解锁
  - 后续类型：globalTotalWeight 达到对应阈值后解锁（第二类≥15，第三类≥30，以此类推，每类递增15）
- 解锁新类型前需在当前类型完成至少2张图片

#### 3.7.4 快照数据流
- **自动保存提示**：当 currentPictureWeight 达到12（阶段4）或15（阶段5）时触发保存提示
- **手动保存**：任何阶段（0-5）用户都可主动点击"保存当前进度"按钮保存快照
- **截图时机**：基于Canvas当前渲染状态生成Base64（质量0.8，单张<100KB）
- **数据关联**：快照记录关联 pictureId，记录当前阶段（0-5）和权重值（0-15）
- **删除习惯影响**：无影响，快照独立存在

## 4. 主题管理系统

### 4.1 深色模式实现方案

#### 4.1.1 Tailwind CSS 配置
- 启用 Tailwind CSS 的 dark 模式：
  ```javascript
  // tailwind.config.js
  module.exports = {
    darkMode: 'class',  // 使用class策略，手动控制
    // ...
  }
  ```

#### 4.1.2 深色模式切换逻辑
- 在 `<html>` 或 `<body>` 标签上动态添加/移除 `dark` 类
- 三种模式处理：
  - `light`: 强制移除 `dark` 类
  - `dark`: 强制添加 `dark` 类
  - `system`: 监听 `prefers-color-scheme` 媒体查询，自动同步系统主题

#### 4.1.3 系统主题监听
```javascript
// 监听系统主题变化
const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
mediaQuery.addEventListener('change', (e) => {
  if (themePreference === 'system') {
    applyTheme(e.matches ? 'dark' : 'light');
  }
});
```

### 4.2 主题颜色实现方案

#### 4.2.1 CSS 变量方案
- 使用 CSS 自定义属性（CSS Variables）定义主题颜色
- 在 `:root` 中定义默认颜色，通过 JavaScript 动态修改
```css
:root {
  --theme-primary: #D97757;
}
.dark {
  --theme-primary: #D97757;  /* 深色模式下可调整饱和度 */
}
```

#### 4.2.2 动态颜色应用
- 主题颜色变化时，更新 CSS 变量值
- 所有使用主题颜色的组件通过 `var(--theme-primary)` 引用
- 实现实时预览：预览区域使用相同的 CSS 变量

#### 4.2.3 颜色配置映射
```typescript
const themeColors = {
  warmOrange: '#D97757',  // 默认
  blue: '#6A9BCC',
  green: '#788C5D',
  purple: '#9B8AA3',
  pink: '#C49BA8',
  yellow: '#C9B896',
  cyan: '#7DA3A8',
  orange: '#E67E22',
  deepPurple: '#8E44AD',
  deepGreen: '#27AE60'
};
```

### 4.3 持久化与初始化

#### 4.3.1 存储策略
- 使用 IndexedDB 的 Settings store 存储外观设置
- 键值：`themePreference` 和 `themeColor`
- 默认值为 `themePreference: 'system'`, `themeColor: '#D97757'`

#### 4.3.2 应用启动流程
1. 从 IndexedDB 读取外观设置
2. 应用深色模式（根据 preference 计算实际模式）
3. 应用主题颜色（更新 CSS 变量）
4. 若 preference 为 'system'，启动系统主题监听

#### 4.3.3 实时同步
- 设置页面修改后立即更新预览
- 点击"保存"后持久化到 IndexedDB
- 全应用实时响应（通过 CSS 变量机制）

## 5. 提醒系统（Capacitor 本地通知）

### 4.1 能力边界与降级
- P0：移动端以 Capacitor 壳内本地通知为验收基准。
- Web/桌面环境：允许降级为应用内提示（Badge/红点/列表提醒），不强制系统级推送一致性。
- **PWA扩展预留**：如后续支持PWA，可添加Service Worker实现离线状态下的本地通知缓存和同步

### 4.2 配置与规则
- 每个习惯最多 3 个提醒时间点。
- 权限：首次开启提醒时请求；拒绝后提供引导到系统设置的入口。
- 豁免：
  - 当日已完成目标（dayCompletion=1）：不提醒
  - paused/archived/completed 状态：不提醒
- 免打扰：落入免打扰时段的提醒将顺延到免打扰结束后立即发送。

### 4.3 提醒内容格式
- 内容包含习惯名称与今日完成进度
- 示例："喝水 - 已完成2/8杯"

### 4.4 权限配置
- iOS：需在 Info.plist 配置 NSUserNotificationUsageDescription
- Android 12+：需 SCHEDULE_EXACT_ALARM 权限

### 4.5 点击行为
- 点击通知打开应用并跳转到今日清单

## 5. 统计系统设计

### 5.1 统计口径
- 日期口径：按本地日历天
- 周口径：周一为周起始，周日为周结束
- 月/年口径：自然月/自然年

### 5.2 核心指标
- dayCompletion = min(1, doneCount / dailyTargetCount)
- periodCompletion：周期内所有"应出现日"的 dayCompletion 取平均；若周期内应出现日为 0，则记为 null
- periodDoneCount：周期内所有日期 doneCount 求和
- 连续性：周期内最长连续打卡天数序列长度

### 5.3 Top 榜
- 默认展示 Top 10
- 排序规则：按 periodDoneCount 从高到低；若相同则按 periodCompletion 从高到低，再按名称排序

## 6. 跨平台与性能设计要点

### 6.1 跨平台打包策略（P0）
- **响应式自适应**：采用真正的响应式自适应布局，不使用手机设备框架模拟
  - 原型中的手机框架仅用于预览移动端显示效果
  - 移动端（< 768px）：单栏垂直布局，内容宽度100%，底部Tab导航
  - 桌面端（≥ 768px）：单栏垂直布局随屏幕宽度扩展，内容区最大宽度1200px居中
  - 所有功能在任意宽度下完整一致，无功能差异
- **单代码库**：Web应用为主分发，Capacitor 为系统能力补丁
- **布局适配**：使用 Tailwind CSS 响应式断点实现布局自适应
- **PWA扩展预留（未来迭代）**：
  - **技术方案**：可集成vite-plugin-pwa插件实现Service Worker、manifest.json、离线缓存
  - **缓存策略参考**：Cache First（静态资源）、Network First（API数据）、Stale While Revalidate（混合策略）
  - **PWA功能清单**：离线页面、后台同步、推送通知、添加到主屏幕提示
  - **当前阶段**：优先保证Web端核心体验，PWA功能作为后续迭代方向

### 6.2 性能基线（P0）
- 冷启动与路由切换：避免大计算阻塞主线程；统计计算按需触发。
- 列表渲染：大列表（100+）保持流畅；必要时使用虚拟列表或分页策略（后续迭代）。
- 激励/图片资源：懒加载与缓存；快照展示避免一次性解码全部大图。

## 7. 关键边界情况清单
- 时区变化：按本地日历天口径，不因 UTC 偏移导致"打卡归日变化"。
- 频率变更：历史口径重算导致完成率跳变，需要在产品说明中可解释且在 UI 上可感知差异。
- 删除习惯：主列表不再出现，但历史记录与快照仍可识别（名称快照 + 已删除标记）。
- 超额完成：doneCount 可超额，但展示完成度封顶；权重每天最多一次。
- 排序稳定性：视图隔离，避免跨页面/跨日期互相污染。

## 8. 与未来云端（Supabase/Postgres）的兼容约束（P0 仅预留）
- 领域模型字段类型与命名尽量稳定，为未来同步做准备。
- Repository 接口保持存储无关，测试尽量基于接口契约而非具体实现。
