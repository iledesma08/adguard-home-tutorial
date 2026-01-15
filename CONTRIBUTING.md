# Contributing to AdGuard Home Tutorial

Thank you for your interest in contributing to this project! This tutorial aims to help users set up a comprehensive network security and privacy solution using a Raspberry Pi Zero 2W.

## How to Contribute

### Reporting Issues

If you find errors, typos, or outdated information:

1. **Check existing issues** to avoid duplicates
2. **Open a new issue** with:
   - Clear title describing the problem
   - Which part of the tutorial is affected
   - What's wrong and what it should be
   - Your Fritz!Box model and software versions (if relevant)

### Suggesting Improvements

Have ideas for making the tutorial better?

1. Open an issue with the "enhancement" label
2. Describe your suggestion clearly
3. Explain why it would be helpful
4. Include examples if possible

### Contributing Code/Configuration

#### Configuration Files

If you have improved configurations:

1. Fork the repository
2. Add your configuration to the `configs/` directory
3. Name it descriptively (e.g., `AdGuardHome-performance.yaml`)
4. Document what makes it special in the filename or a comment
5. Submit a pull request with a clear description

#### Documentation

To improve or expand the documentation:

1. Fork the repository
2. Make your changes to the relevant markdown files
3. Follow the existing style and formatting
4. Test all commands and procedures if possible
5. Submit a pull request

### Style Guidelines

#### Markdown Formatting

- Use clear, descriptive headings
- Include code blocks with appropriate syntax highlighting
- Add examples and use cases
- Include troubleshooting tips where relevant
- Keep line length reasonable (80-120 characters)

#### Code Blocks

```bash
# Always include comments for complex commands
sudo command --with-flags

# Show expected output when helpful
```

#### Configuration Files

- Use YAML format for AdGuard Home configs
- Include comments for non-obvious settings
- Set secure defaults
- Add warnings for settings that need customization

### Testing

Before submitting:

1. **Verify commands work** on a fresh Raspberry Pi OS installation
2. **Test configurations** don't break existing functionality
3. **Check links** are valid and accessible
4. **Proofread** for typos and grammar

### Adding New Sections

If you want to add a new tutorial part:

1. Create a new file in `docs/` following the naming convention
2. Update the main README.md index
3. Link from relevant existing parts
4. Follow the existing structure:
   - Overview/Introduction
   - Prerequisites
   - Step-by-step instructions
   - Configuration examples
   - Testing/Verification
   - Troubleshooting
   - Next steps
   - Additional resources

### Images and Screenshots

When adding images:

1. Place in the `images/` directory
2. Use descriptive filenames with part numbers
3. Compress images to reasonable sizes (<500KB)
4. Use PNG for screenshots, JPG for photos
5. Add alt text for accessibility
6. Update the images README

### Language and Tone

- Write in clear, simple English
- Be friendly and encouraging
- Assume readers are beginners unless in an "Advanced" section
- Explain technical terms when first used
- Provide context for why steps are necessary

### Examples of Good Contributions

✅ "Fixed typo in Part 2, line 45"
✅ "Added troubleshooting section for common UFW issues"
✅ "Updated AdGuard Home configuration for v0.107.x"
✅ "Added Fritz!Box 7590 specific screenshots"
✅ "Improved explanation of split tunneling in Part 3"

### Examples to Avoid

❌ Undocumented configuration changes
❌ Breaking changes without migration guide
❌ Copy-pasted content from other sources without attribution
❌ Untested commands or configurations
❌ Personal promotion or non-relevant links

## Pull Request Process

1. **Fork** the repository
2. **Create a branch** with a descriptive name:
   - `fix/part2-typo`
   - `feature/add-ipv6-guide`
   - `docs/improve-troubleshooting`
3. **Make your changes** following the guidelines above
4. **Test thoroughly** if applicable
5. **Commit** with clear, descriptive messages:
   ```
   Fix typo in Part 2 DNS configuration
   
   - Corrected IP address in example
   - Updated command syntax for newer version
   ```
6. **Submit pull request** with:
   - Clear title
   - Description of changes
   - Reference to related issues (if any)
   - Testing performed (if applicable)

## Code of Conduct

### Our Standards

- Be respectful and constructive
- Focus on what's best for the community
- Show empathy towards others
- Accept constructive criticism gracefully
- Help maintain a welcoming environment

### Unacceptable Behavior

- Harassment or discriminatory language
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information
- Spam or promotional content

## Questions?

Not sure if your contribution is appropriate? Open an issue to discuss it first!

## Recognition

Contributors will be:
- Listed in commit history
- Acknowledged in release notes for significant contributions
- Appreciated by the community!

## License

By contributing, you agree that your contributions will be licensed under the MIT License, the same as the project.

---

Thank you for helping make this tutorial better! 🎉
