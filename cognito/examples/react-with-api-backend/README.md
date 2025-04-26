## Creating a User Pool

To get started, you need to create a user pool in Amazon Cognito:

1. Login into AWS Console and navigate to the Amazon Cognito Console.
2. Click **Create user pool**.
3. Choose **Create a user pool** in the top-right corner to start the user pool creation wizard.
4. In **Configure sign-in experience**, select **User name** for **sign-in options**. Keep all other options as **default** and click **Next**.
5. In **Configure security requirements**, ensure that **Password policy mode **is set to **Cognito defaults**. For **Multi-factor authentication,** choose **Optional MFA** with **Authenticator apps** and **SMS message**. Keep all other options as **default** and click **Next**.
6. In **Configure message delivery**, select **Send email with Cognito** for the email provider. Click **Next**.
7. In **Integrate your app**, enter a **User pool name**. Confirm that the **App type** is set to **Public client** and **Don’t generate a client secret** is selected.
8. Under **Advanced app client settings** add `ALLOW_USER_PASSWORD_AUTH` to the list of authentication flows. Click **Next**.
9. Review your settings and click **Create user pool**.

## Create the react front end:

```
npm create vite@latest web-client -- --template react-ts
cd web-client
npm install
```

Add required packages:

```
npm install react-router-dom
npm install @aws-sdk/client-cognito-identity-provider
```

Update the `web-client\src\App.tsx` file:

```js
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import LoginPage from "./LoginPage";
import HomePage from "./HomePage";
import ConfirmUserPage from "./ConfirmUserPage";
import "./App.css";

const App = () => {
  const isAuthenticated = () => {
    const accessToken = sessionStorage.getItem("accessToken");
    return !!accessToken;
  };

  return (
    <BrowserRouter>
      <Routes>
        <Route
          path="/"
          element={
            isAuthenticated() ? (
              <Navigate replace to="/home" />
            ) : (
              <Navigate replace to="/login" />
            )
          }
        />
        <Route path="/login" element={<LoginPage />} />
        <Route path="/confirm" element={<ConfirmUserPage />} />
        <Route
          path="/home"
          element={
            isAuthenticated() ? <HomePage /> : <Navigate replace to="/login" />
          }
        />
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

Add the required pages:

- `web-client\src\HomePage.tsx`

```js
// Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

import { useNavigate } from "react-router-dom";

function parseJwt(token) {
  const base64Url = token.split(".")[1];
  const base64 = base64Url.replace(/-/g, "+").replace(/_/g, "/");
  const jsonPayload = decodeURIComponent(
    window
      .atob(base64)
      .split("")
      .map((c) => `%${`00${c.charCodeAt(0).toString(16)}`.slice(-2)}`)
      .join("")
  );
  return JSON.parse(jsonPayload);
}

const HomePage = () => {
  const navigate = useNavigate();
  const idToken = parseJwt(sessionStorage.idToken.toString());
  const accessToken = parseJwt(sessionStorage.accessToken.toString());
  console.log(
    `Amazon Cognito ID token encoded: ${sessionStorage.idToken.toString()}`
  );
  console.log("Amazon Cognito ID token decoded: ");
  console.log(idToken);
  console.log(
    `Amazon Cognito access token encoded: ${sessionStorage.accessToken.toString()}`
  );
  console.log("Amazon Cognito access token decoded: ");
  console.log(accessToken);
  console.log("Amazon Cognito refresh token: ");
  console.log(sessionStorage.refreshToken);
  console.log(
    "Amazon Cognito example application. Not for use in production applications."
  );
  const handleLogout = () => {
    sessionStorage.clear();
    navigate("/login");
  };

  return (
    <div>
      <h1>Hello World</h1>
      <p>See console log for Amazon Cognito user tokens.</p>
      <button type="button" onClick={handleLogout}>
        Logout
      </button>
    </div>
  );
};

export default HomePage;
```

- `web-client\src\LoginPage.tsx`

```js
import { useState } from "react";
import { useNavigate } from "react-router-dom";
import { signIn, signUp } from "./authService";

const LoginPage = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [isSignUp, setIsSignUp] = useState(false);
  const navigate = useNavigate();

  const handleSignIn = async (e: { preventDefault: () => void }) => {
    e.preventDefault();
    try {
      const session = await signIn(email, password);
      console.log("Sign in successful", session);
      if (session && typeof session.AccessToken !== "undefined") {
        sessionStorage.setItem("accessToken", session.AccessToken);
        if (sessionStorage.getItem("accessToken")) {
          window.location.href = "/home";
        } else {
          console.error("Session token was not set properly.");
        }
      } else {
        console.error("SignIn session or AccessToken is undefined.");
      }
    } catch (error) {
      alert(`Sign in failed: ${error}`);
    }
  };

  const handleSignUp = async (e: { preventDefault: () => void }) => {
    e.preventDefault();
    if (password !== confirmPassword) {
      alert("Passwords do not match");
      return;
    }
    try {
      await signUp(email, password);
      navigate("/confirm", { state: { email } });
    } catch (error) {
      alert(`Sign up failed: ${error}`);
    }
  };

  return (
    <div className="loginForm">
      <h1>Welcome</h1>
      <h4>
        {isSignUp ? "Sign up to create an account" : "Sign in to your account"}
      </h4>
      <form onSubmit={isSignUp ? handleSignUp : handleSignIn}>
        <div>
          <input
            className="inputText"
            id="email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="Email"
            required
          />
        </div>
        <div>
          <input
            className="inputText"
            id="password"
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            placeholder="Password"
            required
          />
        </div>
        {isSignUp && (
          <div>
            <input
              className="inputText"
              id="confirmPassword"
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              placeholder="Confirm Password"
              required
            />
          </div>
        )}
        <button type="submit">{isSignUp ? "Sign Up" : "Sign In"}</button>
      </form>
      <button type="button" onClick={() => setIsSignUp(!isSignUp)}>
        {isSignUp
          ? "Already have an account? Sign In"
          : "Need an account? Sign Up"}
      </button>
    </div>
  );
};

export default LoginPage;
```

- `web-client\src\ConfirmedUserPage.tsx`

```js
import type React from "react";
import { useState } from "react";
import { useLocation, useNavigate } from "react-router-dom";
import { confirmSignUp } from "./authService";

const ConfirmUserPage = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const [email, setEmail] = useState(location.state?.email || "");
  const [confirmationCode, setConfirmationCode] = useState("");

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    try {
      await confirmSignUp(email, confirmationCode);
      alert("Account confirmed successfully!\nSign in on next page.");
      navigate("/login");
    } catch (error) {
      alert(`Failed to confirm account: ${error}`);
    }
  };

  return (
    <div className="loginForm">
      <h2>Confirm Account</h2>
      <form onSubmit={handleSubmit}>
        <div>
          <input
            className="inputText"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="Email"
            required
          />
        </div>
        <div>
          <input
            className="inputText"
            type="text"
            value={confirmationCode}
            onChange={(e) => setConfirmationCode(e.target.value)}
            placeholder="Confirmation Code"
            required
          />
        </div>
        <button type="submit">Confirm Account</button>
      </form>
    </div>
  );
};

export default ConfirmUserPage;
```

Now create the service that talks with Cognito:

- Create a file `web-client\src\authService.ts`:

```js
import {
  CognitoIdentityProviderClient,
  InitiateAuthCommand,
  SignUpCommand,
  ConfirmSignUpCommand,
  type InitiateAuthCommandInput,
  type SignUpCommandInput,
  type ConfirmSignUpCommandInput,
} from "@aws-sdk/client-cognito-identity-provider";
import config from "./config.json";

export const cognitoClient = new CognitoIdentityProviderClient({
  region: config.region,
  ...(config.endpoint ? { endpoint: config.endpoint } : {}),
});

export const signIn = async (username: string, password: string) => {
  const params: InitiateAuthCommandInput = {
    AuthFlow: "USER_PASSWORD_AUTH",
    ClientId: config.clientId,
    AuthParameters: {
      USERNAME: username,
      PASSWORD: password,
    },
  };
  try {
    const command = new InitiateAuthCommand(params);
    const { AuthenticationResult } = await cognitoClient.send(command);
    if (AuthenticationResult) {
      sessionStorage.setItem("idToken", AuthenticationResult.IdToken || "");
      sessionStorage.setItem(
        "accessToken",
        AuthenticationResult.AccessToken || ""
      );
      sessionStorage.setItem(
        "refreshToken",
        AuthenticationResult.RefreshToken || ""
      );
      return AuthenticationResult;
    }
  } catch (error) {
    console.error("Error signing in: ", error);
    throw error;
  }
};

export const signUp = async (email: string, password: string) => {
  const params: SignUpCommandInput = {
    ClientId: config.clientId,
    Username: email,
    Password: password,
    UserAttributes: [
      {
        Name: "email",
        Value: email,
      },
    ],
  };
  try {
    const command = new SignUpCommand(params);
    const response = await cognitoClient.send(command);
    console.log("Sign up success: ", response);
    return response;
  } catch (error) {
    console.error("Error signing up: ", error);
    throw error;
  }
};

export const confirmSignUp = async (username: string, code: string) => {
  const params: ConfirmSignUpCommandInput = {
    ClientId: config.clientId,
    Username: username,
    ConfirmationCode: code,
  };
  try {
    const command = new ConfirmSignUpCommand(params);
    await cognitoClient.send(command);
    console.log("User confirmed successfully");
    return true;
  } catch (error) {
    console.error("Error confirming sign up: ", error);
    throw error;
  }
};
```

Configure the Cognito Connection:

- Create a file calles `web-client\src\config.json`:

```json
{
  "region": "YOUR_AWS_REGION",
  "userPoolId": "YOUR_COGNITO_USER_POOL_ID",
  "clientId": "YOUR_COGNITO_APP_CLIENT_ID"
}
```

Update the values accordingly.

## Test Locally

Create a `docker-compose.yml` file with this content:

```yml
version: "3.8"
services:
  cognito-local:
    image: jagregory/cognito-local:latest
    ports:
      - "9229:9229" # Expose the debug port
    volumes:
      - cognito-data:/app/.cognito # Mount the .cognito volume
    environment: # You can add environment variables here if needed.
      - CODE=ABC123 # Signup confirmation code is ABC123
    # networks: #Define networks
    #   - your_network_name
#networks: #Define networks
#  your_network_name:
#    driver: bridge
volumes:
  cognito-data:
```

Start the docker image:

`docker compose up -d`

Create dummy user Pool

```
aws --endpoint http://localhost:9229 cognito-idp create-user-pool --pool-name MyUserPool
```

Use the UserPool:Id value and create an app client, for ecample:

```
aws --endpoint http://localhost:9229 cognito-idp create-user-pool-client  --user-pool-id local_1AgkV9U3 --client-name my-app-client --no-generate-secret
```

Update the configuration values accordingly:

```json
{
  "region": "local",
  "userPoolId": "local_1AgkV9U3",
  "clientId": "abu0j1b6of8repfd5y91grp95",
  "endpoint": "http://localhost:9229/"
}
```

Run and Test :)
