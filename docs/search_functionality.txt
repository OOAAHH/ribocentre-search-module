# Ribocentre-Aptamer 搜索功能实现文档

本文档详细说明了 Ribocentre-Aptamer 网站中搜索功能的实现逻辑，包括 header 中的搜索功能和主页的 homepageSearch 功能。
后续计划进行减法，现在两者的行为不同。

## 1. 搜索功能架构概述

Ribocentre-Aptamer 网站的搜索功能采用了模块化设计，主要包含三个核心模块，并共享一套工具函数：

1. **SearchModule (js/search.js)** - 全站通用的搜索功能模块
2. **homepageSearchModule (js/homepage-main.js)** - 主页特定的搜索功能模块
3. **AdvancedSearchModule (js/advanced-search.js)** - 高级搜索页面的专用搜索模块
4. **SearchUtils (js/search-utils.js)** - 提供数据加载、关键词高亮等通用工具

这些模块可以根据不同页面的需求独立运行或协同工作。系统会根据当前页面类型自动选择合适的搜索模块。
所有模块的搜索逻辑均依赖 `SearchUtils.processResults`，从而确保同一关键词在不同位置得到一致的搜索结果。

## 2. 文件依赖关系

### 2.1 Header 搜索功能相关文件

Header 中的搜索功能主要依赖以下文件：

| 文件路径 | 作用 |
|---------|------|
| `js/search.js` | 实现全站通用的搜索功能逻辑 |
| `js/search-utils.js` | 公共搜索工具函数 |
| `css/search.css` | 提供搜索功能的样式定义 |
| `js/simple-jekyll-search.min.js` | 提供基于 Jekyll 的搜索引擎功能 |
| `search.json` | 存储网站内容的搜索索引数据 |

### 2.2 Homepage 搜索功能相关文件

主页的搜索功能主要依赖以下文件：

| 文件路径 | 作用 |
|---------|------|
| `js/homepage-main.js` | 包含 homepageSearchModule 实现 |
| `js/search-utils.js` | 公共搜索工具函数 |
| `css/search.css` | 提供搜索功能的样式定义 |
| `css/homepage.css` | 包含主页特定的搜索样式 |
| `js/homepage-data.js` | 提供主页搜索需要的数据 |
| `apidata/sequences_cleaned.json` | 提供适配体序列数据用于搜索 |

### 2.3 Advanced Search 功能相关文件

高级搜索页面主要依赖以下文件：

| 文件路径 | 作用 |
|---------|------|
| `js/advanced-search.js` | 包含 AdvancedSearchModule 类实现 |
| `js/search-utils.js` | 公共搜索工具函数 |
| `css/advanced-search.css` | 提供高级搜索页面的样式定义 |
| `advanced_search.html` | 高级搜索页面的HTML结构 |
| `search.json` | 同样用作高级搜索的数据源 |

## 3. 搜索功能实现详解

### 3.1 Header 搜索功能 (SearchModule)

#### 3.1.1 初始化流程

```js
// 在 js/search.js 中
const SearchModule = {
    init() {
        // 获取DOM元素
        this.mainSearchInput = document.getElementById('mainSearch');
        this.searchResultsOverlay = document.getElementById('searchResultsOverlay');
        this.searchResults = document.getElementById('searchResults');
        this.searchResultsList = document.getElementById('searchResultsList');
        this.searchResultsCount = document.getElementById('searchResultsCount');
        
        // 检查页面类型，避免与主页搜索冲突
        const isIndexPage = window.location.pathname === '/' || 
                            window.location.pathname.endsWith('index.html');
        const homepageModuleActive = typeof homepageSearchModule !== 'undefined' && 
                                    homepageSearchModule.mainSearchInput;
        
        // 在特定条件下跳过初始化
        if (isIndexPage && !isFluorescencePage && homepageModuleActive) {
            console.log('SearchModule: 在index页面检测到homepageSearchModule，跳过初始化');
            return;
        }
        
        // 如果页面中没有搜索结果容器，创建一个
        if (!this.searchResultsOverlay || !this.searchResults || !this.searchResultsList || !this.searchResultsCount) {
            this.setupSearchContainer();
        }
        
        // 绑定事件
        this.bindEvents();
        this.bindWindowEvents();
    }
}
```

#### 3.1.2 搜索处理流程

1. **输入监听**：监听搜索框的输入事件，使用防抖处理
2. **搜索执行**：满足条件时执行 `performSearch()` 方法
3. **结果处理**：处理搜索结果并显示
4. **结果展示**：在搜索结果覆盖层中显示结果

```js
// 搜索执行函数
async performSearch() {
    const query = this.mainSearchInput.value.trim();
    if (query.length === 0) return;
    
    // 显示加载状态
    this.searchResultsList.innerHTML = '<div class="search-loading">搜索中...</div>';
    this.searchResultsCount.textContent = '搜索中...';
    this.showSearchResults();
    
    const startTime = performance.now();
    
    try {
        // 获取搜索数据
        const response = await fetch('/search.json');
        if (!response.ok) throw new Error('搜索数据加载失败');
        const data = await response.json();
        
        // 处理搜索结果
        this.allSearchResults = this.processSearchResults(data, query);
        
        // 计算搜索耗时
        const searchTime = ((performance.now() - startTime) / 1000).toFixed(3);
        
        // 渲染搜索结果
        this.renderResults(searchTime);
    } catch (error) {
        console.error('搜索过程出错:', error);
        this.searchResultsList.innerHTML = '<div class="search-error">搜索失败，请稍后重试</div>';
        this.searchResultsCount.textContent = '搜索失败';
    }
}
```

### 3.2 Homepage 搜索功能 (homepageSearchModule)

#### 3.2.1 初始化流程

```js
// 在 js/homepage-main.js 中
const homepageSearchModule = {
    init() {
        // 检查是否在index.html页面，以及是否已有SearchModule模块在运行
        const isIndexPage = window.location.pathname === '/' || 
                           window.location.pathname.endsWith('index.html');
        const searchModuleActive = typeof SearchModule !== 'undefined' && 
                                  SearchModule.mainSearchInput;
        
        // 在index页面，并且SearchModule已激活，则不初始化本模块的搜索功能
        if (isIndexPage && searchModuleActive) {
            console.log('homepageSearchModule: 检测到SearchModule已存在，跳过搜索功能初始化');
            return;
        }
        
        this.mainSearchInput = document.getElementById('mainSearch');
        if (!this.mainSearchInput) return;

        this.setupSearchContainer();
        this.bindEvents();
        this.bindWindowEvents();
    }
}
```

#### 3.2.2 搜索处理流程

1. **输入监听**：监听搜索框的输入事件，使用防抖处理
2. **Hero区域高度固定**：当搜索激活时，固定hero区域高度，避免布局抖动
3. **数据获取**：从配置的数据路径获取搜索数据
4. **结果处理**：处理搜索结果，计算相关性得分
5. **结果展示**：在搜索结果容器中显示结果

```js
// 主页搜索执行函数
async performSearch() {
    const query = this.mainSearchInput.value.trim();
    if (query.length === 0) {
        this.mainSearchResults.style.display = 'none';
        return;
    }
    
    // 显示加载状态
    this.mainSearchResults.innerHTML = '<div style="padding: 20px; text-align: center;">搜索中...</div>';
    this.mainSearchResults.style.display = 'block';
    
    try {
        // 获取数据路径
        const config = window.DASHBOARD_CONFIG || { dataPath: '/apidata/sequences_cleaned.json' };
        const dataPath = config.dataPath;
        
        // 获取搜索数据
        const response = await fetch(dataPath);
        if (!response.ok) throw new Error(`Failed to load data from ${dataPath}`);
        const data = await response.json();
        
        // 处理搜索结果
        this.allSearchResults = this.processSearchResults(data, query);
        
        // 渲染搜索结果
        this.renderResults();
    } catch (error) {
        console.error('搜索过程出错:', error);
        this.mainSearchResults.innerHTML = '<div style="padding: 20px; text-align: center; color: red;">搜索失败，请稍后重试</div>';
    }
}
```

### 3.3 Advanced Search 功能 (AdvancedSearchModule)

#### 3.3.1 初始化流程

高级搜索页面使用面向对象的方式实现，通过 `AdvancedSearchModule` 类管理所有功能，并在基础搜索阶段同样调用 `SearchUtils.processResults`，与其他模块保持一致：

```js
// 在 js/advanced-search.js 中
class AdvancedSearchModule {
    constructor() {
        this.searchData = [];
        this.filteredData = [];
        this.currentPage = 1;
        this.itemsPerPageList = 5;
        this.itemsPerPageTable = 20;
        this.itemsPerPage = this.itemsPerPageList; // 默认list
        this.searchTimeout = null;
        this.currentFilters = {};
        this.currentSearchType = 'all';
        this.currentView = 'list';
        this.masonryLoaded = false;
        this.masonryPage = 1;
        this.masonryPageSize = 30;
        this.masonryData = [];
        
        this.init();
    }

    async init() {
        try {
            await this.loadData();
            this.setupEventListeners();
            this.setupFilters();
            console.log('Advanced Search Module initialized');
        } catch (error) {
            console.error('Failed to initialize advanced search:', error);
        }
    }
    
    // 其他方法...
}

// 初始化搜索模块
let advancedSearch;
document.addEventListener('DOMContentLoaded', () => {
    advancedSearch = new AdvancedSearchModule();
});
```

#### 3.3.2 数据加载流程

高级搜索模块的数据加载逻辑更为健壮，包含了多路径尝试和异常处理：

```js
async loadData() {
    try {
        // 尝试多个可能的路径加载数据
        const potentialPaths = [
            `${window.location.origin}${window.DASHBOARD_CONFIG?.baseurl || ''}/search.json`,
            './search.json',
            '../search.json',
            '/search.json',
            'search.json'
        ].filter(Boolean);

        let response = null;
        let successfulPath = null;

        // 逐个尝试直到成功
        for (const path of potentialPaths) {
            try {
                console.log(`Attempting to load data from: ${path}`);
                response = await fetch(path);
                if (response.ok) {
                    successfulPath = path;
                    break;
                }
            } catch (error) {
                console.warn(`Failed to load from ${path}:`, error.message);
                continue;
            }
        }

        if (!response || !response.ok) {
            throw new Error(`Failed to load data from any path. Last status: ${response?.status}`);
        }

        const text = await response.text();
        
        // 尝试解析JSON，处理可能的Jekyll模板问题
        try {
            this.searchData = JSON.parse(text);
        } catch (e) {
            // 尝试清理文本，去除模板标记
            const jsonStart = text.indexOf('[');
            if (jsonStart !== -1) {
                const cleanedText = text.substring(jsonStart);
                this.searchData = JSON.parse(cleanedText);
            } else {
                throw e;
            }
        }
        
        console.log(`Successfully loaded ${this.searchData.length} records from: ${successfulPath}`);
    } catch (error) {
        console.error('Error loading search data:', error);
        console.log('Using mock data as fallback');
        // 使用模拟数据作为备选
        this.searchData = this.generateMockData();
    }
}
```

#### 3.3.3 搜索处理流程

高级搜索提供了更丰富的搜索功能，包括：

1. **多字段搜索**：在多个字段中查找匹配
2. **类型过滤**：根据适配体类型过滤
3. **高级筛选**：提供多种参数筛选结果
4. **结果排序**：支持多种排序方式
5. **多种视图**：支持列表、网格和表格三种视图

```js
async performSearch(query) {
    if (!query) {
        this.showPlaceholder();
        return;
    }

    this.showLoadingState();

    // 模拟搜索延迟
    await new Promise(resolve => setTimeout(resolve, 500));

    const startTime = performance.now();
    
    // 在加载的数据上执行搜索
    let results = this.searchData.filter(item => {
        const searchFields = [
            item.title || '',
            item.content || '',
            item.tags || '',
            item.category || '',
            item.target || ''
        ].join(' ').toLowerCase();

        return searchFields.includes(query.toLowerCase());
    });

    // 应用类型过滤
    if (this.currentSearchType !== 'all') {
        results = this.filterByType(results, this.currentSearchType);
    }

    // 应用其他过滤条件
    results = this.applyCurrentFilters(results);

    const endTime = performance.now();
    const searchTime = ((endTime - startTime) / 1000).toFixed(3);

    this.filteredData = results;
    this.currentPage = 1;
    
    this.showResults(results, searchTime);
}
```

#### 3.3.4 高级过滤功能

高级搜索的一个关键特性是提供丰富的过滤选项：

```js
applyCurrentFilters(data) {
    let filtered = [...data];

    // 年份过滤
    const yearFrom = document.getElementById('yearFrom')?.value;
    const yearTo = document.getElementById('yearTo')?.value;
    if (yearFrom || yearTo) {
        filtered = filtered.filter(item => {
            const year = item.date ? parseInt(item.date.split('-')[0]) : 0;
            return (!yearFrom || year >= parseInt(yearFrom)) && 
                  (!yearTo || year <= parseInt(yearTo));
        });
    }

    // 序列长度过滤
    const lengthFrom = document.getElementById('lengthFrom')?.value;
    const lengthTo = document.getElementById('lengthTo')?.value;
    if (lengthFrom || lengthTo) {
        filtered = filtered.filter(item => {
            const length = item.sequence_length || 0;
            return (!lengthFrom || length >= parseInt(lengthFrom)) && 
                  (!lengthTo || length <= parseInt(lengthTo));
        });
    }

    // GC含量过滤
    const gcFrom = document.getElementById('gcFrom')?.value;
    const gcTo = document.getElementById('gcTo')?.value;
    if (gcFrom || gcTo) {
        filtered = filtered.filter(item => {
            const gc = item.gc_content || 0;
            return (!gcFrom || gc >= parseFloat(gcFrom)) && 
                  (!gcTo || gc <= parseFloat(gcTo));
        });
    }

    // 目标类型过滤
    const targetType = document.getElementById('targetType')?.value;
    if (targetType) {
        filtered = filtered.filter(item => 
            item.tags && item.tags.toLowerCase().includes(targetType.toLowerCase())
        );
    }

    // 应用领域过滤
    const applicationField = document.getElementById('applicationField')?.value;
    if (applicationField) {
        filtered = filtered.filter(item => 
            item.content && item.content.toLowerCase().includes(applicationField.toLowerCase())
        );
    }

    // 亲和力过滤
    const affinityRange = document.getElementById('affinityRange')?.value;
    if (affinityRange) {
        filtered = filtered.filter(item => {
            const kd = parseFloat(item.kd_value) || 0;
            switch (affinityRange) {
                case 'pM': return kd < 1;
                case 'nM': return kd >= 1 && kd < 1000;
                case 'μM': return kd >= 1000 && kd < 1000000;
                case 'mM': return kd >= 1000000;
                default: return true;
            }
        });
    }

    return filtered;
}
```

#### 3.3.5 多视图渲染

高级搜索支持三种不同的结果视图：列表、网格和表格：

```js
renderResults() {
    const resultsList = document.getElementById('resultsList');
    if (!resultsList) return;

    let start, end, pageData;
    switch (this.currentView) {
        case 'grid':
            // 瀑布流无限加载
            if (!this.masonryLoaded) {
                this.masonryLoaded = true;
                this.masonryPage = 1;
                this.masonryPageSize = 30;
                this.masonryData = this.filteredData.slice(0, this.masonryPage * this.masonryPageSize);
                this.renderGridView(resultsList, this.masonryData);
                window.addEventListener('scroll', this.handleMasonryScroll.bind(this));
            } else {
                this.renderGridView(resultsList, this.masonryData);
            }
            // 隐藏分页，显示进度条
            const paginationContainer = document.getElementById('paginationContainer');
            if (paginationContainer) paginationContainer.style.display = 'none';
            this.showMasonryProgressBar();
            break;
        case 'table':
            window.removeEventListener('scroll', this.handleMasonryScroll);
            this.hideMasonryProgressBar();
            this.itemsPerPage = this.itemsPerPageTable;
            start = (this.currentPage - 1) * this.itemsPerPage;
            end = start + this.itemsPerPage;
            pageData = this.filteredData.slice(start, end);
            this.renderTableView(resultsList, pageData);
            this.setupPagination();
            break;
        case 'list':
        default:
            window.removeEventListener('scroll', this.handleMasonryScroll);
            this.hideMasonryProgressBar();
            this.itemsPerPage = this.itemsPerPageList;
            start = (this.currentPage - 1) * this.itemsPerPage;
            end = start + this.itemsPerPage;
            pageData = this.filteredData.slice(start, end);
            this.renderListView(resultsList, pageData);
            this.setupPagination();
            break;
    }
}
```

## 4. 关键功能实现

### 4.1 搜索相关性计算

```js
calculateRelevanceScore(item, query) {
    let score = 0;
    const queryLower = query.toLowerCase();
    
    // 标题匹配权重最高
    if (item.title) {
        const titleLower = item.title.toLowerCase();
        if (titleLower === queryLower) score += 100;
        else if (titleLower.startsWith(queryLower)) score += 80;
        else if (titleLower.includes(queryLower)) score += 60;
    }
    
    // 内容匹配权重其次
    if (item.content) {
        const contentLower = item.content.toLowerCase();
        if (contentLower.includes(queryLower)) {
            // 根据匹配位置计算权重，越靠前权重越高
            const pos = contentLower.indexOf(queryLower);
            const posWeight = Math.max(0, 40 - Math.floor(pos / 20));
            score += posWeight;
        }
    }
    
    // 类别匹配权重最低
    if (item.category && item.category.toLowerCase().includes(queryLower)) {
        score += 20;
    }
    
    return score;
}
```

### 4.2 搜索结果高亮显示

```js
highlightKeywords(text, query) {
    if (!text) return '';
    const pattern = new RegExp(`(${query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi');
    return text.replace(pattern, '<span class="keyword-highlight">$1</span>');
}
```

### 4.3 Hero区域高度固定

为了防止搜索结果显示时导致页面布局跳动，代码会在搜索激活时固定hero区域高度：

```js
fixHeroHeight() {
    const heroSection = document.querySelector('.hero-section');
    const overviewContainer = document.querySelector('.overview-container');
    
    if (heroSection && !heroSection.classList.contains('search-active')) {
        // 保存原始高度
        this.heroOriginalHeight = heroSection.getBoundingClientRect().height;
        
        // 强制禁用transition并固定高度
        heroSection.style.transition = 'none';
        heroSection.style.height = this.heroOriginalHeight + 'px';
        heroSection.style.minHeight = this.heroOriginalHeight + 'px';
        heroSection.style.maxHeight = this.heroOriginalHeight + 'px';
        heroSection.style.setProperty('--hero-fixed-height', this.heroOriginalHeight + 'px');
        heroSection.classList.add('search-active');
    }
    
    if (overviewContainer && !overviewContainer.classList.contains('search-active')) {
        // 类似处理overview容器
        // ...
    }
}
```

### 4.4 高级搜索的滚动加载

高级搜索的网格视图使用滚动加载（类似无限滚动）来实现更好的用户体验：

```js
handleMasonryScroll() {
    if (this.currentView !== 'grid') return;
    
    // 计算滚动进度 - 考虑页面总高度和视窗位置
    const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;
    const scrollProgress = scrollTop / (documentHeight - windowHeight);
    
    // 基于滚动位置的智能加载
    if ((windowHeight + scrollTop) >= (documentHeight - 200)) {
        // 快到底部时加载更多
        this.masonryPage++;
        const moreData = this.filteredData.slice(0, this.masonryPage * this.masonryPageSize);
        if (moreData.length > this.masonryData.length) {
            this.masonryData = moreData;
            const resultsList = document.getElementById('resultsList');
            this.renderGridView(resultsList, this.masonryData);
        }
    }
    
    // 实时更新进度条
    this.updateMasonryProgressBar(scrollProgress);
}
```

### 4.5 高级搜索的UI交互增强

高级搜索页面包含多种UI交互增强，如滚动进度条、筛选面板动画等：

```js
updateMasonryProgressBar(scrollProgress = 0) {
    const bar = document.getElementById('masonryProgressBar');
    const inner = document.getElementById('masonryProgressInner');
    if (!bar || !inner) return;
    
    const total = this.filteredData.length;
    const loaded = this.masonryData.length;
    
    // 多维度进度计算
    const dataLoadProgress = total > 0 ? (loaded / total) : 0;
    const scrollWeight = Math.min(scrollProgress * 1.2, 1); // 滚动权重，略微放大
    
    // 综合进度：70% 基于数据加载，30% 基于滚动位置
    const combinedProgress = (dataLoadProgress * 0.7) + (scrollWeight * 0.3);
    const finalProgress = Math.min(combinedProgress, 1);
    
    // 动态设置进度条样式
    inner.style.height = Math.max(8, Math.round((320 - 4) * finalProgress)) + 'px';
    
    // 根据进度调整进度条颜色渐变
    const hue = 240 + (finalProgress * 60);
    const saturation = 60 + (finalProgress * 30);
    const lightness = 50 + (finalProgress * 20);
    
    inner.style.background = `linear-gradient(180deg, 
        hsl(${hue}, ${saturation}%, ${lightness}%) 0%, 
        hsl(${hue - 20}, ${saturation + 10}%, ${lightness - 15}%) 100%)`;
    
    bar.style.display = 'flex';
}
```

## 5. 模块协同工作方式

### 5.1 模块优先级判断

网站根据当前页面类型和已初始化的模块来决定使用哪个搜索模块：

1. **在主页上**：
   - 如果 `homepageSearchModule` 已初始化，则使用该模块
   - 如果 `homepageSearchModule` 未初始化，则使用 `SearchModule`

2. **在其他页面上**：
   - 始终使用 `SearchModule`

### 5.2 模块互操作性

这两个模块可以在特定情况下互相调用对方的方法：

```js
// 例如，在homepageSearchModule中可以调用SearchModule的方法
if (typeof SearchModule !== 'undefined' && SearchModule.closeSearchResults) {
    SearchModule.closeSearchResults();
}

// 在SearchModule中可以检测homepageSearchModule的存在
if (typeof homepageSearchModule !== 'undefined' && homepageSearchModule.mainSearchInput) {
    // ...相应处理
}
```

## 6. CSS样式实现

搜索功能的样式主要定义在 `css/search.css` 中，包括：

1. **搜索输入框样式**：`.homepage-search-box`, `.homepage-search-input`
2. **搜索结果容器样式**：`.search-results-container`, `.search-results-overlay`
3. **搜索结果项样式**：`.search-result-item`, `.search-result-title`
4. **关键字高亮样式**：`.keyword-highlight`
5. **响应式设计**：针对不同屏幕尺寸的样式调整

## 7. 搜索配置选项

网站提供了全局配置对象来控制搜索行为：

```js
// 在index.html中定义的全局配置
window.SEARCH_CONFIG = {
    disableHeroHeightFix: true, // 是否禁用hero section高度调整
    searchDelay: 300,           // 搜索延迟时间（毫秒）
    minSearchLength: 2          // 最小搜索字符长度
};

window.DASHBOARD_CONFIG = {
    baseurl: '{{ site.baseurl }}',
    dataPath: '{{ site.baseurl }}/apidata/sequences_cleaned.json' // 搜索数据路径
};
```

## 8. 常见问题与解决方案

### 8.1 搜索模块冲突

**问题**：在某些页面上，`SearchModule` 和 `homepageSearchModule` 可能同时初始化，导致冲突。

**解决方案**：
- 每个模块在初始化前检查另一个模块是否已经初始化
- 基于页面类型和特定条件决定哪个模块应该运行
- 在某些特定页面上强制使用某一个模块

### 8.2 搜索结果布局抖动

**问题**：搜索结果显示时可能导致页面布局抖动，特别是在主页的hero区域。

**解决方案**：
- 实现 `fixHeroHeight()` 方法，在搜索激活时固定hero区域高度
- 提供配置选项 `disableHeroHeightFix` 来控制这一行为
- 使用CSS过渡效果使布局变化更平滑

### 8.3 高级搜索的数据加载问题

**问题**：高级搜索可能在不同环境下无法找到正确的数据路径。

**解决方案**：
- 使用多路径尝试策略，依次尝试不同可能的路径
- 提供备用的模拟数据，在数据加载失败时使用
- 记录成功的路径，便于后续调试

## 9. 搜索功能比较

三种搜索实现的比较：

| 特性 | SearchModule | homepageSearchModule | AdvancedSearchModule |
|-----|-------------|---------------------|---------------------|
| 数据源 | search.json | sequences_cleaned.json | search.json (多路径尝试) |
| 实现方式 | 对象模式 | 对象模式 | 类模式 |
| 结果展示 | 覆盖层 | 内联容器 | 专用页面 |
| 高级筛选 | 无 | 无 | 丰富的筛选选项 |
| 结果视图 | 单一列表 | 单一列表 | 列表/网格/表格 |
| 分页方式 | 无分页 | 无分页 | 传统分页+无限滚动 |
| UI交互 | 基础 | 基础 | 丰富的动画和过渡 |
| 数据导出 | 无 | 无 | 支持CSV导出 |

## 10. 搜索功能扩展建议

1. **增加筛选选项**：允许用户按类别、日期等条件筛选搜索结果
2. **添加高级搜索**：实现更复杂的搜索语法，如布尔操作符
3. **搜索历史记录**：保存用户最近的搜索查询
4. **搜索建议**：在用户输入时提供自动完成建议
5. **搜索结果分页**：对大量搜索结果实现分页显示
6. **统一搜索实现**：将三种实现合并为一个统一、可配置的搜索模块
7. **添加搜索分析**：记录用户搜索行为，改进搜索体验

## 11. 总结

Ribocentre-Aptamer 网站的搜索功能采用了模块化设计，通过 `SearchModule`、`homepageSearchModule` 和 `AdvancedSearchModule` 三个核心模块实现了全站、主页和高级搜索功能。这种设计允许不同页面根据需求使用不同的搜索实现，同时保持了用户体验的一致性。

搜索功能的关键特性包括：
- 防抖处理的实时搜索
- 相关性评分的结果排序
- 关键字高亮显示
- 响应式设计适应不同设备
- 布局固定避免页面抖动
- 高级筛选和多视图展示（高级搜索）
- 无限滚动和进度指示（高级搜索）

通过这些实现，网站为用户提供了高效、直观的搜索体验，帮助用户快速找到所需的适配体信息。 