## LAC 2019 Workshop

### Links

The following are a collection of links I am using in this project. The Spatial Workstation is a great free VST I use for binaural decoding. Wigware is an amazing collection of plug-ins used in this project for multi-channel monitoring. Reaper is the DAW of choice for multi-channel audio and JUCE is the C++ framework used to design these plug-ins. I recommend getting all these tools if you are interested in learning about ambisonic software. Sennheiser also has some valuable free tools.

[Facebook Spatial Workstation](https://facebook360.fb.com/spatial-workstation/) //
[Wigware](https://www.brucewiggins.co.uk/?page_id=78) //
[Reaper](https://www.reaper.fm/) //
[JUCE](https://juce.com/) //
[Sennheiser Ambeo Tools](https://en-us.sennheiser.com/ambeo-blueprints-downloads)

### Downloads

This is a collection of downloadables I am using to produce these simple ambisonic plug-ins. They include audio and images. JUCE also allows import of 3D models but I have not yet perfected the use of these.

<a href="https://github.com/gzalles/LAC19_workshop/blob/master/LAC19workshop.pdf" download>Click to Download Slides</a>

<a href="https://github.com/gzalles/LAC19_workshop/blob/master/mark2.JPG" download>Click to Download FOA Mic Encoder Image</a>

<a href="https://github.com/gzalles/LAC19_workshop/blob/master/foaMicEnc.zip" download>Click to Download FOA Mic Encoder Reaper Session</a>

### Code Snippets  
These code snippets are being used to expedite the creation of the plug-ins during the workshop. They are meant to help the reader succeed without any major issues. The code is heavily commented, please read the comments for clarification.

<!-- /////////////////////////////////////////////////////////////// -->
#### FoaMicEncAudioProcessorEditor
This code is used in the FOA Microphone Encoder's editor.

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
#### FoaMicEncAudioProcessor
This code is used for the PluginProcessor part of the FOA mic encoder.

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


<!-- /////////////////////////////////////////////////////////////// -->
#### FoaPanAudioProcessorEditor (constructor declaration - header)

Notice the addition of the listener in the initialization list.

```cpp
class FoaPanAudioProcessorEditor  : public AudioProcessorEditor,
private Slider::Listener //add listener to initialization list!
{
public:
    FoaPanAudioProcessorEditor (FoaPanAudioProcessor&);
    ~FoaPanAudioProcessorEditor();

    //==============================================================================
    void paint (Graphics&) override;
    void resized() override;

    //declare our slider function
    void sliderValueChanged (Slider* slider) override;

    Slider azimuthSlider;   // declare slider for azi
    Slider elevationSlider; // declare slider for elev

    //declare scoped pointers to add sliders to VTS
    ScopedPointer<AudioProcessorValueTreeState::SliderAttachment> azimuthSliderAttachment;
    ScopedPointer<AudioProcessorValueTreeState::SliderAttachment> elevationSliderAttachment;

    //declare some float variables
    float aziPosX, aziPosY; //use to draw smaller circle
    float aziVal, elevVal;  //used to store slider value

private:

    // This reference is provided as a quick way for your editor to
    // access the processor object that created it.
    FoaPanAudioProcessor& processor;

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (FoaPanAudioProcessorEditor)
};
```
<!-- /////////////////////////////////////////////////////////////// -->

#### FoaPanAudioProcessorEditor (constructor definition - cpp)
```cpp
FoaPanAudioProcessorEditor::FoaPanAudioProcessorEditor (FoaPanAudioProcessor& p)
: AudioProcessorEditor (&p), processor (p)
{
    // Make sure that before the constructor has finished, you've set the
    // editor's size to whatever you need it to be.
    setSize (400, 300);

    int textEntryBoxWidth = 100;
    int textEntryBoxHeight = 20;
    float rangeMin = -180.0;
    float rangeMax = +180.0;
    bool readOnly = false;

    // these define the parameters of our azimuth slider object
    azimuthSlider.setRange(rangeMin, rangeMax);
    azimuthSlider.setSliderStyle (Slider::LinearHorizontal);
    azimuthSlider.setTextBoxStyle (Slider::TextBoxBelow, readOnly, textEntryBoxWidth, textEntryBoxHeight);
    azimuthSlider.addListener(this);//this = Slider::Listener
    addAndMakeVisible (&azimuthSlider);

    // these define the parameters of our elevation slider object
    elevationSlider.setRange(rangeMin, rangeMax);
    elevationSlider.setSliderStyle (Slider::LinearVertical);
    elevationSlider.setTextBoxStyle (Slider::TextBoxBelow, readOnly, textEntryBoxWidth, textEntryBoxHeight);
    elevationSlider.addListener(this);//this = Slider::Listener
    addAndMakeVisible (&elevationSlider);

    //add the sliders to the VTS
    azimuthSliderAttachment = new AudioProcessorValueTreeState::SliderAttachment (processor.parameters, "azimuth", azimuthSlider);
    elevationSliderAttachment = new AudioProcessorValueTreeState::SliderAttachment (processor.parameters, "elevation", elevationSlider);
}
```
<!-- /////////////////////////////////////////////////////////////// -->

#### FoaPanAudioProcessorEditor::paint (editor .cpp)
```cpp
void FoaPanAudioProcessorEditor::paint (Graphics& g)
{
    float titleHeight = 15.0;//same as font size
    float subTitleHeight = 10.0;//same as font size

    // fill the whole window silver
    g.fillAll (Colours::silver);
    // set the current drawing colour to blueviolet
    g.setColour (Colours::blueviolet);
    // set the font size and draw text to the screen for title and subtitle
    g.setFont (15.0f);
    g.drawFittedText ("FOA Pan", 0, 0, getWidth(), titleHeight, Justification::centred, 1);//1=maxNumLines
    g.setFont (10.0f);
    g.drawFittedText ("ACN Ordering [WYZX]",
                      0, titleHeight, getWidth(), subTitleHeight, Justification::centred, 1);

    //set font size for slider text and draw text to the screen for both sliders
    g.setFont (12.0f);
    g.drawFittedText ("Azimuth", 20, 215, 200, 20, Justification::centred, 1);//trial and error
    g.drawFittedText ("Elevation", 200, 40, 200, 20, Justification::centred, 1);

    //define large ellipsis params
    int largeDiameter = 150;
    float largeRadius = largeDiameter/2;
    int lineThickness = 2;

    //not "grabbable" [to do]
    //draw the circle/"sphere" [to do]
    //ellipses depict position of source on azimuth plane only
    g.setColour (Colours::blueviolet);
    g.drawEllipse(50, 50, largeDiameter, largeDiameter, lineThickness);//main big circle

    //define small ellipsis params
    int smallDiameter = 15;
    float smallRadius = smallDiameter/2;

    //go to corner of large circle, then its origin, then to the position, multiply it by radius.
    //minus 90 degrees (pi/2) justifies the position so that 0 is above rather than on the right
    aziPosX = 50 + largeRadius + cos(aziVal * (M_PI / 180) - M_PI/2) * largeRadius;
    aziPosY = 50 + largeRadius + sin(aziVal * (M_PI / 180) - M_PI/2) * largeRadius;

    g.setColour (Colours::skyblue);
    //subtract the w and h of circle to have origin of mini circle at the circumference
    //posX, posY, w, h, thickness.
    g.drawEllipse(aziPosX - smallRadius, aziPosY - smallRadius, smallDiameter, smallDiameter, lineThickness);

    repaint();//important, redraws every frame
}
```

<!-- /////////////////////////////////////////////////////////////// -->

#### FoaPanAudioProcessor - constructor definition
This bit is kind of tricky. Note the comma after the macro in lieu of the colon. Also note the added ": parameters (*this, nullptr)"

```cpp
FoaPanAudioProcessor::FoaPanAudioProcessor() : parameters (*this, nullptr)
#ifndef JucePlugin_PreferredChannelConfigurations
, AudioProcessor (BusesProperties()
#if ! JucePlugin_IsMidiEffect
#if ! JucePlugin_IsSynth
                  .withInput  ("Input",  AudioChannelSet::mono(), true)
#endif
                  .withOutput ("Output", AudioChannelSet::ambisonic(1), true)
#endif
                  )
#endif
{
    parameters.createAndAddParameter(std::make_unique<AudioParameterFloat>("azimuth",// parameter ID
                                                                           "azimuth",// parameter name
                                                                           NormalisableRange<float>
                                                                           (-180.0f, 180.0f),// range
                                                                           0.0f,// default value
                                                                           "degrees"));

    parameters.createAndAddParameter(std::make_unique<AudioParameterFloat>("elevation",// parameter ID
                                                                           "elevation",// parameter name
                                                                           NormalisableRange<float> (-180.0f, 180.0f),// range
                                                                           0.0f,// default value
                                                                           "degrees"));

    parameters.state = ValueTree (Identifier ("FoaPanVT"));
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
