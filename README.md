# Cloudinary Skill for Claude Code

A Claude Code skill for managing Cloudinary images — uploads, URL transforms, MCP tool integration, and Astro framework configuration.

## Install

```bash
skillsmith install wrsmith108/cloudinary
```

Or manually copy `SKILL.md` to `~/.claude/skills/cloudinary/SKILL.md`.

## What It Does

- Generates optimized Cloudinary URLs with `f_auto`, `q_auto`, responsive widths
- Guides image uploads via Node.js SDK, CLI, or Cloudinary MCP tools
- Provides transform cheat sheet (blog, mobile, OG cards, retina, thumbnails)
- Configures Astro integration with CDN-passthrough guidance
- Covers folder conventions and troubleshooting

## Triggers

| Type | Values |
|------|--------|
| Explicit | `/cloudinary` |
| Keywords | `upload to cloudinary`, `cloudinary url`, `optimize images`, `image transforms`, `blog images`, `f_auto q_auto`, `image cdn` |

## Tools Used

- `Bash` — CLI uploads, SDK scripts
- `Read` — inspecting config files
- `WebFetch` — Cloudinary API docs

## Requirements

- `CLOUDINARY_URL` environment variable set (`cloudinary://key:secret@cloud_name`)
- Optional: [cloudinary-asset-mgmt MCP server](https://github.com/cloudinary/cloudinary-mcp-server) for direct tool access

## License

[MIT](LICENSE)
