# HW3



```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import re
```


```python
math_data = pd.read_csv('./math-achievement-sch-sy2015-16.csv', low_memory=False)
arts_data = pd.read_csv('./rla-achievement-sch-sy2015-16.csv', low_memory=False)
```

## Problem 1


```python
fiscal_data = pd.read_csv('./Sdf16_1a.txt', sep='\t', low_memory=False)
```


```python
condition = (fiscal_data['TFEDREV']>0)
total_fedrev = fiscal_data[condition].groupby('STNAME')['TFEDREV'].sum().sort_values()

print('Ranking of states by total federal funding:\n')
print(total_fedrev, end='\n\n')

plt.figure(figsize=(16,8))
total_fedrev.plot.bar()
plt.xlabel('State Name')
plt.ylabel('Total Federal Funding ($)');
```

    Ranking of states by total federal funding:
    
    STNAME
    Vermont                  111891000
    Wyoming                  123012000
    Delaware                 144707000
    North Dakota             155453000
    New Hampshire            169166000
    Maine                    186523000
    Rhode Island             188204000
    South Dakota             196644000
    Montana                  220340000
    District of Columbia     226202000
    Idaho                    248546000
    Hawaii                   261131000
    Alaska                   307320000
    Nebraska                 346826000
    West Virginia            360283000
    Nevada                   405789000
    Utah                     419642000
    Kansas                   453922000
    Iowa                     464852000
    Connecticut              484186000
    New Mexico               516289000
    Oregon                   582560000
    Arkansas                 606946000
    Minnesota                685055000
    Mississippi              690724000
    Oklahoma                 703225000
    Colorado                 721719000
    Wisconsin                782647000
    Alabama                  803907000
    Massachusetts            804595000
    Maryland                 823599000
    South Carolina           860867000
    Kentucky                 880296000
    Missouri                 959978000
    Indiana                 1015476000
    Virginia                1058146000
    Tennessee               1096182000
    Washington              1098332000
    Louisiana               1115619000
    New Jersey              1249741000
    Arizona                 1302010000
    North Carolina          1587976000
    Michigan                1731034000
    Georgia                 1815242000
    Ohio                    1837963000
    Pennsylvania            2037315000
    Illinois                2334945000
    Florida                 3147329000
    New York                3374794000
    Texas                   6194317000
    California              7709275000
    Name: TFEDREV, dtype: int64
    



![png](output_4_1.png)



```python
## Or 'TOTALEXP'

condition = ((fiscal_data['V33']>0) & (fiscal_data['TFEDREV']>0))
avg_fedrev = (
    fiscal_data
    .loc[condition,['TFEDREV','V33','STNAME']]
    .groupby('STNAME')
    .sum()
)
avg_fedrev['FEDREV_PS'] = avg_fedrev['TFEDREV']/avg_fedrev['V33']

print('Ranking of states by federal funding per student:')
avg_fedrev['FEDREV_PS'].sort_values()
```

    Ranking of states by federal funding per student:





    STNAME
    Vermont                  522.064223
    Utah                     647.982669
    Iowa                     724.263505
    Minnesota                724.605380
    Colorado                 775.893944
    Michigan                 821.254731
    Pennsylvania             824.053831
    Virginia                 824.426779
    Massachusetts            837.010752
    Idaho                    851.553770
    Nevada                   868.237439
    New Jersey               877.228346
    Connecticut              903.842131
    Wisconsin                911.593612
    Kansas                   916.005610
    Washington               930.943158
    Indiana                  931.212627
    Oregon                   933.838920
    Maryland                 936.763816
    New Hampshire            940.572790
    Ohio                    1005.517091
    Oklahoma                1008.192141
    North Carolina          1028.727054
    Maine                   1029.576113
    Georgia                 1031.580392
    Nebraska                1036.603685
    Missouri                1045.362288
    Illinois                1045.771215
    Delaware                1072.281920
    Alabama                 1080.826686
    Tennessee               1096.977472
    Texas                   1115.349585
    South Carolina          1127.616716
    Florida                 1133.383124
    North Dakota            1177.547234
    Arkansas                1181.100022
    Arizona                 1191.109830
    California              1228.554112
    West Virginia           1234.808183
    Kentucky                1282.407785
    New York                1297.282459
    Wyoming                 1301.562781
    Rhode Island            1306.465837
    Mississippi             1420.526689
    Hawaii                  1434.825133
    Montana                 1435.148549
    South Dakota            1466.999888
    New Mexico              1541.345235
    Louisiana               1556.713877
    Alaska                  2319.798908
    District of Columbia    2703.569793
    Name: FEDREV_PS, dtype: float64



## Problem 2


```python
condition = (fiscal_data['TOTALREV']>0) & (fiscal_data['TOTALEXP']>0) & (fiscal_data['V33']>0)
exp_rev = (
    fiscal_data
    .assign(DEBT = fiscal_data['TOTALEXP'] - fiscal_data['TOTALREV'])
    .loc[:,['DEBT','STNAME','V33']]
    .groupby('STNAME')
    .sum()
)

plt.scatter(fiscal_data.loc[condition,'TOTALREV'], fiscal_data.loc[condition,'TOTALEXP'])
plt.xlabel('Total Revenue')
plt.ylabel('Total Expenditure')
plt.show()

exp_rev['DEBT_PS'] = exp_rev['DEBT']/exp_rev['V33']

print('Ranking of states by debt per student:')
exp_rev.query('DEBT_PS>0')['DEBT_PS'].sort_values()
```


![png](output_7_0.png)


    Ranking of states by debt per student:





    STNAME
    Arkansas                  55.510848
    Kentucky                  58.774265
    Tennessee                 94.398330
    Wisconsin                103.665563
    Iowa                     104.204397
    Texas                    110.359253
    Oklahoma                 112.218429
    Illinois                 113.742436
    Kansas                   131.640193
    Oregon                   145.571553
    South Dakota             150.175716
    Virginia                 185.631094
    Washington               265.801263
    Alabama                  346.284944
    New York                 504.916474
    North Carolina           506.761787
    Montana                  543.625086
    Minnesota                746.329846
    Nebraska                 764.693409
    Alaska                   968.643614
    District of Columbia    1285.117970
    North Dakota            1611.360914
    Name: DEBT_PS, dtype: float64



## Problem 3


```python
# If a number if less than or less than or equal to, then we either decrease the number by 2 or make it 0, whichever is maxmimum
# If a number if greater than or greater than or equal to, then we either increase the number by 2 or make it 100, whichever is minimum
# Any data point containing a hyphen will be the average of 2 numbers in the string

def data_processing(df, col_name):
    col = df[col_name]
    new_col = []
    for x in col:
        if x=='PS':
            num = np.nan
        elif bool(re.search('[a-zA-Z]', x)):
            expr, num = x[0:2], int(x[2:])
            if expr in ['LE','LT']:
                num = max(num - 2,0)
            elif expr in ['GE','GT']:
                num = min(num + 2,100)
        elif bool(re.search('-', x)):
            num1, num2 = [int(y) for y in x.split('-')]
            num = round((num1+num2)/2)
        else:
            num = int(x)
        new_col.append(num)
    df[col_name+'_proc'] = new_col
    return df
```


```python
math_data = data_processing(math_data, 'ALL_MTH00PCTPROF_1516')
plt.figure(figsize=(10,6))
plt.hist(math_data['ALL_MTH00PCTPROF_1516_proc'].dropna(), bins=25)
plt.title('Histogram of percent of proficient students in math');
```


![png](output_10_0.png)


## Problem 4


```python
condition = fiscal_data['TFEDREV']>0
cut_amount = int((fiscal_data.loc[condition, 'TFEDREV'].sum())*0.15)
print('15% of the U.S. federal budget currently being spent on funding school districts:', cut_amount)
```

    15% of the U.S. federal budget currently being spent on funding school districts: 8340411300



```python
condition = (fiscal_data['TOTALREV']>0) & (fiscal_data['TOTALEXP']>0)
cut_rev = fiscal_data.assign(DEBT = fiscal_data['TOTALEXP'] - fiscal_data['TOTALREV']).query('DEBT<0')
cut_rev['DEBT'] = -cut_rev['DEBT']
available_amount = cut_rev['DEBT'].sum()
cut_rev['CUT'] = (cut_amount/available_amount)*cut_rev['DEBT']
cut_rev['CUT'] = round(cut_rev['CUT'])
cut_rev[['LEAID','CUT']]
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
      <th>LEAID</th>
      <th>CUT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0100005</td>
      <td>891226.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0100007</td>
      <td>769226.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0100008</td>
      <td>1409951.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0100012</td>
      <td>118161.0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>0100030</td>
      <td>830669.0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>0100060</td>
      <td>789313.0</td>
    </tr>
    <tr>
      <th>60</th>
      <td>0100195</td>
      <td>156563.0</td>
    </tr>
    <tr>
      <th>62</th>
      <td>0100210</td>
      <td>964191.0</td>
    </tr>
    <tr>
      <th>63</th>
      <td>0100240</td>
      <td>1239209.0</td>
    </tr>
    <tr>
      <th>64</th>
      <td>0100270</td>
      <td>11330421.0</td>
    </tr>
    <tr>
      <th>65</th>
      <td>0100300</td>
      <td>13588.0</td>
    </tr>
    <tr>
      <th>67</th>
      <td>0100360</td>
      <td>243411.0</td>
    </tr>
    <tr>
      <th>68</th>
      <td>0100390</td>
      <td>420652.0</td>
    </tr>
    <tr>
      <th>69</th>
      <td>0100420</td>
      <td>493025.0</td>
    </tr>
    <tr>
      <th>71</th>
      <td>0100480</td>
      <td>48150.0</td>
    </tr>
    <tr>
      <th>72</th>
      <td>0100510</td>
      <td>197624.0</td>
    </tr>
    <tr>
      <th>74</th>
      <td>0100600</td>
      <td>36039.0</td>
    </tr>
    <tr>
      <th>76</th>
      <td>0100660</td>
      <td>4726.0</td>
    </tr>
    <tr>
      <th>77</th>
      <td>0100690</td>
      <td>521679.0</td>
    </tr>
    <tr>
      <th>79</th>
      <td>0100750</td>
      <td>225096.0</td>
    </tr>
    <tr>
      <th>82</th>
      <td>0100840</td>
      <td>132340.0</td>
    </tr>
    <tr>
      <th>83</th>
      <td>0100870</td>
      <td>592575.0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>0100900</td>
      <td>36925.0</td>
    </tr>
    <tr>
      <th>85</th>
      <td>0100930</td>
      <td>355368.0</td>
    </tr>
    <tr>
      <th>86</th>
      <td>0100960</td>
      <td>21269.0</td>
    </tr>
    <tr>
      <th>87</th>
      <td>0100990</td>
      <td>271474.0</td>
    </tr>
    <tr>
      <th>88</th>
      <td>0101020</td>
      <td>219188.0</td>
    </tr>
    <tr>
      <th>89</th>
      <td>0101050</td>
      <td>448715.0</td>
    </tr>
    <tr>
      <th>90</th>
      <td>0101080</td>
      <td>102504.0</td>
    </tr>
    <tr>
      <th>91</th>
      <td>0101110</td>
      <td>389339.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>18644</th>
      <td>5517160</td>
      <td>213871.0</td>
    </tr>
    <tr>
      <th>18645</th>
      <td>5517190</td>
      <td>430695.0</td>
    </tr>
    <tr>
      <th>18646</th>
      <td>5517220</td>
      <td>91870.0</td>
    </tr>
    <tr>
      <th>18655</th>
      <td>5600960</td>
      <td>1005251.0</td>
    </tr>
    <tr>
      <th>18657</th>
      <td>5601090</td>
      <td>125250.0</td>
    </tr>
    <tr>
      <th>18658</th>
      <td>5601260</td>
      <td>254636.0</td>
    </tr>
    <tr>
      <th>18659</th>
      <td>5601420</td>
      <td>110185.0</td>
    </tr>
    <tr>
      <th>18661</th>
      <td>5601470</td>
      <td>191125.0</td>
    </tr>
    <tr>
      <th>18663</th>
      <td>5601980</td>
      <td>1102143.0</td>
    </tr>
    <tr>
      <th>18665</th>
      <td>5602140</td>
      <td>64988.0</td>
    </tr>
    <tr>
      <th>18668</th>
      <td>5602670</td>
      <td>5022.0</td>
    </tr>
    <tr>
      <th>18670</th>
      <td>5602820</td>
      <td>1191650.0</td>
    </tr>
    <tr>
      <th>18672</th>
      <td>5602870</td>
      <td>453146.0</td>
    </tr>
    <tr>
      <th>18674</th>
      <td>5603170</td>
      <td>154790.0</td>
    </tr>
    <tr>
      <th>18675</th>
      <td>5603180</td>
      <td>77395.0</td>
    </tr>
    <tr>
      <th>18677</th>
      <td>5603770</td>
      <td>141793.0</td>
    </tr>
    <tr>
      <th>18678</th>
      <td>5604030</td>
      <td>580759.0</td>
    </tr>
    <tr>
      <th>18679</th>
      <td>5604060</td>
      <td>676469.0</td>
    </tr>
    <tr>
      <th>18680</th>
      <td>5604120</td>
      <td>345620.0</td>
    </tr>
    <tr>
      <th>18682</th>
      <td>5604260</td>
      <td>115502.0</td>
    </tr>
    <tr>
      <th>18684</th>
      <td>5604450</td>
      <td>1850986.0</td>
    </tr>
    <tr>
      <th>18689</th>
      <td>5605090</td>
      <td>300423.0</td>
    </tr>
    <tr>
      <th>18690</th>
      <td>5605160</td>
      <td>248433.0</td>
    </tr>
    <tr>
      <th>18693</th>
      <td>5605680</td>
      <td>49923.0</td>
    </tr>
    <tr>
      <th>18694</th>
      <td>5605690</td>
      <td>346211.0</td>
    </tr>
    <tr>
      <th>18697</th>
      <td>5605762</td>
      <td>1007319.0</td>
    </tr>
    <tr>
      <th>18698</th>
      <td>5605820</td>
      <td>49923.0</td>
    </tr>
    <tr>
      <th>18699</th>
      <td>5605830</td>
      <td>84485.0</td>
    </tr>
    <tr>
      <th>18700</th>
      <td>5606090</td>
      <td>39288.0</td>
    </tr>
    <tr>
      <th>18701</th>
      <td>5606240</td>
      <td>133817.0</td>
    </tr>
  </tbody>
</table>
<p>11536 rows Ã 2 columns</p>
</div>



## Problem 5

Selected the school districts whose expenditure is less than funding. Cutting equal amount of funds from all the selected districts will be biased for the ones with lower funding. So, an equal proportion of funding was cut down from each of the districts to reduce the total federal funding by 15%.
