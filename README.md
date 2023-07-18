# Company Response to Consumer Complaints

[ [Project Description](#project-description) ]
[ [Project Goal](#project-goal) ]
[ [Initial Thoughts](#initial-thoughts) ]
[ [The Plan](#the-plan) ]
[ [Acquire &amp; Prep](#acquire) ]
[ [Explore](#explore) ]
[ [Data Dictionary](#data-dictionary) ]
[ [Modeling](#model) ]
[ [Steps to Reproduce](#steps-to-reproduce) ]
[ [Conclusion](#conclusion) ]
[ [Meet the Team](#meet-the-team) ]

## Project Description

[Back to Top](#company-response-to-consumer-complaints)

The Consumer Financial Protection Bureau (CFPB) has a Consumer Complaint Database that is a collection of complaints about consumer financial products and services that they sent to companies for a response. Complaints are published after the company responds, confirming a commercial relationship with the consumer, or after 15 days, whichever comes first. Complaints referred to other regulators, such as complaints about depository institutions with less than $10 billion in assets, are not published in the Consumer Complaint Database.

This database is not a statistical sample of consumers’ experiences in the marketplace. Complaints are not necessarily representative of all consumers’ experiences and complaints do not constitute “information” for purposes of the Information Quality Act.

Complaint volume should be considered in the context of company size and/or market share. For example, companies with more customers may have more complaints than companies with fewer customers. CFPB encourages users to pair complaint data with public and private datasets for additional context.

The Bureau publishes the consumer’s narrative description of his or her experience if the consumer opts to share it publicly and after the Bureau removes personal information. CFPB doesn’t verify all the allegations in complaint narratives. Unproven allegations in consumer narratives should be regarded as opinions, not facts. CFPB does not adopt the views expressed and makes no representation that the consumers’ allegations are accurate, clear, complete, or unbiased in substance or presentation. Users should consider what conclusions may be fairly drawn from complaints alone.

## Project Goal

[Back to Top](#company-response-to-consumer-complaints)

This project aims to predict a company's response to a complaint made by a consumer to the CFPB to see if the wording of a complaint can affect the response from a company.

### Initial Thoughts

* There are going to be keywords that match a company's response.
* Sentiment analysis will not be useful because most complaints will likely be negative.

## Deliverables

* [Here](https://www.canva.com/design/DAFolc43Z1Y/eeld7BULceqtaXHpmf2mFA/edit?utm_content=DAFolc43Z1Y&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton) is a slide presentation summarizing findings in exploration and results of modeling.

## The Plan

[Back to Top](#company-response-to-consumer-complaints)

### Acquire

* Data acquired from [Google BigQuery](https://console.cloud.google.com/marketplace/product/cfpb/complaint-database)
* 3,458,906 rows × 18 columns *before* cleaning

### Prepare

* Clean the data
  * Drop Columns
  * Rename columns
  * Remove nulls
  * Fixed data type
* Create engineered columns from existing data
  * Bin products
  * Process narrative into clean and lemon
* Encode categorical columns
* Split data (60/20/20)

<details>
  <summary>Show more preparation details</summary>

* **Dropped Columns**
  * **product**
    * 0% null
    * *ENGINEERED FEATURE*
    * **bin related products/services together then drop**
      * bins = credit_report, credit_card, debt_collection, mortgage, bank, loans, and money_service
  * **subproduct**
    * 7% null
  * **issue**
    * 0% nulls
    * 165 unique values
    * **Planned use in future iteration**
  * **subissue**
    * 20% null
    * 221 unique
  * **consumer_complaint_narrative**
    * 64% null
    * renamed to narrative
      * **drop all null values**
      * **drop after NLTK cleaning**
  * **company_public_response**
    * 56% null
    * related to target
  * **zip_code**
    * 1% null
    * mixed data types
      * **Planned use in future iteration**
  * **consumer_consent_provided**
    * 25% null
    * does not relate to the target
  * **submitted_via**
    * 0% null
    * does not relate to the target
  * **date_sent_to_company**
    * 0% null
      * **Planned use in future iteration**
  * **timely_response**
    * 0% null
    * boolean
      * **Planned use in future iteration**
  * **consumer_disputed**
    * 77% null
  * **complaint_id**
    * 0% null

---

* **Cleaned Columns**
  * **date_received**
    * 0% nulls
    * changed date to DateTime
    * keep for purposes of exploration
  * **company_name**
    * 0% nulls
    * 6,694 Companies
  * **state**
    * 1% null
    * keep for purposes of exploration
      * **impute 1% null into UNKNOWN label**
  * **tags**
    * 89% null
      * **impute nulls with "Average Person label**
  * **company_response_to_consumer**
    * Target
    * 4 nulls = 0%
      * **Drop these 4 rows because this is the target column**
    * 8 initial unique values
      * **future: apply the model to in_progress complaints and see what it predicts based on the language**
      * **Drop 'in progress' response because there is no conclusion**

---

#### Post-Cleaning Inspection

[Back to Top](#company-response-to-consumer-complaints)

1,246,736 rows x 8 columns

Used NLTK to clean each document resulting in:

* 2 new columns: *clean* (tokenized, numbers/specials, and XX's removed) and *lemon* (removed stopwords, kept real words, and lemmatized)

Selected columns to explore after cleaning:

* date_received, product_bins, company_name, state, tags, company_response_to_customer (target), clean, lemon

</details>

---

### Explore

[Back to Top](#company-response-to-consumer-complaints)

**1. Are there words that get particular responses and is there a relationship?**

* What are the payout words that got a company response of closed with monetary relief?
* Are there unique words associated with products? Is there a relationship between unique product words and responses?

**2. Do all responses have a negative sentiment?**

* Do narratives with a neutral or positive sentiment analysis relating to bank account products lead to a response of closed with monetary relief?

**3. Are there unique words associated with the most negative and most positive company responses?**

**4. Which product is more likely to have monetary relief?**

<!-- 
---
**FUTURE ITERATION QUESTIONS**

5. Is there a relationship/bias for servicemember tags in relation to company response?

   * good but it's better after MVP
     NEW - Is the company response independent from a consumer being a service member or not?
     * Is the company response independent from a consumer having a tag?
6. Do some companies proportionally give better or worse responses (relief/no relief)?

   * Look at this after MVP
7. Does narrative length relate to company response?

   * plain answer - may be out
8. Do consumers in some states receive monetary relief more frequently than others?

   * Look at this after MVP
9. Are there more complaints during certain seasons of the year?

   * not valid for modeling - OUT -->

### Data Dictionary

[Back to Top](#company-response-to-consumer-complaints)

[Here](https://cfpb.github.io/api/ccdb/fields.html) is a link to the CFPB's official data dictionary

| Feature                               | Definition                                                                                  |
| :------------------------------------ | :------------------------------------------------------------------------------------------ |
| date_received                         | Date the complaint was received by the CFPB                                                 |
| product                               | The type of product the consumer identified in the complaint                                |
| subproduct                            | The type of sub-product the consumer identified in the complaint                            |
| issue                                 | The issue the consumer identified in the complaint                                          |
| subissue                              | The sub-issue the consumer identified in the complaint                                      |
| consumer_complaint_narrative          | A description of the complaint provided by the consumer                                     |
| company_public_response               | The company's optional public-facing response to a consumer's complaint                     |
| company_name                          | Name of the company identified in the complaint by the consumer                             |
| state                                 | Two-letter postal abbreviation of the state of the mailing address provided by the consumer |
| zip_code                              | The mailing ZIP code provided by the consumer                                               |
| tags                                  | Older American is aged 62 and older, Servicemember is Active/Guard/Reserve member or spouse |
| consumer_consent_provided             | Identifies whether the consumer opted in to publish their complaint narrative               |
| submitted_via                         | How the complaint was submitted to the CFPB                                                 |
| date_sent_to_company                  | The date the CFPB sent the complaint to the company                                         |
| company_response_to_consumer (target) | The response from the company about this complaint                                          |
| timely_response                       | Indicates whether the company gave a timely response or not                                 |
| consumer_disputed                     | Whether the consumer disputed the company's response, discontinued as of April 24, 2017    |
| complaint_id                          | Unique ID for complaints registered with the CFPB                                           |
| product_bins                          | Engineered Feature: bin related products together                                           |
| clean                                 | Engineered Feature: tokenized, removed numbers/specials and XXs from privacy sanitization  |
| lemon                                 | Engineered Feature: removed stopwords, kept real words, and lemmatized the clean column    |

## Model

[Back to Top](#company-response-to-consumer-complaints)

### Data Sample

* Calculated the sample size for each class category using a 20% sampling rate.

* Created smaller datasets by sampling the specified number of samples from each class category.

### Term Frequencies used

* Count Vectorizer
* TF-IDF

### Classification Models

* Decision Tree
* Random Forest
* Logistic Regression

### Evaluation Metric

* Accuracy
  * **Baseline: 78.79%</span>**

### Features Sent In

* Top 2,900 words in the 'lemon' column

---

## Steps to Reproduce

[Back to Top](#company-response-to-consumer-complaints)

1) Clone this repo
   * You may need to update your Python Libraries, our libraries were updated in June 2023
2) For a relatively quick run *(possibly 10+ min, depends on system resources)*
   * Verify `df = wrangle_complaints()` is in the cell after imports of final-report
   * Run the final-report notebook
   * This will use a pre-built and cleaned dataset that would be produced from the longer steps below
     * Even after cleaning, the data amounts to just under 1GB
     * Runtime may take some time due to sentiment analysis and modeling

<details>
  <summary>3) For the longer run *(possibly 30+ min, depends on system resources)*</summary>

* ⚠️WARNING⚠️: These are basically the same steps we took to originally acquire the data. The actions take a lot of time (and disk space) and may not even be the best way. We highly recommend doing the quick run in step 2 unless you want to know how we got the data and experience the long wait.
* Verify `df = wrangle_complaints_the_long_way()` is in the cell after imports of final-report
* Install the pandas-gbq package through the terminal/command line
  * `pip install pandas-gbq`
* Go to [Google BigQuery](https://console.cloud.google.com/marketplace/product/cfpb/complaint-database) and create a project
  * Copy the 'SQL_query' variable found in `wrangle.py` and run in [Google BigQuery](console.cloud.google.com/bigquery?ws=!1m5!1m4!4m3!1sbigquery-public-data!2scfpb_complaints!3scomplaint_database)
  * Save the result as a BigQuery table in your project
    * You can look in `wrangle.py` for what we named the project, database, and table (we kept the database and table names the same as the original)
  * Edit and save the 'SQL_query' variable found in `wrangle.py` to the respective table names in your BigQuery project using this format: `FROM database. table` and edit the 'project_ID' variable to your project's ID
  * Create a [Service Account](https://console.cloud.google.com/iam-admin/serviceaccounts) and key for your project
    * Save the key as `service_key.json` into the local repo
* Run the final-report notebook and be patient
  * It may ask for authentication when it tries to query Google BigQuery
  * Try to run again if it stopped for authentication
* This will run through the longer pathway of getting the data from the source and performing the cleaning and natural language preparation
  * It will probably take a while **(3.5 million rows, +2GB)**, hence we do not recommend

</details>

---

## Conclusion

### Takeaways

[Back to Top](#company-response-to-consumer-complaints)

* The analysis of words used in complaints against company responses suggests that specific words can be associated with certain product categories and used to predict the type of response.
* The most common words in consumer complaints include account, credit, report, information, payment, loan, time, debt, company, and day.
* The response "Closed with explanation" is the most likely outcome from a complaint, suggesting that companies provide detailed explanations in their responses rather than relief.
* Different product categories receive different types of responses based on the sentiment of the complaints/narratives.
* A Decision Tree Classifier with a max depth of 9 and minimum sample leaf of 11 performed best on the train, validate, and test data, but all models fell short of expectations.
* Further exploration of feature combinations, n-gram types, and evaluation metrics is underway to enhance prediction accuracy.

### Recommendations and Next Steps

* **Enhance Response Analysis**: The project highlights the need to analyze company responses to consumer complaints. Consider investing in natural language processing (NLP) techniques to extract meaningful insights from response data. By understanding the patterns and sentiments in responses, it might be possible to identify areas for improvement and optimize customer interactions.
* **Monitor Sentiment and Product Categories**: Pay attention to sentiment analysis of consumer complaints across different product categories. Identify trends in sentiment and response types to understand customer expectations and tailor the response strategies accordingly. This can help to improve the overall customer experience and target specific pain points in different product categories.
* **Address Discrimination and Bias**: Conduct further analysis on zip codes, states, and company responses to identify potential discrimination or bias in the complaint resolution process. Ensure fairness and equality by addressing any disparities and taking appropriate actions to eliminate discriminatory practices.
* **Identify Industry Trends**: Look for industry-specific trends by analyzing complaints related to specific companies, such as Silicon Valley Bank and Bank of America. This analysis can help identify emerging issues, detect patterns of non-compliance, and proactively address potential risks.
* **Continuous Improvement**: Treat the project as a starting point and continuously refine the complaint resolution processes. Regularly review customer feedback, complaints, and company responses to identify areas for improvement. Implement a feedback loop to integrate customer insights into operations and drive continuous improvement initiatives.

<!-- 
### Next Steps

* Look at discrimination based on zip code/state and company response
* Apply the model to 'in_progress' complaints and see what it predicts based on the language after company response
* Look at Silicon Valley Bank complaints to identify signs or trends that could have led to their failure
* Look at Bank of America complaints to identify signs of their illegal fees and fake accounts -->

## Meet the Team

[Back to Top](#company-response-to-consumer-complaints)

GitHubs:
[ [Alexia Lewis](https://github.com/lewisalexia) ]
[ [Rosendo Lugo](https://github.com/rosendo-lugo) ]
[ [Chellyann Moreno](https://github.com/chellyann-moreno) ]
[ [Tyler Kephart](https://github.com/tkephart96) ]
