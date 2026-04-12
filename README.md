# Introduction

Do you need to run mega surveys with very many questions - too many for one person to answer? These tools lets you split one large survey with any number of questions into smaller forms and provides mechanisms for intelligent participant distribution and collecting responses.  

This is Google Cloud Platform Project using Google Apps Script, Google Sheets, and Google Forms. It could run on a personal Google account, depending on local area access, storage, quotas, and usage limits. This is an unofficial tool project and is not affiliated with Google.  

These tools were originally created as part of the iVAIS project.

# Creating the main survey sheet

The main survey sheet is where all data is stored: survey questions, forms data, form redirects, and responses.  

Create a Google Sheet with three tabs named: `Forms`, `Questions`, and `Responses`.  

The Forms tab must have these columns:  
`FormKey`	`TemplateFormId`	`Title`	`Description`	`FormId`	`EditUrl`	`LiveUrl`	`FinalThankYouMessage`	`Answers`	`Respondents`	`Redirects`	`MaxRedirects`	`LastRedirect`	`MaxRespondents`	`FailedControl`	`StillNeeded`  
The first 8 columns are forms data, and the last 8 columns are for handling redirect logic and responses.

The Questions tab must have these columns:  
`FormKey`	`QuestionID`	`QuestionType`	`OtherOption`	`Validation`	`NewPage`	`QuestionText`	`Answer Option 1`	`Answer Option 2`	`Answer Option 3`  
All questions and answer options must be added here.   
Questions will be added to the forms you specifiy in FormKey, in the order they appear in the sheet rows.  

The Responses tab must have these columns:  
`FormKey`	`QuestionID`	`RespondentId`	`Timestamp`	`Answer`  
Response data will be updated periodically and automatically when the surveys runs.

# Creating the forms-main script

The forms-main script handles creating forms and collecting responses from answered forms.

Open the main survey sheet, go to Extensions > Apps Script and create a new Google Apps Script project.  
Replace the code with the code from cartesian-forms-main.txt and Save. Do NOT click Deploy.  

Under Project Settings, make sure Chrome V8 runtime is enabled.  

Under Script Properties, set property `HOST_SPREADSHEET_ID` to the ID of the main survey sheet: the key after /spreadsheets/d/ in the URL.  

Google might ask you to allow the following Project OAuth Scopes on first run:  
-Allow this application to run when you are not present  
-See, edit, create, and delete all of your Google Drive files  
-View and manage your forms in Google Drive  
-See, edit, create, and delete all your Google Sheets spreadsheets  

# Creating the forms-url script

The forms-url script is deployed as a Web App with one single purpose: to redirect users to one of our survey forms.

Open the main survey sheet, go to Extensions > Apps Script and create a new Google Apps Script project.  
Replace the code with the code from cartesian-forms-url.txt and Save.

Under Project Settings, make sure Chrome V8 runtime is enabled.  

Under Script Properties, set property `SPREADSHEET_ID` to the ID of the main survey sheet: the key after /spreadsheets/d/ in the URL.  

Deploy as type: Web app. Execute as: Me (your account). Who has access: Anyone.

Google might ask you to allow the following Project OAuth Scopes on first run:  
-See, edit, create, and delete all your Google Sheets spreadsheets  

Under Deploy > Manage deployments > Web app URL you will find the web app URL.  
This is the URL that must be shared with your users (and no one else) when they take your survey.

The forms-url web app with automatically redirect users to one of the forms in the Forms tab of the main survey sheet.  
One redirect form is chosen by the following logic:  
-Form with lowest `Answers` number (all forms will have 0 initially)  
-Lowest `FormKey` number.  
-`MaxRedirects` must be greater than `Redirects`, for any form to be considered for a redirect.  
-If no form can be chosen, the web app gives an error message.  

# Building forms

Currently supported question types are Short Answer Text and Multiple Choice with up to 3 answer options.



# Running the survey
