---
name: seo-optimizer
description: Optimize blog posts for search engines and improve discoverability. Use when improving SEO, analyzing keywords, or enhancing post visibility.
---

# SEO Optimizer

Optimize blog posts for better search engine visibility and discoverability. This skill helps improve content structure, metadata, and overall SEO performance for the Hugo blog.

## When to Use This Skill

Use the seo-optimizer skill when you need to:
- Improve post search engine rankings
- Optimize post titles and descriptions
- Enhance keyword targeting
- Analyze and improve content structure
- Add or optimize meta tags
- Improve internal linking
- Optimize images for SEO
- Audit posts for SEO best practices

## SEO Configuration

The blog is already configured with key SEO features in `config.yml`:

```yaml
baseURL: https://novemberde.github.io/
enableRobotsTXT: true
googleAnalytics: "G-9SF4VDQ4N0"
params:
  env: production  # Enables opengraph, twitter-cards, schema
```

## SEO Checklist for Posts

### Title Optimization

**Best Practices:**
- Keep titles between 50-60 characters
- Include primary keyword near the beginning
- Make titles compelling and click-worthy
- Use title case or sentence case consistently
- For Korean posts, ensure proper Korean SEO keywords

**Good Examples:**
```yaml
# Technical post
title: "AWS Lambda Best Practices: Complete Guide for 2025"

# Korean post
title: "AWS Lambda 모범 사례: 2025년 완벽 가이드"

# Leadership post
title: "10x Engineering Leadership: Lessons from Building Platform Teams"
```

**Bad Examples:**
```yaml
# Too long (>60 chars)
title: "Everything You Ever Wanted to Know About AWS Lambda Functions and Serverless Architecture Best Practices"

# No keywords
title: "My Thoughts"

# Not descriptive
title: "Tutorial"
```

### Meta Description

Hugo PaperMod theme automatically generates descriptions from content. To optimize:

**In Post Content:**
- Write a strong opening paragraph (first 155-160 characters)
- Include primary keywords naturally
- Make it compelling to encourage clicks

**Example Opening:**
```markdown
AWS Lambda revolutionizes serverless computing by letting you run code without managing servers. This guide covers best practices for production Lambda functions, including performance optimization, cost management, and security.
```

### Tags and Keywords

**Tag Strategy:**
- Use 4-8 relevant tags per post
- Mix broad and specific tags
- Include trending keywords when relevant
- Use both English and Korean tags for bilingual SEO
- Maintain tag consistency across similar posts

**Tag Optimization:**
```yaml
# Good - Specific and relevant
tags: ["aws-lambda", "serverless", "nodejs", "performance", "best-practices"]

# Better - Includes Korean for bilingual SEO
tags: ["aws-lambda", "serverless", "서버리스", "nodejs", "모범사례"]

# Bad - Too generic or too many
tags: ["programming", "code", "tech", "aws", "amazon", "cloud", "serverless", "lambda", "tutorial", "guide", "2025"]
```

### URL Structure

**Current URL Pattern:**
```
https://novemberde.github.io/post/YYYY/MM/DD/Post-Title/
```

**URL Optimization Tips:**
- Keep filenames concise and descriptive
- Use hyphens (not underscores)
- Include primary keyword in filename
- Avoid special characters
- Keep URLs under 75 characters when possible

**Good Filenames:**
```
AWS-Lambda-Best-Practices.md
10x-Engineering-Leadership.md
Docker-Container-Optimization.md
```

**Bad Filenames:**
```
post1.md
my_new_article_about_stuff.md
this-is-a-very-long-filename-that-goes-on-and-on.md
```

### Header Structure

Proper header hierarchy helps both SEO and readability:

```markdown
# Post Title (H1 - automatically generated from frontmatter)

## Main Section (H2 - use for primary topics)

### Subsection (H3 - use for subtopics)

#### Detail (H4 - use sparingly)
```

**SEO Tips for Headers:**
- Include keywords in H2 and H3 headers
- Use descriptive headers, not generic ones
- Maintain logical hierarchy
- Don't skip header levels

**Good Header Structure:**
```markdown
## AWS Lambda Performance Optimization

### Cold Start Reduction Techniques

#### Provisioned Concurrency Configuration

### Memory and Timeout Settings

## Cost Optimization Strategies
```

**Bad Header Structure:**
```markdown
## Part 1

### Thing A

#### Another Thing

## Part 2

### Something

## Conclusion
```

### Content Optimization

**Keyword Density:**
- Primary keyword: 1-2% of content
- Related keywords: Naturally throughout
- Avoid keyword stuffing
- Use synonyms and variations

**Content Length:**
- Aim for 1000+ words for technical guides
- 500-800 words for shorter posts
- Longer content tends to rank better (if quality is maintained)

**Content Structure:**
- Start with key information (inverted pyramid)
- Use short paragraphs (3-4 sentences)
- Include code examples for technical posts
- Add lists and bullet points for scannability
- Include images with descriptive alt text

### Image Optimization

**Image SEO Best Practices:**

```markdown
# Good - Descriptive alt text
![AWS Lambda function architecture diagram showing event source, Lambda, and destination](/images/lambda-architecture.png)

# Bad - Generic or missing alt text
![Image](/img/screenshot.png)
![](/image1.png)
```

**Image File Optimization:**
```bash
# Use descriptive filenames
lambda-cold-start-comparison.png  # Good
image1.png                        # Bad

# Compress images
# Use tools like imagemagick, tinypng, or squoosh
convert original.png -quality 85 optimized.png

# Use appropriate formats
# - PNG for diagrams and screenshots
# - JPG for photos
# - WebP for better compression (if supported)
```

### Internal Linking

**Link to Related Posts:**
```markdown
For more information on serverless architecture, see [Building Serverless APIs with AWS Lambda](/post/2024/03/15/serverless-api-guide/).

This builds on concepts from my previous post on [AWS Lambda Best Practices](/post/2024/01/10/lambda-best-practices/).
```

**Internal Linking Strategy:**
- Link to 2-5 related posts per article
- Use descriptive anchor text (not "click here")
- Link to both newer and older relevant content
- Create topic clusters (pillar posts + supporting posts)

### Code Block Optimization

**Always Include Language Tags:**
```markdown
# Good
```javascript
const handler = async (event) => {
  return { statusCode: 200 };
};
```

# Bad
```
const handler = async (event) => {
  return { statusCode: 200 };
};
```
```

**Code Block SEO Benefits:**
- Improves syntax highlighting
- Better for accessibility
- Search engines understand code context
- Helps with featured snippets

## Schema Markup

Hugo PaperMod automatically includes schema markup when `env: production` is set. This provides:
- Article schema
- Breadcrumb schema
- Organization schema
- Social media cards (OpenGraph, Twitter)

**Verify Schema:**
1. Build the site: `hugo`
2. Check generated HTML in `public/`
3. Use Google's Rich Results Test: https://search.google.com/test/rich-results

## Social Media Optimization

**OpenGraph and Twitter Cards** are automatically generated by PaperMod theme.

**To Enhance Social Sharing:**

Add custom parameters to frontmatter (optional):
```yaml
---
title: "Your Post Title"
tags: ["tag1", "tag2"]
date: "2025-12-22T10:00:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
# Optional social media overrides
# cover:
#   image: "/images/post-cover.png"
#   alt: "Description of cover image"
---
```

## SEO Audit Workflow

### Audit a Single Post

```markdown
# 1. Title Check
- [ ] 50-60 characters
- [ ] Includes primary keyword
- [ ] Compelling and clear

# 2. Content Check
- [ ] Strong opening paragraph (155-160 chars)
- [ ] 1000+ words (for guides)
- [ ] Proper header hierarchy (H2, H3)
- [ ] Keywords in headers
- [ ] Code blocks have language tags
- [ ] Lists and bullet points for scannability

# 3. Meta Check
- [ ] 4-8 relevant tags
- [ ] Date is correct
- [ ] Required frontmatter fields present

# 4. Image Check
- [ ] All images have descriptive alt text
- [ ] Filenames are descriptive
- [ ] Images are optimized/compressed

# 5. Link Check
- [ ] 2-5 internal links to related posts
- [ ] External links open in new tab (if desired)
- [ ] No broken links

# 6. URL Check
- [ ] Filename is concise and includes keyword
- [ ] Uses hyphens, not underscores
- [ ] No special characters
```

### Audit All Posts

```bash
# Find posts without proper tags (less than 3 tags)
for file in content/post/**/*.md; do
  tag_count=$(grep -oP '(?<=tags: \[).*(?=\])' "$file" | tr ',' '\n' | wc -l)
  if [ "$tag_count" -lt 3 ]; then
    echo "$file has only $tag_count tags"
  fi
done

# Find posts with missing alt text in images
grep -r "!\[\](" content/post/

# Find very short posts (less than 500 words)
for file in content/post/**/*.md; do
  word_count=$(wc -w < "$file")
  if [ "$word_count" -lt 500 ]; then
    echo "$file: $word_count words"
  fi
done
```

## Keyword Research

### Finding Keywords

**For Technical Posts:**
1. Use Google Trends: https://trends.google.com
2. Check AWS documentation for official terminology
3. Look at Stack Overflow questions
4. Review GitHub discussions

**For Korean Content:**
1. Use Naver search suggestions
2. Check Korean developer communities
3. Look at Korean tech blog trends

### Competitor Analysis

```bash
# Research similar blogs
# 1. Identify top-ranking posts for your target keywords
# 2. Analyze their:
#    - Title structure
#    - Content length
#    - Header usage
#    - Tag strategy
#    - Internal linking

# 3. Create better content:
#    - More comprehensive
#    - Better examples
#    - Clearer explanations
#    - Updated information
```

## Common SEO Improvements

### Improve Post Titles

**Before:**
```yaml
title: "Lambda Tutorial"
```

**After:**
```yaml
title: "AWS Lambda Tutorial: Build Serverless Functions in 2025"
```

### Add Descriptive Alt Text

**Before:**
```markdown
![](/images/img1.png)
```

**After:**
```markdown
![AWS Lambda architecture diagram showing API Gateway triggering Lambda function connected to DynamoDB](/images/lambda-architecture-diagram.png)
```

### Optimize Tags

**Before:**
```yaml
tags: ["aws", "code"]
```

**After:**
```yaml
tags: ["aws-lambda", "serverless", "nodejs", "tutorial", "aws-best-practices"]
```

### Improve Content Structure

**Before:**
```markdown
This post talks about Lambda.

Lambda is great. Here's some code.

[code block]

That's it.
```

**After:**
```markdown
AWS Lambda revolutionizes serverless computing by eliminating server management. This comprehensive guide covers Lambda best practices, performance optimization, and cost management strategies.

## Understanding AWS Lambda

Lambda is a serverless compute service that runs your code in response to events...

### Key Benefits

- No server management
- Automatic scaling
- Pay per use

## Performance Optimization

### Cold Start Reduction

One of the most common Lambda challenges is cold starts...

[detailed explanation with code examples]

## Related Resources

For more on serverless architecture, check out [Building Serverless APIs](/post/2024/03/15/serverless-api-guide/).
```

## Monitoring SEO Performance

### Google Analytics

The blog has Google Analytics enabled (`G-9SF4VDQ4N0`). Monitor:
- Page views per post
- Bounce rate
- Average time on page
- Traffic sources
- Search queries (via Google Search Console)

### Google Search Console

Set up Google Search Console for:
- Search query data
- Click-through rates
- Average position
- Index coverage
- Mobile usability

### Key Metrics to Track

1. **Organic Traffic:** Traffic from search engines
2. **Click-Through Rate (CTR):** % of impressions that result in clicks
3. **Average Position:** Where posts rank in search results
4. **Bounce Rate:** % of visitors who leave immediately
5. **Time on Page:** How long visitors stay

## SEO Best Practices Summary

### Do's ✓
- Write comprehensive, valuable content (1000+ words)
- Use descriptive titles with keywords (50-60 chars)
- Include 4-8 relevant tags
- Add descriptive alt text to all images
- Use proper header hierarchy (H2, H3, H4)
- Include internal links to related posts
- Optimize images (compress, descriptive filenames)
- Write strong opening paragraphs
- Use code blocks with language tags
- Keep URLs concise and descriptive

### Don'ts ✗
- Don't keyword stuff
- Don't use generic titles
- Don't skip alt text
- Don't use generic filenames (img1.png)
- Don't skip header hierarchy levels
- Don't write very short posts (unless intentional)
- Don't forget to proofread
- Don't ignore mobile optimization
- Don't use too many tags (over 10)
- Don't forget bilingual SEO (English + Korean)

## Quick SEO Wins

High-impact, low-effort improvements:

1. **Add Alt Text to Images** (5 min per post)
   - Improves accessibility and SEO
   - Use descriptive, keyword-rich text

2. **Optimize Post Titles** (2 min per post)
   - Add primary keyword
   - Keep under 60 characters
   - Make compelling

3. **Add Internal Links** (5 min per post)
   - Link to 3-5 related posts
   - Use descriptive anchor text

4. **Improve Opening Paragraph** (5 min per post)
   - Make first 160 characters compelling
   - Include primary keyword
   - Summarize key value

5. **Add More Tags** (2 min per post)
   - Ensure 4-8 relevant tags
   - Include both broad and specific
   - Add Korean tags for Korean posts

## Tools and Resources

### SEO Analysis Tools
- Google Search Console
- Google Analytics
- Google PageSpeed Insights
- Google Rich Results Test

### Keyword Research
- Google Trends
- Google Keyword Planner
- Naver Keyword Tool (for Korean)

### Content Optimization
- Yoast SEO (WordPress, but principles apply)
- Hemingway Editor (readability)
- Grammarly (grammar and clarity)

### Image Optimization
- TinyPNG
- Squoosh
- ImageMagick

## Troubleshooting

**Issue: Post Not Indexed**
- Check robots.txt: `enableRobotsTXT: true` in config
- Verify sitemap.xml is generated
- Submit sitemap to Google Search Console
- Check for `noindex` tags (shouldn't be present)

**Issue: Low Click-Through Rate**
- Improve post title (make more compelling)
- Enhance meta description (strong opening paragraph)
- Add structured data markup (already enabled)

**Issue: High Bounce Rate**
- Improve content quality and relevance
- Add internal links to keep visitors engaged
- Ensure mobile responsiveness
- Improve page load speed

**Issue: Posts Not Ranking**
- Research and target less competitive keywords
- Create more comprehensive content
- Build more internal links to the post
- Improve content quality and uniqueness
- Add more related posts (topic clusters)
