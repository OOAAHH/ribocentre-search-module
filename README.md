# 🔍 Ribocentre-Aptamer Search Module

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0.0-green.svg)](https://github.com/OOAAHH/ribocentre-search-module)
[![JavaScript](https://img.shields.io/badge/javascript-ES6+-yellow.svg)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/OOAAHH/ribocentre-search-module/pulls)

> 🧬 A powerful, intelligent search module designed specifically for RNA aptamer databases and bioinformatics research platforms.

## 🌟 Features

- **🔍 Smart Search Algorithm**: Advanced relevance scoring with multi-field search capabilities
- **⚡ Real-time Search**: Instant results with <100ms response time
- **🎯 Advanced Filtering**: Filter by year, sequence length, GC content, and more
- **💡 Keyword Highlighting**: Automatic search term highlighting in results
- **📱 Responsive Design**: Perfect adaptation for desktop and mobile devices
- **🔄 Multi-source Support**: Unified search across Jekyll pages and sequence databases
- **🚀 Easy Integration**: 3-line code integration for basic functionality

## 🎬 Live Demo

🌐 **[View Live Demo](https://OOAAHH.github.io/ribocentre-search-module/examples/basic_search_demo.html)**

Try searching for:
- `ATP aptamer`
- `thrombin`
- `SELEX`
- `binding affinity`

## 📦 Installation

### Option 1: Download Release
```bash
# Download the latest release
wget https://github.com/OOAAHH/ribocentre-search-module/releases/latest/download/ribocentre_search_module_v2.0.zip

# Extract files
unzip ribocentre_search_module_v2.0.zip
```

### Option 2: Clone Repository
```bash
git clone https://github.com/OOAAHH/ribocentre-search-module.git
cd ribocentre-search-module
```

### Option 3: CDN (Coming Soon)
```html
<!-- CSS -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/OOAAHH/ribocentre-search-module@2.0.0/css/search.css">

<!-- JavaScript -->
<script src="https://cdn.jsdelivr.net/gh/OOAAHH/ribocentre-search-module@2.0.0/js/search-utils.js"></script>
<script src="https://cdn.jsdelivr.net/gh/OOAAHH/ribocentre-search-module@2.0.0/js/search.js"></script>
```

## 🚀 Quick Start

### Basic Integration (3 lines of code!)

```html
<!DOCTYPE html>
<html>
<head>
    <!-- 1. Include CSS -->
    <link rel="stylesheet" href="css/search.css">
</head>
<body>
    <!-- 2. Add search input -->
    <input type="text" id="mainSearch" placeholder="Search RNA aptamers...">
    
    <!-- 3. Include JavaScript -->
    <script src="js/search-utils.js"></script>
    <script src="js/search.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            SearchModule.init();
        });
    </script>
</body>
</html>
```

### Advanced Search Integration

```html
<!-- Include advanced search module -->
<script src="js/advanced-search.js"></script>
<link rel="stylesheet" href="css/advanced-search.css">

<script>
    // Initialize advanced search
    const advancedSearch = new AdvancedSearchModule();
</script>
```

## 📁 Project Structure

```
ribocentre-search-module/
├── 📄 README.md                          # You are here
├── 📋 search_module_documentation.md     # Complete technical documentation
├── 📁 js/                               # Core JavaScript modules
│   ├── 🔍 search.js                     # Main search module
│   ├── 🛠️ search-utils.js               # Search utilities
│   ├── ⚙️ advanced-search.js            # Advanced search features
│   └── 🏠 homepage-main.js              # Homepage search integration
├── 📁 css/                              # Stylesheets
│   └── 🎨 search.css                    # Search interface styles
├── 📁 pages/                            # Page templates
│   └── 📄 advanced_search.html          # Advanced search page
├── 📁 includes/                         # Reusable components
│   └── 🧩 search-box.html               # Search box component
├── 📁 data/                             # Sample data files
│   ├── 📊 search.json                   # Search index data
│   └── 🧬 sequences_sample.json         # Sequence data sample
├── 📁 docs/                             # Technical documentation
│   ├── 📖 search_functionality.txt      # Detailed functionality guide
│   └── 🔧 SEARCH_FIX_README.txt        # Search fixes documentation
└── 📁 examples/                         # Usage examples
    ├── 🎮 basic_search_demo.html        # Interactive demo
    └── 📋 integration_guide.md          # Integration guide
```

## ⚙️ Configuration

```javascript
// Configure search behavior
window.SEARCH_CONFIG = {
    disableHeroHeightFix: false,  // Disable page height fixing
    searchDelay: 300,             // Search delay in milliseconds
    minSearchLength: 2            // Minimum search character length
};

// Configure data sources
window.DASHBOARD_CONFIG = {
    dataPath: '/api/search.json', // Custom search data path
    baseurl: '/your-app'          // Application base URL
};
```

## 🎯 Use Cases

| Field | Application |
|-------|-------------|
| 🧬 **Bioinformatics** | RNA/DNA sequence search, protein databases |
| 🔬 **Academic Research** | Literature search, paper databases |
| 🏥 **Medical Research** | Drug databases, clinical data search |
| 📚 **Digital Libraries** | Document search, content management |
| 🏛️ **Museums** | Artifact catalogs, collection databases |

## 📊 Performance

- **⚡ Response Time**: <100ms average
- **📈 Scalability**: 1000+ records real-time search
- **💾 Memory**: Optimized memory usage with caching
- **🌐 Compatibility**: All modern browsers (95%+ coverage)

## 🔌 API Reference

### SearchModule Methods

```javascript
// Initialize search module
SearchModule.init();

// Perform programmatic search
SearchModule.performSearchWithQuery('ATP aptamer');

// Get current search results
const results = SearchModule.allSearchResults;

// Close search results
SearchModule.closeSearchResults();
```

### Events

```javascript
// Listen to search events
document.addEventListener('searchStart', (event) => {
    console.log('Search started:', event.detail.query);
});

document.addEventListener('searchComplete', (event) => {
    console.log('Search completed:', event.detail.results);
});
```

## 🧪 Testing

```bash
# Run the demo
open examples/basic_search_demo.html

# Test different scenarios
# 1. Basic search: "ATP"
# 2. Complex search: "RNA aptamer thrombin"
# 3. Advanced filters: Year range, sequence length
```

## 📖 Documentation

- 📋 **[Complete Technical Documentation](search_module_documentation.md)** - Full system architecture
- 🛠️ **[Integration Guide](examples/integration_guide.md)** - Step-by-step integration
- 🎮 **[Live Demo](examples/basic_search_demo.html)** - Interactive demonstration
- 🔧 **[API Reference](docs/)** - Detailed API documentation

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Setup

```bash
# Clone the repository
git clone https://github.com/OOAAHH/ribocentre-search-module.git
cd ribocentre-search-module

# Start development server (if you have one)
# npm start  # or your preferred method

# Make your changes and test
open examples/basic_search_demo.html
```

### Contribution Types

- 🐛 **Bug Reports**: [Report issues](https://github.com/OOAAHH/ribocentre-search-module/issues)
- ✨ **Feature Requests**: [Suggest enhancements](https://github.com/OOAAHH/ribocentre-search-module/issues)
- 📖 **Documentation**: Improve docs and examples
- 🧪 **Testing**: Add test cases and improve coverage
- 🌐 **Translations**: Help translate to other languages

## 🏷️ Versioning

We use [SemVer](http://semver.org/) for versioning. For available versions, see the [tags on this repository](https://github.com/OOAAHH/ribocentre-search-module/tags).

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📞 Support

- 📋 **Documentation**: Check our comprehensive docs first
- 🐛 **Bug Reports**: [Create an issue](https://github.com/OOAAHH/ribocentre-search-module/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/OOAAHH/ribocentre-search-module/discussions)
- 📧 **Email**: contact@ribocentre-aptamer.org

## 🙏 Acknowledgments

- 🧬 **Ribocentre-Aptamer Team** - Original project development
- 🔬 **Academic Community** - Research and feedback
- 💻 **Open Source Community** - Inspiration and best practices

## 📈 Project Stats

![GitHub stars](https://img.shields.io/github/stars/OOAAHH/ribocentre-search-module?style=social)
![GitHub forks](https://img.shields.io/github/forks/OOAAHH/ribocentre-search-module?style=social)
![GitHub issues](https://img.shields.io/github/issues/OOAAHH/ribocentre-search-module)
![GitHub last commit](https://img.shields.io/github/last-commit/OOAAHH/ribocentre-search-module)

## 🔗 Related Projects

- 🧬 **[Ribocentre-Aptamer](https://github.com/Ribocentre-aptamer/Ribocentre-aptamer.github.io)** - Main aptamer database
- 🔍 **[Jekyll Simple Search](https://github.com/christian-fei/Simple-Jekyll-Search)** - Alternative Jekyll search
- 📊 **[Lunr.js](https://github.com/olivernn/lunr.js)** - Client-side full-text search

## 📊 Citation

If you use this search module in your research, please cite:

```bibtex
@software{ribocentre_search_module,
  author = {Ribocentre-Aptamer Team},
  title = {Ribocentre-Aptamer Search Module: Intelligent Search for Bioinformatics Databases},
  year = {2024},
  url = {https://github.com/OOAAHH/ribocentre-search-module},
  version = {2.0.0}
}
```

---

<div align="center">

**⭐ If you find this project useful, please give it a star! ⭐**

Made with ❤️ by the [Ribocentre-Aptamer Team](https://github.com/Ribocentre-aptamer)

</div> 