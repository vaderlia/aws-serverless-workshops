# Module 1: User Authentication

In this module, you will create an Amazon Cognito User Pool and Identity Pool for the wild rydes application.
The Cognito User Pool will store user profile information and provide sign-up and sign-in capabilities, with the Cognito Identity Pool providing the ability to assume an Identity and Access Management (IAM) role from within the application.

Since Wild Rydes is a ride sharing application, a key requirement is that all users must sign-up and sign-in before they're allowed to request a ride. You will configure the application to integrate with [Amazon Cognito](https://aws.amazon.com/cognito/) for these purposes via the [AWS Amplify JavaScript library](https://aws-amplify.github.io/).

## Solution Architecture

The architecture for this module is very straightforward. All of your static web content including HTML, CSS, JavaScript, images and other files will be served locally from your Cloud9 workspace. As you made changes to the website application code, all changes will be automatically updated and shown in your browser via live reload capabilities.

For this module, we will be creating a Cognito User Pool as our secure user directory then configuring our application to use the AWS Amplify library to easily integrate Amazon Cognito into our application.

![Website architecture](../images/wildrydes-module1-architecture.png)

## Implementation Overview

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### 1. Running the website locally

1. From your Cloud9 workspace, select the terminal window and when you are within your website directory, run the following command to start the local web server 

    ```console
    yarn start
    ```

    Wait for the development server to start. You can ignore any message saying *Compiled with warnings* as we will resolve these warnings as we add our functionality to the application.


2. Now that the development server has started, click **Preview** in the top of the screen next to the Run button.

    ![Cloud9 Preview](../images/cloud9-local-preview.png)  

3. The web application will load in a small window next to the terminal at the bottom of the Cloud9 IDE. Click the **re-size button** next to the word **Browser** to open this window in a new tab.

    ![Cloud9 Preview Re-size](../images/cloud9-resize-live-preview.png)   

   As you make changes to the web application, this tab will automatically refresh to reflect your changes. Leave this tab open and return to the Cloud9 IDE tab to continue the workshop.

   Though the Wild Rydes website may look functional, there is currently no integration for sign-up or sign-in requests to go anywhere.

### 2. Creating a Cognito User Pool

Amazon Cognito lets you add user sign-up, sign-in, and access control to your web and mobile apps quickly and easily. In this step, we'll create a Cognito user pool for use with our Wild Rydes app.

#### High-Level Instructions

Use the AWS console to create an Amazon Cognito User Pool requiring e-mail verification.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **Services** then select **Cognito** under Security, Identity, and Compliance.

2. Choose your desired **Region** in top-right of the console if not already chosen.

3. Choose **Manage User Pools**.

4. Choose **Create a User Pool** in the top right of the console.

5. Provide a name for your user pool such as `WildRydes`.

6. Choose **Step through settings** to configure our user pool options.

	![User Pool Setup Step 1](../images/cognito-userpool-setup-step1.png)

7. Leave **Username** selected, but additionally select *Also allow sign in with verified email address* and *Also allow sign in with verified phone number*.

8. Leave all other attribute defaults as-is.

9. Choose **Next**.

	![User Pool Setup Step 2](../images/cognito-userpool-setup-step2.png)

10. Leave password policies and user sign up settings set to default settings and choose **Next**.

	![User Pool Setup Step 3](../images/cognito-userpool-setup-step3.png)

11. Leave MFA set to Off for this workshop.

12. Leave the default setting selected of requiring e-mail verification.

13. Choose **Save changes**.

	![User Pool Setup Step 4](../images/cognito-userpool-setup-step4.png)

14. Leave all message defaults as-is and choose **Next step**.

15. Skip adding any tags and click **Next step**.

16. Choose **No** to not remember your user's devices then click **Next step**.

	![User Pool Setup Step 5](../images/cognito-userpool-setup-step5.png)

17. Choose **Add an app client**.

18. Input `wildrydes-web-app` as the app client name.

19. **Uncheck** `Generate client secret`. Client secrets are used for server-side applications authentication and are not needed for JavaScript applications.

20. Choose **Create app client**.

21. Choose **Next step**.

	![User Pool Setup Step 6](../images/cognito-userpool-setup-step6.png)

22. Leave all Lambda trigger settings set to **none**. These trigger settings allow you to extend the out-of-the-box sign-up and sign-in flows with your own custom logic, but we will not be using this feature in this workshop.

23. Choose **Next step**.

24. Review summary of all provided settings for accuracy then choose **Create pool**.

25. Within Cloud9, click the **+** symbol and choose to create **New File**. You will use this new blank editor tab as a scratchpad for various resource names and variables.

	![Cloud9 Create Scratchpad Tab](../images/cloud9-createscratchpadtab.png)

26. Back in the AWS Cognito console, copy your new **User Pool Id** into the scratchpad tab.

27. Choose **App clients** heading under **General settings** within the Cognito navigation panel.

28. Copy the **App client ID** over to your scrathpad. You will be using both of these values later on.

</p></details>

### 3. Creating a Cognito Identity Pool

Cognito Identity Pools are used to provide AWS credentials via IAM roles to end-user applications. Since we'll be using IAM integration to integrate our Cognito deployment and users with other AWS services, we'll go ahead and create this identity pool now.

#### High-Level Instructions

You will need to create a Cognito Identity Pool linked to the Cognito User Pool and app client ID you just created. Your application will not require un-authenticated users to access any AWS resources, so you do not need to enable access to unauthenticated identities. 

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the Cognito console, choose **Federated Identities** in the header bar to switch to the console for Cognito Federated Identities.

1. Choose **Create new Identity pool**.

1. Input `wildrydes_identity_pool` as the Identity pool name.

1. Under Authentication providers, within the Cognito tab, input the User Pool ID and App client Id you copied previously to the scratchpad tab.

	![Identity Pool Setup Step 1](../images/cognito-identitypool-setup-step1.png)

1. Choose **Create Pool**.

1. Choose **Allow** to allow Cognito Identity Pools to setup IAM roles for your application's users. Permissions and settings of these roles can be customized later.

1. Copy/paste the **Identity Pool ID**, highlighted in red within the code sample in the Get AWS Credentials section, into your Cloud9 scatchpad editor tab. Make sure not to copy the quotation marks!

	![Copy Identity Pool Id to Cloud9 scratchpad](../images/cognito-identitypool-copyId.png)

</p></details>

### 4. Integrating your application with Amazon Cognito using AWS Amplify

Now that you've created and configured your Cognito User Pool and Identity Pool, you need to configure your application to integrate to Amazon Cognito so it can store user profiles and enable sign-up and sign-in.

#### High-Level Instructions

You will import the AWS Amplify library into the project then add sign-up and sign-in utility classes to integrate with our existing UI and front-end components.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Before using any AWS Amplify modules, we first need to configure Amplify to use our newly created Cognito resources by updating `/website/src/amplify-config.js`.

1. After opening this file in your Cloud9 IDE editor, find an replace the following parameters with values from your previous scratchpad:
	- `identityPoolId`
	- `region`
	- `userPoolId`
	- `userPoolWebClientId`

1. Be sure to **save your changes** to the config file so your new Amplify  settings take effect. Any unsaved changes to a file are indicated by a dot icon in the tab of the editor so if you see a gray dot next to the file name, you may have forgotten to save.

1. Next, edit the `website/src/index.js` file to add the following lines to the **top of the file (but below all the other imports)** to configure Amplify then save your changes:

	```
	import Amplify from 'aws-amplify';
	import awsConfig from './amplify-config';
	
	Amplify.configure(awsConfig);
	```

1. Next, we need to ensure our application evaluates the user's authenticated state. In the same `/website/src/index.js` file, find and replace the **isAuthenticated method** with the code below to use our Amplify library's built-in user session to check this status.

	```
	const isAuthenticated = () => Amplify.Auth.user !== null;
	```

1. Now that we've imported the Amplify and configured the Amplify library, we need to update our application's code to sign-up users using Amplify and Cognito User Pools by finding and replacing the following methods within the `/website/src/auth/SignUp.js` file with the code below then save your changes.

	```
	async onSubmitForm(e) {
	e.preventDefault();
	try {
	    const params = {
	        username: this.state.email.replace(/[@.]/g, '|'),
	        password: this.state.password,
	        attributes: {
	        email: this.state.email,
	        phone_number: this.state.phone
	        },
	        validationData: []
	    };
	    const data = await Auth.signUp(params);
	    console.log(data);
	    this.setState({ stage: 1 });
	} catch (err) {
	    if (err.message === "User already exists") {
	        // Setting state to allow user to proceed to enter verification code
	        this.setState({ stage: 1 });
	    } else {
	        if (err.message.indexOf("phone number format") >= 0) {err.message = "Invalid phone number format. Must include country code. Example: +14252345678"}
	        alert(err.message);
	        console.error("Exception from Auth.signUp: ", err);
	        this.setState({ stage: 0, email: '', password: '', confirm: '' });
	    }
	}
	}
	
	async onSubmitVerification(e) {
	    e.preventDefault();
	    try {
	    const data = await Auth.confirmSignUp(
	        this.state.email.replace(/[@.]/g, '|'),
	        this.state.code
	    );
	    console.log(data);
	    // Go to the sign in page
	    this.props.history.replace('/signin');
	    } catch (err) {
	    alert(err.message);
	    console.error("Exception from Auth.confirmSignUp: ", err);
	    }
	}
	```

1. You additionally need to integrate the sign-in capability to use AWS Amplify and Cognito by finding and replacing the following methods within the `/website/src/auth/SignIn.js` file with the code below then save your changes.

    ```
    async onSubmitForm(e) {
        e.preventDefault();
        try {
            const userObject = await Auth.signIn(
                this.state.email.replace(/[@.]/g, '|'),
                this.state.password
            );
            console.log('userObject', userObject);
            if (userObject.challengeName) {
            // Auth challenges are pending prior to token issuance
            this.setState({ userObject, stage: 1 });
            } else {
            // No remaining auth challenges need to be satisfied
            const session = await Auth.currentSession();
            // console.log('Cognito User Access Token:', session.getAccessToken().getJwtToken());
            console.log('Cognito User Identity Token:', session.getIdToken().getJwtToken());
            // console.log('Cognito User Refresh Token', session.getRefreshToken().getToken());
            this.setState({ stage: 0, email: '', password: '', code: '' });
            this.props.history.replace('/app');
            }
        } catch (err) {
            alert(err.message);
            console.error('Auth.signIn(): ', err);
        }
    }

    async onSubmitVerification(e) {
        e.preventDefault();
        try {
            const data = await Auth.confirmSignIn(
                this.state.userObject,
                this.state.code
            );
            console.log('Cognito User Data:', data);
            const session = await Auth.currentSession();
            // console.log('Cognito User Access Token:', session.getAccessToken().getJwtToken());
            console.log('Cognito User Identity Token:', session.getIdToken().getJwtToken());
            // console.log('Cognito User Refresh Token', session.getRefreshToken().getToken());
            this.setState({ stage: 0, email: '', password: '', code: '' });
            this.props.history.replace('/app');
        } catch (err) {
            alert(err.message);
            console.error('Auth.confirmSignIn(): ', err);
        }
    }
    ```

</p></details>

### 5. Validate sign-up and sign-in are now working with Amazon Cognito

Now that you have integrated our Amplify code into our application, you need to test the site to see that authentication is working end-to-end.

#### High-Level Instructions

Return to your browser tab where you started your Wild Rydes application earlier after popping out from the Cloud9 IDE once in preview mode. This page automatically refreshes after you save any code changes so should now reflect all of your changes and be ready for testing.

Sign-up as a new user then sign-in with that user. Provide a valid phone number with `+country_code` first preceeding the number. For a US-based phone number, an example would be `+14251234567`. You should then see that a verification message is sent with a one-time code upon registering, which is required before you're able to sign-in.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Visit `/register` path of your Cloud9's website to go to the registration page.

1. Input your e-mail address, phone number with `+country_code` first preceeding the number, as well as your password twice. For a US-based phone number, an example would be `+14251234567`.

1. Choose **Let's Ryde** to submit registration.

1. On the verify e-mail screen, enter the one-time code sent to your e-mail address provided then choose **Verify**.

1. Assuming no errors were encountered, you will be redirected to the Sign-in screen. Now, re-enter the same e-mail address and password you chose at registration.

1. **If prompted for an MFA code**, check your phone entered previously for an SMS message. Enter your **SMS MFA code** at the verification code prompt then choose **Verify**. If not prompted for an MFA code, skip this step.

1. If the page then loads a map, sign-in was successful and you have successfully integrated Cognito for app authentication. 

</p></details>

Once you have finished this section, you may proceed to the next module, [Serverless API Auth](../2_ServerlessAPI).

