

# Introduction
This Javascript code and instructions will allow you to capture slider data from Qualtrics as a continuous response, like dial testing. 

This code adds an event listener to a Qualtrics survey slider that captures the value of a slider question every second and saves it as an embedded data element. The slider values are captured in "sliderData_" variables starting with "sliderData_1" up to however many seconds you record the slider question. It also saves the number of seconds the slider values were captured for in a separate embedded data element: "sliderDataSeconds".

*Note*: I have developed this code to use on a **single slider question** in Qualtrics, like this one below. 

<img width="689" alt="Screenshot 2023-03-03 at 16 53 04" src="https://user-images.githubusercontent.com/47788764/222863475-8f79533b-8b38-4aca-ac9a-fc15735fc61a.png">


I'll provide two solutions to create your continuous slider and accompanying variables: one *easier, but less foolproof* where you need to hand-craft the "sliderData_" variables in Qualtrics and have these immediately visible in the dataset you export, and one *slightly more complex, but also more foolproof* option. The reason why the second one is more complex is becauce it invoves R, which I can understand not everyone may have. However, this being the more foolproof option, allowing to capture as many seconds as you need automatically without having to define them by hand a priori, I would recommend installing R. Also, it's just a great tool!

# How To
## General How To's
### Qualtrics Embedded Data
Since we're saving the slider values as embedded data, we're going to need to define those variables first in the Survey Flow. It's important that these variables are defined **before** the question itself, in this case the slider, is shown to the respondent. That's why we'll create an Embedded Data Block in the Survey flow that precedes the slider block. Like so:

### Where Do I Put The Code? 

In the survey builder, you'll select the slider question and scroll down the left-hand menu of that question. At the bottom, you'll see an option to add Javascript:

<img width="311" alt="Screenshot 2023-03-03 at 17 02 19" src="https://user-images.githubusercontent.com/47788764/222864071-8ff6331d-e173-427b-a9dd-ffa05b5929a5.png">

Click on that, and then delete everything that's there and replace it with the code below. Click save! 

All that's left to do is publish your survey! Good luck. 


### Option with R.
In the option with R, you won't have to create all your sliderData_ second variables by hand, but instead will be encoding these into a JSON object that you can later parse for analysis. 

Instead, you'll include the variables "sliderDataCount" for the total number of seconds the slider was presented, and "sliderData" for the JSON object that'll store all your second-by-second data in a key-value structure. 

<img width="880" alt="Screenshot 2023-03-03 at 23 06 56" src="https://user-images.githubusercontent.com/47788764/222881407-316e89d6-ff93-4f38-a7b6-0fb04ddf1b7a.png">

Look at the "Where Do I put the Code" section above, because you'll be pasting your code in the same place, but the code will be different, see below.

#### The Javascript Code

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

#### Parsing the JSON Code
When you've collected some data, the JSON string field will look like this:

<img width="746" alt="Screenshot 2023-03-03 at 23 10 27" src="https://user-images.githubusercontent.com/47788764/222881565-2ed5afbe-5fd0-483b-b813-5f18940d206a.png">

You can add new columns for each of these seconds captured in your JSON string in R. From there, you can keep analyzing that dataset in R or save it as a .csv and import it into SPSS, for instance, if you're more comfortable analyzing data there. 

You'll first want to export your Qualtrics data as a .csv and open that in R. Then, you can use the code below to separate all the second-by-second slider data in their own column.

##### The R Code

If you copy the R code from here, keep in mind that you'll have to delete all the "//"! To be safe you can always download the file with the R code. 

    //# Get the maximum number of seconds/keys across all rows
        max_seconds <- max(sapply(df$sliderData, function(x) length(fromJSON(x))))

    //# Create an empty data frame with the correct number of columns
        new_cols <- paste0("sliderData_", 1:max_seconds)
        new_df <- data.frame(matrix(ncol = length(new_cols), nrow = nrow(df)))
        colnames(new_df) <- new_cols

            //# Loop through the rows of the data frame
            for (i in 1:nrow(df)) {
          //# Get the JSON data for the current row
          json_data <- df$sliderData[i]
          //# Convert the JSON data to a list
          json_list <- fromJSON(json_data)
          //# Loop through the maximum number of seconds/keys and assign the slider values to the new variables
          for (j in 1:max_seconds) {
            # Check if the current second/key exists in the JSON data
            if (as.character(j) %in% names(json_list)) {
              # Get the value for the current second/key
              slider_val <- json_list[[as.character(j)]]
            } else {
              # Assign NA if the current second/key does not exist in the JSON data
              slider_val <- NA
            }
            # Assign the slider value to the current second/key variable
            new_df[i, paste0("sliderData_", j)] <- slider_val
          }
        }

        //# Combine the new data frame with the original data frame
        new_df <- cbind(df, new_df)



# Multiple Sliders

If you'd like to capture continuous data from multiple sliders, you can re-use the code, but you'll have to change the variables "sliderData_" and "sliderDataSeconds". For instance, after the word sliders in each of these names, you can add the number of the slider (e.g., "slider1Data_" and so on). Also, don't forget to define these variables in the survey flow.
