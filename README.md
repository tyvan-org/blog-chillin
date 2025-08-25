# Blog Chillin 🚀

A modern, feature-rich personal blog built with [Hugo](https://gohugo.io/) and the [DoIt](https://github.com/HEIGE-PCloud/DoIt) theme. This blog focuses on providing an excellent reading experience with support for various content types, mathematical formulas, and modern web features.

## ✨ Features

### 🎨 Theme & Design
- **DoIt Theme v0.4.X** - Clean, modern, and responsive design
- **Multiple Theme Modes** - Light, dark, black, and auto-switching
- **TypeIt Animation** - Animated subtitle typing effect
- **Responsive Layout** - Optimized for all devices

### 📝 Content Features
- **Markdown Support** - Full Markdown with extended syntax
- **Mathematical Formulas** - KaTeX support for LaTeX math expressions
- **Code Highlighting** - Syntax highlighting with line numbers
- **Table of Contents** - Auto-generated TOC with sidebar navigation
- **Image Optimization** - Automatic image resizing and WebP conversion
- **Social Sharing** - Built-in social media sharing buttons

### 🔍 Search & Navigation
- **Fuse.js Search** - Fast client-side search functionality
- **Taxonomy Support** - Categories, tags, series, and authors
- **Pagination** - Configurable post pagination
- **Breadcrumb Navigation** - Clear site structure

### 📱 Modern Web Features
- **PWA Ready** - Progressive Web App capabilities
- **Cookie Consent** - GDPR-compliant cookie management
- **Analytics Ready** - Support for Google Analytics, Fathom, and more
- **SEO Optimized** - Open Graph, Twitter Cards, and sitemap
- **RSS Feeds** - Multiple RSS feed formats

### 💬 Comments & Engagement
- **Multiple Comment Systems** - Disqus, Gitalk, Valine, Waline, and more
- **Social Links** - Extensive social media integration
- **Reading Time** - Estimated reading time for posts
- **Word Count** - Post length statistics

## 🏗️ Project Structure

```
blog-chillin/
├── archetypes/          # Hugo content templates
├── assets/              # Source assets (SCSS, JS, images)
├── content/             # Blog content
│   └── posts/          # Blog posts
├── public/              # Generated static site (build output)
├── resources/           # Hugo-generated resources
├── themes/              # Hugo themes
│   └── DoIt/           # DoIt theme files
├── config.toml          # Hugo configuration
├── hugo.toml            # Hugo configuration (alternative)
└── README.md            # This file
```

## 🚀 Getting Started

### Prerequisites
- [Hugo Extended](https://gohugo.io/installation/) (version 0.80.0 or higher recommended)
- Git

### Installation

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd blog-chillin
   ```

2. **Install Hugo theme** (if not already present)
   ```bash
   git submodule add https://github.com/HEIGE-PCloud/DoIt.git themes/DoIt
   git submodule update --init --recursive
   ```

3. **Run the development server**
   ```bash
   hugo server -D
   ```

4. **Build for production**
   ```bash
   hugo
   ```

### Configuration

The main configuration files are:
- `hugo.toml` - Primary Hugo configuration
- `config.toml` - Alternative configuration format

Key configuration sections:
- **Site Settings** - Title, description, base URL
- **Theme Settings** - DoIt theme customization
- **Content Settings** - Post pagination, date formats
- **Social Media** - Social network links
- **Analytics** - Google Analytics and other tracking
- **Comments** - Comment system configuration

## 📝 Creating Content

### New Post
```bash
hugo new posts/my-new-post.md
```

### Post Front Matter
```yaml
---
title: "Your Post Title"
date: 2025-01-01T00:00:00+00:00
draft: false
tags: ["tag1", "tag2"]
categories: ["category1"]
series: ["series1"]
---

Your content here...
```

### Content Features
- **Mathematical Formulas**: Use `$...$` for inline and `$$...$$` for block math
- **Code Blocks**: Fenced code blocks with syntax highlighting
- **Images**: Place images in `assets/images/` or `static/images/`
- **Tables**: Markdown tables with sorting capabilities
- **Footnotes**: Extended Markdown footnote support

## 🎨 Customization

### Theme Customization
- Modify `assets/scss/` files for custom styling
- Override theme templates in `layouts/`
- Custom CSS/JS in `assets/`

### Site Configuration
- Update site information in `hugo.toml`
- Configure social media links
- Set up analytics and comments
- Customize navigation menus

## 🌐 Deployment

### GitHub Pages
1. Build the site: `hugo`
2. Push `public/` folder to `gh-pages` branch
3. Configure GitHub Pages to use `gh-pages` branch

### Netlify
1. Connect your repository to Netlify
2. Set build command: `hugo`
3. Set publish directory: `public`

### Vercel
1. Import your repository to Vercel
2. Set build command: `hugo`
3. Set output directory: `public`

## 📚 Available Content

Currently, the blog contains:
- **1 Sample Post**: `first_post.md` - A test post to demonstrate the theme
- **Sample Images**: Avatar images for demonstration

## 🔧 Development

### Local Development
```bash
# Start development server with live reload
hugo server -D

# Build with drafts
hugo -D

# Build for production
hugo --minify
```

### File Watching
```bash
# Watch for changes and rebuild
hugo server --watch
```

## 📖 Documentation

- [Hugo Documentation](https://gohugo.io/documentation/)
- [DoIt Theme Documentation](https://hugo-theme-doit-doc.netlify.app/)
- [Markdown Guide](https://www.markdownguide.org/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with `hugo server`
5. Submit a pull request

## 📄 License

This project is licensed under the [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) license.

## 🙏 Acknowledgments

- [Hugo](https://gohugo.io/) - The static site generator
- [DoIt Theme](https://github.com/HEIGE-PCloud/DoIt) - The beautiful theme
- [KaTeX](https://katex.org/) - Mathematical formula rendering
- [Fuse.js](https://fusejs.io/) - Fuzzy search functionality

---

**Happy Blogging! 🎉**

For questions or support, please open an issue in the repository.
