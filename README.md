# dsg-gtimeline #

This fork builds on the functionality of the gtimeline library by making a number of additions and alterations to the D3.js timeline behavior which are currently exclusivly useful for the [Northeastern DSG](https://dsg.northeastern.edu)'s MHS Lab space. This space is a collobarative project with the [Mass. Historical Society](https://www.masshist.org) and their [Primary Source Co-operative](https://www.primarysourcecoop.org) to create interesting visuazliations with XML-encoded primary source documents. The following changes have been enacted: 

## Changes ## 
1. The tooltip now displays on clicking the timeline entries (instead of hover)
2. The contents of the tooltip have been changed, to only show the year (instead of fulld date) additionally, there is a clickable link that brings users to the entry on the primary source co-operative site.   
3. Added a "Count" vairable for each entry. On the tooltip, this impacts the text in the clickable link, it will be "```Count``` Docs"

      <img width="177" alt="New Tooltip" src="https://github.com/user-attachments/assets/8eb7befe-849d-4393-804b-8a9edab9b7f3" />
      
5. Added an "umbrella" system where entries can fall under another, with the umbrella being bolded and denoted with special characters and center justification.

    <img width="324" alt="Umbrella" src="https://github.com/user-attachments/assets/acb464b2-f7f1-4372-b76e-f0b8947875ed" />

6. Alongside the umbrella system, users can now expand and contract these umbrellas to either show every topic itself or show a summary for the entire umbrella. 

    <img width="1179" alt="Expanded vs Contracted" src="https://github.com/user-attachments/assets/251250a7-45ac-49af-9c28-df77c972b47a" />
    
7. The y axis (both umbrellas and regular topics) are now clickable links that take users to that topic's listing in the MHS PSC.
   
<img width="321" alt="Clickable Words" src="https://github.com/user-attachments/assets/5a80f12c-c791-407a-a187-999cdd91f2c7" />

8. Some general presentation improvements, such as adding tick marks that span the entire chart, and having an x (date) axis on both the top and bottom of the chart, also made it so that the y axis could no longer have its size changed.

## How do I edit this? ## 

first, run ```npm install``` to make sure any (d3) dependancies are instealled. 

To make changes to this library, you need to edit the files in the /src directory. Specifically, I found myself editing the timeline.js, timelineaxis.js, and tooltip.js files, whose name describe what they effect. Then, you have to run ```npm run build``` in the repository. This will place files in /dist, which are useful for browser graphing. 

## Data Format ## 
|                     **Role**                    |                               **Name**                               |     **Start**    |     **End**    |                     **Count**                    |
|:-----------------------------------------------:|:--------------------------------------------------------------------:|:----------------:|:--------------:|:------------------------------------------------:|
| The y axis label the rectangle should appear on | The label that should be added to the rectangle (appears in tooltip) | Start date (UTC) | End date (UTC) | "```Count``` Docs" is the text for the clickable links |

The above table showcase the paramters needed for each rectangular entry on the chart.


### More on The Data.. ### 
Below is an excerpt of the csv data, with each rows' purpose explained. As noted, this libary charts these rectangles in the order they appear in the CSV, which allows this to work. 
<img width="777" alt="TimelineDesc" src="https://github.com/user-attachments/assets/8a41f713-9653-4bc2-bf8e-0dc80482db72" />

**As a quick synposis..** 
The red rows are copies of the data under a particular umbrella. It's only purpose is to handle the animation when something is expanded or contracted. The 2 blue groups each represent one topic row under an umbrella, these appear on their own line. The green lines are inclusive ranges for all the topics under an umbrella. They are plotted in a way that hides the red rows below them. As shown, for somethign to be seen as a "umbrella group" it most follow the bulleted format. 

**And the functinality..**
All the rows start out expanded, when you **collapse** a row the following happens: 
1. All the topic rows under it animate their rectangles up to the umbrella row, the topic rows are hidden. (blue groups) 
2. Those rctangles are hidden (blue groups) 
3. Those rectangles are then moved back to their original positions (blue groups) 
4. The = umbrella ranges are then unhidden on the umbrella line. With the cunulative ones shown over the duplicates. (green groups, red groups)

When you **expand** a row this happens: 
1. The cumulative rectangles are hidden (green groups)
2. The duplicate rectagles are animated down to the correct topic row, as they are shown again. (red groups) 
3. The topic rectangles are shown (blue groups)
4. The duplicate rectangles are hidden and then reset to their original posistion on the umbrella line (red groups)

**Additionally,** the timelein chart takes a paramter of a color list, which handles the color fo every row. So to set all of those entries to be the same color, there must be 21 entries of the same color code. 

All of that eventually becomes the functionality shown in the below gif. 
![TimelineGif](https://media3.giphy.com/media/J6Y4VHR7naYz7Xkolo/giphy.gif)

This is a lot of lines for the relatively small output shown above, and someone attempting to manually create data in this sturcture would likely spend a lot of time and effort to do so. Rather, data processing steps should be completed to do this. For the DSG project, this processing took in indiviual references of this subjects by year, and along with a API JSON of the umbrella -> topic relationship, created the date tanges, as well as all the important duplicates and umbrella rows. It also uses a python script to create the color csv that makes each topic in an umbrella be colored the same. 

## Examples ## 
<img width="1140" alt="Example JQA Timeline" src="https://github.com/user-attachments/assets/b255ff55-a7c7-46cb-8815-f8b035e014e8" />
The above is an image example of this chart that showcases subject references 

* To view online examples of this site in action, visit [https://lab.dsg.northeastern.edu](https://lab.dsg.northeastern.edu)
* To view how to correctly call to run this on a page, view [this file](https://github.com/NEU-DSG/mhs-web/blob/main/src/_includes/visualizations/timeline.njk) in the DSG Lab space.
* To view full example data, click [here](https://github.com/NEU-DSG/mhs-web/tree/main/src/data).

This is a NPM package, to view it, click [here](https://www.npmjs.com/package/dsg-gtimeline)! 

## How would I change this so it works for my project/any project?  ## 

1. 1st and foremost, having a plug and play solution for data processing is essential to make this work for projects. Something like a universal python file that takes either date ranges, or individual dates and create those ranges, should be paired with a JSON file representing Umbrella: list(topics) to create both the csv data and the color list.

2. Removing DSG specificifty from the timeline code

**src/timeline.js**
```
        .on("click", function (event, d) {
          // Cleans text for link 
          const cleanedText = d.replace(/ • /g, "").replace("Topic, ", "");
          const searchSubject = cleanedText.replace(" ", "%20");  // Replace spaces with %20 for the URL

          const url = `https://www.primarysourcecoop.org/publications/${collection}/search#q%3D%2Bsubject%3A%22${searchSubject}%22`;
          window.open(url, "_blank");
```
Found on lines 553 to 559, this takes the data of the y-axis text and uses it as search parameters on the primary source co-operative. Changing the extracted paramters and the base URL, or making them arguments would allow you to change where this link points to. To catch an edge-case, this logic is repeated on lines 645 to 655.



```
 function tooltip_html(event, d) {
    const format = pipe(d3.isoParse, d3.timeFormat("%Y"));

    // Header with the name and a horizontal rule
    const header = `<b>${names(d)}</b><hr style="margin: 2px 0 2px 0">${format(starts(d))}`;

    // Body with the formatted end date and duration (if applicable)
    const body = ends(d) - starts(d)
      ? ` - ${format(ends(d))}, ${durationFormat(starts(d), ends(d))}`
      : "";

    // Generate the dynamic URL for the docs link
    const subjectTitle = String(d[1]);
    let searchSubject = subjectTitle.replace(/ /g, "%20").replace("Topic,%20", "");

    const startDateStr = formatDate(d[2]);  // Format start date
    const endDateStr = formatDate(d[3]);    // Format end date

    const url = `https://www.primarysourcecoop.org/publications/${collection}/search#q%3D%2Bsubject%3A%22${searchSubject}%22%20%2Bdate_when%3A%5B${startDateStr}%20TO%20${endDateStr}%5D%7Crows=20%7Cstart=0%7Chl=true%7Chl.fl=text_merge%7Csort=date_when%20asc%7Cff=person_keyword;subject%7Cfl=id%20index%20title%20filename%20resource_group_name%20date_when%20date_to%20author%20recipient%20person_keyword%20subject%20doc_beginning`;

    // Add the clickable link to the tooltip
    const link = `<br><a href="${url}" target="_blank">${d[4]} Docs</a>`;
    // Combine the header, body, and link into the final tooltip content
    return `${header}${body}${link}`;
  }
}
```
This function, at the very end of the file create the html for each tooltip, which has DSG specific ajustments with the link. 
Specifically, 
* On line 817, ```d[4]``` is accessing the "Count" variable for that rectangle. Which may not be applicable. You could either replace this link text with just a message like "Click here" or change the "Count" column to something relavent in your data
* Lines 919 to 814 extract the dates and name of the rectangle and then use that to fill a MHS PSC search query. You can edit this to make it point to your own URL, and change the extracted search paramters to something needed for your search. 

   

