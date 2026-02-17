---
name: cloudinary
version: 1.1.0
description: Use this skill when uploading images to Cloudinary, generating optimized image URLs, managing assets via Cloudinary MCP tools, or configuring Cloudinary transforms for blogs, social cards, or responsive images.
author: William Smith
tags: [cloudinary, images, cdn, optimization, upload, transforms, media]
triggers:
  keywords:
    - upload to cloudinary
    - cloudinary url
    - optimize images
    - image transforms
    - cloudinary upload
    - blog images
    - f_auto q_auto
    - image cdn
  explicit:
    - /cloudinary
tools:
  - Bash
  - Read
  - WebFetch
---

# Cloudinary Image Management

## Behavioral Classification

**Type**: Guided Decision

**Directive**: ASK, THEN EXECUTE

Ask the user for the target folder, transform preferences, and image source before uploading. Execute the upload and URL generation workflow based on their choices.

## Overview

Cloudinary is an image CDN that serves optimized formats (AVIF/WebP) with on-the-fly transforms. This skill covers uploading, URL generation, and asset management for blog images and other media.

## Environment Variables

| Variable | Required | Sensitive | Description |
|----------|----------|-----------|-------------|
| `CLOUDINARY_URL` | Yes | Yes | Full connection string `cloudinary://key:secret@cloud_name` |
| `CLOUDINARY_CLOUD_NAME` | No | No | Cloud name (extracted from URL if not set) |
| `CLOUDINARY_API_KEY` | No | Yes | API key (extracted from URL if not set) |
| `CLOUDINARY_API_SECRET` | No | Yes | API secret (extracted from URL if not set) |

Ensure `CLOUDINARY_URL` is set in your environment before running upload commands. Never echo or log Cloudinary secrets.

## URL Transform Reference

Base URL pattern:
```
https://res.cloudinary.com/<cloud_name>/image/upload/<transforms>/<public_id>
```

### Common Transforms

| Use Case | Transforms | Description |
|----------|-----------|-------------|
| Blog body | `f_auto,q_auto,w_1200` | Auto-format, auto-quality, 1200px wide |
| Mobile | `f_auto,q_auto,w_600` | Optimized for mobile viewports |
| OG / social card | `f_auto,q_auto,w_1200,h_630,c_fill` | 1200x630 social card crop |
| Retina (2x) | `f_auto,q_auto,w_2400` | Double resolution for HiDPI |
| Thumbnail | `f_auto,q_auto,w_400,h_300,c_fill` | Small preview crop |
| Original | _(none)_ | Full-size original |

### Key Transform Parameters

- `f_auto` - Serves AVIF/WebP/PNG based on browser `Accept` header
- `q_auto` - Perceptual quality optimization (~40-60% savings). Variants: `q_auto:low`, `q_auto:eco`, `q_auto:good`, `q_auto:best`
- `w_N` - Scale width to N pixels (preserves aspect ratio)
- `h_N` - Scale height to N pixels
- `c_fill` - Crop to exact dimensions (use with `w_` and `h_`)
- `c_fit` - Resize to fit within bounds without cropping
- `g_auto` - Auto-detect focal point for cropping
- `dpr_2.0` - Device pixel ratio for responsive images

## Upload Workflow

### Programmatic Upload (Node.js SDK)

```javascript
import { v2 as cloudinary } from 'cloudinary';
// Auto-configures from CLOUDINARY_URL env var

const result = await cloudinary.uploader.upload(filePath, {
  public_id: imageName,
  folder: 'blog/<article-slug>',
  overwrite: true,
  resource_type: 'image',
});
console.log(result.secure_url);
```

Install: `npm install -D cloudinary --save-exact`

### CLI Upload

```bash
# Upload single file
cld uploader upload <file> public_id=<name> folder=blog/<slug>

# Upload directory
cld uploader upload "dir/*.png" folder=blog/<slug>

# List assets in folder
cld search "folder:blog/<slug>" -f public_id -f format -f bytes
```

Install CLI: `pipx install cloudinary-cli`

### Batch Upload Script Pattern

A reusable upload script should accept a folder slug and a local image directory:

```bash
node upload-images.mjs <slug> <image-dir>
```

The script should:
1. Read all image files (PNG, JPG, WebP, AVIF) from the directory
2. Upload each to `blog/{slug}/` folder on Cloudinary
3. Print optimized URLs with `f_auto,q_auto,w_1200` transforms
4. Save a metadata JSON file with public IDs, dimensions, and original sizes

## Folder Conventions

| Cloudinary Path | Content |
|----------------|---------|
| `blog/{slug}/` | Blog article images |
| `brand/` | Logos, icons, brand assets |
| `social/` | Social media cards and banners |

## Cloudinary MCP Tools

When the `cloudinary-asset-mgmt` MCP server is configured, these tools are available:

| Tool | Description |
|------|-------------|
| `upload-asset` | Upload a file to Cloudinary |
| `search-assets` | Search by folder, tags, or metadata |
| `get-asset-details` | Get asset info (dimensions, format, URL) |
| `transform-asset` | Apply transforms to an existing asset |
| `list-images` | List images in account/folder |
| `delete-asset` | Remove an asset |
| `asset-rename` | Rename an asset's public ID |
| `create-folder` | Create a new folder |

### MCP Upload Example

```
Use the cloudinary upload-asset tool to upload hero.png
to the blog/my-article folder with overwrite enabled.
```

## Astro Integration

Add Cloudinary to your Astro image config if you want Astro to optimize images at build time:

```javascript
// astro.config.mjs
image: {
  domains: ['res.cloudinary.com'],
  remotePatterns: [{
    protocol: 'https',
    hostname: 'res.cloudinary.com',
    pathname: '/<cloud_name>/**',
  }],
}
```

**Note**: If you prefer Cloudinary's CDN delivery (`f_auto`, `q_auto`, edge caching), do NOT add it to `image.domains` — Astro would re-encode the images locally at build time, defeating Cloudinary's optimizations.

## Markdown Usage

```markdown
![Alt text](https://res.cloudinary.com/<cloud_name>/image/upload/f_auto,q_auto,w_1200/blog/<slug>/<name>)
```

For OG images in frontmatter:
```yaml
ogImage: "https://res.cloudinary.com/<cloud_name>/image/upload/f_auto,q_auto,w_1200,h_630,c_fill/blog/<slug>/<name>"
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| 401 Unauthorized | Verify `CLOUDINARY_URL` is set in your environment |
| Image not loading in Astro | Add `res.cloudinary.com` to `image.domains` in astro config, or verify the URL is valid |
| Wrong quality / too large | Use `q_auto:low` or `q_auto:eco` for smaller files |
| Blurry on retina | Use `w_2400` (2x) or `dpr_2.0` transform |
| Upload overwrites existing | Set `overwrite: false` or use a unique slug |
| CLI `cld` not found | Install with `pipx install cloudinary-cli` |
| Python mismatch on CLI | Fix with `pipx reinstall cloudinary-cli` |
