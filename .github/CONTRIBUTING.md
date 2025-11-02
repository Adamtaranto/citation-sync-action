# Contributing to Citation Sync Action

Thank you for your interest in contributing to Citation Sync Action! We welcome contributions from the community.

## Ways to Contribute

- **Report bugs**: Open an issue describing the bug and how to reproduce it
- **Suggest features**: Open an issue describing the feature and its use case
- **Improve documentation**: Submit pull requests to improve README, comments, or examples
- **Fix bugs**: Submit pull requests with bug fixes
- **Add features**: Submit pull requests with new features

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/citation-sync-action.git
   cd citation-sync-action
   ```
3. **Create a branch** for your changes:
   ```bash
   git checkout -b feature/your-feature-name
   ```

## Development Guidelines

### Code Style

- Use clear, descriptive variable names
- Add comments for complex logic
- Follow existing code style and conventions
- Use `set -e` and `set -o pipefail` for bash scripts
- Quote variables in bash scripts: `"${VARIABLE}"`

### Testing

- Test your changes thoroughly before submitting
- Add test cases for new features
- Ensure existing tests still pass
- Test with different input combinations

### Commit Messages

Use clear, descriptive commit messages:
- Use present tense ("Add feature" not "Added feature")
- Use imperative mood ("Move file to..." not "Moves file to...")
- First line should be 50 characters or less
- Reference issues and pull requests when relevant

Examples:
```
Add support for custom version prefixes

Fix version parsing for pre-release tags

Update documentation for match mode
```

### Pull Request Process

1. **Update documentation** if you're adding or changing features
2. **Add tests** for new functionality
3. **Update CHANGELOG.md** with a description of your changes
4. **Ensure your code passes all checks**:
   - No syntax errors
   - Follows style guidelines
   - Tests pass
5. **Submit your pull request** with:
   - Clear title describing the change
   - Description of what was changed and why
   - Reference to any related issues

### Pull Request Template

When opening a pull request, include:

```markdown
## Description
Brief description of the changes

## Motivation and Context
Why is this change needed? What problem does it solve?

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
Describe the tests you ran to verify your changes

## Checklist
- [ ] My code follows the style guidelines of this project
- [ ] I have performed a self-review of my own code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
```

## Reporting Bugs

When reporting bugs, please include:

1. **Description**: Clear description of the bug
2. **Steps to reproduce**: Detailed steps to reproduce the issue
3. **Expected behavior**: What you expected to happen
4. **Actual behavior**: What actually happened
5. **Environment**:
   - GitHub Actions runner OS
   - Action version
   - Repository details (if public)
6. **Logs**: Relevant error messages or logs (use debug mode if needed)
7. **Additional context**: Any other relevant information

## Suggesting Features

When suggesting features, please include:

1. **Use case**: Describe the problem this feature would solve
2. **Proposed solution**: How you envision the feature working
3. **Alternatives considered**: Other solutions you've thought about
4. **Additional context**: Examples, mockups, or references

## Code of Conduct

This project adheres to the Contributor Covenant [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior by opening an issue or contacting @Adamtaranto.

## Questions?

If you have questions about contributing, feel free to:
- Open an issue with the "question" label
- Start a discussion in GitHub Discussions (if enabled)
- Contact @Adamtaranto directly

## License

By contributing to this project, you agree that your contributions will be licensed under the MIT License.
