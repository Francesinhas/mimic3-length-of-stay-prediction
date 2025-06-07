## **Plan**
- chosing the disease
	- sql scripts to sort diseases by frequencies - *doc*
	- #q more data - better model results VS less data - potentially more interpretable results
	- #q should we model data for each and every patient?
- ### **Preprocessing**
	- relevant tables
		- ADMISSIONS
		- CHARTEVENTS    ^
		- DIAGNOSES_ICD
		- ICUSTAYS   ^
		- PATIENTS   ^
		- D_ICD_DIAGNOSIS
		- D_ITEMS
	- exclude patients that are far outliers #q
		- very young and very old
		- very short and very long stays
	- exclude patients with not enough data
		- not the case
	- exclude patients with invalid data
		- dismiss before admission
		- negative or other invalid values 
	- deal with missing data
		- ~ missigno library
		- strategies...
	- normalize continous features
		- ? how do we normalize Values in CHARTEVENTS ?
			- values of different measurement units #q 
			- should we use different feeatures for each measurement (ITEM) to preserve the individual meanings?
	- aggregate irregular time-stamped events into fixed time windows (h0, h1, h2, ...)
		- normalization of time windows starting from the admission time
	- encode categorigal features into numerical values using one-hot encoding
		- or should we just leave individual features?
		- #a label encoding
	- #### **Feature engineering**
		- #todo corelation matrix - define importance of features
			- especially to find out which ITEM to take into account
		- include higher-level related clinical concepts
			- aggregate granular data into more meaningful concepts
			- Charlson Comorbidity Index (CCI)
			- APS, SAPS, ...
	- **tslearner python** #imp
		- for time series
	- 
- ### **Preparation**
	- patieint-wise train-test split


## **Progress**
- [ ] select disease with ~100 patients
	- potential:
		- 85201 - Subarachnoid hemorrhage, without loss of consciousness - 142 patients
		- 85200 - ..-.., unspecified state of consciousness - 103
		- 4660 - acute bronchitis - 125
		- [[#^813fa3]] - query for more
	- 


## **Notes**
### query for diseases of some numbers

^813fa3

```
COPY (
    SELECT 
        diags.ICD9_CODE AS disease_id,
        dict.LONG_TITLE AS disease_name,
        COUNT(DISTINCT diags.SUBJECT_ID) AS patient_count
    FROM read_csv_auto('DIAGNOSES_ICD.csv') AS diags
    JOIN read_csv_auto('D_ICD_DIAGNOSES.csv') AS dict
      ON diags.ICD9_CODE = dict.ICD9_CODE
    GROUP BY diags.ICD9_CODE, dict.LONG_TITLE
    HAVING COUNT(DISTINCT diags.SUBJECT_ID) BETWEEN 100 AND 120
    ORDER BY patient_count DESC
) TO 'results.csv' (HEADER, DELIMITER ',');
```



