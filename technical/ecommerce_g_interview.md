# 💼 E-commerce G Interview Experience

## 🧪 Round: NLP & Data Processing
**📅 Date**:  
**🏷️ Tags**: `Customer Support`, `PII Data`, `LLM`, `NLP`, `Data Preprocessing`, `Regex`

---

## ❓ Q1. How to preprocess user queries and remove Personally Identifiable Information (PII)?

---

<details>
<summary>🙋 My Answer</summary>

### 🛠️ Approach

1. **PII Removal:**
   - **Account Number:** Match patterns like account number or bank details and replace them.
   - **IFSC Code & Branch:** Regex to filter out IFSC and branch code details.
   - **Dollar Amount:** Regex pattern for detecting currency symbols or amounts.

2. **Message Preprocessing:**
   - **Remove Repetition:** Detect repeated words (e.g., "where where is my deposit?!") and correct.
   - **Remove Extra Spaces/Newlines:** Clean up unneeded spaces and line breaks.
   - **Emoji Removal:** Detect and remove emojis using regex.
   - **Spacy for NLP:** Use SpaCy to process the message and extract entities (e.g., account details, amounts).

### 📝 Code Example

```python
import re
import spacy

# Load the SpaCy model
nlp = spacy.load('en_core_web_sm')

class PreprocessMessage:
    def __init__(self, msg):
        self.msg = msg

    def remove_pii(self):
        # Example regex for account numbers
        account_pattern = r"(\d{2}-\d{4}-\d{4})"  # Example pattern for account number
        self.msg = re.sub(account_pattern, "[REDACTED_ACCOUNT]", self.msg)

        # Remove IFSC Code
        ifsc_pattern = r"\b[A-Z]{4}\d{7}\b"
        self.msg = re.sub(ifsc_pattern, "[REDACTED_IFSC]", self.msg)

        # Remove Amounts
        amount_pattern = r"\$\d+(\.\d{2})?"
        self.msg = re.sub(amount_pattern, "[REDACTED_AMOUNT]", self.msg)

        return self.msg

    def preprocess(self):
        self.msg = self.remove_pii()
        # Remove extra spaces or newlines
        self.msg = re.sub(r'\s+', ' ', self.msg)
        # Process with SpaCy
        doc = nlp(self.msg)
        return doc

# Example usage
message = "Where is my deposit of $500 made yesterday (Account Number: 1234-5678-1234)?"
preprocessor = PreprocessMessage(message)
processed_msg = preprocessor.preprocess()
print(processed_msg)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧑‍💻 Approach

1. **PII Removal:**

   * **Account Information:** Apply regex patterns to detect account numbers, IFSC codes, or bank-related data and mask them.
   * **Amount Detection:** Detect any monetary values or amounts in the message and replace them.

2. **Message Preprocessing:**

   * Clean up any noisy elements like emojis, extra spaces, and redundant phrases.
   * Use SpaCy's NLP capabilities to tokenize, recognize named entities, and ensure proper cleanup of the message.

```python
import re
import spacy

# Load SpaCy model
nlp = spacy.load("en_core_web_sm")

class TextPreprocessing:
    def __init__(self, message):
        self.message = message

    def remove_pii(self):
        # Example pattern for account number
        account_pattern = r"\b\d{4}-\d{4}-\d{4}-\d{4}\b"  # Simplified regex
        self.message = re.sub(account_pattern, "[REDACTED_ACCOUNT]", self.message)

        # IFSC Code pattern
        ifsc_pattern = r"\b[A-Z]{4}\d{7}\b"
        self.message = re.sub(ifsc_pattern, "[REDACTED_IFSC]", self.message)

        # Dollar Amount
        amount_pattern = r"\$\d+(\.\d{2})?"
        self.message = re.sub(amount_pattern, "[REDACTED_AMOUNT]", self.message)

        return self.message

    def clean_message(self):
        # Remove unwanted spaces and newlines
        self.message = re.sub(r'\s+', ' ', self.message).strip()
        doc = nlp(self.message)
        return doc

# Usage Example
query = "Where is my deposit of $1000 made yesterday (Account Number: 1234-5678-9876)?"
processor = TextPreprocessing(query)
cleaned_message = processor.clean_message()
print(cleaned_message)
```

</details>

---

## ❓ Q2. How to process user data (transaction data) and identify product categories?

---

<details>
<summary>🙋 My Answer</summary>

### 🛠️ Approach

1. **Transaction Aggregation:**

   * **Group by product and transaction ID:** Aggregate transactions for each user.
   * **Concatenate product names:** Create a list of products per user per transaction.

2. **Example User Data**:

   * Aggregated product list by transaction.
   * Example: User A bought "Perfume" and "Chocolate", which are grouped into one transaction.

### 📝 Code Example

```python
import pandas as pd

# Sample Data
data = {
    'user': ['a', 'a', 'b', 'c', 'c', 'd', 'd', 'd', 'a'],
    'product': ['perfume', 'chocolate', 'coke', 'coke', 'chocolate', 'perfume', 'coke', 'chocolate', 'perfume'],
    'txn_id': [1234, 1234, 1235, 1236, 1236, 1237, 1237, 1237, 1238]
}

df = pd.DataFrame(data)

# Group products by transaction
grouped = df.groupby(['txn_id'])['product'].apply(lambda x: ', '.join(x)).reset_index()

# Display grouped transaction data
print(grouped)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧑‍💻 Approach

* **Group Data by Transaction ID:** Create a list of products per transaction.
* **Concatenate Products:** Combine product names using a delimiter (e.g., commas).
* **Transaction-level Aggregation:** This allows understanding of which products are purchased together.

```python
import pandas as pd

# Sample Transaction Data
data = {
    'user': ['a', 'a', 'b', 'c', 'c', 'd', 'd', 'd', 'a'],
    'product': ['perfume', 'chocolate', 'coke', 'coke', 'chocolate', 'perfume', 'coke', 'chocolate', 'perfume'],
    'txn_id': [1234, 1234, 1235, 1236, 1236, 1237, 1237, 1237, 1238]
}

df = pd.DataFrame(data)

# Group and Concatenate Products per Transaction
grouped_products = df.groupby(['txn_id'])['product'].apply(', '.join).reset_index()
print(grouped_products)
```

</details>

---

## ❓ Q3. How to track transaction pairs and calculate total transaction count?

---

<details>
<summary>🙋 My Answer</summary>

1. **Set Hashmap:**

   * Create a dictionary of product pairs and track their occurrence.
   * Update counts for each pair as they appear in the data.

### 📝 Code Example

```python
# Create a dictionary to store product pair frequencies
product_pairs = {}

def update_count(df):
    for txn_id, group in df.groupby('txn_id'):
        products = group['product'].tolist()
        for i in range(len(products)):
            for j in range(i + 1, len(products)):
                pair = tuple(sorted([products[i], products[j]]))
                if pair in product_pairs:
                    product_pairs[pair] += 1
                else:
                    product_pairs[pair] = 1

# Apply the function
update_count(df)

# Display result
print(product_pairs)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

1. **Track Pair Frequencies:**

   * Use a dictionary to count product pairs across transactions.
   * Store counts for each pair and update on encountering new transaction pairs.

```python
# Sample transaction data
product_pairs = {}

def track_pairs(df):
    for txn_id, group in df.groupby('txn_id'):
        products = group['product'].tolist()
        for i in range(len(products)):
            for j in range(i + 1, len(products)):
                pair = tuple(sorted([products[i], products[j]]))
                product_pairs[pair] = product_pairs.get(pair, 0) + 1

# Run the function to count pairs
track_pairs(df)
print(product_pairs)
```

</details> 