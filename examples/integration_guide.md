# 搜索功能模块集成指南

## 📖 概述

本指南详细说明如何将 Ribocentre-Aptamer 搜索功能模块集成到你的项目中。

## 🚀 快速集成

### 1. 文件准备

将以下文件复制到你的项目中：

```
your-project/
├── js/
│   ├── search-utils.js      # 必需 - 搜索工具函数
│   ├── search.js           # 必需 - 基础搜索模块  
│   └── advanced-search.js  # 可选 - 高级搜索模块
├── css/
│   └── search.css          # 必需 - 搜索样式
└── data/
    └── search.json         # 必需 - 搜索数据
```

### 2. HTML结构

在你的HTML页面中添加以下结构：

```html
<!DOCTYPE html>
<html>
<head>
    <!-- 引入样式文件 -->
    <link rel="stylesheet" href="css/search.css">
</head>
<body>
    <!-- 搜索框 -->
    <input type="text" id="mainSearch" placeholder="搜索...">
    
    <!-- 搜索结果容器（可选，如果不存在会自动创建） -->
    <div id="searchResultsOverlay" class="search-results-overlay">
        <div id="searchResults" class="search-results-container">
            <div class="search-results-header">
                <span id="searchResultsCount" class="search-results-count"></span>
                <button class="close-search-btn">×</button>
            </div>
            <div id="searchResultsList" class="search-results-list"></div>
        </div>
    </div>

    <!-- 引入JavaScript文件 -->
    <script src="js/search-utils.js"></script>
    <script src="js/search.js"></script>
    
    <!-- 初始化搜索模块 -->
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            SearchModule.init();
        });
    </script>
</body>
</html>
```

## ⚙️ 配置选项

### 基础配置

```javascript
// 在SearchModule.init()之前设置配置
window.SEARCH_CONFIG = {
    disableHeroHeightFix: false,  // 是否禁用页面高度固定
    searchDelay: 300,             // 搜索延迟时间(毫秒)
    minSearchLength: 2            // 最小搜索字符长度
};

SearchModule.init();
```

### 数据源配置

```javascript
// 自定义数据路径
window.DASHBOARD_CONFIG = {
    dataPath: '/api/search.json',  // 自定义搜索数据路径
    baseurl: '/your-app'           // 应用基础路径
};
```

## 🔧 高级集成

### 1. 集成高级搜索

```html
<!-- 高级搜索页面 -->
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="css/search.css">
    <link rel="stylesheet" href="css/advanced-search.css">
</head>
<body>
    <!-- 高级搜索界面 -->
    <div class="advanced-search-container">
        <input type="text" id="mainSearchInput" placeholder="输入搜索关键词...">
        
        <!-- 筛选器 -->
        <div class="filter-section">
            <input type="number" id="yearFrom" placeholder="开始年份">
            <input type="number" id="yearTo" placeholder="结束年份">
            <input type="number" id="lengthFrom" placeholder="最小长度">
            <input type="number" id="lengthTo" placeholder="最大长度">
        </div>
        
        <!-- 结果显示区域 -->
        <div id="resultsList"></div>
    </div>

    <script src="js/search-utils.js"></script>
    <script src="js/advanced-search.js"></script>
    <script>
        // 初始化高级搜索
        const advancedSearch = new AdvancedSearchModule();
    </script>
</body>
</html>
```

### 2. 自定义搜索算法

```javascript
// 重写相关性评分算法
SearchUtils.calculateRelevanceScore = function(item, query) {
    let score = 0;
    const queryLower = query.toLowerCase();
    
    // 自定义评分逻辑
    if (item.title.toLowerCase().includes(queryLower)) {
        score += 100;
    }
    
    // 添加更多评分规则...
    
    return score;
};
```

### 3. 自定义结果渲染

```javascript
// 重写结果渲染函数
SearchModule.renderResults = function() {
    // 自定义渲染逻辑
    const results = this.allSearchResults;
    let html = '';
    
    results.forEach(item => {
        html += `<div class="custom-result-item">
            <h3>${item.title}</h3>
            <p>${item.content}</p>
        </div>`;
    });
    
    this.searchResultsList.innerHTML = html;
};
```

## 📊 数据格式规范

### 搜索索引数据 (search.json)

```json
[
  {
    "title": "页面标题",              // 必需
    "category": "分类",              // 可选
    "tags": "标签1, 标签2",          // 可选
    "url": "/page-url/",            // 必需
    "date": "2024-01-01",           // 可选
    "content": "页面内容摘要"        // 必需
  }
]
```

### 序列数据格式 (sequences_cleaned.json)

```json
{
  "Sheet1": [
    {
      "ID": "序列ID",                // 必需
      "Named": "序列名称",           // 可选
      "Sequence": "ATCG...",        // 可选
      "Ligand": "目标分子",          // 可选
      "Affinity": "Kd: 1 nM",      // 可选
      "Year": "2024",              // 可选
      "Type": "RNA aptamer",       // 可选
      "Category": "蛋白质"          // 可选
    }
  ]
}
```

## 🎨 样式定制

### 1. 基础样式定制

```css
/* 自定义搜索框样式 */
#mainSearch {
    border: 2px solid #your-color;
    border-radius: 25px;
    padding: 12px 20px;
}

/* 自定义搜索结果样式 */
.search-result-item {
    border-left: 3px solid #your-accent-color;
    padding: 15px;
}

/* 自定义高亮样式 */
.keyword-highlight {
    background-color: #your-highlight-color;
    color: #your-text-color;
}
```

### 2. 响应式设计

```css
/* 移动设备适配 */
@media (max-width: 768px) {
    .search-results-container {
        width: 95%;
        margin-top: 10px;
    }
    
    #mainSearch {
        font-size: 16px; /* 避免iOS自动缩放 */
    }
}
```

## 🔌 API接口

### 1. 搜索事件监听

```javascript
// 监听搜索事件
document.addEventListener('searchStart', (event) => {
    console.log('搜索开始:', event.detail.query);
});

document.addEventListener('searchComplete', (event) => {
    console.log('搜索完成:', event.detail.results);
});
```

### 2. 编程式搜索

```javascript
// 主动触发搜索
SearchModule.performSearchWithQuery('ATP aptamer');

// 获取当前搜索结果
const results = SearchModule.allSearchResults;

// 清空搜索结果
SearchModule.closeSearchResults();
```

### 3. 数据源扩展

```javascript
// 添加自定义数据源
SearchUtils.addDataSource('/api/custom-data.json');

// 自定义数据处理
SearchUtils.transformData = function(rawData) {
    return rawData.map(item => ({
        title: item.name,
        content: item.description,
        url: `/item/${item.id}`,
        category: item.type
    }));
};
```

## 🐛 常见问题

### 1. 搜索结果不显示

检查以下项目：
- 确保 `search.json` 文件路径正确
- 检查浏览器控制台是否有错误
- 确认 HTML 元素 ID 是否正确

### 2. 样式显示异常

解决方案：
- 确保 CSS 文件正确引入
- 检查是否有样式冲突
- 验证 CSS 选择器优先级

### 3. 移动设备兼容性

注意事项：
- 使用 `viewport` 标签
- 设置合适的触摸响应区域
- 避免使用 hover 效果

## 📈 性能优化

### 1. 数据懒加载

```javascript
// 延迟加载搜索数据
const lazyLoadSearchData = async () => {
    if (!window.searchDataLoaded) {
        await SearchUtils.fetchData(['search.json']);
        window.searchDataLoaded = true;
    }
};
```

### 2. 搜索结果缓存

```javascript
// 启用搜索结果缓存
SearchModule.enableCache = true;
SearchModule.cacheTimeout = 5 * 60 * 1000; // 5分钟
```

### 3. 防抖优化

```javascript
// 调整防抖延迟
window.SEARCH_CONFIG = {
    searchDelay: 500  // 增加延迟减少请求频率
};
```

## 🔒 安全考虑

### 1. XSS防护

```javascript
// 确保输出内容经过转义
const escapeHtml = (text) => {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
};
```

### 2. 输入验证

```javascript
// 验证搜索输入
const validateSearchInput = (query) => {
    // 长度限制
    if (query.length > 100) return false;
    
    // 特殊字符过滤
    const allowedChars = /^[a-zA-Z0-9\s\-_.()]+$/;
    return allowedChars.test(query);
};
```

## 📞 技术支持

如果在集成过程中遇到问题：

1. 查阅完整技术文档
2. 检查演示示例
3. 联系开发团队获取支持

---

*本指南涵盖了搜索功能模块的主要集成方法，更多高级用法请参考技术文档。* 