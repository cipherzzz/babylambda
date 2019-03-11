# AWS Lambda with API Gateway
The purpose of this article is to help you setup a simple `AWS lambda` function and
expose it as a public `REST` service via the `API Gateway`. We hope to show you how
to do the following:
- Create the lambda function
- Create a test for it
- Create an API Gateway
- Wire up a REST call to the lambda function with mapped parameters

## Requirements
- AWS account

## Create Lambda Function
- Login to AWS console, search `Lambda`, and click resulting item, `Lambda`
- Click `Create function`, `Author from scratch`
- Enter `baby-lambda` as `name`, `Node.js 8.10` as `Runtime`  
- Select `Create Role`, enter `babyLambdaRole` as name, use `Simple microservice permissions` template

## Boilerplate Test
At this point, you should have a boilerplate `hello world` lambda function.

- Click the `Test` button in the upper right of your screen
- Enter `babyLambdaEvent` into the `name` field
- Leave the payload alone, it doesn't matter at this point
- Click `Create`  
- You should now have a `babyLambdaEvent` in the dropdown, click `Test` again
- Check the `details` of the response and see the hardcoded, `Hello from Lambda!` response

## Initial baby-lambda.js
So far we have setup a simple boilerplate `hello world` lambda function. Let's go ahead and
add our first pass of the `baby-lambda.js` code. Enter the code below into the `Function Code` section

```
exports.handler = async (event) => {

    const months = 23;
    const lbs = 100;

    let message = "not a baby";
    if(months < 24) {
        message = lbs > 30?"it's a big baby":"it's a baby"
    }

    const response = {
        statusCode: 200,
        body: JSON.stringify({months, lbs, message}),
    };
    return response;
};
```

## Setup AWS API Gateway
The lambda function above is simply a function living in the aws lambda environment
but we have not wired it up to receive RESTful requests and pass external parameters
to it. In order to expose this lambda function we will create an api gateway and connect
an http action to the lambda entrypoint.

### Create
- Search `API Gateway` and click resulting item, `API Gateway`
- Click `Get Started`
- Select `REST` as protocol, Select `New API`, Enter `babyLambdaAPI` as the name, Enter a description
- Click `Create API`

### Configure
- Select `Actions` => `Create Method`, Select `GET` action, Click `checkmark`
- Select `Lambda Function` as `Integration Type`, Enter `baby-lambda` as `Lambda Function`
- Click `Save`
- Click `OK` on the `Add Permission to Lambda Function`

### Handle Variables
- Select the `Integration Request` cell in the `GET - Method Execution` detail
- Select `when there are no templates defined` in the `Mapping Templates` => `Request Body Passthrough`
- Select `Add mapping template`, enter `application/json`, click the `checkmark`

Add the json template below into the `application/json` template
```
{
  "lbs": "$input.params('lbs')",
  "months": "$input.params('months')"
}
```

### Deploy API Gateway
- Select `Actions` => `Deploy API`
- Select `[New Stage]` for `Deployment Stage` => Enter `test` for `name`
- Click the `Invoke URL` and you should see a similar response to the `Test` you created

## Update the baby-lambda.js function
Our lambda function has hardcoded variables and now we'd like to pass through the `lbs` and `months`
from the api gateway to the lambda function itself.

### Update Code
- Search for `Lambda` and select the `baby-lambda` function
- Change the code in the function to use the `event.lbs` and `event.months`

```
exports.handler = async (event) => {

    const months = event.months;
    const lbs = event.lbs;

    if( !months || !lbs) {
      return {
        statusCode: 400,
        body: JSON.stringify({error: "You must give a weight in lbs and age in months"}),
      }
    }

    let message = "not a baby";
    if(months < 24) {
        message = lbs > 30?"it's a big baby":"it's a baby";
    }

    const response = {
        statusCode: 200,
        body: JSON.stringify({months, lbs, message}),
    };
    return response;
};
```

### Update Test(s)
Select the `babyLambdaEvent`, Select `configure test events`, select the `babyLambdaEvent`
and change its content to
```
{
  "lbs": 10,
  "months": 19
}
```

- Create another event named `bigBabyEvent` and change the `lbs` to `40`
- Create another event named `errorEvent` and make it read `{}`
- Select each event and run a `Test` for it and verify the output is correct

## Verify the API Gateway hits the Lambda function
- Search `API Gateway` and click resulting item, `API Gateway`
- Click `babyLambdaAPI`, `Stages`, `test`, `GET`, `Invoke URL`
- You should get `{"statusCode":400,"body":"{\"error\":\"You must give a weight in lbs and age in months\"}"}`
- Add `?lbs=9&months=36` to the `Invoke URL` and you should get - `{"statusCode":200,"body":"{\"months\":\"36\",\"lbs\":\"9\",\"message\":\"not a baby\"}"}`

# Summary
Congratulations! You just created a simple serverless `lambda` function that is accessible via the `api gateway`.
These functions are a useful tool for services development when you have bitesized functionality
that can exist outside of a heavy enterprise service. Typically a lambda service would interact
with a persistence mechanism or another service oriented peer. That however, is left as an exercise to the reader. 
