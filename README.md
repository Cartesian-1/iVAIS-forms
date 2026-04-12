# Introduction

Do you need to run mega surveys with very many questions - too many for one person to answer? These tools lets you split one large survey with any number of questions into smaller forms and provides mechanisms for intelligent participant distribution and collecting responses.  

This is Google Cloud Platform Project using Google Apps Script, Google Sheets, and Google Forms. It could run on a personal Google account, depending on local area access, storage, quotas, and usage limits. This is an unofficial tool project and is not affiliated with Google.  

These tools were originally created as part of the iVAIS project.

# The main survey sheet

The main survey sheet is where all data is stored: survey questions, forms data, form redirects, and responses.  

Create a Google Sheet with three tabs named: `Forms`, `Questions`, and `Responses`.  

- The `Forms` tab must have these columns:  
`FormKey`	`TemplateFormId`	`Title`	`Description`	`FormId`	`EditUrl`	`LiveUrl`	`FinalThankYouMessage`	`Answers`	`Respondents`	`Redirects`	`MaxRedirects`	`LastRedirect`	`MaxRespondents`	`FailedControl`	`StillNeeded`  
The first 8 columns are forms data, and the last 8 columns are for handling redirect logic and responses.
The forms you want to build must be specified here including metadata.  
Specify in `MaxRespondents` how many respondents you need for each form.

- The `Questions` tab must have these columns:  
`FormKey`	`QuestionID`	`QuestionType`	`OtherOption`	`Validation`	`NewPage`	`QuestionText`	`Answer Option 1`	`Answer Option 2`	`Answer Option 3`  
All questions and answer options must be added here.   
Questions will be added to the forms you specify in `FormKey`, in the order they appear in the sheet rows.  

- The `Responses` tab must have these columns:  
`FormKey`	`QuestionID`	`RespondentId`	`Timestamp`	`Answer`  
Response data will be updated periodically and automatically when the surveys runs.

# The forms-main script

The `forms-main` script handles creating forms and collecting responses from answered forms.

Open the main survey sheet, go to Extensions > Apps Script and create a new Google Apps Script project.  
Replace the code with the code from `cartesian-forms-main.txt` and save. Do NOT click Deploy.  

Under Project Settings, make sure Chrome V8 runtime is enabled.  

Under Script Properties, set property `HOST_SPREADSHEET_ID` to the ID of the main survey sheet: the key after /spreadsheets/d/ in the URL.  

Google might ask you to allow some Project OAuth Scopes permissions on first run.   

# The forms-url script

The `forms-url` script is deployed as a web app with one single purpose: to redirect users to one of your survey form URLs.

Open the main survey sheet, go to Extensions > Apps Script and create a new Google Apps Script project.  
Replace the code with the code from `cartesian-forms-url.txt` and save.

Under Project Settings, make sure Chrome V8 runtime is enabled.  

Under Script Properties, set property `SPREADSHEET_ID` to the ID of the main survey sheet: the key after /spreadsheets/d/ in the URL.  

Deploy as type: Web app. Execute as: Me (your account). Who has access: Anyone.

Google might ask you to allow some Project OAuth Scopes permissions on first run.  

Under Deploy > Manage deployments > Web app URL you can copy the web app URL.  
This is the URL that must be shared with your users (and no one else) when they take your survey.

The `forms-url` web app will automatically redirect users to one of the forms in the `Forms` tab of the main survey sheet.  
One redirect form is selected by the following logic:  
-Form with lowest `Answers` number in the `Forms` tab of the main survey sheet (all forms will have 0 initially).  
-Form with lowest `FormKey` number.  
-`MaxRedirects` must be strictly greater than `Redirects` for a form to be considered for a redirect.  
-If no form can be chosen, the web app returns an error message.  

# Building forms

- Specify all forms metadata in the `Forms` tab of the main survey sheet:

`FormKey`	 For each form you want to build, specify for each row a unique natural number in a rising sequence.  
`TemplateFormId`	If you want to use a template for visual style, create a Google Forms template in the style you want, publish it, and enter the key after /spreadsheets/d/ in the template live URL.    
`Title`	 Enter the forms title.  
`Description`	 An introductionary text on the first page of the form.  
`FormId`	A unique form identifier. Leave this empty. It will be filled automatically by the `forms-main` script.  
`EditUrl`	The URL to edit the Google Form. Leave this empty. It will be filled automatically by the `forms-main` script.  
`LiveUrl`	The live URL to answer the Google Form. This is not normally used and should NOT be shared with users. Leave this empty. It will be filled automatically by the `forms-main` script.  
`FinalThankYouMessage` An appreciative text message on the last page of the form.  

- Add all your survey questions to the `Questions` tab in the main survey sheet:  

`FormKey`	The number you specify here is the form the question goes into.  
`QuestionID` A unique question identifier.  
`QuestionType` Currently supported question types are `Short Answer Text` and `Multiple Choice` with up to three answer options.  
`OtherOption`	Set to `TRUE` if the question is `Multiple Choice` with an Other (write in) option. Set to `FALSE` otherwise.  
`Validation`	Set to `Number` if a `Short Answer Text` question answer field has number validation. Leave empty otherwise.  
`NewPage`	Set to `TRUE` to insert a page break after this question. Set to `FALSE` to stay on the current page.  

`QuestionText`	Your question text.  

`Answer Option 1`	`Answer Option 2`	`Answer Option 3` The answer options for `Multiple Choice` questions.  
Leave all three empty for `Short Answer Text` questions.  
If a `Multiple Choice` question has fewer than three answer options, leave `Answer Option 2` and/or `Answer Option 3` empty.  

- When all forms metadata and all questions are ready, you can start building the Google Forms:

The following functions are called manually from the functions menu of the `forms-main` script.

`BuildFormsFromSheet` 
This tries to build all the forms you have specified in the `Forms` tab, with the questions you have specified in `Questions`. 
It is generally NOT recommended to run this, unless you have a small number of forms. It is likely to exceed the maximum time allowed for a function to run.  

`buildNext10Forms` `buildNext2Forms` `buildNext3Forms` `buildNext5Forms`  
These will build the next N forms you have specified in the `Forms` tab, with the questions you have specified in `Questions` tab.  
`buildNext10Forms` is generally recommended for most purposes.

`resetBuildBatchProgress`  
 Not normally used. This can be used to restart the building progress measure, if you need to restart the form building process.  

`countGeneratedAndPublishedForms`  
This can be used to check if all forms have been generated and published. Recommended to run before going live.



# Running the survey

`resetResponseSyncState`  
`setSyncBatchSize_25`  
`setSyncBatchSize_3`  
`getSyncBatchSettings`  
`enable10minSync`  
`disable10minSync`  
