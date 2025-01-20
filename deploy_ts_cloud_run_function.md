# GCP Functions

## Install gcloud CLI

```bash
brew install --cask google-cloud-sdk
```

```bash
gcloud init
```

```bash
gcloud --version
```

## Create Project

```bash
npm init -i
```

## Install Dependencies

```bash
npm i @google-cloud/functions-framework
npm i -D typescript
```

## Important Note

To satisfy TypeScript, I had to install additional packages. Without them, you might encounter compilation errors.

```bash
npm i dlx @yarnpkg/sdks vscode
```

## Write the Function

Create a file named `index.ts` with the following content:

```typescript
import { Request, Response } from 'express';

export const hello = (req: Request, res: Response): void => {
  res.send(`Hello from GCP function ${new Date().toISOString()}`);
};
```

## Configure TypeScript

Generate a `tsconfig.json` file:

```bash
npx tsc --init
```

Update the `tsconfig.json` file to include the following:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "outDir": "lib",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

Move the `index.ts` file to a `src` directory to match this configuration.

## Build the Project

```bash
npm run gcp-build
```

Add the following `scripts` section to your `package.json` if it doesn’t already exist:

```json
"scripts": {
  "gcp-build": "tsc",
  "start": "functions-framework --target hello --port 8081"
}
```

## Start the Project Locally

```bash
npm run start
```

Output:

```bash
Serving function...
Function: hello
Signature type: http
URL: http://localhost:8081/
```

Go to the URL, and you should see:

```bash
Hello from GCP function 2025-01-20T07:32:42.067Z
```

## Deploy to GCP Cloud

If you are not logged in, authenticate your account:

```bash
gcloud auth login
```

Deploy the function:

```bash
gcloud functions deploy hello \
    --gen2 \
    --runtime=nodejs20 \
    --region=us-west1 \
    --source=. \
    --entry-point=hello \
    --trigger-http \
    --allow-unauthenticated
```

The command will produce a console output with the URL to invoke the function. Example output:

```bash
https://us-west1-gketest-448214.cloudfunctions.net/hello
```

## Function installed on the GCP console

<img width="970" alt="Screenshot 2025-01-20 at 17 42 48" src="https://github.com/user-attachments/assets/d010def1-78d2-4800-9347-697bd27e279b" />

<img width="986" alt="Screenshot 2025-01-20 at 17 43 58" src="https://github.com/user-attachments/assets/2606fffb-0cb2-4287-a6bc-04352579f27b" />

## Testing the Deployed Function

Visit the URL provided in the output, and you should see:

```bash
Hello from GCP function 2025-01-20T07:32:42.067Z
```

## Cleanup Resources

To avoid unnecessary charges, delete the deployed function when it is no longer needed:

```bash
gcloud functions delete hello --region=us-west1
```

link to repo:https://github.com/EmiRoberti77/gcp_sample_cloud_ts_function

Emi Roberti - Happy Coding
