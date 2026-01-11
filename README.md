# Complete Beginner Guide to Azure Data Factory Parameterized Data Ingestion

## Table of Contents
1. [Understanding Azure Data Storage Hierarchy](#understanding-azure-data-storage-hierarchy)
2. [Why This Architecture?](#why-this-architecture)
3. [Prerequisites Setup](#prerequisites-setup)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Understanding Each Component](#understanding-each-component)

---

## 1. Understanding Azure Data Storage Hierarchy

### The Big Picture: What is Azure Data Lake Storage Gen2?

Think of Azure as a massive cloud computing platform. Within Azure, you need a designated location to store your data files, such as CSV, JSON, and Parquet files. **Azure Data Lake Storage Gen2 (ADLS Gen2)** is the storage service designed for this purpose.

### The Storage Hierarchy (Top to Bottom)

![Storage Hierarchy Diagram 1](https://github.com/user-attachments/assets/5fa406ea-3d2d-4eb8-8242-806206b8b79c)
![Storage Hierarchy Diagram 2](https://github.com/user-attachments/assets/3db84d14-8ae9-4824-b1c4-c65f02e5c96f)

### Detailed Explanation of Each Level

#### **1. Azure Subscription**
- **What it is**: Your Azure account and billing container
- **Real-world analogy**: Similar to your Microsoft account that receives billing charges
- **Example**: "My Company's Azure Subscription"

#### **2. Resource Group**
- **What it is**: A logical container that groups related Azure resources
- **Real-world analogy**: A project folder on your computer containing all files related to a single project
- **Purpose**: Facilitates easier management, deletion, monitoring, and access control for all resources together
- **Example**: "RG-DataEngineering-Project"
- **Contains**: Storage accounts, Data Factories, databases, and other Azure resources

#### **3. Storage Account**
- **What it is**: The actual storage service in Azure (ADLS Gen2 is a type of storage account)
- **Real-world analogy**: A physical hard drive or cloud storage service like Google Drive
- **Naming requirements**: Must be globally unique, lowercase only, 3-24 characters
- **Example**: "mydatalakestorage2024"
- **Why it is separate from Resource Group**: You can create multiple storage accounts for different purposes within the same resource group

#### **4. Container (Filesystem)**
- **What it is**: The top-level organizational unit within a storage account
- **Real-world analogy**: Similar to the C:\ drive on Windows or main folders in Google Drive
- **Purpose of multiple containers**: To separate different data zones or environments
- **Common naming patterns**:
  - `raw-data` or `bronze` - Raw, unprocessed data from sources
  - `processed-data` or `silver` - Cleaned and transformed data
  - `curated-data` or `gold` - Business-ready, aggregated data
  - `staging` - Temporary landing zone
  - `archive` - Historical data backup

#### **5. Directories (Folders)**
- **What they are**: Subfolders within a container
- **Real-world analogy**: Regular folders on your computer
- **Purpose**: Organize data by date, source, project, or any logical grouping
- **Example structure**:
```
raw-data/
    ├── brazilian-ecommerce/
    │   ├── customers/
    │   ├── orders/
    │   └── products/
    ├── sales-data/
    └── marketing-data/
```

#### **6. Files**
- **What they are**: Your actual data files
- **Supported formats**: CSV, JSON, Parquet, Avro, ORC, Excel, and others
- **Example**: `olist_customers_dataset.csv`

---

## 2. Why This Architecture?

### The Problem We're Solving

Imagine you need to copy 100 CSV files from GitHub to Azure. You could:
- **Inefficient approach**: Create 100 separate Copy activities (time-consuming and difficult to maintain)
- **Optimal approach**: Use parameterization with a ForEach loop (scalable and maintainable)

### Benefits of Parameterized Pipelines

1. **Scalability**: Add or remove files by simply updating a parameter
2. **Maintainability**: Change logic once, and it applies to all files
3. **Reusability**: The same pipeline can be used for different data sources
4. **Efficiency**: Enable parallel processing to copy multiple files simultaneously
5. **Reduced complexity**: One Copy activity instead of 100

### The Data Flow Architecture

```
GitHub Repository (Source)
    ↓
Azure Data Factory Pipeline (Orchestration)
    ↓
ForEach Loop (Iterate through file list)
    ↓
Copy Activity (Copy each file)
    ↓
Azure Data Lake Gen2 (Destination)
    ↓
Container → Folder → File Saved
```

---

## 3. Prerequisites Setup

### Step 1: Create a Resource Group

**What it is**: A logical container for all your project resources

**How to create**:
1. Navigate to the [Azure Portal](https://portal.azure.com)
2. Click **"Resource groups"** in the left menu or search for "Resource group"

![Resource Group Navigation](https://github.com/user-attachments/assets/d6606d6d-acda-4520-8e7c-ee9b9ab7cf07)

3. Click **"+ Create"**
4. Complete the following fields:
   - **Subscription**: Select your subscription
   - **Resource group name**: `RG-DataEngineering-Demo`
   - **Region**: Choose the region closest to you (e.g., `East US`, `Central India`)
5. Click **"Review + Create"** then **"Create"**

**Why region matters**: Data transfer is faster when resources are located in the same region

---

### Step 2: Create Azure Data Lake Storage Gen2

**What it is**: Your data storage account

**How to create**:
1. In the Azure Portal, click **"+ Create a resource"**
2. Search for **"Storage account"**

![Storage Account Search](https://github.com/user-attachments/assets/59330e59-6d5c-4993-9f0a-4d53947d10c2)

3. Click **"Create"**
4. Complete the configuration tabs:

#### **Basics Tab**:
   - **Subscription**: Your subscription
   - **Resource group**: Select `RG-DataEngineering-Demo`
   - **Storage account name**: `datalake[yourname]2024` (must be globally unique, lowercase, no spaces)
   - **Region**: Same as Resource Group (e.g., `East US`)
   - **Performance**: Standard (cost-effective for most use cases)
   - **Redundancy**: LRS (Locally-redundant storage) - most economical option for demonstrations

#### **Advanced Tab**:
   - **Hierarchical namespace**: **ENABLE THIS** (This is what makes it Data Lake Gen2)
   - Leave other settings as default

#### **Networking Tab**:
   - **Network access**: Enable public access (for demonstration purposes)
   - For production environments: Use private endpoints

#### **Data Protection Tab**:
   - Leave defaults (you can disable soft delete for cost savings in demonstrations)

5. Click **"Review + Create"** then **"Create"**
6. Wait for deployment to complete (approximately 1-2 minutes)

**Why Hierarchical Namespace is important**: It enables a file and folder structure instead of only blob storage

---

### Step 3: Create Container in Storage Account

**What it is**: A top-level folder in your storage account

**How to create**:
1. Navigate to your newly created storage account
2. In the left menu, click **"Containers"** under "Data storage" or search for "container"

![Container Navigation](https://github.com/user-attachments/assets/0374c340-a947-4fa2-95c5-62f22e804147)

3. Click **"+ Container"**
4. Complete the fields:
   - **Name**: `raw-data` (lowercase, hyphens allowed)
   - **Public access level**: Private (default - most secure)
5. Click **"Create"**

**Create a folder inside the container** (Optional but recommended):
1. Click on the `raw-data` container
2. Click **"+ Add Directory"**
3. Name it: `brazilian-ecommerce`
4. Click **"Create"**

**Your storage structure**:
```
datalake[yourname]2024 (Storage Account)
    └── raw-data (Container)
        └── brazilian-ecommerce (Folder)
            └── [Files will be copied here]
```

---

### Step 4: Create Azure Data Factory

**What it is**: The orchestration service that moves and transforms data

**How to create**:
1. In the Azure Portal, click **"+ Create a resource"**
2. Search for **"Data Factory"**
3. Click **"Create"**
4. Complete the configuration tabs:

#### **Basics Tab**:
   - **Subscription**: Your subscription
   - **Resource group**: Select `RG-DataEngineering-Demo`
   - **Region**: Same as Storage Account
   - **Name**: `adf-datapipeline-demo` (must be globally unique)
   - **Version**: V2 (default)

#### **Git Configuration Tab**:
   - Select **"Configure Git later"** (for simplicity)

5. Click **"Review + Create"** then **"Create"**
6. Wait for deployment to complete (approximately 2-3 minutes)
7. Click **"Go to resource"**
8. Click **"Launch Studio"** - This opens Azure Data Factory Studio

![ADF Studio Launch](https://github.com/user-attachments/assets/ae262c75-d74c-4169-9ddf-e5a83d791f26)

---

## 4. Step-by-Step Implementation

### Phase 1: Create Linked Services (Connections)

**What are Linked Services**: Connection configurations that specify how Data Factory connects to external systems.

#### **A. Create HTTP Linked Service (for GitHub)**

**Purpose**: Connect to GitHub to read CSV files

**Steps**:
1. In ADF Studio, click **"Manage"** (toolbox icon on the left sidebar)
2. Click **"Linked services"**

![Linked Services](https://github.com/user-attachments/assets/26275d5a-dbc6-4b18-83e7-602ffb2cf1fc)

3. Click **"+ New"**
4. Search for **"HTTP"**
5. Click **"Continue"**
6. Complete the configuration:
   - **Name**: `LS_GitHub_Raw`
   - **Description**: "Connection to GitHub raw files"
   - **Connect via integration runtime**: `AutoResolveIntegrationRuntime`
   - **Base URL** (the GitHub repository from which we will fetch data): 
   ```
   https://raw.githubusercontent.com/mayank953/BigDataProjects/main/
   ```
   **IMPORTANT**: Must end with `/` (forward slash)
   - **Authentication type**: `Anonymous`

7. Click **"Test connection"** (should display "Connection successful")
8. Click **"Create"**

**Why the Base URL ends with `/`**: The relative path will be appended to this base URL

---

#### **B. Create ADLS Gen2 Linked Service (for Storage)**

**Purpose**: Connect to your Azure Data Lake to write CSV files

**Steps**:
1. While still in **"Linked services"**, click **"+ New"**
2. Search for **"Azure Data Lake Storage Gen2"**
3. Click **"Continue"**
4. Complete the configuration:
   - **Name**: `LS_ADLS_Gen2`
   - **Description**: "Connection to Data Lake Storage"
   - **Connect via integration runtime**: `AutoResolveIntegrationRuntime`
   - **Authentication method**: `Account key` (most straightforward for beginners)
   
   **For Account Key Method**:
   - **Account selection method**: From Azure subscription
   - **Azure subscription**: Select your subscription
   - **Storage account name**: Select your storage account (e.g., `datalake[yourname]2024`)

5. Click **"Test connection"** (should display "Connection successful")
6. Click **"Create"**

---

### Phase 2: Create Datasets (Data Structures)

**What are Datasets**: Templates that describe the structure and location of your data.

#### **A. Create HTTP Dataset (Source - GitHub CSV)**

**Purpose**: Define how to read CSV files from GitHub

**Steps**:
1. Click **"Author"** (pencil icon on the left sidebar)
2. Click **"+"** next to "Datasets" in the left panel
3. Search for **"HTTP"**

![HTTP Dataset](https://github.com/user-attachments/assets/360a5688-8113-4ba1-bc25-f9f74426b496)

4. Click **"Continue"**
5. Select format: **"DelimitedText"** (CSV)
6. Click **"Continue"**
7. Complete the configuration:

#### **Properties Tab**:
   - **Name**: `DS_GitHub_CSV`
   - **Linked service**: Select `LS_GitHub_Raw`
   - **File path**: Leave the "Relative URL" field EMPTY for now
   - **First row as header**: Check this box

#### **Parameters Tab** (Important):
   - Click **"+ New"**
   - **Name**: `RelativePath`
   - **Type**: `String`
   - Click outside to save

![Dataset Parameters](https://github.com/user-attachments/assets/ba78b3d1-dfed-49cb-98db-33f84797909b)

#### **Connection Tab**:
   - In the **"Relative URL"** field, click the small **dynamic content icon**
   - In the expression builder, paste:
   ```
   @dataset().RelativePath
   ```
   - Click **"OK"**

![Dataset Connection](https://github.com/user-attachments/assets/c37de0b3-20e4-4248-921f-9a58adfac08c)

8. Click **"OK"** to close the dataset

**What this accomplishes**: You have created a dataset that will dynamically retrieve its file path from a parameter

**Understanding the Expression**:
- `@dataset()` - Refers to the current dataset
- `.RelativePath` - Retrieves the value of the RelativePath parameter
- This allows the same dataset to read different files

---

#### **B. Create ADLS Gen2 Dataset (Destination - Azure Storage)**

**Purpose**: Define where to write CSV files in Azure

**Steps**:
1. Click **"+"** next to "Datasets"
2. Search for **"Azure Data Lake Storage Gen2"**
3. Click **"Continue"**
4. Select format: **"DelimitedText"**
5. Click **"Continue"**
6. Complete the configuration:

#### **Properties Tab**:
   - **Name**: `DS_ADLS_CSV_Output`
   - **Linked service**: Select `LS_ADLS_Gen2`
   - **File path**: 
     - **Container**: Type `raw-data`
     - **Directory**: Type `brazilian-ecommerce`
     - **File name**: Leave empty (we will make it dynamic)
   - **First row as header**: Check this box

#### **Parameters Tab** (Important):
   - Click **"+ New"**
   - **Name**: `FileName`
   - **Type**: `String`
   - Click outside to save

#### **Connection Tab**:
   - In the **"File name"** field (under File path), click the **dynamic content icon**
   - In the expression builder, paste:
   ```
   @dataset().FileName
   ```
   - Click **"OK"**

7. Click **"OK"** to close the dataset

**What this accomplishes**: You have created a dataset that will dynamically save files with different names

---

### Phase 3: Create the Pipeline

**What is a Pipeline**: A logical grouping of activities that together perform a task

#### **Step 1: Create Pipeline**

1. In **"Author"**, click **"+"** next to "Pipelines"
2. Click **"Pipeline"**
3. Name it: **"PL_Ingest_GitHub_CSV"**

![Pipeline Creation](https://github.com/user-attachments/assets/95ba851d-f054-44e7-b701-83752f2d97f8)

---

#### **Step 2: Add Pipeline Parameter (Array of File Names)**

**Purpose**: Store the list of files to copy

**Steps**:
1. Click on the **empty canvas** (ensure no activity is selected)
2. At the bottom, click the **"Parameters"** tab
3. Click **"+ New"**
4. Complete the fields:
   - **Name**: `FileNames`
   - **Type**: Select **Array** from dropdown (MUST BE ARRAY)
   - **Default value**: Click in the box and paste this JSON array (example files):
   ```json
   ["olist_customers_dataset.csv","olist_geolocation_dataset.csv","olist_order_items_dataset.csv","olist_order_payments_dataset.csv","olist_order_reviews_dataset.csv","olist_orders_dataset.csv","olist_products_dataset.csv","olist_sellers_dataset.csv","product_category_name_translation.csv"]
   ```

![Pipeline Parameters](https://github.com/user-attachments/assets/97adc242-9b1a-47a9-afc2-cdf35161f516)

5. Click **outside the box** to confirm

---

#### **Step 3: Add ForEach Activity**

**Purpose**: Loop through each file name in the array

**Steps**:
1. In the **"Activities"** panel (left side), expand **"Iteration & conditionals"**
2. **Drag "ForEach"** activity onto the canvas
3. Click on the ForEach activity to select it
4. At the bottom, in the **"General"** tab:
   - **Name**: `ForEach_CSV_Files`

![ForEach General](https://github.com/user-attachments/assets/3642abec-cc9f-4227-99be-3def54a8a4d4)

5. Navigate to the **"Settings"** tab:
   - **Sequential**: Uncheck (enables parallel processing for faster execution)
   - **Batch count**: Leave default or set to `20` (number of parallel threads)
   - **Items**: Click inside, then click **"Add dynamic content"**
   - In the expression builder, paste:
   ```
   @pipeline().parameters.FileNames
   ```
   - Click **"OK"**

**What this accomplishes**: 
- The ForEach activity will iterate through each file name in the `FileNames` array
- `@pipeline().parameters.FileNames` references the parameter we created
- During each iteration, the current file name is accessible via `@item()`

**Sequential vs Parallel**:
- **Sequential (checked)**: Files are copied one by one (slower but uses fewer resources)
- **Parallel (unchecked)**: Multiple files are copied simultaneously (faster)

---

#### **Step 4: Add Copy Activity Inside ForEach**

**Purpose**: Copy each individual file from GitHub to Azure

**Steps**:
1. **Click on the ForEach activity**
2. Click the **pencil icon** (Edit) that appears on the ForEach box
3. You are now **inside the ForEach loop** (notice the breadcrumb at the top: "Pipeline > ForEach_CSV_Files")
4. In the **"Activities"** panel, expand **"Move & transform"**
5. **Drag "Copy data"** activity onto the canvas inside ForEach
6. Click on the Copy activity, in the **"General"** tab:
   - **Name**: `Copy_CSV_Files`

![Copy Activity](https://github.com/user-attachments/assets/52dfeb8d-dfc9-4d58-a82c-db1ab34b8c97)

---

#### **Step 5: Configure Copy Activity - Source (GitHub)**

**Purpose**: Specify where the Copy activity should read data from

**Steps**:
1. Click on the **Copy activity**
2. Navigate to the **"Source"** tab at the bottom
3. **Source dataset**: Select `DS_GitHub_CSV` from the dropdown
4. The **Dataset properties** section appears:
   - **RelativePath**: Click inside, then click **"Add dynamic content"**
   - In the expression builder, paste:
   ```
   @concat('Project-Brazilian%20Ecommerce/Data/', item())
   ```
   - Click **"OK"**

**Understanding the Expression**:
```
@concat('Project-Brazilian%20Ecommerce/Data/', item())
```
- `concat()` - Joins strings together
- `'Project-Brazilian%20Ecommerce/Data/'` - The folder path (note `%20` represents a space)
- `item()` - The current file name from the ForEach loop
- **Result**: `Project-Brazilian%20Ecommerce/Data/olist_customers_dataset.csv`

**Complete URL formed**:
```
Base URL (from Linked Service): https://raw.githubusercontent.com/mayank953/BigDataProjects/main/
+
Relative Path: Project-Brazilian%20Ecommerce/Data/olist_customers_dataset.csv
=
Full URL: https://raw.githubusercontent.com/mayank953/BigDataProjects/main/Project-Brazilian%20Ecommerce/Data/olist_customers_dataset.csv
```

**Why `%20`**: URLs cannot contain spaces. `%20` is the URL-encoded format for a space character.

---

#### **Step 6: Configure Copy Activity - Sink (Azure Storage)**

**Purpose**: Specify where the Copy activity should write data to

**Steps**:
1. While still on the **Copy activity**, navigate to the **"Sink"** tab
2. **Sink dataset**: Select `DS_ADLS_CSV_Output` from the dropdown
3. The **Dataset properties** section appears:
   - **FileName**: Click inside, then click **"Add dynamic content"**
   - In the expression builder, paste:
   ```
   @item()
   ```
   - Click **"OK"**

**What this accomplishes**:
- `@item()` retrieves the current file name from the ForEach loop
- Each file will be saved with its original name
- **Example**: If the current item is `olist_customers_dataset.csv`, that will be the file name used

**Where files will be saved**:
```
Storage Account: datalake[yourname]2024
└── Container: raw-data
    └── Folder: brazilian-ecommerce
        └── File: olist_customers_dataset.csv  ← Saved here
```

---

#### **Step 7: Navigate Back to Main Pipeline**

1. Click **"PL_Ingest_GitHub_CSV"** in the breadcrumb at the top (above the canvas)
2. You should now see the ForEach activity on the main canvas

![Pipeline Overview](https://github.com/user-attachments/assets/6c3a9fde-d976-492b-b5c5-48294a5f8b38)

---

### Phase 4: Validate and Publish

#### **Step 1: Validate**

**Purpose**: Check for any configuration errors

**Steps**:
1. Click the **"Validate all"** button in the top toolbar
2. A panel opens on the right showing validation results

![Validation](https://github.com/user-attachments/assets/f85b2449-ea1f-4d5d-9815-8d3585416011)

3. **Look for**:
   - "Your pipeline has been validated. No errors found." - This indicates success
   - Any error messages - These must be resolved before proceeding

---

#### **Step 2: Publish**

**Purpose**: Save all your changes to Azure Data Factory

**Steps**:
1. Click the **"Publish all"** button in the top toolbar
2. A **"Pending changes"** panel opens showing all modifications
3. Review the changes (you should see your pipeline, datasets, and linked services)
4. Click **"Publish"**
5. Wait for the success message: "Publishing succeeded"

**What happens**: All your configurations are now saved to Azure and ready to execute

---

### Phase 5: Test the Pipeline

#### **Step 1: Debug Run (Testing)**

**Purpose**: Run the pipeline in test mode to verify functionality

**Steps**:
1. Ensure you are on the main pipeline canvas (not inside ForEach)
2. Click the **"Debug"** button in the top toolbar
3. A **"Pipeline run parameters"** panel appears on the right
4. You should see the `FileNames` parameter with the array of file names
5. **Verify the parameter value is correct** (should display the JSON array)
6. Click **"OK"** to start the run

---

#### **Step 2: Monitor Execution**

**Purpose**: Watch the pipeline execute in real-time

**Steps**:
1. At the bottom, click the **"Output"** tab
2. You will see a row for the pipeline run with status "In Progress"
3. Monitor the **ForEach_CSV_Files** activity:
   - Status will show "In Progress" with a spinning icon
   - Duration will increase as it runs
   - Click the **glasses icon** to view inside the ForEach

4. **Inside ForEach view**:
   - You will see multiple **Copy_CSV_Files** activities (one for each file)
   - Each displays:
     - **Activity name**: Copy_CSV_Files
     - **Status**: In Progress then Succeeded (or Failed)
     - **Duration**: Execution time
     - Click on any activity to view details

5. **Success indicators**:
   - Green checkmarks next to all activities
   - Status: "Succeeded"
   - All 9 copy activities completed

6. **If any activity fails**:
   - A red X icon appears
   - Click on the failed activity
   - Click the **error icon** to view the error message

![Pipeline Execution](https://github.com/user-attachments/assets/c5897f50-0082-48aa-a76c-23cae0974f01)

---

#### **Step 3: Verify Files in Storage**

**Purpose**: Confirm files were successfully copied to Azure

**Steps**:
1. Return to the **Azure Portal**
2. Navigate to your **Storage Account** (`datalake[yourname]2024`)
3. Click **"Storage browser"** in the left menu
4. Click **"Blob containers"**
5. Click on the **"raw-data"** container
6. Click on the **"brazilian-ecommerce"** folder
7. **You should see all 9 CSV files**:

![Verified Files](https://github.com/user-attachments/assets/a8945c96-f24a-45a4-86a1-06a7d593e520)

8. **Click on any file** to:
   - Download it
   - View its properties (size, last modified time)
   - Preview the content

**Success!** Your parameterized pipeline is functioning correctly.

---

## 5. Understanding Each Component in Depth

### What is a Linked Service?

**Definition**: A connection configuration that stores information about how to connect to external data sources or destinations.

**Real-world analogy**: Similar to a phone number in your contacts - it specifies how to reach someone

**Key components**:
1. **Connection information**: URL, server address, account name
2. **Authentication**: Credentials proving access authorization
3. **Integration Runtime**: The compute resource that executes the connection

**Types commonly used**:
- HTTP/REST API - Connect to web services
- Azure Data Lake Storage Gen2 - Connect to Azure storage
- Azure SQL Database - Connect to databases
- Azure Blob Storage - Connect to blob storage
- SFTP - Connect to file servers
- Amazon S3 - Connect to AWS storage

**Why use separate Linked Services**: 
- Reusable across multiple datasets
- Easier credential updates in a single location
- Enhanced security - can integrate with Azure Key Vault for secrets

---

### What is a Dataset?

**Definition**: A named view of data that points to or references the data you want to use in activities

**Real-world analogy**: Similar to a template or blueprint that describes the structure of your data

**Key components**:
1. **Linked Service**: Which connection to use
2. **Structure**: File format (CSV, JSON, Parquet)
3. **Schema**: Column names and data types (optional)
4. **Parameters**: Variables to enable dynamic behavior

**Why use parameters in datasets**:

Without parameters:
```
Dataset1: Read file1.csv from GitHub
Dataset2: Read file2.csv from GitHub
Dataset3: Read file3.csv from GitHub
... (requires 100 datasets for 100 files)
```

With parameters:
```
Dataset_Generic: Read {FileName} from GitHub
→ Can be used for any file by passing the file name as a parameter
```

---

### What is a Pipeline?

**Definition**: A logical grouping of activities that together perform a task

**Real-world analogy**: Similar to a recipe with multiple steps to create a dish

**Key components**:
1. **Activities**: The individual steps (Copy, Transform, Execute)
2. **Parameters**: Input values that make the pipeline reusable
3. **Variables**: Values that change during pipeline execution
4. **Dependencies**: Execution order (Activity A must complete before Activity B)

**Types of activities**:
- **Move & Transform**: Copy data, Data flow
- **Iteration & Conditionals**: ForEach, If Condition, Until, Switch
- **Execute**: Execute pipeline, Stored procedure, Databricks notebook
- **General**: Web (call REST API), Get Metadata, Wait

---

### What is a ForEach Activity?

**Definition**: An iteration control flow that repeats a set of activities for each item in a collection

**Real-world analogy**: Similar to a `for` loop in programming

**How it works**:
```python
# In programming terms:
files = ["file1.csv", "file2.csv", "file3.csv"]
for file in files:
    copy_file(file)
```

**In Data Factory**:
```
Items: @pipeline().parameters.FileNames
For each file in FileNames:
    Execute Copy Activity with current file
```

**Sequential vs Parallel**:
- **Sequential**: Processes items one after another (similar to standing in a queue)
- **Parallel**: Processes multiple items simultaneously (similar to multiple cashiers at a store)

**Batch count**: Number of parallel threads to use (default: 20, maximum: 50)

---

### What are Expressions and Functions?

**Definition**: Dynamic content that is evaluated at runtime

**Syntax**: Always begins with `@`

**Common expressions**:

1. **Reference a parameter**:
```
@pipeline().parameters.ParameterName
```

2. **Reference a variable**:
```
@variables('VariableName')
```

3. **Current item in ForEach**:
```
@item()
```

4. **Concatenate strings**:
```
@concat('string1', 'string2', variable)
```

5. **Get current date**:
```
@utcnow()
@formatDateTime(utcnow(), 'yyyy-MM-dd')
```

6. **Conditional logic**:
```
@if(equals(pipeline().parameters.env, 'prod'), 'production-container', 'dev-container')
```

---

### Understanding the Data Flow

Let's trace exactly what occurs when you execute this pipeline:

#### **Step-by-Step Execution**:

1. **Pipeline starts**:
   - Reads parameter: `FileNames = ["file1.csv", "file2.csv", "file3.csv"]`

2. **ForEach activity executes**:
   - Takes the `FileNames` array
   - Creates iterations: 3 iterations (one for each file)

3. **First iteration**:
   - `@item()` = `"file1.csv"`
   - Copy activity starts:
     - **Source**: 
       - Uses `DS_GitHub_CSV` dataset
       - Passes `RelativePath = "Project-Brazilian%20Ecommerce/Data/file1.csv"`
       - Forms full URL: `https://raw.githubusercontent.com/.../file1.csv`
     - **Sink**:
       - Uses `DS_ADLS_CSV_Output` dataset
       - Passes `FileName = "file1.csv"`
       - Saves to: `raw-data/brazilian-ecommerce/file1.csv`

4. **Second iteration**: Same process with `file2.csv`

5. **Third iteration**: Same process with `file3.csv`

6. **Pipeline completes**: All files copied successfully
