# Introduction

What if you need to run a survey with very many questions - way too many for one person to answer? This project lets you split any number of questions into smaller forms. It also provides mechanisms for intelligent participant distribution and collecting responses.  
This is a Google Cloud Platform Project using Google Apps Script, Sheets, and Forms. It could run on a personal Google account. This is an unofficial project and is not affiliated with, endorsed by, or sponsored by Google.

# The main Google Sheet

Create a Google Sheet with three tabs named: Forms, Questions, and Responses.  

The Forms tab must have these columns:  
FormKey	TemplateFormId	Title	Description	FormId	EditUrl	LiveUrl	FinalThankYouMessage	Answers	Respondents	Redirects	MaxRedirects	LastRedirect	MaxRespondents	FailedControl	StillNeeded  

The Questions tab must have these columns:  
FormKey	QuestionID	QuestionType	OtherOption	Validation	NewPage	QuestionText	Answer Option 1	Answer Option 2	Answer Option 3  
Add any number of questions and answer options. Supported question types are Multiple Choice and Short Answer Text.  
Questions will be added to the Google Form you specifiy in FormKey, in the order they appear in Questions tab.

The Responses tab must have these columns:  
FormKey	QuestionID	RespondentId	Timestamp	Answer  
Response data will be updated automatically here when the surveys runs.
