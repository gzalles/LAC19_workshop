## LAC 2019 Workshop

### Downloads
<a href="https://github.com/gzalles/LAC19_workshop/blob/master/mark2.JPG" download>Click to Download FOA Mic Encoder Image</a>

### FOA Mic Encoder
```cpp
void FoaMicEncAudioProcessorEditor::paint (Graphics& g)
{
    // fill the whole window black
    g.fillAll (Colours::black);
    // set the current drawing colour to white
    g.setColour (Colours::white);
    // set the font size and draw text to the screen
    g.setFont (22.0f);

    //set some values for fitted text functions
    int titleHeight = 50;
    int subTitleOffset = 20;

    //draw the title
    g.drawFittedText ("FOA Mic Encoder", 0, 0, getWidth(), titleHeight, Justification::centred, 1);

    //change the font size
    g.setFont (18.0f);

    //draw the subtitle
    g.drawFittedText ("ACN + SN3D", 0, subTitleOffset, getWidth(), titleHeight, Justification::centred, 1);

    //draw the image from cache
    Image background = ImageCache::getFromMemory (BinaryData::mark2_JPG, BinaryData::mark2_JPGSize);

    //set values for image offset
    int imageOffsetX = 56;
    int imageOffsetY = 60;

    //draw the image on the canvas
    g.drawImageAt (background, imageOffsetX, imageOffsetY);
}
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
