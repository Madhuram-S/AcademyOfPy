
## Analysis of performance of city school based Math and Reading scores

In this analysis, we analyse the performance data for 15 schools (both charter and district) with varied student sizes. Math Score and Reading score are considered to be the key performance factors.

To compare performance, we find out averages of math and reading scores, % passing in math and reading and % overall passing rate. 

Then we find the relationsips between, % overall passing rate and school size, school type and per student budget

### Analysis indicates that

- Charter schools have significantly out-performed district school across both math and reading. 
- School funding or per student budget seems to have an inverse relationshiop with performance. Schools with higer spending have under-performed than schools with lower funding.
- Interestingly, most of the Charter schools per student budget are less than USD 615 where as all district schools have funding greater than USD 615. This means, there are other reason like school policies, teaching methods that could be underlying reasons for better or worse performance. 
- Small (< 1000 students) and mid-size schools (1000-2000 students) have significantly performed better than large size schools( > 2000 students) especially in Math



```python
# Import Dependcies and libraries
import pandas as pd
import numpy as np
import re
import warnings
warnings.filterwarnings('ignore')
```


```python
#Set data file path
school_datFile = "Resources/schools_complete.csv"
student_datFile = "Resources/students_complete.csv"

#read data files to dataframes
school_rawdata = pd.read_csv(school_datFile)
student_rawdata = pd.read_csv(student_datFile)

#merge both dataframes to one combined dataframe
combinedDF = pd.merge(school_rawdata, student_rawdata, on='school_name', how = 'right')

# add two columns for pass for math and reading. this will be used to calculate count
combinedDF['math_pass'] = [ score >= 70 for score in combinedDF['math_score']]
combinedDF['read_pass'] = [ score >= 70 for score in combinedDF['reading_score']]

```


```python
# set formatting standards that can be applied using applymap()
format_int = "{0:,.0f}".format
format_float = "{:,.6f}".format
format_cur = "${:,.2f}".format
```

## District Summary
In this section, we calculate the performance metrics at district level, We display
- Total number of schools
- Total number of students
- Total budget
- Average math score 
- Average reading score
- Overall passing rate 
- Percentage of students with a passing math score (70 or greater)
- Percentage of students with a passing reading score (70 or greater)


```python
# high level snapshot (in table form) of the district's key metrics, including:
#otal Schools,Total Students, Total Budget, Average Math Score, Average Reading Score, 
# % Passing Math, % Passing Reading, % Overall Passing Rate

district_summary = {'Total Schools' : combinedDF['school_name'].nunique(),
'Total Students' : len(combinedDF.index),
'Total Budget' : school_rawdata['budget'].sum(),
'Average Math Score' : combinedDF['math_score'].mean(),
'Average Reading Score' : combinedDF['reading_score'].mean(),
'% Passing Math' : ((sum(combinedDF["math_pass"]) / len(combinedDF.index))*100),
'% Passing Reading' : ((sum(combinedDF["read_pass"]) / len(combinedDF.index))*100)
}
overall_pass_rate = (district_summary['Average Math Score']+district_summary['Average Reading Score'] ) / 2

#create summary DF for disctrict data
district_SDF = pd.DataFrame(district_summary, index = [0])
district_SDF['% Overall Passing Rate'] = overall_pass_rate

# apply row formatting
district_SDF.iloc[:, 0:2] = district_SDF.iloc[:,0:2].applymap(format_int)
district_SDF.loc[:,['Total Budget']] = district_SDF.loc[:,['Total Budget']].applymap(format_cur)
district_SDF.iloc[:,3:7] = district_SDF.iloc[:,3:7].applymap(format_float)

#display summary results
district_SDF

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <td>81.877840</td>
      <td>74.980853</td>
      <td>85.805463</td>
      <td>80.431606</td>
    </tr>
  </tbody>
</table>
</div>



## School Wise Metrics
After summary view of the district, we find out how each schools have performed against each metrics. So we calculate
- Total number of students
- Total budget
- Per Student Budget
- Average math score 
- Average reading score
- Overall passing rate 
- Percentage of students with a passing math score (70 or greater)
- Percentage of students with a passing reading score (70 or greater)


```python
# Create an overview table that summarizes key metrics about each school, including:
# School Name,School Type,Total Students,Total School Budget,Per Student Budget,Average Math Score,Average Reading Score,
# % Passing Math,% Passing Reading,Overall Passing Rate (Average of the above two)


school_grpDF = combinedDF.groupby(['School ID'])

school_summary = { 'School Name' : school_grpDF['school_name'].max(),
                  'School Type' : school_grpDF['type'].max(),
                  'Total Students' : school_grpDF['student_name'].count(),
                  'Total School Budget' : school_grpDF['budget'].mean(),
                  'Per Student Budget' : school_grpDF['budget'].mean() / school_grpDF['student_name'].count(),
                  'Average Math Score' : school_grpDF['math_score'].mean(),
                  'Average Reading Score' : school_grpDF['reading_score'].mean(),
                  '% Passing Math' : (school_grpDF['math_pass'].sum() / school_grpDF['student_name'].count())*100,
                  '% Passing Reading' : (school_grpDF['read_pass'].sum() / school_grpDF['student_name'].count())*100
}

overall_pass_rate = (school_summary['% Passing Math'] + school_summary['% Passing Reading'])/2

school_SDF = pd.DataFrame(school_summary)
school_SDF['% Overall Passing Rate'] = overall_pass_rate
#school_SDF.to_csv('Resources/schoolsummary.csv', index = False)
```


```python
#Sort School summary dataframe by %Overall Passing Rate in descending order

school_SDF_sorted = school_SDF.sort_values('% Overall Passing Rate', ascending = False)
school_SDF_sorted.set_index('School Name',inplace = True)
```

### Display the Top 5 schools based on % Overall passing Rate


```python
# Display the 5 top performing schools
school_top5 = school_SDF_sorted.head(5)

# Apply Formatting
school_top5.loc[:,'Total Students'] = school_top5.loc[:,'Total Students'].map(format_int)

school_top5.loc[:,['Total School Budget','Per Student Budget']] \
                        = school_top5.loc[:,['Total School Budget','Per Student Budget']].applymap(format_cur)

school_top5.iloc[:,5:9] = school_top5.iloc[:,5:9].applymap(format_float) 

school_top5

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>95.586652</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>95.290520</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>95.270270</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>95.265668</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>95.203679</td>
    </tr>
  </tbody>
</table>
</div>



### Display the Bottom 5 schools based on % Overall passing Rate


```python
# Display the 5 poorly performing schools
school_worst5 = school_SDF_sorted.tail(5).sort_values('% Overall Passing Rate')

# Apply Formatting
school_worst5.loc[:,'Total Students'] = school_worst5.loc[:,'Total Students'].map(format_int)

school_worst5.loc[:,['Total School Budget','Per Student Budget']] \
                        = school_worst5.loc[:,['Total School Budget','Per Student Budget']].applymap(format_cur)

school_worst5.iloc[:,5:9] = school_worst5.iloc[:,5:9].applymap(format_float) 

school_worst5
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>73.293323</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>73.363852</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>73.500171</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>73.639992</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>73.804308</td>
    </tr>
  </tbody>
</table>
</div>



## School-wise and grade-wise performance
 Here we analyse how each school has performed across grades (9th to 12th) for Math and Reading
 
 Average Math and Reading score is calculated and displayed for each school across 9th to 12th grades


```python

# get the order in which columns need to appear while displaying school-wise grade information
avbl_grades = list(combinedDF['grade'].unique())

avbl_grades = ",".join(avbl_grades)
avbl_grades = sorted([int(s) for s in re.findall(r'\d+', avbl_grades)])

avbl_grades = [f'{i}th' for i in avbl_grades]
```

### Peformance in Math across grades


```python
# Summarize the average math score by grade

# group  by schools and grades
grade_grpDF = combinedDF.groupby(['school_name','grade'])

# extract avg of math_scores by school and grade
grade_math_SDF = pd.DataFrame(grade_grpDF['math_score'].mean())

# undo group by
grade_math_SDF.reset_index(inplace = True)

# use pivot() to display as matrix of information
grade_math_SDF = grade_math_SDF.pivot('school_name','grade','math_score')

grade_math_SDF = grade_math_SDF.reindex(columns = avbl_grades)

grade_math_SDF
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
    <tr>
      <th>school_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



### Performance in Reading across Grades


```python
# Summarize the average reading score by grade

# use pivot_table directly summarize

#before using pivot_table(), convert reading_score to numeric value

combinedDF['reading_score'] = pd.to_numeric(combinedDF['reading_score'])

grade_read_SDF = combinedDF.pivot_table(index = 'school_name', columns = 'grade', values = 'reading_score', aggfunc = np.mean)

#change the column arrangement
grade_read_SDF = grade_read_SDF.reindex(columns = avbl_grades)

grade_read_SDF
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
    <tr>
      <th>school_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



## Relationship between Per Student Budget and School Performance

Here, we answer the question whether higher budget means higher performance?

The per student budget is grouped into 4 groups of <\$585, \$585-615, \$615-645, \$645-675

Average Math and Reading score, % Passing Rate in Math and reading and % overall passing rate is analysed for each group


```python
#Break down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. 
spending_bins = [0, 585, 615, 645, 675]
spendBins_names = ["<$585", "$585-615", "$615-645", "$645-675"]

# divide the dataframe by spending bins and add a column to specify the bin the row belongs 
school_SDF["Spending Ranges (Per Student)"] = pd.cut(school_SDF['Per Student Budget'], spending_bins, labels = spendBins_names)

# select only required columns and group by the spending bins
spendingGrp_SDF = school_SDF[["Spending Ranges (Per Student)",'Average Math Score','Average Reading Score','% Passing Math','% Passing Reading' \
                             ,'% Overall Passing Rate']].groupby(["Spending Ranges (Per Student)"])

#aggregate the columns by average
spendingGrp_SDF.agg(np.mean)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>% Overall Passing Rate</th>
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
      <th>&lt;$585</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>93.460096</td>
      <td>96.610877</td>
      <td>95.035486</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>94.230858</td>
      <td>95.900287</td>
      <td>95.065572</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>79.079225</td>
      <td>81.891436</td>
      <td>75.668212</td>
      <td>86.106569</td>
      <td>80.887391</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>66.164813</td>
      <td>81.133951</td>
      <td>73.649382</td>
    </tr>
  </tbody>
</table>
</div>



## Impact of School Size on School Performance

Here, we look at whether size(# of students) has an impact on performance.

The school size is grouped into 3 groups of Small (<1000), Medium (1000-2000), Large (2000-5000)

Average Math and Reading score, % Passing Rate in Math and reading and % overall passing rate is analysed for each group


```python
# Calculate the school performance by school size (no. of students)
size_bins = [0, 1000, 2000, 5000]
sizeBins_names = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

# divide the dataframe by spending bins and add a column to specify the bin the row belongs 
school_SDF['School Size'] = pd.cut(school_SDF['Total Students'], size_bins, labels = sizeBins_names)

# select only required columns and group by the spending bins
sizeGrp_SDF = school_SDF[['School Size','Average Math Score','Average Reading Score','% Passing Math','% Passing Reading' \
                             ,'% Overall Passing Rate']].groupby('School Size')

#aggregate the columns by average
sizeGrp_SDF.agg(np.mean)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>% Overall Passing Rate</th>
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
      <th>Small (&lt;1000)</th>
      <td>83.821598</td>
      <td>83.929843</td>
      <td>93.550225</td>
      <td>96.099437</td>
      <td>94.824831</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.374684</td>
      <td>83.864438</td>
      <td>93.599695</td>
      <td>96.790680</td>
      <td>95.195187</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>77.746417</td>
      <td>81.344493</td>
      <td>69.963361</td>
      <td>82.766634</td>
      <td>76.364998</td>
    </tr>
  </tbody>
</table>
</div>



## Does School Type Influence School Performance ?
Average Math and Reading score, % Passing Rate in Math and reading and % overall passing rate is analysed for Charter and District schools. 


```python
# Summarize by School Type
# select only required columns and group by the spending bins
schoolType_SDF = school_SDF[['School Type','Average Math Score','Average Reading Score','% Passing Math','% Passing Reading' \
                             ,'% Overall Passing Rate']].groupby('School Type')

#aggregate the columns by average
schoolType_SDF.agg(np.mean)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>% Overall Passing Rate</th>
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
      <td>93.620830</td>
      <td>96.586489</td>
      <td>95.103660</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>66.548453</td>
      <td>80.799062</td>
      <td>73.673757</td>
    </tr>
  </tbody>
</table>
</div>


