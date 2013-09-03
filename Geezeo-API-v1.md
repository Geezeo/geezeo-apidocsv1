#Geezeo REST API

##Overview

The Geezeo REST API provides a method of read and partial write access to a users PFM data. This API is for Partner use only, and is not designed or intended for use by the end user. In addition, security is limited to a Partner-wide access token at this time, so care should be taken that the token is not exposed to an end user.

There are several types of data that are available via the API. They include:

* Accounts
* Transactions
* Cashflow Bills & Transactions
* Budgets
* Expenses (spending)
* Goals
* Users

Every request is scoped to a single user. There is no method to receive data for multiple users within a single request.

##Technical Overview

As this is a REST based API, all request are standard HTTP requests. Read requests are GETs.

In order to make a request, the following pieces of data are required:

* API endpoint
* Partner API key
* User's unique identifier (partner customer id)
* Specific URI for the object type

The current API endpoint is: https://[api key]@[partner domain]/api/v1/

This endpoint will be referred to as [base] in the documentation below.

The Partner API Key will be provided to you by Geezeo.

The specific URI for each object type is below.

The format of the data returned can either be XML or JSON. To specify which format is desired, end the URI with either ".json" or ".xml". For example, https://[partner domain]/api/v1/users/1.json

##Objects

###User

URI: [base]/users/[user id].[format]

####Read Example

    curl http://abc123@geezeo.dev/api/v1/users/1.xml
    
    <?xml version="1.0" encoding="UTF-8"?>
    <user>
    <id type="integer">1</id>
    <partner-customer-id nil="true"></partner-customer-id>
    <first-name>Dev</first-name>
    <last-name>Devidson</last-name>
    </user>

####Write Example

    curl -d "user%5Bemail%5D=user%40host.com&user%5Blogin%5D=abc123def&user%5Bpartner_customer_id%5D=testapiuser001&user%5Bpassword%5D=password&user%5Bpassword_confirmation%5D=password&user%5Burl%5D=mysupserusername&user%5Bchallenge_question%5D%3D5678&user%5Bchallenge_answer%5D%3D12345" http://abc123@geezeo.dev/api/v1/users/

Response:

    {
      "message": {
        "message": "User created."
      }
    }

Required field:

* email
* login
* partner_customer_id
* password
* password_confirmation
* url
* challenge_question
* challenge_answer

If there are errors, a response describing all the errors will be returned, similar to the following:

    {
      "error": {
        "message": "Error creating User",
        "data": {
          "partner_customer_id": "has already been taken",
          "url": "is already taken"
        }
      }
    }

###Accounts

URI: [base]/users/[user id]/accounts.[format]

####Read Example

    curl http://abc123@geezeo.dev/api/v1/users/1/accounts.xml

    <?xml version="1.0" encoding="UTF-8"?>
    <accounts type="array">
      <account>
        <id type="integer">1</id>
        <name>eChecking</name>
        <balance type="decimal">300.54</balance>
        <reference-id>789274930</reference-id>
        <aggregation-type>partner</aggregation-type>
        <state>active</state>
      </account>
      <account>
        <id type="integer">2</id>
        <name>Complete Savings</name>
        <balance type="decimal">1000.0</balance>
        <reference-id>278914032</reference-id>
        <aggregation-type>partner</aggregation-type>
        <state>active</state>
      </account>
    </accounts>

####Deleting accounts

Make a HTTP DELETE request to the accounts URI (from above).

###Transactions

URI: [base]/users/[user id]/accounts/[account id]/transactions.[format]

#### Read Example

    curl http://abc123@geezeo.dev/api/v1/users/1/accounts/1/transactions.xml

    <?xml version="1.0" encoding="UTF-8"?>
    <hash>
      <pages>
        <total-pages type="integer">1</>
        <current-pages type="integer">1</>
      </pages>
      <transactions type="array">
        <transaction>
          <id type="integer">1540</id>
          <reference-id nil="true"></reference-id>
          <transaction-type>Debit</transaction-type>
          <memo>GameStop</memo>
          <balance type="decimal">59.99</balance>
          <posted-at type="datetime">2012-05-24T00:00:00Z</posted-at>
          <created-at type="datetime">2012-05-24T19:24:07Z</created-at>
          <nickname>GameStop</nickname>
          <original-name>GameStop</original-name>
          <check-number nil="true"></check-number>
          <deleted-at nil="true"></deleted-at>
          <tags type="array">
            <tag>
              <name>Entertainment</name>
              <balance type="decimal">59.99</balance>
            </tag>
          </tags>
        </transaction>
        <transaction>
          <id type="integer">1553</id>
          <reference-id nil="true"></reference-id>
          <transaction-type>Debit</transaction-type>
          <memo>CVS</memo>
          <balance type="decimal">1.74</balance>
          <posted-at type="datetime">2012-05-16T00:00:00Z</posted-at>
          <created-at type="datetime">2012-05-24T19:24:09Z</created-at>
          <nickname>CVS</nickname>
          <original-name>CVS</original-name>
          <check-number nil="true"></check-number>
          <deleted-at nil="true"></deleted-at>
          <tags type="array">
            <tag>
              <name>Health</name>
              <balance type="decimal">1.74</balance>
            </tag>
          </tags>
        </transaction>
      </transactions>
    </hash>

###Transaction Search

URI: [base]/users/[user id]/transactions/search[.format]

Optional parameters:

* q - The search query (Example: Starbucks). If this is omitted, all transactions will be returned.
* begin_on (default: one month ago from the current date) - The beginning of the date range to limit the transactions returned.
* end_on (default: the current date) - The end of the date range to limit the transactions returned.
* untagged (default: false) - Only return untagged transactions.

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/transactions/search.xml?query=GameStop

Response:
    
    <?xml version="1.0" encoding="UTF-8"?>
    <hash>
      <transactions type="array">
        <transaction>
          <id type="integer">1540</id>
          <reference-id nil="true"></reference-id>
          <transaction-type>Debit</transaction-type>
          <memo>GameStop</memo>
          <balance type="decimal">59.99</balance>
          <posted-at type="datetime">2012-05-24T00:00:00Z</posted-at>
          <created-at type="datetime">2012-05-24T19:24:07Z</created-at>
          <nickname>GameStop</nickname>
          <original-name>GameStop</original-name>
          <check-number nil="true"></check-number>
          <deleted-at nil="true"></deleted-at>
          <tags type="array">
            <tag>
              <name>Entertainment</name>
              <balance type="decimal">59.99</balance>
            </tag>
          </tags>
        </transaction>
    </hash>


###Budgets

URI: [base]/users/[user id]/budgets.[format]

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/budgets.xml

Response:

    <?xml version="1.0" encoding="UTF-8"?>
    <budgets type="array">
      <budget>
       <id type="integer">35668</id>
       <name>transportation</name>
       <month type="integer">5</month>
       <year type="integer">2013</year>
       <tag-names type="array">
        <tag-name>Transportation</tag-name>
       </tag-names>
       <spent type="integer">0</spent>
       <budget-amount type="integer">200</budget-amount>
      </budget>
    </budgets>

###Cashflow Incomes

URI: [base]/users/[user id]/incomes.[format]

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/incomes.xml

Response:

    <?xml version="1.0" encoding="UTF-8"?>
    <incomes type="array">
      <income>
       <id type="integer">445</id>
       <name>Paper Route</name>
       <interval type="integer">2</interval>
       <interval-type>weeks</interval-type>
       <amount type="decimal">500.0</amount>
       <start-date type="date">2011-03-11</start-date>
       <stopped-on nil="true"/>
      </income>
    </incomes>


###Cashflow Bills

URI: [base]/users/[user id]/bills.[format]

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/bills.xml
   
Response:
 
    <?xml version="1.0" encoding="UTF-8"?>
    <bills type="array">
      <bill>
       <id type="integer">4496</id>
       <name>Xmas Gifts</name>
       <interval type="integer">6</interval>
       <interval-type>months</interval-type>
       <amount type="decimal">-100.0</amount>
       <start-date type="date">2012-06-25</start-date>
       <stopped-on nil="true"/>
      </bill>
    </bills>

###Alerts

URI: [base]/users/[user id]/alerts.[format]

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/alerts.xml
   
Response:
 
    <?xml version="1.0" encoding="UTF-8"?>
    <alerts type="array">
      <alert>
        <id type="integer">1</id>
        <options>
          <threshold-type>minimum</threshold-type>
          <threshold-amount type="float">250.0</threshold-amount>
        </options>
        <email-delivery type="boolean">true</email-delivery>
        <sms-delivery type="boolean">false</sms-delivery>
        <source-type>Account</source-type>
        <source-id type="integer">1</source-id>
        <type>AccountThresholdAlert</type>
      </alert>
    </alerts>

###Goals

URI: [base]/users/[user id]/goals.[format]

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/goals.xml

Response:
    
    <?xml version="1.0" encoding="UTF-8"?>
    <goals type="array">
     <goal>
      <id type="integer">266</id>
      <name>Save for a Vacation</name>
      <state>active</state>
      <status type="symbol">under</status>
      <initial-value type="decimal">3897.52</initial-value>
      <current-value type="decimal">3897.52</current-value>
      <target-value type="decimal">10000.0</target-value>
      <monthly-contribution type="integer">120</monthly-contribution>
      <percent-complete type="integer">39</percent-complete>
      <complete type="boolean">false</complete>
     </goal>
    </goals>

###Expenses

URI: [base]/users/[user id]/expenses.[format]

Optional parameters:

* threshold (default: 80) - This parameter controls at what point the remaining expenses are grouped into an 'other' tag. With the default, individual tags will be provided until 80% of the expenses are accounted for, then the remaining ones will be grouped as 'other'. To disable this behavior, supply '100' as the value.
* fallback_to_latest_transactions - This parameter, if non-blank, will cause the API to fallback to the last 30 days of activity in the database, rather than activity within the last 30 calendar days, if the latest transaction for the user is older than 4 days. For example, if the last transaction for the user occurred 7 days ago, the system will look back 37 days for activity, rather than 30.

####Example

    curl http://abc123@geezeo.dev/api/v1/users/1/expenses.xml

Response:
    
    <?xml version="1.0" encoding="UTF-8"?>
    <expense type="array">
      <expense>
        <tag>Utilities</tag>
        <amount type="float">1291.96</amount>
      </expense>
      <expense>
        <tag>Personal</tag>
        <amount type="float">325.0</amount>
      </expense>
      <expense>
        <tag>Other</tag>
        <amount type="float">95.02</amount>
      </expense>
    </expense>

####Example with Threshold

    curl http://abc123@geezeo.dev/api/v1/users/1/expenses.xml?threshold=100

Response:
    
    <?xml version="1.0" encoding="UTF-8"?>
    <expense type="array">
      <expense>
        <tag>Utilities</tag>
        <amount type="float">1291.96</amount>
      </expense>
      <expense>
        <tag>Personal</tag>
        <amount type="float">325.0</amount>
      </expense>
      <expense>
        <tag>Entertainment</tag>
        <amount type="float">83.0</amount>
      </expense>
      <expense>
        <tag>Household</tag>
        <amount type="float">6.0</amount>
      </expense>
      <expense>
        <tag>Diningout</tag>
        <amount type="float">4.42</amount>
      </expense>
      <expense>
        <tag>Health</tag>
        <amount type="float">2.0</amount>
      </expense>
      <expense>
        <tag>Other</tag>
        <amount type="float">0.0</amount>
      </expense>
    </expense>

##Aggregation API

The Aggregation API is currently a synchronous API. Any calls the result in a query to a third party will block the return of the API call. Timeouts should be set high (most likely upwards of 120s), accordingly.

The Aggregation API is part of the REST API. Currently, as of 2012-07-19, functionality that is supported is:

* Searching for FIs
* Retrieving login parameters for an FI
* Adding FIs (both MFA and non-MFA)
* Classifying newly added accounts
* Harvesting a user

###Overview

The process of adding new aggregated accounts to a user is a multi-step one. Briefly, the user will search for the FI, either by name or URL. The particular login parameters specific to that FI will be retrieved from the API, and displayed to the user. The user input will be submitted and processed. If the credentials are valid, a list of accounts and account types will be returned, along with all valid account types for that FI. The user then has the opportunity to rename and reclassify any or all accounts, as well as ignoring any or all accounts. Those choices will be sent back via the API, and the accounts will be added and updated.

###Common Responses

There are two common responses that may be returned when a request does not return the expected data, Message and Error.

The Error response indicates that something went wrong with the request, and provides the message received from the backend services.

The Message response usually will indicate that the response conditionally succeeded, but requires more data to continue processing. An example of this is adding a FI that requires a MFA answer. The response from the "Add FI" request will be a Message, containing the necessary data to continue processing the request.

####Message Example

    {
      "response": {
        "message": "Harvest complete",
        "data": {
          "key": "value"
        }
      }
    }

####Error Example

    {
      "error": 
      {
        "message": "Something unexpected happened"
      }
    }

####Example - non MFA

All URLs relative to /api/v1

Any text in the response bodies preceded by ## are inline comments and will not be returned in the actual response.

###Searching for an FI

Calling ce_fis/?query=CashEdge+Test+Bank+%28Agg%29+-+Retail+Non+2FA via GET with no parameters.

Here, the "query" parameter would be the user entered search string for FI. An additional "scope" parameter is allowed, with the values of "name" or "url".

####Example

    {
      "ce_fis": [
        {
          "id": 2, ## The Geezeo ID, and will be used in subsequent requests
          "fi_id": 20349,
          "name": "CashEdge Test Bank (Agg) - Retail Non 2FA", ## The FI name
          "url": "https://cashbank.cashedge.com/cashedgeBank/CashedgeBankSite/LoginPage.jsp",
          "ce_login_parameters": [ ## This data is necessary to actually add the FI. Each parameter must be presented to the user, and their response captured.
            {
              "ce_login_parameter": {
                "id": 3,
                "parameter_id": "42971", ## This id will be used in subsequent requests
                "parameter_caption": "UserName", ## This should be used as the label of the field.
                "parameter_type": "login", ## If this is "password", the field should be masked
                "parameter_max_length": 20
              }
            },
            {
              "ce_login_parameter": {
                "id": 4,
                "parameter_id": "42972",
                "parameter_caption": "Password",
                "parameter_type": "password",
                "parameter_max_length": 20
              }
            }
          ]
        }
      ]
    }

###Validating FI login credentials

Calling users/pcid/ce_fis via POST with credentials[login_params][42971]=script1&credentials[login_params][42972]=cashedge1&id=2

Here, the user supplied credentials are submitted. The integer keys are the values in the previous response "parameter_id" fields. The "parameter_caption" values are what the fields should be labeled as when presented to the user.

In this case, the "UserName" value is "script1", and the "Password" value is "cashedge1".

####Example

    {
      "accounts": [
        {
          "id": 657,            ## The Geezeo ID of the account
          "name": "Banking",    ## The name of the account at the FI
          "balance": "3897.52", ## The current balance on the account
          "reference_id": null,
          "aggregation_type": "cashedge",
          "state": "active",
          "account_type": { ## This data is used to classify the account.
            "name": "Savings",  ## The account type to show the user
            "acct_type": "SDA", ## This and the ext_type are used to classify
            "ext_type": "SDA",  ## the account in subsequent requests
            "group": "Cash"
          }
        },
        {
          "id": 658,
          "name": "Red Wing Shoe Company, Inc. 401k Profit Sharing Plan",
          "balance": "21099.0",
          "reference_id": null,
          "aggregation_type": "cashedge",
          "state": "active",
          "account_type": {
            "name": "Investment",
            "acct_type": "INV",
            "ext_type": "INV",
            "group": "Investment"
          }
        },
        ...
      ]
    }

###Retrieving list of support account types

If the user desires to reclassify any or all accounts, the full list of supported account types for the FI can be retrieved via the API.

A user may with to reclassify accounts if the initial account type is incorrect (Savings selected for a Checking account, for example).

The account types control how the transaction and balance data for the account are processed, and choosing an incorrect account type may result in invalid data being returned.

When classifying an account, the user should be presented with the "name", and value of the "account_type" data structure. The "acct_type" and "ext_type" values will be joined with a comma and provided back to the API.

#### Example Calling ce_fis/2 via GET with no parameters

    {
      "id": 2, ## These details are the same returned in the search response
      "fi_id": 20349,
      "name": "CashEdge Test Bank (Agg) - Retail Non 2FA",
      "url": "https://cashbank.cashedge.com/cashedgeBank/CashedgeBankSite/LoginPage.jsp",
      "ce_login_parameters": [
        {
          "ce_login_parameter": {
            "id": 3,
            "parameter_id": "42971",
            "parameter_caption": "UserName",
            "parameter_type": "login",
            "parameter_max_length": 20
          }
        },
        {
          "ce_login_parameter": {
            "id": 4,
            "parameter_id": "42972",
            "parameter_caption": "Password",
            "parameter_type": "password",
            "parameter_max_length": 20
          }
        }
      ],
      "ce_accounts": [ ## A list of all supported account types. The data is the same returned in the account list response.
        {
          "ce_account": {
            "name": "Bill",
            "acct_type": "BPA",
            "ext_type": "BPA",
            "group": "Bill"
          }
        },
        {
          "ce_account": {
            "name": "Cash Management",
            "acct_type": "DDA",
            "ext_type": "CMA",
            "group": "Cash"
          }
        },
        ...
      ]
    }

###Classifying accounts

This is required even if the user does not wish to reclassify their accounts. In that case, the existing "acct_type" and "ext_type" must be supplied back to the API. If this step is omitted, the accounts will not actually be added to the users account. Additionally, the account can be renamed at this step by providing the "details" parameter.

Calling users/pcid/accounts/classify via PUT with

    accounts[657][type_code]=SDA%2CSDA&accounts[657][details]=My+Checking&accounts[658][type_code]=ignore

Here the integer key of "accounts" is the "id" value from the account details in the last request. The "type_code" value is made up of the concatenated "acct_type" and "ext_type", joined by a comma (ie: SDA,SDA). This is our best guess as to what the account is.

    {"response"=>{"message"=>"1 added."}}

###MFA

MFA (Multi Factory Authentication) is a method of identify verification that some FIs require. After the normal username/password credentials, there is a 2nd verification step. this step differs for each FI. The aggregation API returns the appropriate prompts, and accepts them for verification.

The overall process is the same as a non-MFA FI, with a fork for the secondary verification step. After the "Validating FI login credentials" step in the non-MFA steps, instead of returning a list of accounts at the FI, the API will return an error response, with the necessary MFA prompts and parameters to continue adding the FI.

####MFA Example

Calling ce_fis/?query=CashEdge+Test+Bank+%28Agg%29+-+Retail+2FA via get with no parameters

    {"ce_fis"=>[{"id"=>1, "fi_id"=>20404, "name"=>"CashEdge Test Bank (Agg) - Retail 2FA", "url"=>"https://cashbank.cashedge.com/cashedgeBank/aggrretail2FA/cashedge_login.html", "ce_login_parameters"=>[{"ce_login_parameter"=>{"id"=>1, "parameter_id"=>"42969", "parameter_caption"=>"User ID", "parameter_type"=>"login", "parameter_max_length"=>30}}, {"ce_login_parameter"=>{"id"=>2, "parameter_id"=>"42970", "parameter_caption"=>"Password", "parameter_type"=>"password", "parameter_max_length"=>30}}]}]}

Calling users/pcid/ce_fis via post with credentials%5Blogin_params%5D%5B42969%5D=test&credentials

    {"response"=>{"message"=>"The account requires further authentication", "data"=>{"mfa_parameters"=>[{"ce_login_parameter"=>{"ce_fi_id"=>nil, "parameter_caption"=>"What is your favorite color?", "parameter_id"=>"answer", "parameter_max_length"=>nil, "parameter_type"=>"password"}}], "session_key"=>"464a674e7367646e2b77665576726947426b627879413d3d", "harvest_id"=>"123770714", "login_id"=>"19349692"}}}

Calling users/pcid/ce_fis/1 via put with harvest_id=123770714&login_id=19349692&mfa_responses%5Banswer%5D=red&session_key=464a674e7367646e2b77665576726947426b627879413d3d

    {"accounts"=>[{"id"=>658, "name"=>"Checking", "balance"=>"70.0", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Checking", "acct_type"=>"DDA", "ext_type"=>"DDA", "group"=>"Cash"}}, {"id"=>659, "name"=>"Checking1", "balance"=>"150.0", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Checking", "acct_type"=>"DDA", "ext_type"=>"DDA", "group"=>"Cash"}}, {"id"=>660, "name"=>"Checking2", "balance"=>"6348.9", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Checking", "acct_type"=>"DDA", "ext_type"=>"DDA", "group"=>"Cash"}}, {"id"=>661, "name"=>"Checking7", "balance"=>"328.9", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Checking", "acct_type"=>"DDA", "ext_type"=>"DDA", "group"=>"Cash"}}, {"id"=>662, "name"=>"Checking8", "balance"=>"1248.9", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Checking", "acct_type"=>"DDA", "ext_type"=>"DDA", "group"=>"Cash"}}, {"id"=>663, "name"=>"Checking9", "balance"=>"28.9", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Checking", "acct_type"=>"DDA", "ext_type"=>"DDA", "group"=>"Cash"}}, {"id"=>664, "name"=>"Saving3", "balance"=>"543.0", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Savings", "acct_type"=>"SDA", "ext_type"=>"SDA", "group"=>"Cash"}}, {"id"=>665, "name"=>"Saving4", "balance"=>"10003.0", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Savings", "acct_type"=>"SDA", "ext_type"=>"SDA", "group"=>"Cash"}}, {"id"=>666, "name"=>"Saving5", "balance"=>"0.0", "reference_id"=>nil, "aggregation_type"=>"cashedge", "state"=>"active", "account_type"=>{"name"=>"Savings", "acct_type"=>"SDA", "ext_type"=>"SDA", "group"=>"Cash"}}]}

Calling users/pcid/accounts/classify via put with: 

    accounts%5B658%5D%5Btype_code%5D=DDA%2CDDA&accounts%5B659%5D%5Btype_code%5D=ignore&accounts%5B660%5D%5Btype_code%5D=ignore&accounts%5B661%5D%5Btype_code%5D=ignore&accounts%5B662%5D%5Btype_code%5D=ignore&accounts%5B663%5D%5Btype_code%5D=ignore&accounts%5B664%5D%5Btype_code%5D=ignore&accounts%5B665%5D%5Btype_code%5D=ignore&accounts%5B666%5D%5Btype_code%5D=ignore

Response:

    {"response"=>{"message"=>"1 added."}}


###Pending Accounts

In some circumstances, the account will be added at the aggregator, but not finalized/classified, leaving the account in an inconsistent state.

To view these account, make a GET request to /api/v1/users/:user_id/pending_accounts.

    [
      {
        "id": 2,
        "name": "CashEdge Test Bank (Agg) - Retail Non 2FA",
        "account_ids": [
          "19435010"
        ]
      }
    ]

To remove a pending account, make a DELETE request to /api/v1/users/:user_id/pending_accounts/:login_id where the :login_id is the value of the :account_id from the GET request. It is possible to have multiple pending accounts for a single FI. A DELETE request is required to remove each of them.

##Change CE FI Login API

### URL

`PUT /api/v1/accounts/:account_id/update_credentials`

### Non-MFA Request

#### Parameters

The new login credentials are in the same form as when adding the FI initially.
* `credentials[login_params][:login_parameter_id]`
* `credentials[login_params][:password_parameter_id]`

#### Response

On success, the response will be the json of the account that has been updated. When an MFA challenge occurs, the challenge will be returned so the answer may be resubmitted. On error, a message describing the error will be returned.

#### Successful Example

`PUT /api/v1/accounts/1/update_credentials` with a body of `credentials[login_params][1234]=username&credentials[login_params][1235]=newpassword`

```json
{
  "id":86,
  "name":"account-name",
  "balance":"42.5",
  "reference_id":"42",
  "aggregation_type":"cashedge",
  "state":"active",
  "harvest_updated_at":"2013-03-07T14:42:00Z",
  "account_type":{
    "name":"Checking",
    "acct_type":"DDA",
    "ext_type":"DDA",
    "group":null
  },
  "fi":{
    "fi_id":99,
    "name":"FI"
  }
}
```

### Example that requires MFA

`PUT /api/v1/accounts/1/update_credentials` with a body of `credentials[login_params][1234]=username&credentials[login_params][1235]=newpassword`

```json
{
  "response":"The account requires further authentication",
  "data":{
    "mfa_parameters":[
      {
        "ce_login_parameter":{
          "ce_fi_id":null,
          "parameter_caption":"What is your favorite color?",
          "parameter_id":"answer",
          "parameter_max_length":null,
          "parameter_type":"password"
        }
      }
    ],
    "session_key":"464a674e7367646e2b77665576726947426b627879413d3d",
    "harvest_id":"123770714",
    "login_id":"19349692"
  }
}
```

The answer to the MFA challenge can then be submitted to continue the process. These are required parameters for this request:
* `mfa_responses[answer]` The answer to the MFA challenge.
* `harvest_id` From the previous request.
* `login_id` From the previous request.
* `session_key` From the previous request.

`PUT /api/v1/accounts/1/update_credentials` with a body of `mfa_responses[answer]=red&harvest_id=123770714&login_id=19349692&session_key=464a674e7367646e2b77665576726947426b627879413d3d`

```json
{
  "id":86,
  "name":"account-name",
  "balance":"42.5",
  "reference_id":"42",
  "aggregation_type":"cashedge",
  "state":"active",
  "harvest_updated_at":"2013-03-07T14:42:00Z",
  "account_type":{
    "name":"Checking",
    "acct_type":"DDA",
    "ext_type":"DDA",
    "group":null
  },
  "fi":{
    "fi_id":99,
    "name":"FI"
  }
}
```

### Error Response

```json
{
  "error": {
    "message":"Something unexpected happened.",
    "data":{}
  }
}
```
