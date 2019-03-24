## LAC 2019 Workshop

### Downloads
<a href="https://github.com/gzalles/LAC19_workshop/blob/master/mark2.JPG" download>Click to Download FOA Mic Encoder Image</a>

### Code Highlighting

```cpp

//make 4 write pointers (output)
auto* outDataW = buffer.getWritePointer (0);
auto* outDataY = buffer.getWritePointer (1);
auto* outDataZ = buffer.getWritePointer (2);
auto* outDataX = buffer.getWritePointer (3);

//make 4 read pointers (input)
auto* inDataFLU = buffer.getReadPointer (0);
auto* inDataFRD = buffer.getReadPointer (1);
auto* inDataBLD = buffer.getReadPointer (2);
auto* inDataBRU = buffer.getReadPointer (3);

```

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/gzalles/LAC19_workshop/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
