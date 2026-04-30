# Markdown Complete Guide (.md)

`Markdown` is an efficient and simple way to write clean, readable web documentation. Easier than HTML, it offers a faster and more natural workflow for creating GitHub project documentation.
Markdown has become a useful tool for developers around the world. I hope this small guide can help new developers and serve as a quick reference for developers and enthusiasts alike

---

## Basic Commands Table

| Command | Explanation |
|--------|------------|
| `# Heading` | Level 1 heading |
| `## Heading` | Level 2 heading |
| `### Heading` | Level 3 heading |
| `### ...` | Up to `######` (Level 6) |
| `- text` | Bullet point (list) |
| `* text` | Bullet point as well |
| `1. text` | Numbered list |
| `` `text` `` | Inline code |
| ```` ``` ```` | Code block |
| `**text**` | Bold |
| `*text*` | Italic |
| `***text***` | Bold italic |
| `~~text~~` | Strikethrough|
| `> text` | Blockquote|
| `---` | Horizontal line |
| `[text](url)` | Link |
| `![alt](url)` | Image |
| `- [ ] task` | Empty checkbox |
| `- [x] task` | Filled checkbox |

---

## Code block

For larger code snippets:

```c
int main()
{
    return 0;
}
```

Also works for other languages:
- `c`
- `bash`
- `python`
- `cpp`

---

## Tables

```markdown
| Column 1 | Column 2 | Column 3 |
| :--- | :---: | ---: |
| text | text | text |
| text | text | text |
```

### Alignment

| symbol | Alignment |
|--------|----------|
| `:---` | Left |
| `:---:` | Center|
| `---:` | Right |

---

## Links

```markdown
[Google](https://google.com)
```

---

## Images

### Basic syntax

```markdown
![Alternative text](images_url)
```

Example:

```markdown
![cat](https://example.com/cat.jpg)
```

---

### How to get an image URL

#### Option 1: From the internet
1. Right-click the image   
2. Select "Copy image url"  
3. Paste it into Markdown  

---

#### Option 2: local image

Structure:

```
project/
 ├── README.md
 └── image.png
```

Use:

```markdown
![my image](image.png)
```

---

#### Option 3: Inside a folder

```
project/
 ├── README.md
 └── image/
      └── photo.png
```

```markdown
![photo](image/photo.png)
```

---

### Important

- Use relative paths (`./image.png`)
- GitHub supports local images
- Some other platforms do not 

---

## Checkboxes

```markdown
- [ ] Learn Markdown
- [x] Install Git
```

---

## Blockquote

```markdown
> This is a Blockquote
```

Nested:

```markdown
> Level 1
>> Level 2
```

---

## Horizontal lines

```markdown
---
```

or

```markdown
***
```

---

## PRO tips

### Mix styles

```markdown
**Text _important_**
```

---

### New line 

- Add two spaces at the end of the line  
- Or leave an empty line  

---

### Note

Markdown may behave differently depending on the platform:
- GitHub
- Notion
- Other editors

---

## Complete example

```markdown
# My Project

## Description
This is a **project** in Markdown.

## List
- Item 1
- Item 2

## Code
```bash
ls -la
```

## Table
| Name | age |
| :--- | ---: |
| John | 30 |

## Image
![logo](image/logo.png)
```

---

---
```
## When to use Markdown vs HTML

| Situation | Can I use Markdown? | Use HTML? | Example |
|----------|------------------------|------------|--------|
| Normal headings | `Yes` | `No` | `## Heading` |
| Center text |  `No` | `Required` | `<h1 align="center">Title</h1>` |
| Bold / Italic | `Yes` | `No` | `**text**`, `*text*` |
| List | `Yes` | `No` | `- item` |
| Code | `Yes` | `No` | `` `code` `` or ``` ``` |
| Tables | `Yes` | `No` | `| col | col |` |
| Links | `Yes` | `No` | `[text](url)` |
| Basic images | `Yes` | `No` | `![alt](img.png)` |
| Center image | `No` | `Required` | `<p align="center"><img src="img.png"></p>` |
| Change image size | `No` | `Required` | `<img src="img.png" width="200">` |
| Align text (left/right/center) | `No` | `Required` | `<p align="center">text</p>` |
| Text colors | `No` | `Required` | `<span style="color:red">text</span>` |
| Controlled line breaks | `Limited` | `Recommended` | `<br>` |
| Complex layouts | `No` | `Required` | `<div>...</div>` |

---

## Fast rules

-  Use **Markdown by default**
-  Use **HTML only when markdown cannot do it**
-  Don't mix HTML unnecessarily

---

## Mixed example (correct use)

```markdown
<h1 align="center"> C++ Guide</h1>

## Introduction

Normal text with **Markdown**

<p align="center">
  <img src="images/logo.png" width="200">
</p>
```

---