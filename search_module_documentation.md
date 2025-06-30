# Ribocentre-Aptamer 搜索功能模块完整文档

## 📋 项目概述

本文档详细介绍了 Ribocentre-Aptamer 网站的搜索功能模块实现。该搜索系统专为RNA适配体数据库设计，支持多种搜索方式，包括全文搜索、高级筛选和智能关键词匹配。

## 🏗️ 系统架构

### 搜索模块组成

```
搜索功能模块
├── SearchModule (js/search.js)                 # 全站通用搜索
├── homepageSearchModule (js/homepage-main.js)  # 主页搜索
├── AdvancedSearchModule (js/advanced-search.js) # 高级搜索
├── SearchUtils (js/search-utils.js)           # 搜索工具函数
├── 样式文件 (css/search.css)                  # 搜索界面样式
├── 数据文件 (search.json)                     # 搜索索引数据
└── 高级搜索页面 (_pages/advanced_search.html) # 高级搜索界面
```

### 核心特性

1. **多数据源支持**: 
   - Jekyll生成的页面内容 (search.json)
   - 序列数据库 (sequences_cleaned.json)
   - 统一的搜索接口

2. **智能搜索算法**:
   - 相关性评分机制
   - 关键词高亮显示
   - 防抖输入处理
   - 搜索结果缓存

3. **多种搜索方式**:
   - 即时搜索 (实时显示结果)
   - 高级筛选 (按年份、序列长度、GC含量等)
   - 分类搜索 (按适配体类型、目标分子等)

4. **用户界面优化**:
   - 响应式设计
   - 搜索结果分页
   - 多种视图模式 (列表/网格/表格)
   - 键盘快捷键支持

## 🔧 核心代码实现

### 1. 统一搜索工具 (SearchUtils)

```javascript
const SearchUtils = {
  // 数据获取
  async fetchData(paths) {
    for (const path of paths) {
      try {
        const response = await fetch(path);
        if (response.ok) {
          const text = await response.text();
          return JSON.parse(text);
        }
      } catch (e) {
        console.warn('Failed to load search data from', path, e);
      }
    }
    throw new Error('All search data paths failed');
  },

  // 搜索结果处理
  processResults(data, query) {
    const queryLower = query.toLowerCase();
    const terms = queryLower.split(/\s+/).filter(t => t.length > 0);
    const results = [];
    
    data.forEach(item => {
      const titleLower = (item.title || '').toLowerCase();
      const categoryLower = (item.category || '').toLowerCase();
      const tagsLower = this.sanitizeMeta(item.tags).toLowerCase();
      const contentLower = this.sanitizeMeta(item.content).toLowerCase();
      
      let matched = false;
      if (titleLower.includes(queryLower) ||
          categoryLower.includes(queryLower) ||
          tagsLower.includes(queryLower) ||
          contentLower.includes(queryLower)) {
        matched = true;
      } else {
        for (const term of terms) {
          if (titleLower.includes(term) ||
              categoryLower.includes(term) ||
              tagsLower.includes(term) ||
              contentLower.includes(term)) {
            matched = true;
            break;
          }
        }
      }
      
      if (matched) {
        item.relevanceScore = this.calculateRelevanceScore(item, query);
        results.push(item);
      }
    });
    
    results.sort((a, b) => b.relevanceScore - a.relevanceScore);
    return results;
  },

  // 相关性评分算法
  calculateRelevanceScore(item, query) {
    let score = 0;
    const terms = query.toLowerCase().split(/\s+/).filter(Boolean);
    const fullQuery = query.toLowerCase();

    const title = (item.title || '').toLowerCase();
    const category = (item.category || '').toLowerCase();
    const tags = this.sanitizeMeta(item.tags).toLowerCase();
    const content = this.sanitizeMeta(item.content).toLowerCase();
    
    // 完整匹配得分更高
    if (title.includes(fullQuery)) score += 200;
    if (content.includes(fullQuery)) score += 100;
    if (category.includes(fullQuery)) score += 80;
    if (tags.includes(fullQuery)) score += 90;
    
    // 单词匹配得分
    terms.forEach(term => {
      if (title === term) score += 100;
      else if (title.startsWith(term)) score += 80;
      else if (title.includes(term)) score += 60;
      if (category.includes(term)) score += 30;
      if (tags.includes(term)) score += 40;
      if (content.includes(term)) score += Math.min((content.match(new RegExp(term,'gi'))||[]).length*3,30);
    });
    
    return score;
  },

  // 关键词高亮
  highlightKeywords(text, query) {
    if (!text) return '';
    const terms = query.toLowerCase().split(/\s+/).filter(t => t.length > 1);
    let result = text;
    terms.forEach(term => {
      const regex = new RegExp(`(${term})`, 'gi');
      result = result.replace(regex, '<span class="keyword-highlight">$1</span>');
    });
    return result;
  }
};
```

### 2. 全站搜索模块 (SearchModule)

```javascript
const SearchModule = {
  mainSearchInput: null,
  searchResultsOverlay: null,
  searchResults: null,
  allSearchResults: [],
  searchTimeout: null,

  init() {
    this.mainSearchInput = document.getElementById('mainSearch');
    this.setupSearchContainer();
    this.bindEvents();
    this.bindWindowEvents();
  },

  async performSearch() {
    const query = this.mainSearchInput.value.trim();
    if (!query) return;

    this.showSearchResults();
    this.searchResultsList.innerHTML = '<div class="search-loading">搜索中...</div>';

    try {
      const data = await SearchUtils.fetchData(['./search.json', '/search.json']);
      this.allSearchResults = SearchUtils.processResults(data, query);
      this.renderResults();
    } catch (error) {
      console.error('搜索过程出错:', error);
      this.searchResultsList.innerHTML = '<div class="search-error">搜索失败，请稍后重试</div>';
    }
  },

  renderResults() {
    if (this.allSearchResults.length === 0) {
      this.searchResultsList.innerHTML = '<div class="search-no-results">未找到相关结果</div>';
      return;
    }

    let html = '';
    const query = this.mainSearchInput.value.trim();

    this.allSearchResults.forEach((item, idx) => {
      const highlightedTitle = SearchUtils.highlightKeywords(item.title || 'Untitled', query);
      
      html += `<div class="search-result-item" data-url="${item.url || '#'}">
        <div class="result-header">
          <span class="result-index">${idx + 1}</span>
          <h3 class="result-title">${highlightedTitle}</h3>
        </div>
        <div class="result-description">${SearchUtils.getContentPreview(item.content || '', query)}</div>
      </div>`;
    });

    this.searchResultsList.innerHTML = html;
  }
};
```

### 3. 高级搜索模块 (AdvancedSearchModule)

```javascript
class AdvancedSearchModule {
  constructor() {
    this.searchData = [];
    this.filteredData = [];
    this.currentPage = 1;
    this.currentFilters = {};
    this.currentView = 'list';
    this.init();
  }

  async loadData() {
    // 多路径尝试加载数据
    const potentialPaths = [
      `${window.location.origin}/search.json`,
      './search.json',
      '/search.json'
    ];

    for (const path of potentialPaths) {
      try {
        const response = await fetch(path);
        if (response.ok) {
          this.searchData = await response.json();
          break;
        }
      } catch (error) {
        continue;
      }
    }

    // 增强数据处理
    this.searchData = this.searchData.map(item => {
      if (item.tags) {
        const tagMap = this.parseTagsToMap(item.tags);
        item.structured_tags = tagMap;
        item.sequence_length = tagMap['Length'] ? parseInt(tagMap['Length']) : undefined;
        item.gc_content = tagMap['GC'] ? parseFloat(tagMap['GC']) : undefined;
        item.pub_year = tagMap['Year'] ? parseInt(tagMap['Year']) : undefined;
      }
      return item;
    });
  }

  async performSearch(query) {
    const startTime = performance.now();
    
    let results = this.searchData.filter(item => {
      const searchFields = [
        item.title || '',
        item.content || '',
        item.tags || '',
        item.category || ''
      ].join(' ').toLowerCase();

      return searchFields.includes(query.toLowerCase());
    });

    // 应用高级筛选
    results = this.applyCurrentFilters(results);
    
    const endTime = performance.now();
    const searchTime = ((endTime - startTime) / 1000).toFixed(3);

    this.filteredData = results;
    this.showResults(results, searchTime);
  }

  applyCurrentFilters(data) {
    let filtered = [...data];

    // 年份筛选
    const yearFrom = document.getElementById('yearFrom')?.value;
    const yearTo = document.getElementById('yearTo')?.value;
    if (yearFrom || yearTo) {
      filtered = filtered.filter(item => {
        const year = item.pub_year || 0;
        return (!yearFrom || year >= parseInt(yearFrom)) && 
               (!yearTo || year <= parseInt(yearTo));
      });
    }

    // 序列长度筛选
    const lengthFrom = document.getElementById('lengthFrom')?.value;
    const lengthTo = document.getElementById('lengthTo')?.value;
    if (lengthFrom || lengthTo) {
      filtered = filtered.filter(item => {
        const length = item.sequence_length || 0;
        return (!lengthFrom || length >= parseInt(lengthFrom)) && 
               (!lengthTo || length <= parseInt(lengthTo));
      });
    }

    // GC含量筛选
    const gcFrom = document.getElementById('gcFrom')?.value;
    const gcTo = document.getElementById('gcTo')?.value;
    if (gcFrom || gcTo) {
      filtered = filtered.filter(item => {
        const gc = item.gc_content || 0;
        return (!gcFrom || gc >= parseFloat(gcFrom)) && 
               (!gcTo || gc <= parseFloat(gcTo));
      });
    }

    return filtered;
  }
}
```

## 📊 数据结构

### 搜索索引数据格式 (search.json)

```json
[
  {
    "title": "ATP aptamer",
    "category": "aptamer",
    "tags": "Type:RNA aptamer, Length:27, GC:63.0, Year:1995",
    "url": "/posts/ATP-aptamer/",
    "date": "1995-01-01",
    "content": "This aptamer binds to ATP with high specificity..."
  }
]
```

### 序列数据格式 (sequences_cleaned.json)

```json
{
  "Sheet1": [
    {
      "ID": "1_aptamer",
      "Named": "ATP aptamer clone 1",
      "Sequence": "ACCTGGGGGAGTATTGCGGAGGAAGG",
      "Ligand": "ATP",
      "Affinity": "Kd: 6 μM",
      "Year": "1995",
      "Type": "RNA aptamer",
      "Category": "Nucleotides"
    }
  ]
}
```

## 🎨 界面设计

### CSS 样式系统

```css
/* 搜索结果覆盖层 */
.search-results-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 10000;
  background: rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(2px);
  transition: opacity 0.3s ease;
}

/* 搜索结果容器 */
.search-results-container {
  position: absolute;
  width: 90%;
  max-width: 600px;
  left: 50%;
  transform: translateX(-50%);
  background: white;
  border-radius: 15px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.2);
  max-height: 500px;
  overflow: hidden;
}

/* 搜索结果项 */
.search-result-item {
  padding: 15px 20px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
  transition: all 0.2s ease;
}

.search-result-item:hover {
  background: #f8f9fa;
}

/* 关键词高亮 */
.keyword-highlight {
  background-color: #ffeaa7;
  font-weight: bold;
  padding: 0 2px;
}
```

## 🚀 使用指南

### 基础搜索

1. **即时搜索**: 在搜索框输入关键词，系统自动显示匹配结果
2. **回车搜索**: 按Enter键执行搜索
3. **关键词高亮**: 搜索词在结果中会被高亮显示

### 高级搜索

1. **访问高级搜索页面**: `/advanced_search.html`
2. **筛选选项**:
   - 发表年份范围
   - 序列长度范围  
   - GC含量范围
   - 适配体类型
   - 目标分子类别

3. **结果视图**:
   - 列表视图: 简洁的搜索结果列表
   - 网格视图: 卡片式布局
   - 表格视图: 详细的数据表格

### 搜索优化建议

1. **关键词选择**: 使用具体的科学术语，如"thrombin aptamer"、"ATP binding"
2. **组合搜索**: 使用多个关键词提高搜索精度
3. **筛选功能**: 利用高级筛选缩小结果范围

## ⚙️ 技术特性

### 性能优化

1. **防抖处理**: 避免频繁的搜索请求
2. **结果缓存**: 缓存搜索结果提高响应速度  
3. **分页加载**: 大量结果分页显示
4. **懒加载**: 按需加载搜索数据

### 兼容性

- 支持现代浏览器 (Chrome, Firefox, Safari, Edge)
- 响应式设计，适配桌面和移动设备
- 渐进式增强，JavaScript失效时仍可基本使用

### 可扩展性

- 模块化设计，易于添加新的搜索功能
- 数据源可配置，支持多种数据格式
- 搜索算法可定制，支持不同的评分机制

## 📁 文件清单

### 核心文件
- `js/search.js` - 全站搜索模块
- `js/search-utils.js` - 搜索工具函数
- `js/advanced-search.js` - 高级搜索模块
- `js/homepage-main.js` - 主页搜索模块
- `css/search.css` - 搜索样式文件

### 数据文件
- `search.json` - 主要搜索索引
- `apidata/sequences_cleaned.json` - 序列数据

### 页面文件
- `_pages/advanced_search.html` - 高级搜索页面
- `_includes/search-box.html` - 搜索框组件

### 文档文件
- `doc/search_functionality.txt` - 详细技术文档
- `doc/SEARCH_FIX_README.txt` - 搜索修复说明

## 🔄 更新日志

### v2.0 (当前版本)
- 统一了多个搜索模块的接口
- 新增高级筛选功能
- 改进了相关性评分算法
- 优化了用户界面交互

### v1.5
- 添加了序列数据搜索支持
- 实现了搜索结果缓存
- 改进了关键词高亮显示

### v1.0
- 基础搜索功能实现
- Jekyll内容索引支持
- 响应式界面设计

## 📞 技术支持

如需了解更多技术细节或有任何问题，请参考项目文档或联系开发团队。

---
*本文档详细介绍了 Ribocentre-Aptamer 项目的搜索功能模块，展示了完整的技术实现和使用方法。* 