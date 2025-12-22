---
name: blog-post-creator
description: Create new Hugo blog posts with proper frontmatter and directory structure. Use when creating new blog posts, articles, or content for the Hugo site.
---

# Blog Post Creator

Create new blog posts for the Hugo static site with proper frontmatter, directory structure, and naming conventions.

## Post Structure

All posts follow this directory pattern:
```
content/post/YYYY/MM/DD/Post-Title.md
```

## Frontmatter Template

Every post must include this frontmatter:
```yaml
---
title: "Your Post Title"
tags: ["tag1", "tag2", "tag3"]
date: "YYYY-MM-DDTHH:MM:SS+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
```

## Instructions

When creating a new blog post:

1. **Generate the file path**:
   - Use current date: `content/post/YYYY/MM/DD/`
   - Create a descriptive filename with hyphens: `Post-Title.md`
   - Example: `content/post/2025/01/28/AWS-unified-studio.md`

2. **Create proper frontmatter**:
   - title: Clear, descriptive title in quotes
   - tags: Array of relevant tags (lowercase, use hyphens for multi-word tags)
   - date: ISO 8601 format with timezone
   - Include the three boolean flags for Hugo theme settings

3. **Add content structure**:
   - Start with a clear introduction
   - Use proper markdown headers (##, ###)
   - Include code blocks with language tags when relevant

## Common Tags

Based on the blog's existing content, common tags include:

**Technical/AWS:**
- aws, serverless, lambda, ec2, s3
- docker, kubernetes, devops
- nodejs, python, golang, typescript
- ml, ai, mlops

**Leadership/Career:**
- leadership, career, engineering-culture
- 리더십 (Korean for leadership)
- 개발자 리더십 (Korean for developer leadership)
- 엔지니어 리더십 (Korean for engineer leadership)

**Note:** This blog supports both English and Korean tags. You can use Korean tags for Korean-language content.

## Examples

### Creating a Technical Tutorial
```markdown
---
title: "Getting Started with AWS Lambda"
tags: ["aws", "lambda", "serverless", "tutorial"]
date: "2025-01-28T10:00:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Introduction

Brief overview of what this tutorial covers...

## Prerequisites

- AWS Account
- Basic understanding of...
```

### Creating a Career/Leadership Post
```markdown
---
title: "Lessons from Leading a Platform Team"
tags: ["leadership", "platform-engineering", "career", "engineering-culture"]
date: "2025-01-28T10:00:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Context

Setting the stage...
```

## Best Practices

1. **Title**: Make it clear and searchable
   - For Korean content, use Korean titles (e.g., "-10x 리더 되기, 그리고 10x 리더 되기")
   - For English content, use English titles

2. **Tags**: Use 3-8 relevant tags, prefer existing tags from the blog
   - Mix English and Korean tags when appropriate
   - Use descriptive Korean tags for Korean content (e.g., "개발자 리더십", "엔지니어 리더십")
   - Use quotes for multi-word tags in array format

3. **Date**: Use current date/time in UTC (ISO 8601 format)
   - Format: `YYYY-MM-DDTHH:MM:SS+00:00`
   - Example: `2025-02-09T00:30:00+00:00`

4. **Directory Structure**: Always follow the YYYY/MM/DD pattern
   - Example: `content/post/2025/02/09/`

5. **Filename**: Use hyphens, avoid spaces, keep it descriptive but concise
   - For English: `AWS-unified-studio.md`
   - For Korean: `Plus-minus-10x-leader-ko.md` (add `-ko` suffix for Korean)
   - Use capitalized first letters for better readability

6. **Language Considerations**:
   - This blog is bilingual (English and Korean)
   - Korean content should use Korean tags and appropriate filename suffix
   - Ensure proper UTF-8 encoding for Korean characters

7. **Required Frontmatter Fields**:
   - Always include: `ShowBreadCrumbs: true`, `ShowReadingTime: true`, `ShowPostNavLinks: true`
   - These enable PaperMod theme features for better user experience
