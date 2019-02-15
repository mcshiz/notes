# notes
---

- [Use ssh keys for github](#github-ssh)
- [Mandatory code reviews](#mandatory-code-reviews)

---
<a name="github-ssh"></a>
### Use ssh keys for github
Create ssh key on your maching 
- `ssh-keygen -t rsa`

Add public key to github repo and enable
- Go to github/<YOUR REPO>/settings/SSH and GPG Keys  
- Copy public key contents from step 1 <NAME OF KEY>.pub into proper text box
- enable key

Create ssh alias
- make or edit this file ~/.ssh/config
    ```
    Host <ALIAS OF YOUR CHOICE: github-work>
    Hostname github.com
    PreferredAuthentications publickey
    IdentityFile <PATH TO PRIVATE KEY>
    User git
    ```

Change github repo urls from https to SSH
  - git remote set-url origin git@<ssh-alias><user_name>/<repo_name>.git
Use github commands
  - git pull git@<ssh-alias>:<user_name>/<repo_name>.git
      - i.e git pull git@github-work:mcshiz/happy-faces.git
<a name="mandatory-code-reviews"></a> 
### Set mandatory code reviews by specific people on all pull requests
- Create a file in the root of the project called CODEOWNERS . 
- Add github usernames to it - i.e @your_mom 
- Commit and push that file  
- Log into github and navigate to the repo, go to settings > branches > add rule
- Change branch to `staging`
- Click the "Require pull request reviews before merging" button
- Check the "Require review from Code Owners" box
    
    
 ### Export cloudformation values from one stack to be used in other stacks

<p>

<b>Note:</b>
 when passing paramaters to cloudformation use lowercase variables. Some things will maintain caseing other things won't<br/>

i.e Passing PROD below doesn't work because the exported variable ends up as prod-MyExportResource but the import looks for PROD-MyExportResource 😢

</p>

File 1

```

Outputs:

MyExportResource:

Description: The SNS Topic ARN which gets populated when products get updated/added to BODS

Value: !Ref preparedProductOutSNS

Export:

Name: !Sub "${AWS::StackName}-MyExportResource"

```

File 2

```

Resources:

# Subscribe to the SNS topic that is exported from another file

MyExportResourceSubscription:

Type: AWS::SNS::Subscription

Properties:

Endpoint: !GetAtt BODSToSFCCSQS.Arn

Protocol: sqs

Region: us-west-2

TopicArn: 

Fn::ImportValue:

!Sub

- ${stage}-preparedData-stack-MyExportResource

- { stage: !Ref Stage }

```



### Dynamodb Sort Keys and begins_with

```

Consider how you might group locations. In the United States, locations are usually grouped by city

state, and country. Let’s say we create a sort key that stores an item’s 

location in a string with the template of city-state-country.



The order of city-state-country starts with the most specific part of the location data.

As a result, your queries using the begins_with operator are limited to the city.

If I switch the order of the location string to be country-state-city, the begins_with operator

instead provides three levels of querying:



begins_with(USA) – Returns all items located in the United States.

begins_with(USA-TX) – Returns only items located in Texas.

begins_with(USA-TX-Houston) – Returns only items located in Houston.

```



### Sending custom metrics to cloudformation from lambda

<p> When you want to monitor something particular that your lambda is doing such as number of SQS records processed or whatever
 you can send custom metrics to cloudwatch and setup alerts to monitor those metrics.



Below I have an alarm that will trigger if a product pulled from an SQS queue doesn't have all of the required attributes or fails to insert into dynamodb



```

let metric = {

MetricData : [

{

MetricName: 'PRODUCTS_TO_BODS_FAIL',

Dimensions: [

{

Name: 'TYPE',

Value: 'fail'

}

],

Timestamp: new Date(),

Unit: 'Count',

Value: fails

},

{

MetricName: 'PRODUCTS_TO_BODS_SUCCESS',

Dimensions: [

{

Name: 'TYPE',

Value: 'success'

}

],

Timestamp: new Date(),

Unit: 'Count',

Value: successes

}

],

Namespace: 'BRIAN_TEST'

} 

```



### Dynamodb marshaller

Browser side

`npm install --save @aws/dynamodb-data-marshaller`

It's also part of the aws-sdk for use by lambdas

`AWS.DynamoDB.Converter.marshall()` and
`AWS.DynamoDB.Converter.unmarshall()`

<b>Note:</b>When
 using aws dynamodb's document client marshalling is done for you automatically!

```

var unmarshalled = AWS.DynamoDB.Converter.unmarshall({

string: {S: 'foo'},

list: {L: [{S: 'fizz'}, {S: 'buzz'}, {S: 'pop'}]},

map: {

M: {

nestedMap: {

M: {

key: {S: 'value'}

}

}

}

},

number: {N: '123'},

nullValue: {NULL: true},

boolValue: {BOOL: true}

});

```

```

var unmarshalled = AWS.DynamoDB.Converter.unmarshall({

string: 'foo',

list: ['fizz', 'buzz', 'pop'],

map: {

nestedMap: {

key: 'value'

}

},

number: 123,

nullValue: true,

boolValue: true

```



### SNS --> SQS

<p>To easily enable future expansion of an application it's a good idea to put messages into a SNS topic first then subscribe
 your SQS topic(s) to that.

This will allow you to later add more SQS subscriptions to process the same messages for a different use.

If you put them directly into SQS they can only be processed once (unless you put them back into the queue or add them to a different queue 🤢)</p>





### Stream a CSV from S3 to Lambda for processing

<p>Sometimes I don't want to npm install a csv parser to keep my lambdas nice and small

`readline` is part of the NodeJs API 

</p>



```

const readline = require('readline');

const rl = readline.createInterface({

input: s3.getObject(params).createReadStream()

});

rl.on('line', (line) => {

let columns = line.split(',')

}).on('close' () => {

console.log('Done reading the lines of the file')

})



```
