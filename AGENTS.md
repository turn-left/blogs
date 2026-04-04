# AGENTS.md - Agent Coding Guidelines for This Repository

## Overview

This is a personal technical wiki/documentation site built with Docsify. The site contains technical notes about backend (Java, Python) and frontend (Vue) development. Content is primarily in Chinese.

---

## Project Structure

```
D:\wiki\
├── docs/                    # Root docs folder (legacy)
├── 2026/docs/              # Current docs folder
│   ├── _coverpage.md       # Docsify cover page
│   ├── _navbar.md          # Navigation bar
│   ├── _sidebar.md         # Sidebar navigation
│   └── index.html          # Docsify entry point
└── README.md               # Repository readme
```

---

## Build & Development Commands

### Running the Documentation Site

Since this is a Docsify-based documentation site, there is no traditional build process.

- **Serve locally**: Use any static file server or open `2026/docs/index.html` directly in a browser
- **Docsify CLI** (optional): `npx docsify serve docs` or `npx docsify serve 2026/docs`

### Linting

No JavaScript/TypeScript linting is configured. For Markdown linting:

- **markdownlint** (optional): `npx markdownlint "**/*.md"`

### Testing

No test framework is configured for this documentation project.

---

## Code Style Guidelines

### General Principles

1. **Language**: Primary content is in Chinese (Simplified). Use Chinese for documentation text.
2. **Code examples**: Can be in any language (Java, Python, JavaScript, etc.)
3. **File encoding**: UTF-8

### Markdown Formatting

- Use standard Markdown syntax
- Use ATX-style headers (`#`, `##`, `###`)
- Use fenced code blocks with language hints:

```markdown
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```
```

- Use `-` for unordered lists
- Use `1.` for ordered lists
- Use `>` for blockquotes
- Use `---` for horizontal rules

### File Naming

- Use lowercase with hyphens for file names: `java-basics.md`, `python-notes.md`
- Use English for file names even if content is Chinese
- Use descriptive names: `vue-practical-guide.md` not `vue.md`

### Content Organization

- Follow the sidebar structure in `_sidebar.md`
- Group related content under category headers
- Keep each document focused on a single topic
- Include table of contents for long documents using `[[TOC]]`

### Code Examples

- Include comments in code when helpful
- Show complete, runnable examples where possible
- Use meaningful variable and function names
- Follow language-specific best practices (see below)

### Java Code Style

- Use camelCase for variable and method names
- Use PascalCase for class names
- Use descriptive names: `userList` not `ul`
- Use 4 spaces for indentation
- Place opening brace on same line

### Python Code Style

- Follow PEP 8
- Use snake_case for variables and functions
- Use PascalCase for classes
- Use 4 spaces for indentation

### JavaScript/Vue Code Style

- Use camelCase for variables and functions
- Use PascalCase for components
- Use const by default, let when mutation needed
- Avoid var

### Error Handling

- Document common errors and solutions
- Include troubleshooting sections where appropriate
- Use clear, actionable headings

### Links

- Use relative links for internal docs: `[Java 基础](backend/java.md)`
- Use absolute URLs for external resources: `[MDN](https://developer.mozilla.org)`
- Verify external links are working

### Images

- Place images in an `images/` subfolder within the same directory
- Use descriptive alt text
- Keep images reasonably sized

---

## Working With This Repository

### Adding New Documents

1. Create the markdown file in the appropriate category folder
2. Update `_sidebar.md` to add the new document to navigation
3. Test locally by serving the docs

### Editing Existing Documents

1. Find the file in `2026/docs/` directory
2. Make changes following the style guidelines
3. Verify the changes render correctly

### Documentation Conventions

- Keep front matter minimal (none needed for Docsify)
- Use consistent terminology
- Update the sidebar when adding new sections
- Include last-updated dates for time-sensitive content

---

## Notes for AI Agents

- This is a documentation-only repository with no build system
- Focus on content quality and consistency
- Verify links and code examples work correctly
- Do not add unnecessary complexity or tooling
- Maintain the Chinese/English mix appropriately
