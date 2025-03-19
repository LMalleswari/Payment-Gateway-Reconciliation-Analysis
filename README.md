# **Payment Gateway Reconciliation Analysis**

## **Problem Statement:**
Task is to analyze three data sets related to payment gateway reconciliation and match the data using common identifiers as mentioned.And also need to document the matching logic and any assumptions made during the process.
 
### **Objective of the assignment is following:**
 
1.Find Duplicate records in three dataset.

2.Find matching records between all three dataset by tracking payment Prepare summary of matching/unmatching record and amount.

3.Identify cases of any amount mismatch Prepare logic document including various assumptions made during the process.

4.What would be your recommendations to the customer?

Also,prepare a project plan in order to automate this process for a similar new set of data for upcoming days. This should have all the assumptions that have been taken during the planning.

###  **We will use Python to automate the reconciliation process by following these logical steps:**

- *Step 1 : Understanding the data*
  
   #### Collection Sheet :
  Represents the payments collected from customers.

      GOAL ID: Unique identifier assigned to a collection goal or category.  
      GROUP NAME:  Group or category of the transaction.
      AMOUNT: Transaction amount.
      INSTALLMENT:  Installment details.
      TRANS TYPE:  Transaction type.
      PAYMENT MODE:  Payment method.
      RECEIVED DATE:  Date the payment was received.
      FTR NUMBER:  Internal reference number.
      BANK REF NUMBER: Bank reference number.
      TRANS DATE: Transaction date.
      TRANS TIME:  Transaction time.
      PAYMENT GATEWAY : The payment gateway through which the transaction was processed 
      RECON FLAG: Indicates whether the transaction has been reconciled (Yes or No).
    #### Key Fields for Reconciliation: AMOUNT, TICKET_NO, PAYMENT_GATEWAY, FTR_NUMBER.

  ####  SY Sheet :  
    Represents transaction records from an internal system.

         AMOUNT: The payment amount processed.
         CHITS GROUP ID: Group ID assigned to the transaction.
         CUSTOMER ID: Identifies the customer.
         PAYMENT DATE TIME:  The date and time the payment was recorded.
         PAYMENT DATE:  The date of the payment.
         GOAL ID:  An identifier related to a goal or objective.
         UTR NO:  Unique Transaction Reference number.
         VENDOR TXN ID: Vendor's transaction identifier.

    ####  Key Fields for Reconciliation: UTR_NO, AMOUNT, VENDOR_TXN_ID.

    #### CF Sheet :
     Contains detailed records of transactions processed by the payment gateway.

      ORDER ID: Unique order ID linked to the payment.
      REFERENCE ID: Internal reference ID used for tracking the transaction
      AMOUNT: The transaction amount.
      SETTLEMENT AMOUNT:  The final settlement amount after charges and taxes.
      PAYMENT MODE:The method used for payment .
      BANK NAME: The name of the bank involved.
      TRANS DATE TIME:  The date and time of the transaction.
      TRANS DATE: The date of the transaction.
      TRXNS STATUS: The status of the transaction 
      SETTLEMENT: The settlement status
      UTR NO:  The Unique Transaction Reference number.
      SETTLED_DATE TIME: The date and time the transaction was settled.
      SETTLED DATE: The date the transaction was settled.
      CUSTOMER REFERENCE_ID:  A reference ID associated with the customer.
      
- *Step 2: Data Cleaning and processing*

      Read the Excel file and load each sheet into a separate DataFrame.

             import pandas as pd 

             filepath = r'D:\Training\Data analyst\real data set\Data for assignment_vf.xlsx'

              sy_df = pd.read_excel(filepath,sheet_name="SY",header=1)
              cf_df = pd.read_excel(filepath, sheet_name="CF",header=1)
              collection_df = pd.read_excel(filepath, sheet_name="Collection",header=1)
      
              sy_df=sy_df.drop(columns=sy_df.columns[0])
              cf_df=cf_df.drop(columns=cf_df.columns[0])
              collection_df=collection_df.drop(columns=collection_df.columns[0])      

              sy_df.head(2)
              cf_df.head(2)
              collection_df.head(2)
      
         # Identify duplicate records
               sy_duplicates = sy_df[sy_df.duplicated()]
               cf_duplicates = cf_df[cf_df.duplicated()]
               collection_duplicates = collection_df[collection_df.duplicated()]

         # Separate non-duplicate records
               sy_non_duplicates = sy_df.drop_duplicates()
               cf_non_duplicates = cf_df.drop_duplicates()
               collection_non_duplicates = collection_df.drop_duplicates()


- *Step3: Identify & remove Duplicates*
      
              # Identify duplicate records
                     sy_duplicates = sy_df[sy_df.duplicated()]
                     cf_duplicates = cf_df[cf_df.duplicated()]
                     collection_duplicates = collection_df[collection_df.duplicated()]

              # Separate non-duplicate records
                     sy_df = sy_df.drop_duplicates()
                     cf_df = cf_df.drop_duplicates()
                     collection_df = collection_df.drop_duplicates()
   
   
- *Step4: Understand the Matching Criteria for Reconciliation*

               collection_df["FTR_NUMBER"] = collection_df["FTR_NUMBER"].astype(str).str.strip() 
               sy_df["VENDOR_TXN_ID"] = sy_df["VENDOR_TXN_ID"].astype(str).str.strip()
               cf_df["ORDER_ID"] =cf_df["ORDER_ID"].astype(str).str.strip()


            #Merge collection_df and sy_df on FTR_NUMBER and VENDOR_TXN_ID
                  sy_collections=pd.merge(collection_df,sy_df,left_on="FTR_NUMBER",right_on="VENDOR_TXN_ID",how = "inner")
                  sy_collections.head(2)


            #Merge collection_df and cf_df on FTR_NUMBER and VENDOR_TXN_ID
                  cf_collections=pd.merge(collection_df,cf_df,left_on="FTR_NUMBER",right_on="ORDER_ID",how = "inner")
                  cf_collections.head(2)

- *step5: Identifying Matched & Unmatched Transactions*

             #Check if Amounts Match
                   sy_collections["Amount_Match"] = sy_collections["AMOUNT_x"] == sy_collections["AMOUNT_y"]
                   cf_collections["Amount_Match"] = cf_collections["AMOUNT_x"] == cf_collections["AMOUNT_y"] 

             #Store Matched & Unmatched records in DataFrames
                    matched_collection_sy_df = sy_collections[sy_collections["Amount_Match"] == True]
                    unmatched_collection_sy_df = sy_collections[sy_collections["Amount_Match"] == False]
                    matched_cf_collections_df = cf_collections[cf_collections["Amount_Match"] == True]
                    unmatched_cf_collections_df = cf_collections[cf_collections["Amount_Match"] == False]

- *step6: Summary of Matching & Unmatching Records*
          
              #Summary for Collection & SY
                    summary_sy = {
                    "Total Records in Collection & SY": len(sy_collections),
                    "Matched Records": len(matched_collection_sy_df),
                    "Unmatched Records": len(unmatched_collection_sy_df)
                    }

              # Summary for Collection & CF
                    summary_cf = {
                    "Total Records in Collection & CF": len(cf_collections),
                    "Matched Records": len(matched_cf_collections_df),
                    "Unmatched Records": len(unmatched_cf_collections_df)
                    }

               # Print Summary
                    print("Summary of Collection & SY:", summary_sy)
                    print("Summary of Collection & CF:", summary_cf)

- *Step7: Identifying Amount Mismatches cases*

               # Find Amount Mismatches in Collection & SY
                    amount_mismatch_sy_df = sy_collections[
                    sy_collections["AMOUNT_x"] != sy_collections["AMOUNT_y"]
                    ]

               # Find Amount Mismatches in Collection & CF
                    amount_mismatch_cf_df = cf_collections[
                    cf_collections["AMOUNT_x"] != cf_collections["AMOUNT_y"]
                    ]

               # Create Summary Dictionary
                    amount_mismatch_summary = {
                    "Total Records in Collection & SY": len(sy_collections),
                    "Amount Mismatch Cases (Collection & SY)": len(amount_mismatch_sy_df),
                    "Total Records in Collection & CF": len(cf_collections),
                    "Amount Mismatch Cases (Collection & CF)": len(amount_mismatch_cf_df),
                    }


               # Display Summary
                    print("Cases of amount mismatch : ",amount_mismatch_summary)


 
   *Recommendations:*

   1.Investigate Amount Mismatches

   2.Cross-check mismatches with actual bank/payment records.

   3.Implement a dashboard for real-time tracking.

   4.Audit Unmatched Transactions

   5.Investigate missing transactions.

   6.Regular Data Validation & Cleanup

--

## **Project Plan for Automating Payment Reconciliation Process**

## **Objective**
To automate the payment gateway reconciliation process for upcoming datasets by developing a scalable, efficient, and error-free system that identifies duplicate records, matches payments, summarizes matching/unmatching records, and identifies mismatches.

## **Project Phases & Timeline**

### **Phase 1: Requirement Gathering & Data Understanding**
- Identify data sources and formats.
- Define reconciliation rules and edge cases.
- Gather business requirements for automation.


### **Phase 2: Data Preparation & Preprocessing**
- Develop Python scripts to ingest data from multiple sources.
- Implement data cleaning (removal of duplicates, standardization of formats, handling missing values).
- Store cleaned data in a structured database.

### **Phase 3: Data Matching & Validation**
- Implement logic to match transactions across datasets using identifiers.
- Identify matched, unmatched, and mismatched transactions.
- Apply amount verification rules and flag discrepancies.
- Store results in a structured format.

### **Phase 4: Reporting & Monitoring**
- Generate automated reconciliation reports.
- Create a dashboard for real-time monitoring.
- Implement email notifications for unmatched/mismatched transactions.

### **Phase 5: Testing & Deployment**
- Conduct end-to-end testing with historical data.
- Perform performance tuning.
- Deploy automation scripts on a scheduled basis.
- Train users on monitoring and handling exceptions.

### **Phase 6: Maintenance & Optimization)**

- Incorporate feedback and continuously improve automation.

## **Technology Stack**
- **Programming Language:** Python (Pandas, NumPy)
- **Data Storage:** SQL Database / Cloud Storage
- **Automation & Scheduling:** Apache Airflow / Cron Jobs
- **Reporting & Visualization:** Power BI / Tableau / Excel Reports
- **Alerting System:** Email Notifications / Slack Alerts

---

## **Expected Benefits**
1. **Efficiency:** Reduces manual effort and processing time.
2. **Accuracy:** Minimizes errors in transaction matching.
3. **Scalability:** Can handle large volumes of transactions.
4. **Transparency:** Provides clear audit trails and reports.
5. **Proactive Monitoring:** Alerts for mismatches and missing transactions.

---


