# Introduction
This Javascript code and instructions will allow you to capture slider data from Qualtrics as a continuous response, like dial testing. 

This code adds an event listener to a Qualtrics survey slider that captures the value of a slider question every second and saves it as an embedded data element. The slider values are captured in "sliderData_" variables starting with "sliderData_0" up to however many seconds you record the slider question. It also saves the number of seconds the slider values were captured for in a separate embedded data element: "sliderDataSeconds".

*Note*: I have developed this code to use on a **single slider question** in Qualtrics, like this one below. 

<img width="689" alt="Screenshot 2023-03-03 at 16 53 04" src="https://user-images.githubusercontent.com/47788764/222863475-8f79533b-8b38-4aca-ac9a-fc15735fc61a.png">

# How To
## Qualtrics Embedded Data
Since we're saving the slider values as embedded data, we're going to need to define those variables first in the Survey Flow. It's important that these variables are defined **before** the question itself, in this case the slider, is shown to the respondent. That's why we'll create an Embedded Data Block in the Survey flow that precedes the slider block. Like so:

<img width="858" alt="Screenshot 2023-03-03 at 16 55 50" src="https://user-images.githubusercontent.com/47788764/222863636-5cef7674-d3f8-4b2a-947d-254b620f49fd.png">

You'll also need to **define all the variables of the seconds you want to capture the data for**; If you don't define all these variables, you won't record their data. In my specific use case, the reason I wanted to make this code, I want to capture everyone's data per second, as this allows a full second-by-second analysis and captures different flows of responding per participant (some may report changes often, some may not). This is interesting information for me and that's why all this data will be captured and recorded. 

## Where Do I Put The Code? 

In the survey builder, you'll select the slider question and scroll down the left-hand menu of that question. At the bottom, you'll see an option to add Javascript:

<img width="311" alt="Screenshot 2023-03-03 at 17 02 19" src="https://user-images.githubusercontent.com/47788764/222864071-8ff6331d-e173-427b-a9dd-ffa05b5929a5.png">

Click on that, and then delete everything that's there and replace it with the code below. Click save! 

All that's left to do is publish your survey! Good luck. 

### The Code

    //
      Qualtrics.SurveyEngine.addOnload(function() {
     // Set up the slider element
    var slider = jQuery("#" + this.questionId + " .ResultsInput");

    // Set up the interval to capture the slider value every second
    var interval = setInterval(function() {
       saveSliderValue();
    }, 1000);

    // Set up the variables for the slider data
    var sliderDataCount = 0;
    var sliderData = {};

    // Define a function to save the slider value to a hidden variable
    function saveSliderValue() {
      // Get the current value of the slider
      var value = slider.val();

    // Save the slider value to a hidden variable
    var variableName = "sliderData_" + sliderDataCount;
    Qualtrics.SurveyEngine.setEmbeddedData(variableName, value);

    // Add the slider value to the slider data object
    sliderData[sliderDataCount] = value;

    // Increment the count variable
    sliderDataCount++;
      }

     // Save the number of seconds captured as an embedded data variable on survey submission
      Qualtrics.SurveyEngine.addOnUnload(function() {
        clearInterval(interval);
       Qualtrics.SurveyEngine.setEmbeddedData("sliderDataSeconds", sliderDataCount);
          });
       });

# Multiple Sliders

If you'd like to capture continuous data from multiple sliders, you can re-use the code, but you'll have to change the variables "sliderData_" and "sliderDataSeconds". For instance, after the word sliders in each of these names, you can add the number of the slider (e.g., "slider1Data_" and so on). Also, don't forget to define these variables in the survey flow.
