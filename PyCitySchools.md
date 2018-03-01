
# PyCity Schools Analysis
- Schools with less spending budgets per student have higher math, reading, and overall scores.
- Schools with a smaller student population have higher math, reading, and overall scores.
- Charter schools have higher math, reading, and overall scores than those of district schools.


```python
import pandas as pd 
import numpy as np
```


```python
#Open and read files for school and students csv
school_csv = "schools_complete.csv"
students_csv = "students_complete.csv"

school_df = pd.read_csv(school_csv)
students_df = pd.read_csv(students_csv)
```

# District Summary


```python
total_schools = len(school_df["name"].unique())
total_students = students_df["name"].count()
total_budget = school_df["budget"].sum()
avg_math = students_df["math_score"].mean()
avg_reading = students_df["reading_score"].mean()

#calculate percent of students that passed based on failing score of 65%

pass_math = students_df.loc[students_df["math_score"] > 65, ["math_score"]].count()
percent_pass_math = round((pass_math/total_students) * 100, 0)

pass_reading = students_df.loc[students_df["reading_score"] > 65, ["reading_score"]].count()
percent_pass_reading = round((pass_reading/total_students) * 100, 0)

#% overall pass = students who scored >65 on math and reading
overall_pass = students_df.loc[(students_df["math_score"] > 65)  & (students_df["reading_score"] > 65), "name"].count()
percent_overall_pass = round((overall_pass/total_students) * 100, 0)
```


```python
#creating summary table
district_summary = pd.DataFrame({"Total Schools": [total_schools],
                              "Total Students": [total_students],
                              "Total Budget": [total_budget],
                              "Average Math Score": [avg_math],
                              "Average Reading Score": [avg_reading],
                              "% Passing Math": [percent_pass_math.values[0]],
                              "% Passing Reading": [percent_pass_reading.values[0]],
                              "% Overall Passing Rate": [percent_overall_pass]})

#reorganize the format
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)
district_summary["Total Students"] = district_summary["Total Students"].map("{:,}".format)
organized_district_summary = district_summary[["Total Schools", "Total Students", "Total Budget", "Average Math Score", 
                                         "Average Reading Score", "% Passing Math", "% Passing Reading", "% Overall Passing Rate"]]
organized_district_summary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428.00</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>83.0</td>
      <td>94.0</td>
      <td>79.0</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
#group student csv by high schools 
school_groups = students_df.groupby(["school"])

school_df = school_df.rename(columns={"name":"High School", "type":"School Type", "size":"Total Students", "budget":"Total School Budget"})
#del school_df["School ID"]

school_df = school_df.set_index("High School")
school_df["Per Student Budget"] = (school_df["Total School Budget"]/school_df["Total Students"])
school_df["Average Math Score"] = school_groups["math_score"].mean()
school_df["Average Reading Score"] = school_groups["reading_score"].mean()

#reformat columns
school_df["Total School Budget"] = school_df["Total School Budget"].map("${:,.2f}".format)
school_df["Per Student Budget"] = school_df["Per Student Budget"].map("${:.2f}".format)

#reset index 
school_df.reset_index(inplace=True)
```


```python
#get the necessary columns and drop all NaN
math_students_df = students_df.loc[:,["school", "math_score"]]
reading_students_df = students_df.loc[:,["school", "reading_score"]]

clean_math_students_df = math_students_df.dropna(how="all")
clean_reading_students_df = reading_students_df.dropna(how="all")

#total number of scores per high school
total_math_groups = clean_math_students_df.groupby(["school"]).count()
total_reading_groups = clean_reading_students_df.groupby(["school"]).count()

#filter to find math scores > 65 and group by school
math_scores = clean_math_students_df.loc[clean_math_students_df["math_score"] > 65]
math_groups = math_scores.groupby(["school"]).count()

#filter to find reading scores > 65 and group by school
reading_scores = clean_reading_students_df.loc[clean_reading_students_df["reading_score"] > 65]
reading_groups = reading_scores.groupby(["school"]).count()

#find percent passing math and percent passing reading
percent_math_groups = round((math_groups/total_math_groups) * 100, 0)
percent_reading_groups = round((reading_groups/total_reading_groups) * 100, 0)

#reset index to merge the two score groups
percent_math_groups.reset_index(inplace=True)
percent_reading_groups.reset_index(inplace=True)

school_scores = percent_math_groups.merge(percent_reading_groups, how = "right")
school_scores.head()

#rename columns
school_scores = school_scores.rename(columns={"school": "High School", "math_score": "% Passing Math", "reading_score": "% Passing Reading"})

#calculate overall passing rate of each high school
school_scores["Overall Passing Rate"] = school_scores.mean(axis=1)
```


```python
#merge "school_df" and "school_scores" tables
schools_summary = school_df.merge(school_scores, how = "right")

#set index to "High School
schools_summary = schools_summary.set_index("High School")
schools_summary.index.name = None
schools_summary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School ID</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>0</td>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>75.0</td>
      <td>92.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>1</td>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>75.0</td>
      <td>92.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>2</td>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>3</td>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>75.0</td>
      <td>91.0</td>
      <td>83.0</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>4</td>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>5</td>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>6</td>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>7</td>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>$628.00</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>76.0</td>
      <td>92.0</td>
      <td>84.0</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>8</td>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>9</td>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>10</td>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>11</td>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>76.0</td>
      <td>92.0</td>
      <td>84.0</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>12</td>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>75.0</td>
      <td>92.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>13</td>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>76.0</td>
      <td>91.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>14</td>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing Schools (By Passing Rate)


```python
top_schools = schools_summary.sort_values("Overall Passing Rate", ascending=False)
top_schools.index.name = None
top_schools.head(5)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School ID</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Shelton High School</th>
      <td>2</td>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>4</td>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>5</td>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>6</td>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>8</td>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>



# Bottom Performing Schools (By Passing Rate)


```python
bottom_schools = schools_summary.sort_values("Overall Passing Rate")
bottom_schools.index.name = None
bottom_schools.head(5)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School ID</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Hernandez High School</th>
      <td>3</td>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>75.0</td>
      <td>91.0</td>
      <td>83.0</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>0</td>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>75.0</td>
      <td>92.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>1</td>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>75.0</td>
      <td>92.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>12</td>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>75.0</td>
      <td>92.0</td>
      <td>83.5</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>13</td>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>76.0</td>
      <td>91.0</td>
      <td>83.5</td>
    </tr>
  </tbody>
</table>
</div>



# Math Scores by Grade


```python
student_math = students_df.loc[:,["school", "grade", "math_score"]]

#group by both school and grades to find sum of math scores
grades_by_school_math = student_math.groupby(["school", "grade"])

sum_by_grade_math = grades_by_school_math.aggregate(np.sum)
sum_by_grade_math = sum_by_grade_math.reset_index()
#print(sum_by_grade_math.head())

student_count_by_grade_math = grades_by_school_math.count()
student_count_by_grade_math = student_count_by_grade_math.reset_index()
#print(student_count_by_grade_math.head())

#average math score per grade
avg_grade_math = (sum_by_grade_math["math_score"] / student_count_by_grade_math["math_score"])

#create pivot table
math_scores_grade = sum_by_grade_math[["school", "grade"]].copy()
math_scores_grade["Average Math Score"] = avg_grade_math

math_by_grade = math_scores_grade.pivot_table("Average Math Score", ["school"], "grade")
math_by_grade = math_by_grade[["9th", "10th", "11th", "12th"]]
math_by_grade.index.name = None
math_by_grade.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>grade</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Score by Grade


```python
student_reading = students_df.loc[:,["school", "grade", "reading_score"]]

#group by both school and grades to find sum of math scores
grades_by_school_reading = student_reading.groupby(["school", "grade"])

sum_by_grade_reading = grades_by_school_reading.aggregate(np.sum)
sum_by_grade_reading = sum_by_grade_reading.reset_index()
#print(sum_by_grade_reading.head())

student_count_by_grade_reading = grades_by_school_reading.count()
student_count_by_grade_reading = student_count_by_grade_reading.reset_index()
#print(student_count_by_grade_reading.head())

#average math score per grade
avg_grade_reading = (sum_by_grade_reading["reading_score"] / student_count_by_grade_reading["reading_score"])

#create pivot table
reading_scores_grade = sum_by_grade_reading[["school", "grade"]].copy()
reading_scores_grade["Average Reading Score"] = avg_grade_reading

reading_by_grade = reading_scores_grade.pivot_table("Average Reading Score", ["school"], "grade")
reading_by_grade = reading_by_grade[["9th", "10th", "11th", "12th"]]
reading_by_grade.index.name = None
reading_by_grade.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>grade</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending


```python
school_spending = schools_summary.loc[:,["Per Student Budget", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
school_spending.reset_index(inplace=True)
school_spending["Per Student Budget"] = school_spending["Per Student Budget"].str.replace("[\$,]", " ").astype("float")
school_spending = school_spending.rename(columns={"Per Student Budget": "Spending Ranges (Per Student)"})
school_spending
#print(school_spending["Per Student Budget"].max())
#print(school_spending["Per Student Budget"].min())

#create bins 
bins = [0, 585, 615, 645, 675]
group_labels = ["$0-585", "$585-615", "$615-645", "$645-675"]

#cut data and place into bins
pd.cut(school_spending["Spending Ranges (Per Student)"], bins, labels=group_labels)

#create new column for data series
school_spending["Spending Ranges (Per Student)"] = pd.cut(school_spending["Spending Ranges (Per Student)"], bins, labels=group_labels)

spending_group = school_spending.groupby("Spending Ranges (Per Student)")
spending_group.mean()

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>$0-585</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>79.079225</td>
      <td>81.891436</td>
      <td>83.833333</td>
      <td>94.500000</td>
      <td>89.166667</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>75.000000</td>
      <td>91.666667</td>
      <td>83.333333</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Size


```python
school_size = schools_summary.loc[:,["Total Students", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
school_size.reset_index(inplace=True)
school_size = school_size.rename(columns={"Total Students": "School Size"})

#print(school_size["Total Students"].max())
#print(school_size["Total Students"].min())

#create bins 
bins = [0, 1500, 3500, 5000]
group_labels = ["Small (<1500)", "Medium (1500-3500)", "Large (3500-5000)"]

#cut data and place into bins
pd.cut(school_size["School Size"], bins, labels=group_labels)

#create new column for data series
school_size["School Size"] = pd.cut(school_size["School Size"], bins, labels=group_labels)

size_group = school_size.groupby("School Size")
size_group.mean()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1500)</th>
      <td>83.664898</td>
      <td>83.892148</td>
      <td>100.00</td>
      <td>100.000</td>
      <td>100.0000</td>
    </tr>
    <tr>
      <th>Medium (1500-3500)</th>
      <td>80.904987</td>
      <td>82.822740</td>
      <td>90.75</td>
      <td>96.875</td>
      <td>93.8125</td>
    </tr>
    <tr>
      <th>Large (3500-5000)</th>
      <td>77.063340</td>
      <td>80.919864</td>
      <td>75.50</td>
      <td>91.750</td>
      <td>83.6250</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Type


```python
school_type = schools_summary.loc[:,["School Type", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
school_type.reset_index(inplace=True)

#replace string with number value in "School Type" column
school_type.replace(["Charter", "District"], [1, 2], inplace=True)
school_type

#create bins 
bins = [0, 1, 2]
group_labels = ["Charter", "District"]

#cut data and place into bins
pd.cut(school_type["School Type"], bins, labels=group_labels)

#create new column for data series
school_type["School Type"] = pd.cut(school_type["School Type"], bins, labels=group_labels)

type_group = school_type.groupby("School Type")
type_group.mean()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>75.428571</td>
      <td>91.714286</td>
      <td>83.571429</td>
    </tr>
  </tbody>
</table>
</div>


