# iVAIS-forms

If you need to run surveys with very many questions - too many for one person to answer - these tools lets you split one large survey with any number of questions into smaller forms, and provides mechanisms for controlling participant distribution and collecting responses.  

This is a Google Cloud Platform Project using Google Apps Script, Google Sheets, and Google Forms. It could run on a personal Google account, depending on local area access, storage, quotas, and usage limits. This is an unofficial tool project and is not affiliated with Google.  

These tools were created as part of the iVAIS project.

# The main survey sheet

**The main survey sheet is where all data is stored: survey questions, forms data, redirects, and responses.**    

Create a Google Sheet with three tabs named: `Forms`, `Questions`, and `Responses`.  

**The `Forms` tab must have these column headers (A1-P1):**  
`FormKey`	`TemplateFormId`	`Title`	`Description`	`FormId`	`EditUrl`	`LiveUrl`	`FinalThankYouMessage`  
`Answers`	`Respondents`	`Redirects`	`MaxRedirects`	`LastRedirect`	`MaxRespondents`	`FailedControl`	`StillNeeded`  
The first 8 columns are forms data, and the last 8 columns are for handling redirect logic and responses.

The `Answers` column must contain this formula (I2 example): `=COUNTIF(Responses!$A:$A,$A2)`  
The `Respondents` column must contain this formula (J2 example): `=$I2/N` where N is the number of questions in the form (including background questions, etc.).  
The `StillNeeded` column must contain this formula (P2 example): `=MAX($N2-$J2+$O2,0)`  

**The `Questions` tab must have these column headers (A1-J1):**   
`FormKey`	`QuestionID`	`QuestionType`	`OtherOption`	`Validation`	`NewPage`	`QuestionText`  
`Answer Option 1`	`Answer Option 2`	`Answer Option 3`  
All questions and answer options must be added here.   
Questions will be added to the forms you specify in `FormKey`, in the order they appear in these rows.  

**The `Responses` tab must have these column headers (A1-E1):**    
`FormKey`	`QuestionID`	`RespondentId`	`Timestamp`	`Answer`  
Response data will be updated here periodically and automatically when the surveys runs.

# The forms-main script

**The `forms-main` script handles creating forms and collecting responses from answered forms.**  

Open the main survey sheet, go to Extensions > Apps Script and create a new Google Apps Script project.  
Replace the code with the content of `iVAIS-forms-main.txt` and save. Do NOT click Deploy.  

Under Project Settings, make sure Chrome V8 runtime is enabled.  

Under Script Properties, set property `HOST_SPREADSHEET_ID` to the ID of the main survey sheet: the key after /spreadsheets/d/ in the URL.  

Google might ask you to allow some Project OAuth Scopes permissions on first run.   

# The forms-url script

**The `forms-url` script is deployed as a web app with one purpose: Redirect users to a form in the `Forms` tab.**  

Open the main survey sheet, go to Extensions > Apps Script and create a new Google Apps Script project.  
Replace the code with the content of `iVAIS-forms-url.txt` and save.

Under Project Settings, make sure Chrome V8 runtime is enabled.  

Under Script Properties, set property `SPREADSHEET_ID` to the ID of the main survey sheet: the key after /spreadsheets/d/ in the URL.  

Deploy as type: Web app. Execute as: Me (your account). Who has access: Anyone.

Google might ask you to allow some Project OAuth Scopes permissions on first run.  

**Survey URL** 

Under Deploy > Manage deployments > Web app URL you can copy the web app URL.  
This is the URL that must be shared with your users (and no one else) when they answer your survey.

The `forms-url` web app will automatically redirect users to one of the forms in the `Forms` tab of the main survey sheet.  

**Redirect logic**  

One form in `Forms` is selected for a redirect by the following logic:  
-form with lowest `Answers` number in the `Forms` tab of the main survey sheet (all forms will have 0 initially).  
-form with lowest `FormKey` number.  
-`MaxRedirects` must be strictly greater than `Redirects` for a form to be considered for a redirect.  
-if no form can be chosen, the web app returns an error message.  

# Building the forms

**Specify all forms metadata in the `Forms` tab of the main survey sheet:**  

This description is based on 1,000 different questions plus 6 background questions divided between 100 forms with 16 questions each. Larger volumes are possible, but results may vary due to timing and cloud platform limitations.  

`FormKey`	 For each form you want to build, specify for each row a unique natural number in a rising sequence.  
`TemplateFormId`	If you want to use a template for visual style, create a Google Forms template in that style, publish it, and enter the key after /spreadsheets/d/ in the template live URL.    
`Title`	 The form title.  
`Description`	 An introductionary text on the first page of the form.  
`FormId`	A unique form identifier. Leave this empty. It will be filled automatically by the `forms-main` script.  
`EditUrl`	The URL to edit the Google Form. Leave this empty. It will be filled automatically by the `forms-main` script.  
`LiveUrl`	The live URL to answer the Google Form. This is not normally used and should NOT be shared with users. Leave this empty. It will be filled automatically by the `forms-main` script.  
`FinalThankYouMessage` An appreciative text message on the last page of the form.  

**Add all your survey questions to the `Questions` tab in the main survey sheet:**    

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

**Control question**  

In case you have a control question with some `FormKey` (say 87) designed to disqualify some respondents (if they answer say 1 or 3), the `FailedControl` column in the `Forms` tab must contain this formula (O2 example):  
`=COUNTIFS(Responses!$A:$A,$A2,Responses!$B:$B,87,Responses!$E:$E,1)+COUNTIFS(Responses!$A:$A,$A2,Responses!$B:$B,87,Responses!$E:$E,3`

**When all forms metadata and all questions are ready, you can start building the Google Forms:**  

The following functions are called manually from the functions menu of the `forms-main` script.

`BuildFormsFromSheet` 
It is generally NOT recommended to run this function, unless you have a small number of forms. It tries to build all the forms at once, and it is likely to exceed the maximum time allowed for a function to run.  

`buildNext10Forms` `buildNext5Forms` `buildNext3Forms` `buildNext2Forms`      
These functions will build the next 10, 5, 3, or 2 forms you have specified in the `Forms` tab, with the questions you have specified in `Questions` tab. Running `buildNext10Forms` repeatedly is generally recommended for most building purposes.

`resetBuildBatchProgress`  
Not normally used. Can be used to restart the building progress measure, if you need to restart the form building process.  

**When all forms have been built, they must be published:**    

Currently there is no way to publish forms automatically. You must click the `EditUrl` of each form and publish it manually.  
Run the function `countGeneratedAndPublishedForms` to check if all forms have been generated and published.

# Running the survey

Specify in the `MaxRespondents` column how many respondents you need for each form.  

**Automated synchronization**

Run the function `enable10minSync` in `forms-main` one hour before you start the survey. This will start the automated synchronization of responses which runs with a time based trigger every 10 minutes. 

Completed and submitted forms are added to the `Responses` tab in the main survey sheet. Partially answered forms are ignored.

The default batch size of forms that each 10-minute synchronization updates is 20. The batch size can be changed to 3 or 25 by running the functions `setSyncBatchSize_3` or `setSyncBatchSize_25`.  
Run the function `getSyncBatchSettings` if you want to know the current progress and batch size.  

Only new responses will be synchronized. If you wish to re-synchronize all responses, run `resetResponseSyncState`.  

**Start the survey** 

Invite the respondents to your survey with the single `forms-url` web app URL link as your survey invitation link.  

When your users visit the web app URL, they will be redirected to one of the forms in the `Forms` tab. 
Each time this happens, the `forms-url` script will update the columns `Redirects` and `LastRedirect` (timestamp).  

The automated synchronization will update `MaxRedirects` for all forms in a batch to allow more redirects if and only if:  
-`StillNeeded` is greater than 0, and  
-at least 30 minutes has passed since the `LastRedirect` timestamp.  

**Stop the survey** 

The automated synchronization will stop if all values in the `StillNeeded` column are 0, or if you run the function `disable10minSync`.
