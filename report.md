# **BigMart Sales Data Analysis**

## **Overview**
This project focuses on exploratory data analysis (EDA) on the BigMart Sales dataset. The goal is to uncover insights about sales trends, identify factors influencing sales, and gain a better understanding of product and outlet performance.

---

## **Dataset Description**
The dataset contains details about items sold at various outlets, their sales figures, and related attributes. Key columns include:
- `Item_Identifier`: Unique product ID
- `Item_Weight`: Weight of the product
- `Item_Fat_Content`: Indicates whether the product is low fat or regular
- `Item_Visibility`: The visibility of the product in the store
- `Item_Type`: The category to which the product belongs
- `Item_MRP`: Maximum Retail Price (MRP) of the product
- `Outlet_Identifier`: Unique outlet ID
- `Outlet_Size`: Size of the outlet (Small, Medium, High)
- `Outlet_Location_Type`: Tier of the city where the outlet is located
- `Outlet_Type`: Type of outlet (Grocery Store, Supermarket Type1, Type2, Type3)
- `Item_Outlet_Sales`: Total sales for the product in the outlet

---

## **Data Cleaning**
### **1. Handling Missing Values**
- **`Item_Weight`**: 
  - Missing values were first filled using the mean weight for each `Item_Identifier`.
  - Remaining missing values were filled with the overall mean.
- **`Outlet_Size`**: 
  - Missing values were filled using the most frequent size (`mode`) for each `Outlet_Type`.

### **Code**:
```python
# Filling missing values in Item_Weight
df['Item_Weight'] = df.groupby('Item_Identifier')['Item_Weight'].transform(
    lambda x: x.fillna(x.mean())
)
df['Item_Weight'].fillna(df['Item_Weight'].mean(), inplace=True)

# Filling missing values in Outlet_Size
outlet_size_mode = df.groupby('Outlet_Type')['Outlet_Size'].agg(pd.Series.mode)

def fill_outlet_size(row):
    if pd.isnull(row['Outlet_Size']):
        return outlet_size_mode.get(row['Outlet_Type'], row['Outlet_Size'])
    return row['Outlet_Size']

df['Outlet_Size'] = df.apply(fill_outlet_size, axis=1)
```

### **2. Standardizing `Item_Fat_Content`**
The `Item_Fat_Content` column contained values like `'Low Fat'`, `'Regular'`, `'low fat'`, `'LF'`, and `'reg'`.
I standardized these values to two categories: **'Low Fat'** and **'Regular'**.

### **Code**:
```python
# Standardizing Item_Fat_Content
df['Item_Fat_Content'] = df['Item_Fat_Content'].replace({
    'low fat': 'Low Fat',
    'LF': 'Low Fat',
    'reg': 'Regular'
})
```
---
### 3. Feature Engineering

Feature engineering was performed to enhance the dataset's value and provide additional insights:

### Outlet Operating Years
- A new column, `Outlet_Operating_Years`, was created by subtracting the `Outlet_Establishment_Year` from the current year (assumed to be **2023** for this analysis).
- The `Outlet_Establishment_Year` column was subsequently dropped as it became redundant.
```python
  #Creating a new column for Outlet operating years
current_year= datetime.datetime.now().year

df['Outlet_Operating_Years']= current_year-df['Outlet_Establishment_Year']

#Droping Outlet Establishment Year
df.drop(columns=['Outlet_Establishment_Year'], inplace=True)
```


##  Exploratory Data Analysis (EDA)

#### **Distribution of Item Outlet Sales**
The `Item Outlet Sales` feature has a **right-skewed distribution**, indicating that most sales figures are low.

#### **Segregating Item Outlet Sales**
To further analyze sales, I segregated `Item_Outlet_Sales` into low, medium, and high sales.

```python
# Segregating Item_Outlet_Sales
bins = [0, 1500, 3500, df['Item_Outlet_Sales'].max()]
labels = ['Low', 'Medium', 'High']
df['Sales_Category'] = pd.cut(df['Item_Outlet_Sales'], bins=bins, labels=labels)
```
## **Barplot of Item Type in Low Sales Data**

After segregating low sales data, a barplot of `Item_Type` revealed that Fruits and Vegetables have the highest sales in low sales data, while Seafood has the lowest.

```python
#Low sales items categorized by item type
low_sales_item_type = low_sales_data['Item_Type'].value_counts()
sns.barplot(x=low_sales_item_type.index, y=low_sales_item_type.values)
plt.xticks(rotation=90)
plt.title('Low Sales by Item Type')
plt.show()
```
### **Barplot of Item Type on Whole Sales Data**
Upon analyzing the entire sales data, the barplot of `Item_Type` showed a similar graph to the low sales data, reinforcing the consistency of sales trends across data subsets.

```python
# Aggregate total sales by item type
item_sales = df.groupby('Item_Type')['Item_Outlet_Sales'].sum().sort_values(ascending=False)

# Create a barplot
plt.figure(figsize=(12, 6))
sns.barplot(x=item_sales.index, y=item_sales.values, palette='viridis')

# Customize the plot
plt.title('Total Sales by Item Type', fontsize=16)
plt.xlabel('Item Type', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()

# Show the plot
plt.show()
```
## **Boxplot of MRP Distribution for Item Type on Whole Sales Data**
The boxplot for different `Item_Type` shows that most item types have similar box plots in terms of price range and variability.

### **Code**:
```python
# Boxplot of MRP distribution for Item Type
plt.figure(figsize=(14, 8))
sns.boxplot(x='Item_Type', y='Item_MRP', data=df, palette='Set3')

# Customize the plot
plt.title('Distribution of MRP by Item Type', fontsize=16)
plt.xlabel('Item Type', fontsize=14)
plt.ylabel('Item MRP', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()

plt.show()
```
### **Boxplot of MRP Distribution in Low Sales Data**
The MRP (Maximum Retail Price) distribution for low sales shows:

- Q1 and Q3 are near 50 and 150.
- The median is around 100.
- Whiskers extend to 200.

### **Code**:
```python
# Boxplot of MRP distribution for Low Sales Data
sns.boxplot(data=low_sales_data, x='Sales_Category', y='Item_MRP')
plt.title('MRP Distribution in Low Sales')
plt.show()
```
### **Scatter Plot: Item Visibility vs Item Outlet Sales**
The scatter plot shows no clear pattern, and correlation analysis revealed a negative correlation between `Item_Visibility` and `Item_Outlet_Sales`.

### **Code**:
```python
#How item visibility affects low sales
sns.scatterplot(data=low_sales_data, x='Item_Visibility', y='Item_Outlet_Sales')
plt.title('Item Visibility vs Sales in Low Sales Category')
plt.show()
```
### Barplot of Outlet Type in Low Sales Data

### Code:
```python
# Grouping sales by Outlet Type for low sales data
low_sales_sales_by_outlet_type = low_sales_data.groupby('Outlet_Type')['Item_Outlet_Sales'].sum().sort_values(ascending=False)

# Creating the barplot
plt.figure(figsize=(12, 6))
sns.barplot(x=low_sales_sales_by_outlet_type.index, y=low_sales_sales_by_outlet_type.values, palette='viridis')

# Customizing the plot
plt.title('Low Sales by Outlet Type', fontsize=16)
plt.xlabel('Outlet Type', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()
plt.show()
```
## Insights:
- Supermarket Type 1 has the highest sales in low sales data.
- Grocery Stores and Supermarket Type 2 have similar sales figures, but lower compared to Supermarket Type 1.
- Supermarket Type 3 consistently shows the lowest sales among all outlet types.

## Interpretation:
The barplot of outlet types in low sales data demonstrates that Supermarket Type 1 is the dominant outlet type with significantly higher sales, while Supermarket Type 3 performs poorly.

### Barplot of Outlet Type in Total Sales Data

### Code:
```python
# Group by Outlet Type and sum the total sales
total_sales_by_outlet_type = df.groupby('Outlet_Type')['Item_Outlet_Sales'].sum().sort_values(ascending=False)


# Plotting total sales by Outlet Type
plt.figure(figsize=(8, 6))
sns.barplot(x=total_sales_by_outlet_type.index, y=total_sales_by_outlet_type.values, palette='viridis')

# Customize the plot
plt.title('Total Sales by Outlet Type', fontsize=16)
plt.xlabel('Outlet Type', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()

plt.show()
```
## Insights:
- Supermarket Type 1 leads in total sales, followed closely by Supermarket Type 3.
- Supermarket Type 2 follows in third place, with Grocery Stores performing the lowest in total sales.
- The sales difference between Supermarket Type 1 and other outlet types is significant

## Interpretation:
In total sales data, Supermarket Type 1 dominates overall sales, followed by Supermarket Type 3, Supermarket Type 2, and Grocery Stores at the lowest sales.

### Barplot of Outlet Size in Low Sales Data
### Code:
```python
# Group by Outlet Size and sum sales in low sales data
low_sales_sales_by_outlet_size = low_sales_data.groupby('Outlet_Size')['Item_Outlet_Sales'].sum().sort_values(ascending=False) 

plt.figure(figsize=(8, 6))
sns.barplot(x=low_sales_sales_by_outlet_size.index, y=low_sales_sales_by_outlet_size.values, palette='viridis')

# Customize the plot
plt.title('Low Sales by Outlet Size', fontsize=16)
plt.xlabel('Outlet Size', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=40, ha='right', fontsize=12)
plt.tight_layout()

plt.show()
```
## Insights:
- In low sales data, small outlets contribute the highest sales.
- Medium-sized outlets follow, with large outlets performing the least.

## Interpretation:
The barplot of outlet size in low sales data indicates that small outlets perform the best, followed by medium outlets, while large outlets consistently contribute the lowest sales in this segment.


### Barplot of Outlet Size in Total Sales Data
### Code:
```python
# Group by Outlet Size and sum the total sales
total_sales_by_outlet_size = df.groupby('Outlet_Size')['Item_Outlet_Sales'].sum().sort_values(ascending=False)

# Plotting total sales by Outlet Size
plt.figure(figsize=(8, 6))
sns.barplot(x=total_sales_by_outlet_size.index, y=total_sales_by_outlet_size.values, palette='viridis')

# Customize the plot
plt.title('Total Sales by Outlet Size', fontsize=16)
plt.xlabel('Outlet Size', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()

plt.show()

```
## Insights:
- In total sales data, small outlets are the highest contributors, followed by medium-sized outlets.
- Large outlets have the lowest sales figures in total sales.
- Both low sales data and total sales data show that small outlets perform the best, followed by medium, and large outlets contribute the least.

## Interpretation:
The barplot of outlet size in total sales data demonstrates that small outlets are the highest contributors to sales, similar to low sales data. Medium-sized outlets follow, with large outlets consistently performing poorly in sales.

### Barplot of Outlet Location Type in Low Sales Data
### Code:
```python
# Group by Outlet Location Type and sum sales in low sales data
low_sales_sales_by_location_type = low_sales_data.groupby('Outlet_Location_Type')['Item_Outlet_Sales'].sum().sort_values(ascending=False)

plt.figure(figsize=(8, 6))
sns.barplot(x=low_sales_sales_by_location_type.index, y=low_sales_sales_by_location_type.values, palette='viridis')

# Customize the plot
plt.title('Low Sales by Outlet Location Type', fontsize=16)
plt.xlabel('Outlet Location Type', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()

plt.show()


```
## Insights:
- In low sales data, there isn’t a big difference in the bars, but Tier 3 locations lead sales slightly more than Tier 2 and Tier 1.

## Interpretation:
The barplot of outlet location types in low sales data shows that Tier 3 has the highest sales, followed closely by Tier 2 and Tier 1, with minimal differences between them.

### Barplot of Outlet Location Type in Total Sales Data
### Code:
```python
# Group by Outlet Location Type and sum the total sales
total_sales_by_location_type = df.groupby('Outlet_Location_Type')['Item_Outlet_Sales'].sum().sort_values(ascending=False)

# Plotting total sales by Outlet Location Type
plt.figure(figsize=(8, 6))
sns.barplot(x=total_sales_by_location_type.index, y=total_sales_by_location_type.values, palette='viridis')

# Customize the plot
plt.title('Total Sales by Outlet Location Type', fontsize=16)
plt.xlabel('Outlet Location Type', fontsize=14)
plt.ylabel('Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.tight_layout()

plt.show()

```
## Insights:
- In total sales data, Tier 3 locations perform slightly better than Tier 1 and Tier 2.
- The differences between Tier 1, Tier 2, and Tier 3 are more noticeable compared to low sales data.
## Interpretation:
The barplot of outlet location types in total sales data indicates that Tier 3 locations dominate overall sales, with Tier 1 and Tier 2 performing similarly but with more pronounced differences than in low sales.

### **Correlation Matrix**
A correlation matrix was created to analyze relationships between `Item_Visibility`, `Item_MRP`, `Item_Weight`, and `Item_Outlet_Sales`:

### **Code**:
```python
print(df[['Item_Visibility', 'Item_MRP', 'Item_Weight', 'Item_Outlet_Sales']].corr())
```

#### **Insights from the Correlation Matrix**
- `Item_Visibility` has a negative correlation with `Item_Outlet_Sales`, indicating that higher visibility may not always result in higher sales.
- `Item_MRP` has a positive correlation with `Item_Outlet_Sales`, showing that higher-priced items tend to have higher sales.
- `Item_Weight` has a weak correlation with sales, suggesting it doesn’t significantly affect sales performance.

---

### **Key Insights**
- **Sales Segmentation**: The majority of sales are low sales, as reflected in the right-skewed distribution of `Item_Outlet_Sales`.
- **Product Demand**: Fruits and Vegetables dominate sales, while Seafood has the lowest sales.
- **MRP Distribution**: Price ranges across different item types are fairly similar, with boxplots revealing consistent pricing patterns.
- **Visibility Impact**: `Item_Visibility` shows a negative correlation with sales, implying that visibility alone doesn’t drive sales.
- **Outlet Performance**: Performance varies by outlet type, size, and location:
  - Supermarket 3 shows the highest sales, while smaller outlets and certain locations like Tier 1 and Tier 3 have similar performance.
## **Conclusion**

The analysis of the BigMart sales dataset provides valuable insights into the factors influencing sales performance. Below are the key conclusions derived from the project:

### **Sales Segmentation**
- The majority of sales are categorized as low sales, as reflected in the right-skewed distribution of `Item_Outlet_Sales`. 
- Understanding low sales is crucial for identifying opportunities for growth and addressing underperforming segments.

### **Product Performance**
- **Fruits and Vegetables** dominate both low sales data and all sales data, indicating their broad appeal and higher product availability. 
- Seafood consistently ranks the lowest in sales across all datasets, suggesting limited demand or availability.

### **MRP Distribution**
- The `Item_MRP` distribution reveals consistent pricing patterns across item types.
- Boxplots for low sales data indicate that the first and third quartiles (Q1 and Q3) of MRP lie near 50 and 150, with a median around 100 and whiskers extending to 200.

### **Impact of Item Visibility**
- The scatter plot of `Item_Visibility` and `Item_Outlet_Sales` in the low sales data shows no clear pattern, and correlation analysis reveals a weak negative relationship.
- This suggests that higher visibility alone does not drive higher sales and other factors, like pricing and promotions, may play a larger role.

### Outlet Type:
- **Supermarket Type 3** outperforms others in **low** and **total sales**, while **Grocery Stores** consistently show poor performance.
- **Supermarket Type 1** has the highest sales in **low sales data**, followed by **Grocery Stores** and **Supermarket Type 2**, with **Supermarket Type 3** consistently showing the lowest sales.
- In **total sales data**, **Supermarket Type 1** dominates overall sales, followed by **Supermarket Type 3**, **Supermarket Type 2**, and **Grocery Stores** performing the lowest.

### Outlet Size:
- **Small outlets** perform the best in **low sales** data, followed by **medium-sized outlets**, while **high-sized outlets** consistently contribute the lowest sales.
- In **total sales data**, it similar as **low sales** data .

### Outlet Location:
- In **low sales data**, **Tier 3** locations lead slightly more in sales compared to **Tier 2** and **Tier 1** locations, with minimal differences between them.
- In **total sales data**, **Tier 3** locations perform slightly better than **Tier 1** and **Tier 2**, with more noticeable differences compared to low sales data.

### **Correlation Analysis**
- `Item_MRP` has a strong positive correlation with `Item_Outlet_Sales`, indicating that higher-priced items contribute significantly to revenue.
- `Item_Weight` shows a very weak correlation with sales, suggesting it has minimal impact.
