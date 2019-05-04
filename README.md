# An AWS AppSync Chat Starter App written in Angular


## Introduction

This is a Starter Angular Progressive Web Application (PWA) that uses AWS AppSync to implement offline and real-time capabilities in a chat application as part of the blog post https://aws.amazon.com/blogs/mobile/building-a-serverless-real-time-chat-application-with-aws-appsync/. In the chat app, users can have conversations with other users and exchange messages. The application demonstrates GraphQL Mutations, Queries and Subscriptions using AWS AppSync. You can use this for learning purposes or adapt either the application or the GraphQL Schema to meet your needs.



#### Prerequisites

* [AWS Account](https://aws.amazon.com/mobile/details) with appropriate permissions to create the related resources
* [NodeJS](https://nodejs.org/en/download/) with [NPM](https://docs.npmjs.com/getting-started/installing-node)
* [AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) `(pip install awscli --upgrade --user)`
* [AWS Mobile CLI](https://github.com/aws/awsmobile-cli) (configured for a region where [AWS AppSync is available](https://docs.aws.amazon.com/general/latest/gr/rande.html#appsync_region)) `(npm install -g awsmobile-cli)`
* [Angular CLI](https://github.com/angular/angular-cli) `(npm install -g @angular/cli)`


### Backend Setup

1. First, clone this repository and navigate to the created folder:
    ```bash
    $ git clone https://github.com/aws-samples/aws-mobile-appsync-chat-starter-angular.git
    $ cd aws-mobile-appsync-chat-starter-angular
    ```

1. Set up your AWS resources in AWS Mobile Hub with the awsmobile cli:
    ```bash
    $ awsmobile init
    ```
    Provide the following details and name the project:
      * Source directory: `<Press ENTER to accept defaults>`
      * Distribution directory that stores build artifacts: `dist`
      * Build command: `ng build --prod`
      * Start command for local test run: `<Press ENTER to accept defaults>`

1. Copy the file `./backend/mobile-hub-project.yml` to `./awsmobilejs/backend/` and push the configuration change to AWS Mobile Hub.

    ```bash
    $ cp ./backend/mobile-hub-project.yml ./awsmobilejs/backend/
    $ awsmobile push
    ```

    This deploys User Sign-In, Analytics and Hosting features, and downloads your project's `aws-exports.js` file to the `./src` folder.

1. In `./src/aws-exports.js`, look up the ID of the Cognito User Pool that was created

    ```bash
    $ grep aws_user_pools_id src/aws-exports.js
    ```

    From here there are 2 options:
    * Deploy from the AWS Console
    * Deploy with CloudFormation

### Deploy from the AWS Console    

1. Navigate to the AWS AppSync console using the URL: http://console.aws.amazon.com/appsync/home

1. Click on **Create API**, go to **Start from a Sample Project** and select the **Chat App** option. Enter a API name of your choice, select the Region and the Amazon Cognito User Pool ID you retrieved from `./src/aws-exports.js`. Click **Create**.

1. Scroll down to the **Integrate your GraphQL API** section,  select **Web** and *download the AWS AppSync.js config file*. Place the `AppSync.js` file in your project's `./src` directory.

    ```bash
    $ cp <download-directory>/AppSync.js ./src/AppSync.js
    ```
1. Start the application:

    ```bash
    $ awsmobile run
    ```

### Deploy with CloudFormation

1. Deploy the included Cloudformation template to launch a stack that will create an IAM Service Role, DynamoDB tables, an AppSync API, a GraphQL schema, Data Sources, and Resolvers.

    ```bash
    $ aws cloudformation create-stack --stack-name ChatQL --template-body file://backend/deploy-cfn.yml --parameters ParameterKey=userPoolId,ParameterValue=<AWS_USER_POOLS_ID> --capabilities CAPABILITY_IAM --region <YOUR_REGION>
    ```

    When the stack is done deploying, you can view its output. Make note of the GraphQL API Identifier 'ChatQLApiId'.

    ```bash
    aws cloudformation describe-stacks --stack-name ChatQL --query Stacks[0].Outputs --region <YOUR_REGION>
    ```

1. Point your browser to the [AWS AppSync Console](https://console.aws.amazon.com/appsync/home) (keeping mind of the region), and select the API (named 'ChatQL') created in the previous step. Scroll down to the **Integrate your GraphQL API** section,  select **Web** and *download the AWS AppSync.js config file*. Place the `AppSync.js` file in your project's `./src` directory.

    ```bash
    $ cp <download-directory>/AppSync.js ./src/AppSync.js
    ```

1. Finally, execute the following command to install your project package dependencies and run the application locally:
      ```bash
      $ awsmobile run
      ```

1. Access your ChatQL app at http://localhost:4200. Sign up different users and test real-time/offline messaging using different browsers.

### Application Walkthrough

The application is composed of the main app module and 2 independent feature modules:

* auth.module
* chat-app.module

Both modules make use of AWS Amplify, which is initialized with the `./src/aws-exports.js` configuration in `./src/app.module.ts`. The auth.module allows to easily onboard users in the app. To sign up to use the app, a user provides his username, password and email address. Upon succesful sign-up, a confirmation code is sent to the user's email address to confirm the account. The code can be used in the app to confirm the account, after which a user can sign in and access the chat portion of the app.

In the chat, a user can see a list of other users who have registered after sign in to the app for the first time. A user can click on a name to initiate a new conversation (`./src/app/chat-app/chat-user-list`) or can click on an existing conversation to resume it (`./src/app/chat-app/chat-convo-list`). Inside of a conversation, a user automatically receives new messages via subscriptions (`./src/app/chat-app/chat-message-view/`). When a user sends a message, the message is initially displayed with a grey check mark. This indicates that receipt confirmation from the backend server has not been received. The checkmark turns green upon confirmation the message was received by the backend. This is easily implemented by including the `isSent` flag, set to false, when we send the `createMessage` mutation. In the mutation, we request for the `isSent` flag to be returned from the backend. The flag is set to true on the backend and when returned, our template updates the css class associated with the checkmark (see `./src/src/app/chat-app/chat-message`).

![Application Overview](/media/chatql-app.png)

