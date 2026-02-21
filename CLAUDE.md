# Claude Code instructions for git-hooks

## README

Always update README.md when making any of the following changes:
- Adding, modifying, or removing a check script — update both the structure tree and the checks table
- Adding or removing a scope directory — update the structure tree
- Changing EXT_MAP entries in `pre-commit`
- Changing the commit-msg validation rules

## Commits

All commits must follow Conventional Commits format: https://www.conventionalcommits.org/

## New check scripts

When adding a new check, always:
1. Place it in the correct `pre-commit.d/<scope>/` directory
2. Make it executable (`chmod +x`)
3. Include the standard file header (repository, author, license lines)
4. Update README.md with the new check in the appropriate table
5. Test it against both passing and failing cases before committing
