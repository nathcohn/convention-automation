# convention-automation

## Description

In the Hubspot CRM, the primary object "Transaction" is used to define a course taken by a certain student with details about it (start_date, end_date...). Here, for each Transaction we create a convention that is sent to the student's email for him to sign it and send it back. This is done through the help of the Pandadoc API, allowing to fill out predefined convention models and sending them to emails. The models are models Datascientest created, that we can find on the account the company has on Pandadoc.

## Data
### Input:
Transaction object (on Hubspot)
### Output
No output, but: 
  - few variables are updated in the Transaction after the convention is sent. They are parameters that define the course taken by the student that we get from a DynamoDB and the "convention link" and the "convention status" ("sent" to the student). Here they are:
    - objective
    - total_hours
    - pre_requisites
    - hours_spread
    - end_of_formation_document_list
    - convention_link
    - convention_status
  - an email is sent to the student with the convention filled out with his details and the details of the course he takes.

## Routes
- GET createConvention

## Technologies
- Node.js 16.x
- AWS
- Hubspot
- Pandadoc API

## How to Install and Run the Project
- Client side (Hubspot):
  - Create a workflow. The workflow will be triggered when "Phase de la transaction est l'un des Financing in progress - La closance (Deals - B2C)".
  - As a second action, add a code section where you will copy paste the code in hubspotCreateConvention.js. Add there:
    - in the ```Secrets``` section, the secret ```DEV_ACCESS_TOKEN```.
    - in the ```Properties to include to the code``` field, include all the variables that are of type ```event.inputFields['var']``` (where "var" is the property to include).

- Server side:
-   create a lambda function on AWS.
  - You need to create/use an IAM role that permits to connect the function to an API Gateway (in order for the client to communicate with it), to a read and write to a DynamoDB and that has access to "Secrets Manager".
  - Link to it an API Gateway. 
  - Copy paste the code in createConvention.js.
  - The lambda functions require external packages to be added as layers in order for the code to use them. The different packages needed to run are: axios and aws-sdk. In order to use each package, you need to dowload it and zip it. You then add a layer to the lambda function, where you will upload the zipped folder.

## Tests
You can either:
- test the lambda functions directly on the console:
  - example of input: 
    ```
    {
      "courseType": "DST - Machine Learning Engineer",
      "startDate": "1698811200000",
      "endDate": "1699811200000",
      "courseLanguage": "French",
      "amount": "4",
      "format": "Bootcamp",
      "dealname": "Nathaniel Cohn",
      "address": "70 boulevard Flandrin",
      "zip": "75116",
      "city": "Paris",
      "country": "France",
      "firstname": "Nathaniel",
      "lastname": "Cohn",
      "email": "nathaniel.c.ext@datascientest.com",
      "dateOfBirth": "1993-02-27"
    }
    ```

  - Note:
    - Not all the variables are included in the previous examples, they are here just to show the structure of the input. 
    - In order to run a test on the AWS console, remove the field ['queryStringParameters']" since you are not receiving those parameters from a client through a query but locally. Practically, instead of:
    ```
      courseType = event['queryStringParameters']['courseType']
      format = event['queryStringParameters']['format']
      startDate = event['queryStringParameters']['start_date']
    ```
    you should write:
    ```
      courseType = event['courseType']
      format = event['format']
      startDate = event['start_date']
    ```
- or test it from Hubspot (meaning sending a request from hubspot). For example:
  - Create a Transaction for Martin Dupont that takes the DataScientist course etc (fill out all the other necessary variables, that are of type ```event.inputFields['var']`` where "var" is the variable). Link it to the correspponding Contact (Martin Dupont). If it is a B2B convention, you also need to:
    - link the Transaction to the corresponding "Entreprise" (company).
    - create a "Financing" of type "Corporate Funding" that you will link to the Transaction. It defines the amount the company is ready to pay for this student.
    - if the student also pays a part of the course, create other "Financing"(s) corresponding to the types of payments the student will use to pay the rest of the course (for example Personal Funding, CPF etc) and link them to the Transaction. For "Contact", "Entreprise" and "Financing", check in the Hubpost code which variables we get from them in order to know which ones to fill.
  - Create a workflow as previously explained.
  - Run the code, by clicking on "test" and selecting the Financing you created.
  - If the status is "Success", you can verify if the code worked properly. Click on "Afficher la Transasction" and on the new page, look for the "Afficher toutes les proprietes" button and click on it. You will then search in the Search bar for the "Convention Link" and the other properties that are supposed to be filled out and see if they are filled. If yes, check that they are correct (for example, check that the url of "Convention Link" leads to the convention you intended to create) and check that the email was sent with the correct filling out of the convention.
  - Note: you do not need to remove the field ['queryStringParameters']", since you are not receiving those parameters from a client through a query.
