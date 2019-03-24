## LAC 2019 Workshop

### Downloads
<a href="https://github.com/gzalles/LAC19_workshop/blob/master/mark2.JPG" download>Click to Download FOA Mic Encoder Image</a>

### FoaMicEncAudioProcessorEditor
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
<!-- /////////////////////////////////////////////////////////////// -->
### FoaMicEncAudioProcessor
```cpp
void FoaMicEncAudioProcessor::processBlock (AudioBuffer<float>& buffer, MidiBuffer& midiMessages)
{
    //default JUCE code, leave as is...
    ScopedNoDenormals noDenormals;
    auto totalNumInputChannels  = getTotalNumInputChannels();
    auto totalNumOutputChannels = getTotalNumOutputChannels();

    //clear buffer
    for (auto i = totalNumInputChannels; i < totalNumOutputChannels; ++i)
        buffer.clear (i, 0, buffer.getNumSamples());

    /* DESCRIPTION
     This takes 4 channels and spits back out 4 new channels after encoding.

     W = FLU + FRD + BLD + BRU
     Y = FLU - FRD + BLD - BRU
     Z = FLU - FRD - BLD + BRU
     X = FLU + FRD - BLD - BRU

     ordering ACN, normalization SN3D.
     */

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

    //calculate SN3D normalization coefficient
    //pi declared in header (public), defined in this file (PrepareToPlay())
    float wNormFactor = sqrt(0.25f*pi);

    //traverse samples
    for(int index = 0; index < buffer.getNumSamples(); index++) {

        //ACN = W Y Z X
        outDataW[index] = wNormFactor * (inDataFLU[index] + inDataFRD[index] +
                                         inDataBLD[index] + inDataBRU[index]);

        outDataY[index] = inDataFLU[index] - inDataFRD[index] + inDataBLD[index] - inDataBRU[index];
        outDataZ[index] = inDataFLU[index] - inDataFRD[index] - inDataBLD[index] + inDataBRU[index];
        outDataX[index] = inDataFLU[index] + inDataFRD[index] - inDataBLD[index] - inDataBRU[index];

    }

}
```

<!-- ### Markdown

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

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out. -->
