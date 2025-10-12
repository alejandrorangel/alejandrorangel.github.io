# Site Maintenance Guide

This is a Jekyll-based personal blog hosted on GitHub Pages.

## Prerequisites

- Ruby (currently using 2.6.10)
- Bundler (currently using 1.17.2)

## Initial Setup

If you're setting up the project for the first time or on a new machine:

```bash
# Install dependencies
bundle install --path vendor/bundle
```

## Common Tasks

### Adding a New Blog Post

1. Create a new file in the `_posts` directory with the format: `YYYY-MM-DD-title.markdown`
2. Add front matter at the top:
   ```yaml
   ---
   layout: post
   title:  "Your Post Title"
   date:   YYYY-MM-DD HH:MM:SS
   categories: blog entry
   ---
   ```
3. Write your content in Markdown below the front matter
4. Build and preview locally before committing

### Building the Site

```bash
# Build the site (generates _site directory)
bundle exec jekyll build

# Build and watch for changes
bundle exec jekyll build --watch
```

### Running Locally

```bash
# Serve the site at http://localhost:4000
bundle exec jekyll serve

# Serve on a different port
bundle exec jekyll serve --port 4001

# Serve without watching for changes (faster)
bundle exec jekyll serve --no-watch
```

### Updating Dependencies

```bash
# Update all gems to latest compatible versions
bundle update

# Update a specific gem
bundle update jekyll
```

## Deploying to GitHub Pages

GitHub Pages automatically builds and deploys your site when you push to the master branch. Simply:

```bash
git add .
git commit -m "Your commit message"
git push origin master
```

The site will be live at https://rangelalejandro.com (or https://alejandrorangel.github.io) within a few minutes.

## Project Structure

```
.
├── _config.yml          # Site configuration
├── _includes/           # Reusable HTML components
│   ├── footer.html
│   ├── head.html
│   └── header.html
├── _layouts/            # Page templates
│   ├── default.html
│   ├── page.html
│   └── post.html
├── _posts/              # Blog posts (markdown files)
├── _sass/               # Sass stylesheets
├── assets/              # Images and other assets
├── css/                 # Compiled CSS
├── about.md             # About page
├── index.html           # Homepage
└── Gemfile              # Ruby dependencies
```

## Configuration

Edit `_config.yml` to change:
- Site title
- Email
- Description
- Social media links (Twitter, GitHub)
- Other Jekyll settings

**Note:** After changing `_config.yml`, you must restart the Jekyll server for changes to take effect.

## Troubleshooting

### "Jekyll not found" error
Make sure to use `bundle exec` before Jekyll commands:
```bash
bundle exec jekyll serve
```

### Port already in use
Either kill the process using the port or specify a different port:
```bash
bundle exec jekyll serve --port 4001
```

### Changes not showing up
1. Restart the Jekyll server
2. Clear your browser cache
3. If editing `_config.yml`, server restart is required

### Build errors after updating gems
Try removing and reinstalling:
```bash
rm -rf vendor/bundle
bundle install --path vendor/bundle
```

## Additional Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Markdown Guide](https://www.markdownguide.org/)
