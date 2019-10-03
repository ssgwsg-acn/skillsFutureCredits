
# SkillsFuture Credit (SFC) Payment Gateway Services

Guidelines and interface specifications to the SkillsFuture Credit (SFC) Payment Gateway. This document is intended for the following users from Training Providers (TPs) for their e-Services:

•	Functional Users who require instructions on preparing their systems for technical integration with the SFC Payment Gateway <br>
•	Business Users who require an overview of the technical processes


The SFC Payment Gateway provides users with the opportunity to use their SFC at the point of registration for the course. During the registration process, users will be redirected to the SFC Payment Gateway to indicate the amount of credit they would like to use to offset their payment. 
The figure below shows a summary of the process and the steps to be taken by e-Services

<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_process.png">
</p>

#### STEP 1 - CREATING THE REQUEST PAYLOAD
The e-Service will create a request payload, which comprises of information (e.g. Course Start Date and Course Fee) that is necessary for SSG to allow the user to use their SkillsFuture Credit.
<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_processS1.png">
</p>

With the SFC Payment Gateway, users will be redirected to the payment page for an option to utilise their credit to offset against their course fees. e-Services must prepare a set of information, known as the payload, that is required by SSG to validate the user’s request to use their SFC.
The following sections will provide technical information regarding the requirements of the request payload. 

FIELD ATTRIBUTES

|Parameters|Mandatory|Type/Length|Description|
|--- |--- |--- |--- |
|NRIC|Y|String/9|To validate if the same NRIC is used for SingPass login|
|Mobile Number*|N|String/8|To pre-populate in contact details in SFC Payment Gateway only if the information is not available|
|Home Number*|N|String/8||
|Email Address*|N|String/241||
|Course ID|Y|String/50|External course reference number|
|Course Run ID|N|String/50|External course run reference number|
|Course Start Date|Y|String/10|Start date of the registered course. The format must be: yyyy-mm-dd|
|Course Fee Payable|Y|String/9|Net amount (including GST) of the course to validate against SFC Payment amount. The format must be to 2 decimal placing|
|Additional Information|N|String/512|For Training Provider's reference|

\* Note that for the initial claim request, contact information shared by the e-Service will be pre-populated if the user has not transacted with SSG/WSG before.

#### STEP 2 – CALLING THE ENCRYPTION API
To cater for increased security and confidentiality of data, all data must be encrypted before being transferred to and from government organisations. Given that the request payload will contain sensitive information (such as NRIC), the request payload must be encrypted before being submitted to SSG. At the same time, encryption ensures the integrity of the data transactions between systems since they are sent using trusted certificates.

<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_processS2.png">
</p>

Once the e-Service has the required information, it can map the details to the encryption API request. The details about the encryption API are available in the [swagger](https://developer.ssg-wsg.sg/webapp/docs/product/7KU1xrpxljJZnsIkJP6QNF/group/2RTLOUTuE3Dkgf7MOdn0Cm#).The API will then encrypt the request payload using Trust Certificates provided by SSG. This is to ensure that the data transfer between the e-Service and SSG is secure and all information is kept confidential.
From the details gathered in Step 1, the e-Service should map the information to the structure given below and call the encryption API.
```
{
    "claimRequest": {
        "individual":  {
            "nric":  "T5001072J",
            "mobileNumber":  98760000,
            "homeNumber":  87654321,
            "email":  "someone@example.com"
        },  
        "course":  {  
            "id":  "STMI-CS-CSXY",  
            "runId":  "C123",  
            "startDate":  "2019-05-22",  
            "fee":  12  
        },  
        "additionalInformation":  "This is additional information"  
    }  
}
```

#### STEP 3 – SENDING THE ENCRYPTED REQUEST
<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_processS3.png">
</p>

After encrypting the request payload, the e-Service should send the encrypted payload to the SFC Payment Gateway via a form post. This would allow the SFC Payment Gateway to receive the necessary information and continue the user’s application to submit SFC claims. 

#### STEP 4 – DECRYPTING THE ENCRYPTED RESPONSE
<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_processS4.png">
</p>

After the user has completed the SFC claim process, the user will be redirected back to the e-Service, together with an encrypted response payload. The e-Service should then call the decryption API to decrypt the encrypted response. The decryption API details are available in the [swagger](https://developer.ssg-wsg.sg/webapp/docs/product/7KU1xrpxljJZnsIkJP6QNF/group/2RTLOUTuE3Dkgf7MOdn0Cm#). The response of the decryption API will look like:
```
{
  "status": 200,
  "data": {
    "claimRequestStatus": {
      "transactionStatus": "X",
      "individual": {
        "nric": "T5001072J"
      },
      "course": {
        "id": "STMI-CS-CSXY",
        "runId": "C123",
        "startDate": "2019-05-22"
      },
      "claim": {
        "id": "2000217512",
        "amount": "1.0",
        "date": "2019-09-13"
      },
      "additionalInformation": "This is additional information"
    }
  },
  "errors": []
}
```

#### Step 5 – READING THE RESPONSE PAYLOAD
<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_processS5.png">
</p>

The e-Service will read the response payload after decryption. The response payload comprises of information that is necessary for the e-Service to continue the registration process with the user and continue the payment process for any outstanding amount.

#### Step 6 – CONSUMING APIs FOR FURTHER PROCESSING
<p align="center">
  <img src="https://github.com/ssgwsg-acn/TestPayment/raw/master/img/payment_processS6.png">
</p>

The e-Service will upload supporting documents to SSG to support the SFC claim by the user. The e-Service may also choose to view or cancel the user’s claim. All these functionalities are made available as APIs to the consuming e-Services.

The successful submission of claims by users is dependent on the completion of payment and the upload of supporting documents. All claim submissions are subjected to approval by SSG.<br><br>
