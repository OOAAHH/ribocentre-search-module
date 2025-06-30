# Contributing to Ribocentre-Aptamer Search Module

Thank you for your interest in contributing to the Ribocentre-Aptamer Search Module! 🎉

## 🚀 How to Contribute

### 🐛 Reporting Bugs

Before submitting a bug report:
1. **Check existing issues** to avoid duplicates
2. **Use the latest version** to ensure the bug hasn't been fixed
3. **Provide clear reproduction steps**

When filing a bug report, please include:
- **Environment details** (browser, OS, version)
- **Steps to reproduce** the issue
- **Expected vs actual behavior**
- **Screenshots or code snippets** if helpful

### ✨ Suggesting Features

We welcome feature suggestions! Please:
1. **Check existing feature requests** first
2. **Describe the problem** your feature would solve
3. **Provide implementation ideas** if you have them
4. **Consider backwards compatibility**

### 🛠️ Development Setup

```bash
# 1. Fork the repository on GitHub

# 2. Clone your fork
git clone https://github.com/yourusername/ribocentre-search-module.git
cd ribocentre-search-module

# 3. Create a feature branch
git checkout -b feature/your-feature-name

# 4. Make your changes
# Edit files as needed

# 5. Test your changes
open examples/basic_search_demo.html
# Test various search scenarios

# 6. Commit your changes
git add .
git commit -m "Add: your descriptive commit message"

# 7. Push to your fork
git push origin feature/your-feature-name

# 8. Create a Pull Request
```

### 📝 Code Style Guidelines

#### JavaScript
- Use **ES6+ features** where appropriate
- Follow **camelCase** naming convention
- Add **JSDoc comments** for functions
- Keep functions **small and focused**
- Use **meaningful variable names**

```javascript
/**
 * Calculate relevance score for search results
 * @param {Object} item - Search result item
 * @param {string} query - Search query
 * @returns {number} Relevance score
 */
function calculateRelevanceScore(item, query) {
    // Implementation here
}
```

#### CSS
- Use **kebab-case** for class names
- Follow **BEM methodology** where applicable
- Include **browser prefixes** for new features
- **Group related properties** together

```css
/* Good */
.search-result-item {
    padding: 15px;
    border-radius: 8px;
    transition: background-color 0.3s ease;
}

.search-result-item:hover {
    background-color: #f8f9fa;
}
```

#### HTML
- Use **semantic HTML5** elements
- Include **ARIA labels** for accessibility
- Follow **proper nesting** structure
- Use **meaningful IDs and classes**

### 🧪 Testing Guidelines

Before submitting a PR, please test:

1. **Basic Search Functionality**
   ```
   - Type "ATP" → Should show ATP-related results
   - Type "aptamer" → Should show aptamer results
   - Clear search → Should hide results
   ```

2. **Advanced Search Features**
   ```
   - Apply year filter → Should filter results
   - Apply length filter → Should work correctly
   - Combine filters → Should work together
   ```

3. **Responsive Design**
   ```
   - Test on desktop (1920x1080)
   - Test on tablet (768px width)
   - Test on mobile (320px width)
   ```

4. **Browser Compatibility**
   ```
   - Chrome latest
   - Firefox latest  
   - Safari latest
   - Edge latest
   ```

### 📚 Documentation

When contributing:
- **Update README.md** if adding new features
- **Add code comments** for complex logic
- **Update API documentation** if changing interfaces
- **Include usage examples** for new features

### 🔍 Pull Request Process

1. **Create a descriptive title**
   ```
   Good: "Add: Advanced filtering for sequence length"
   Bad: "Fix stuff"
   ```

2. **Fill out the PR template**
   - Describe what your PR does
   - Reference any related issues
   - Include screenshots for UI changes

3. **Keep PRs focused**
   - One feature/fix per PR
   - Avoid mixing unrelated changes

4. **Respond to feedback**
   - Address review comments promptly
   - Be open to suggestions

### 🎯 Areas We Need Help With

- 🐛 **Bug fixes** - Help us squash bugs
- 📱 **Mobile optimization** - Improve mobile experience  
- 🌐 **Internationalization** - Add multi-language support
- 📖 **Documentation** - Improve guides and examples
- 🧪 **Testing** - Add automated tests
- ♿ **Accessibility** - Enhance accessibility features
- 🎨 **UI/UX** - Design improvements

### 🤝 Code of Conduct

This project follows a simple code of conduct:

- **Be respectful** and inclusive
- **Be patient** with newcomers
- **Focus on the issue**, not the person
- **Give constructive feedback**
- **Help others learn and grow**

### 📞 Getting Help

Need help? We're here for you!

- 💬 **GitHub Discussions** - Ask questions and get help
- 🐛 **Issues** - Report bugs or request features  
- 📧 **Email** - contact@ribocentre-aptamer.org

### 🎉 Recognition

Contributors will be:
- **Listed in our README** acknowledgments
- **Added to the contributors page** (if we create one)
- **Mentioned in release notes** for significant contributions

## 📝 Commit Message Guidelines

Use clear, descriptive commit messages:

```
Add: New feature
Fix: Bug fix
Update: Existing feature modification
Remove: Remove feature/code
Docs: Documentation changes
Style: Code style changes
Refactor: Code refactoring
Test: Add or modify tests
```

Examples:
```
Add: Advanced search filtering by GC content
Fix: Search results not highlighting on mobile
Update: Improve search performance for large datasets
Docs: Add API documentation for SearchUtils
```

---

Thank you for contributing to the Ribocentre-Aptamer Search Module! Your contributions help make bioinformatics research more accessible to everyone. 🧬✨ 