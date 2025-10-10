# KH BYUN Development Blog

Personal development blog built with Hugo and the PaperMod theme.

**Live Site:** [https://novemberde.github.io](https://novemberde.github.io)

## Prerequisites

- [Hugo](https://gohugo.io/) (extended version recommended)
- Git

## Getting Started

### 1. Install Hugo

**macOS:**
```bash
brew install hugo
```

**Linux:**
```bash
# Debian/Ubuntu
sudo apt-get install hugo

# Arch Linux
sudo pacman -S hugo
```

**Windows:**
```bash
choco install hugo-extended
```

Or download from [Hugo releases](https://github.com/gohugoio/hugo/releases).

### 2. Clone the Repository

```bash
git clone https://github.com/novemberde/novemberde.github.io.git
cd novemberde.github.io
```

### 3. Run Local Development Server

```bash
hugo server
```

The site will be available at `http://localhost:1313/`

To run with drafts visible:
```bash
hugo server -D
```

## Project Structure

```
.
├── content/          # Blog posts and pages
│   └── post/        # Blog posts organized by date
├── layouts/         # Custom layout overrides
├── static/          # Static assets (images, etc.)
├── themes/          # Hugo themes (PaperMod)
└── config.yml       # Site configuration
```

## Creating New Content

Create a new blog post:
```bash
hugo new post/YYYY/MM/DD/post-title.md
```

## Building for Production

```bash
hugo
```

Generated files will be in the `public/` directory.

## Deployment

This site is automatically deployed to GitHub Pages via GitHub Actions when changes are pushed to the main branch.

## Theme

This blog uses the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## License

Content is personal property. Code is available under the repository license.
