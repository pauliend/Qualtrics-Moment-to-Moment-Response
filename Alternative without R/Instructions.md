
# Alternative without R.
I get it, you don't know R and don't feel like learning an entire new program for this one thing. That's fine! This option works, but is less foolproof.

Here, you'll have to **define all your "sliderData_" variables in the survey flow separately**. This is the catch, since defining all the accurate seconds you'll need is not easy, and not always possible. That's why I recommend the main solution, because that allows you to simply collect and see how many seconds you end up with. 
You'll define your "sliderData_" variables by adding the second to "sliderData_", resulting in "sliderData_1", "sliderData_2", and so forth for every second you want to capture. 
You'll also need to **define all the variables of the seconds you want to capture the data for**; If you don't define all these variables, you won't record their data. In my specific use case, the reason I wanted to make this code, I want to capture everyone's data per second, as this allows a full second-by-second analysis and captures different flows of responding per participant (some may report changes often, some may not). 

<img width="858" alt="Screenshot 2023-03-03 at 16 55 50" src="https://user-images.githubusercontent.com/47788764/222863636-5cef7674-d3f8-4b2a-947d-254b620f49fd.png">

It's important that these variables are defined **before** the question itself, in this case the slider, is shown to the respondent. That's why we'll create an Embedded Data Block in the Survey flow that precedes the slider block.

## Where Do I Put The Code? 

In the survey builder, you'll select the slider question and scroll down the left-hand menu of that question. At the bottom, you'll see an option to add Javascript:

<img width="311" alt="Screenshot 2023-03-03 at 17 02 19" src="https://user-images.githubusercontent.com/47788764/222864071-8ff6331d-e173-427b-a9dd-ffa05b5929a5.png">

Click on that, and then delete everything that's there and replace it with the code below. Click save! 

All that's left to do is publish your survey! Good luck. 

### The Javascript Code

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

    // Define a function to save the slider value to an embedded data field
    function saveSliderValue() {
      // Get the current value of the slider
      var value = slider.val();

    // Save the slider value to an embedded data field
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
