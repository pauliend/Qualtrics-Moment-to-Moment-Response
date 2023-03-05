

# Introduction
This Javascript code and instructions will allow you to capture slider data from Qualtrics as a continuous response, like dial testing. 

This code adds an event listener to a Qualtrics survey slider that captures the value of a slider question every second and saves it as an embedded data element. The slider values are captured in a "sliderData" variables with a JSON string, with up to however many seconds you record the slider question. It also saves the number of seconds the slider values were captured for in a separate embedded data element: "sliderDataCount".

*Note*: I have developed this code to use on a **single slider question** in Qualtrics, like this one below. 

<img width="689" alt="Screenshot 2023-03-03 at 16 53 04" src="https://user-images.githubusercontent.com/47788764/222863475-8f79533b-8b38-4aca-ac9a-fc15735fc61a.png">

I recommend the solution explained below for capturing continuous self-report data with your sliders. However, it does require R which I understand might not be your cup of tea or might not be worth the hassle of installing for this relatively simple Qualtrics variable. However, this being the more foolproof option, allowing to **capture as many seconds as you need automatically without having to define them by hand a priori**, I would recommend installing R. Also, it's just a great tool in general! You might need it someday, so why not install it now? 

# How To
## Qualtrics Embedded Data
Since we're saving the slider values as embedded data, we're going to need to define those variables first in the Survey Flow. It's important that these variables are defined **before** the question itself, in this case the slider, is shown to the respondent. That's why we'll create an Embedded Data Block in the Survey flow that precedes the slider block. Like so:

<img width="880" alt="Screenshot 2023-03-03 at 23 06 56" src="https://user-images.githubusercontent.com/47788764/222881407-316e89d6-ff93-4f38-a7b6-0fb04ddf1b7a.png">

The variable **"sliderDataCount"** records the total number of seconds the slider was presented, and **"sliderData"** is a JSON object that'll store all your second-by-second data in a key-value structure. 

## Inserting the Javascript Code

In the menu above you'll find a file labeled "JavaScript Code". You can copy that code from there, or straight from this window: 

    // Start of Code
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

         // Increment the count variable
         sliderDataCount++;

         // Add the slider value to the slider data object
         sliderData[sliderDataCount-1] = value;

         // Save the slider value to an embedded data field
         Qualtrics.SurveyEngine.setEmbeddedData("sliderData", JSON.stringify(sliderData));
         Qualtrics.SurveyEngine.setEmbeddedData("sliderDataCount", sliderDataCount);
             }
  
        // Save the slider data as embedded data on survey submission
          Qualtrics.SurveyEngine.addOnUnload(function() {
         clearInterval(interval);
             });
        });

### Where Do I Put The Javascript Code? 

In the survey builder, you'll select the slider question and scroll down the left-hand menu of that question. At the bottom, you'll see an option to add Javascript: 

<img width="311" alt="Screenshot 2023-03-03 at 17 02 19" src="https://user-images.githubusercontent.com/47788764/222864071-8ff6331d-e173-427b-a9dd-ffa05b5929a5.png">

Click on that, and then delete everything that's there and replace it with the code below. Click save! 

All that's left to do is publish your survey and you can collect this data!

## Parsing the JSON Code
When you've collected some data, the JSON string field will look like this:

<img width="746" alt="Screenshot 2023-03-03 at 23 10 27" src="https://user-images.githubusercontent.com/47788764/222881565-2ed5afbe-5fd0-483b-b813-5f18940d206a.png">

In this string, the first value represents the captured second (key), and the second value (after the :) is the slider value recorded during that second (value). This is the structure of the JSON string. 

Now, we'll need to transform this string into separate second-by-second variables so that it's more legible and you can analyze it better. This is where R comes in. With R and the code I provided, you can add new columns for each of these seconds captured in your JSON string. From there, you can keep analyzing that dataset in R or save it as a .csv and import it into SPSS, for instance, if you're more comfortable analyzing data there. Here are the steps to do that:

1. You'll first want to export your Qualtrics data as a .csv and open that in R. 
2. Then, you can use the code attached in the .rmd file to separate all the second-by-second slider data in their own column.

# Multiple Sliders

I haven't used this solution to capture continuous data from multiple sliders in the same Qualtrics survey yet. 

I think this should be possible and you can re-use the code, but you'll have to change the variables "sliderData" and "sliderDataCount". For instance, after the word "slider" in each of these variable names, you can add the number of the slider (e.g., "slider1Data"). Also, don't forget to define these variables in the survey flow.
