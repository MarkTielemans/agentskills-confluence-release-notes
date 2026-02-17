# Publish release notes to Confluence

This skill relies heavily on the [Atlassian MCP Server](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/#Confluence-workflows).

The easiest way to set this up in Claude Code is using the [Plugin Marketplace](https://code.claude.com/docs/en/discover-plugins).

## Caveats
- There are no instructions or restrictions to the Tenant ID in the committed [SKILL.md](.claude/skills/confluence-release-notes/SKILL.md). Consider pinning the Tenant ID here.
- The parent page for release notes is pinned in [SKILL.md](.claude/skills/confluence-release-notes/SKILL.md).
- While not strictly necessary, Python is used to inline-generate a unique ID

## Usage
```sh
/confluence-release-notes @MYFILE.md SPACEKEY
```