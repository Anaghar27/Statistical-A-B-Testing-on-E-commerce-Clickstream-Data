# Statistical A/B Testing on E-commerce Clickstream Data

This project applies statistical A/B testing methods to an e-commerce clickstream dataset to measure feature adoption and conversion lift. The dataset contains user-level interactions such as product views, add-to-cart events, and purchases. The goal is to simulate product experiments and evaluate whether new features lead to higher adoption rates.

---

## Step 1: Dataset Setup

This project uses the **E-commerce Behavior Data from Multi-category Store** dataset from Kaggle. Each monthly CSV file is between 5GB and 9GB compressed and expands to 15–30GB when unzipped.

### Step 1A: Enable Kaggle API
1. Log in to Kaggle and go to your account settings.  
2. Under the **API** section, click **Create New API Token**.  
3. A file named `kaggle.json` will be downloaded.  
4. Place `kaggle.json` inside the `.kaggle` folder in your home directory:  
   - Mac/Linux: `~/.kaggle/kaggle.json`  
   - Windows: `C:\Users\<YourName>\.kaggle\kaggle.json`  
5. On Mac/Linux, set permissions to secure the file:  
   ```bash
   chmod 600 ~/.kaggle/kaggle.json
   ```

### Step 1B: Install Kaggle CLI
Install the Kaggle command-line tool:
```bash
pip install kaggle
```

Verify installation:
```bash
kaggle --version
```

### Step 1C: Download the Dataset
Download the dataset using the Kaggle CLI:
```bash
kaggle datasets download -d mkechinov/ecommerce-behavior-data-from-multi-category-store
```

To automatically unzip the dataset into a `data/` folder:
```bash
kaggle datasets download -d mkechinov/ecommerce-behavior-data-from-multi-category-store -p data/ --unzip
```

After unzipping, the `data/` folder will contain monthly CSV files such as:
- `2019-Oct.csv`  
- `2019-Nov.csv`  

### Notes on Dataset Size
- The dataset is very large (~30GB per month when unzipped).  
- Loading the full dataset into memory with Pandas is not practical.  
- For small-scale experiments, use a subset with the `nrows` option in Pandas.  
- For full-scale analysis, distributed frameworks such as **PySpark** or **DuckDB** can be used on a local machine.

---

## Step 2: Loading Data with PySpark

Since the dataset is too large for Pandas, **PySpark** is used to process it.  
- Java and Apache Spark were installed and configured locally on the system.  
- A PySpark session was started in Jupyter Notebook.  
- The dataset files (`2019-Oct.csv` and `2019-Nov.csv`) were loaded into a Spark DataFrame.  
- Schema inspection confirmed columns such as `event_time`, `event_type`, `product_id`, `category_code`, `brand`, `price`, `user_id`, and `user_session`.  
- The combined dataset had around **110 million rows**.  

---

## Step 3: Data Preprocessing

The preprocessing steps included:  
1. **Handling Missing Values**  
   - Filled missing `category_code` and `brand` values with `"unknown"`.  
   - Dropped rows with missing `price` and `user_session` values.

2. **Filtering Events**  
   - Only relevant funnel events were retained: **view → cart → purchase**.  

3. **Feature Engineering**  
   - Extracted `event_date` from the timestamp.  
   - Randomly assigned each unique user to either **Group A** or **Group B** to simulate an experiment.  

4. **Defining Conversion**  
   - A user was considered converted if they had at least one purchase event.  

5. **Aggregating Metrics**  
   - Conversion rates were calculated for Group A and Group B.  
   - Both groups had nearly identical conversion rates (~13.1%), as expected from random assignment.  

---

## Step 4: Statistical Hypothesis Testing

After preprocessing and calculating conversion rates for both groups, we performed statistical tests to determine whether the difference between Group A and Group B was significant.

### Hypotheses:
- Null Hypothesis (H₀): Conversion rates for Group A and Group B are equal.
- Alternative Hypothesis (H₁): Conversion rates for Group A and Group B are different.

### Tests Performed:
- Chi-Square Test for Independence
  - Used a 2×2 contingency table of conversions vs. non-conversions.
  - Result: p-value ≈ 0.95 → fail to reject H₀.

-Two-Proportion Z-Test
  - Compared the two conversion proportions directly.
  - Result: p-value ≈ 0.93 → fail to reject H₀.

### Interpretation:
- Both tests showed p > 0.05, which means there is no statistically significant difference in conversion rates between Group A (~13.12%) and Group B (~13.18%).
- This outcome is expected since users were randomly split into groups without any real feature change applied.
- The analysis validates the experiment setup and statistical testing pipeline. In a real scenario, the same methodology would reveal whether a new feature or product change significantly impacts conversions.

