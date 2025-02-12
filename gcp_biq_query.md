# 🚀 Getting Started with Google BigQuery Using CLI & TypeScript

Google BigQuery is a **serverless, highly scalable, and cost-effective** cloud data warehouse designed for fast SQL queries using the processing power of **Google's infrastructure**. This tutorial covers:

✅ Setting up BigQuery on **Google Cloud Platform (GCP)** via CLI.
✅ Creating and managing datasets and tables.
✅ Querying BigQuery using **TypeScript**.

---

## **1️⃣ Setting Up BigQuery on GCP**

### **Prerequisites**
- A **Google Cloud Platform (GCP)** account ([Sign up here](https://cloud.google.com/)).
- Installed **Google Cloud SDK (gcloud CLI)** ([Download](https://cloud.google.com/sdk/docs/install)).
- Node.js (LTS version) with **TypeScript & npm** installed.

### **Step 1: Enable BigQuery API**
Run the following command to enable the **BigQuery API** for your project:

```sh
 gcloud services enable bigquery.googleapis.com
```

### **Step 2: Authenticate & Set Project**
```sh
 gcloud auth login  # Authenticate your GCP account
 gcloud config set project [PROJECT_ID]  # Replace with your actual GCP project ID
```

### **Step 3: Create a BigQuery Dataset**
A dataset in BigQuery acts as a **container** for your tables.

```sh
 gcloud bigquery datasets create my_dataset
```

### **Step 4: Create a Table in BigQuery**
```sh
gcloud bigquery tables create my_dataset.users \
--schema="id:STRING,name:STRING,age:INTEGER,email:STRING" \
--description="User Information Table"
```

### **Step 5: Insert Sample Data**
```sh
gcloud bq query \
--use_legacy_sql=false \
"INSERT INTO `my_project.my_dataset.users` (id, name, age, email) VALUES ('1', 'John Doe', 30, 'john@example.com');"
```

---

## **2️⃣ Querying BigQuery Using TypeScript**

### **Step 1: Install Required Packages**
Ensure you have Node.js installed, then install the **BigQuery SDK**:

```sh
 npm install --save @google-cloud/bigquery typescript dotenv
```

### **Step 2: Set Up Authentication**
To authenticate, download your **GCP service account key**:
1. Go to the **Google Cloud Console** → IAM & Admin → Service Accounts.
2. Create a service account and download the **JSON key**.
3. Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable:

```sh
 export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your-service-account.json"
```

Or, create a `.env` file:
```env
GOOGLE_APPLICATION_CREDENTIALS=/path/to/your-service-account.json
```

### **Step 3: Write a TypeScript Script to Query BigQuery**

Create a `queryBigQuery.ts` file:

```typescript
import { BigQuery } from "@google-cloud/bigquery";
import dotenv from "dotenv";

dotenv.config(); // Load env variables

const bigquery = new BigQuery();

async function queryBigQuery() {
  const query = `SELECT * FROM \`my_project.my_dataset.users\` LIMIT 10;`;

  const options = { query, location: "US" };
  const [rows] = await bigquery.query(options);
  
  console.log("🚀 Query Results:", rows);
}

queryBigQuery().catch(console.error);
```

### **Step 4: Run the Query Script**
```sh
 ts-node queryBigQuery.ts
```

Expected Output:
```json
[
  { "id": "1", "name": "John Doe", "age": 30, "email": "john@example.com" }
]
```

---

## **3️⃣ Bonus: Creating & Inserting Data from TypeScript**

### **Create a Table from TypeScript**
```typescript
async function createTable() {
  const dataset = bigquery.dataset("my_dataset");
  const table = dataset.table("users");
  const schema = [
    { name: "id", type: "STRING" },
    { name: "name", type: "STRING" },
    { name: "age", type: "INTEGER" },
    { name: "email", type: "STRING" },
  ];
  
  await table.create({ schema });
  console.log("✅ Table created!");
}

createTable().catch(console.error);
```

### **Insert Data from TypeScript**
```typescript
async function insertData() {
  const dataset = bigquery.dataset("my_dataset");
  const table = dataset.table("users");
  
  const rows = [
    { id: "2", name: "Alice Smith", age: 28, email: "alice@example.com" },
  ];
  
  await table.insert(rows);
  console.log("✅ Data inserted successfully!");
}

insertData().catch(console.error);
```

---

## **🎯 Conclusion**
🎉 Congratulations! You’ve successfully:
✅ Set up **Google BigQuery** using CLI.  
✅ Queried BigQuery from **TypeScript**.  
✅ Created tables and inserted data dynamically.  


