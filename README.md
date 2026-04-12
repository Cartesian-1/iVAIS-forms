# Introduction

What if you need to run a survey with very many questions - too many for one person to answer? These tools lets you split a large survey with any number of questions into smaller forms. It also provides mechanisms for intelligent participant distribution to, and collecting responses from those surveys.  

This is Google Cloud Platform Project using Google Apps Script, Sheets, and Forms. It could run on a personal Google account. This is an unofficial tool project and is not affiliated with, endorsed by, or sponsored by Google.  

These tools were originally created as part of the iVAIS project.

# The main survey sheet

Create a Google Sheet with three tabs named: `Forms`, `Questions`, and `Responses`.  

The Forms tab must have these columns:  
`FormKey`	`TemplateFormId`	`Title`	`Description`	`FormId`	`EditUrl`	`LiveUrl`	`FinalThankYouMessage`	`Answers`	`Respondents`	`Redirects`	`MaxRedirects`	`LastRedirect`	`MaxRespondents`	`FailedControl`	`StillNeeded`  

The Questions tab must have these columns:  
`FormKey`	`QuestionID`	`QuestionType`	`OtherOption`	`Validation`	`NewPage`	`QuestionText`	`Answer Option 1`	`Answer Option 2`	`Answer Option 3`  
Add all your questions and answer options here. Supported question types are Multiple Choice and Short Answer Text.  
Questions will be added to the forms you specifiy in FormKey, in the order they appear here.

The Responses tab must have these columns:  
`FormKey`	`QuestionID`	`RespondentId`	`Timestamp`	`Answer`  
Response data will be updated periodically and automatically when the surveys runs.

# The forms-main project

Create a Google Apps Script project 
