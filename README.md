# Python-Global-Superstore-RFM-Analysis
## I. Introduction
This project analyzes the RFM (Recency, Frequency, Monetary) metrics to segment customers for the marketing department of Superstore. The goal is to provide the necessary insights to help the marketing team launch tailored marketing campaigns for each customer group, thereby boosting sales during the upcoming Christmas and New Year season.
## II. Dataset
This is a transnational data set which contains all the transactions occurring between 01/12/2010 and 09/12/2011 for a UK-based and registered non-store online retail. The company mainly sells unique all-occasion gifts. Many customers of the company are wholesalers.

**Dataset Dictionary**
- **InvoiceNo**: Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'C', it indicates a cancellation.
- **StockCode**: Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product.
- **Description**: Product (item) name. Nominal.
- **Quantity**: The quantities of each product (item) per transaction. Numeric.
- **InvoiceDate**: Invoice Date and time. Numeric, the day and time when each transaction was generated.
- **UnitPrice**: Unit price. Numeric, Product price per unit in sterling.
- **CustomerID**: Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer.
- **Country**: Country name. Nominal, the name of the country where each customer resides.
### III. Explore Data

#### 1. Cleaning Data
Data cleaning involves removing missing data, replacing missing values, and adjusting data types to ensure consistency. The steps include removing missing data when the proportion of missing values is low across the dataset, imputing missing values using metrics like mode, median, etc., eliminating duplicates, and correcting data types to maintain data integrity. Additionally, identifying and handling outlier values is crucial to ensuring the accuracy and consistency of the data.

**Actions**
```
df_raw = pd.DataFrame(df)
print(df_raw.info())
print(df_raw.describe())
# drop missing value in "CustomerID" by delete rows
df_raw= df_raw.dropna(subset=["CustomerID"])
# drop missing value in "Description" by delete columns
df_raw=df_raw.drop('Description',axis=1)
# Change datatype of UnitPrice
df_raw["UnitPrice"]=df_raw["UnitPrice"].astype(int)
# Change datatype of CustomerID
df_raw["CustomerID"]=df_raw["CustomerID"].astype(str)
# Handle outlier of Quantity and UnitPrice
df_raw=df_raw[(df_raw['Quantity'] > 0)]
df_raw=df_raw[(df_raw['UnitPrice'] > 0)]
print(df_raw.info())
print(df_raw.describe())
print(df_raw.head())
```

**RFM Calculation and Segmentation**

After cleaning the data, the **Recency**, **Frequency**, and **Monetary** values were calculated separately for each customer. **Recency** was determined based on the number of days since the last purchase, **Frequency** measured the total number of transactions, and **Monetary** represented the total spending of each customer. Once these individual metrics were calculated, they were combined to form the complete RFM score for each customer.
```
#Create RFM Table
RFM_data=df_trans.groupby('CustomerID').agg({
    'InvoiceDate': lambda x: (present_date - x.max()).days,
    'InvoiceNo' : 'count',
    'purchase_price':'sum'})
RFM_data.rename(columns={'InvoiceDate': 'Recency', 'InvoiceNo': 'Frequency','purchase_price':'Monetary'}, inplace=True)
#Calculate RFMCore
RFM_data["Rscore"]=pd.qcut(RFM_data['Recency'],q=5,labels=[5,4,3,2,1])
RFM_data["Fscore"]=pd.qcut(RFM_data['Frequency'],q=5,labels=[1,2,3,4,5])
RFM_data["Mscore"]=pd.qcut(RFM_data['Monetary'],q=5,labels=[1,2,3,4,5])
RFM_data['RFMscore']=RFM_data["Rscore"].astype(str)+RFM_data["Fscore"].astype(str)+RFM_data["Mscore"].astype(str)
RFM_data = RFM_data.reset_index()
print(RFM_data.head())
```

**Segmentation**

Next, process the Segment file that has classified each group according to the RFM index to apply it to each customer.

```jsx
df1=pd.read_excel('Segment.xlsx')
df_seg = pd.DataFrame(df1)
df_seg.rename(columns={'RFM Score': 'RFMscore'}, inplace=True)
print(df_seg)
```

```
# Convert 'RFMscore' to string type
df_seg['RFMscore'] = df_seg['RFMscore'].astype(str)

# Split the string by commas to create a list
df_seg['RFMscore'] = df_seg['RFMscore'].str.split(',')

# Strip leading and trailing spaces from each item in the list
df_seg['RFMscore'] = df_seg['RFMscore'].apply(lambda x: [i.strip() for i in x])

# Explode the list so each item in the list becomes a separate row
df_seg = df_seg.explode('RFMscore').reset_index(drop=True)

# Print the DataFrame
print(df_seg)
```

<img width="198" alt="seg1" src="https://github.com/user-attachments/assets/17506913-6cc9-4e05-8b7a-4ac8ac022e32" />


```
RFM_table=pd.merge_ordered(RFM_data,df_seg,on='RFMscore',how='left')
print(RFM_table)
```
<img width="482" alt="seg2" src="https://github.com/user-attachments/assets/5c6b2685-188a-47c3-8b9a-d0bb56141c6b" />

**Data Visualization and Insight**

***RFM Distribution Analysis***

 - Distribution of Recency
```
  # Distribution of Recency
fig,ax = plt.subplots(figsize=(12,3))
sns.histplot(RFM_table['Recency'])
ax.set_title('Distribution of Recency')
for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```

<img width="766" alt="R" src="https://github.com/user-attachments/assets/5bdae8c7-a4d1-4f69-8972-c67cd113c16a" />

**Key observations**: A large proportion of customers made repeat purchases within 100 days, accounting for over 2,600 customers. Notably, the highest number of customers returned for transactions within 20 days, indicating a strong customer retention rate. This is a promising sign for the sustainable growth of the business.

- Distribution of Frequency
```
#Distribution of Frequency
binsF = [0, 2, 5, 20, np.inf]
labelsF = ['1-2', '2-5', '5-20', '20+']
RFM_table['FrequencyGroup'] = pd.cut(RFM_table['Frequency'], bins=binsF, labels=labelsF)
fig, ax = plt.subplots(figsize=(8, 3))
sns.countplot(x=RFM_table['FrequencyGroup'], ax=ax)
ax.set_title('Distribution of Frequency')
for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```
<img width="597" alt="F" src="https://github.com/user-attachments/assets/ceedaea7-da95-4a00-894c-8b95b41cf44d" />

**Key observations**: The number of customers making more than 20 transactions accounts for two-thirds of the total customer base, which is a very positive sign. This indicates that the company has built a large base of loyal customers, helping to maintain stable revenue. Additionally, it also reflects the quality of the company’s products and services, as they effectively meet customer needs and expectations. The repeated purchases by customers are a clear sign of satisfaction and trust in the brand, providing a strong foundation for the company’s sustainable growth.

- Distribution of Monetary
```
#Distribution of Monetary
binsF = [0,1000,5000,10000, np.inf]
labelsF = ['0-1k', '1k-5k', '5k-10k', '10k+']
RFM_table['MonetaryGroup'] = pd.cut(RFM_table['Monetary'], bins=binsF, labels=labelsF)
fig, ax = plt.subplots(figsize=(8, 3))
sns.countplot(x=RFM_table['MonetaryGroup'], ax=ax)
ax.set_title('Distribution of Monetary')
for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```

<img width="535" alt="M" src="https://github.com/user-attachments/assets/aaa87991-fa72-4fa2-acb1-802cde79da0a" />

**Key observations**: The majority of the company's customers belong to the low-spending group, with 3,087 customers spending under 1k, accounting for two-thirds of the total customer base. To increase revenue, the company needs to implement strategies to nurture and tap into the potential of customers with higher spending levels.

***RFM Segments of Customer Count***

```
import squarify
import numpy as np
colors=['#FF0000',"#00FFFF","#FFFF00","#A52A2A","#800080","#00FF00","#808000","#FFC0CB","#FFA500","#FF00FF","#736F6E"]
fig, ax = plt.subplots(1,figsize=(15,8))
squarify.plot(sizes=grouped['Cust_Count_seg'],
              label=grouped['Segment'],
              value=[f'{x*100:.2f}%' for x in grouped['count_share']],
              alpha=.8,
              color=colors,
              bar_kwargs=dict(linewidth=1.5,edgecolor="white"))
plt.title('RFM Segments of Customer Count',fontsize=16)
plt.axis('off')
plt.show()
```
<img width="616" alt="ALL" src="https://github.com/user-attachments/assets/65707da1-8fab-471b-84fe-a783124cfd30" />

**Key observations**:
The Champion and Hibernating customer segments account for the highest proportion in the company.

**1. Champion Customers (18.01%)**:

**Insights**:

The company has a notable group of loyal customers, contributing to a stable and sustainable revenue stream—a strong indicator of business performance.

**Risks**:
Over-reliance on this limited customer group poses significant risks. Any decline in this segment, due to market changes, competition, or other factors, could severely impact the company's revenue.

**2. Hibernating Customers (18.63%)**:

**Insights**:
Hibernating customers make up the largest segment, suggesting that the company has lost touch with a substantial portion of previous buyers. This is a warning sign regarding the company’s ability to maintain long-term customer relationships.

**Opportunities**:
If the company can reconnect with this group, it has the potential to boost revenue and maximize long-term customer value.

**Challenges**:
This situation might highlight inefficiencies in the company’s strategies for customer retention and long-term engagement.

## V. Recommendations

**For Champions**: Diversify the customer base to reduce dependency on this group while continuing to nurture loyalty.

**For Hibernating Customers**: Launch targeted reactivation campaigns and evaluate current retention strategies to identify gaps and improve customer engagement efforts.
