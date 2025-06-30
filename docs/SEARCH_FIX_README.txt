# 搜索框布局闪烁问题修复说明

## 问题描述
主页搜索框在输入时会导致 hero section 的高度动态变化，每次输入都会触发重新布局，导致页面出现闪烁效果。

## 修复方案

### 1. CSS 样式修复
在 `css/homepage.css` 中添加了以下样式：
- `.hero-section.search-active` - 固定 hero section 高度
- `.overview-container.search-active` - 固定 overview 容器高度
- 新增的搜索结果样式优化

### 2. JavaScript 优化
**主要修复位置**: `js/homepage-main.js` 中的 `SearchModule`
- **防抖机制**：增加了智能防抖延迟，减少频繁搜索触发
- **最小搜索长度**：设置最少2个字符才开始搜索
- **配置化管理**：通过 `window.SEARCH_CONFIG` 进行配置
- **高度锁定功能**：在搜索时固定 hero section 和 overview container 的高度

### 3. 配置选项

在 `index.html` 中可以通过修改 `window.SEARCH_CONFIG` 来调整搜索行为：

```javascript
window.SEARCH_CONFIG = {
    disableHeroHeightFix: false, // 设置为 true 可以完全禁用hero section高度调整
    searchDelay: 300,            // 搜索延迟时间（毫秒）
    minSearchLength: 2           // 最小搜索字符长度
};
```

## 快速禁用修复

如果您想要完全禁用 hero section 的高度调整功能，只需要将配置中的 `disableHeroHeightFix` 设置为 `true`：

```javascript
window.SEARCH_CONFIG = {
    disableHeroHeightFix: true,  // 禁用hero section高度调整
    searchDelay: 300,
    minSearchLength: 2
};
```

## 修复效果

1. **消除闪烁**：搜索时页面布局保持稳定
2. **改善性能**：减少不必要的重新布局计算
3. **更好的用户体验**：搜索过程更加流畅
4. **可配置性**：可以根据需要启用或禁用修复功能

## 技术细节

- 使用 CSS 变量 `--hero-fixed-height` 和 `--overview-fixed-height` 来动态设置固定高度
- `SearchModule` 在搜索开始时计算并锁定当前高度
- 搜索结束时恢复原始布局行为
- 支持响应式设计，窗口大小变化时会重新计算高度
- 添加了 ESC 键快捷关闭和点击外部区域关闭功能

## 修复文件

1. **`css/homepage.css`** - CSS 样式修复
2. **`js/homepage-main.js`** - SearchModule 功能修复
3. **`index.html`** - 配置选项和移除重复代码

## 注意事项

1. 修复功能默认启用，如不需要可通过配置禁用
2. 修复适用于所有屏幕尺寸和设备
3. 不影响其他页面功能和现有样式
4. 所有搜索功能现在统一由 `SearchModule` 管理，避免了代码重复 