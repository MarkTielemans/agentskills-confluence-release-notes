# Publish release notes to Confluence


## Caveats
- There are no instructions or restrictions to the Tenant ID in the committed [SKILL.md](.claude/skills/confluence-release-notes/SKILL.md). Consider pinning the Tenant ID here.
- The parent page for release notes is pinned in [SKILL.md](.claude/skills/confluence-release-notes/SKILL.md).
- While not strictly necessary, Python is used to inline-generate a unique ID

## Usage
```sh
/confluence-release-notes @MYFILE.md SPACEKEY
```