# Ai-biases-GSCE/A level-algorithm-


I) Intrododuction: 


II) Methodological approach:

import pandas as pd
import numpy as np

type_descriptor = {
    'year': 'int',
    'Qualification': 'string',
    'Local Authority': 'string',
    'URN': 'string',
    'School or college name': 'string',
    'School or college type': 'string',
    'Subject': 'string',
    'A*': 'float',
    'A': 'float',
    'B': 'float',
    'C': 'float',
    'D': 'float',
    'E': 'float',
    'Fail/No results': 'float'
}

# load data files
dt_2017 = pd.read_csv("data-2017.csv", dtype=type_descriptor)
dt_2018 = pd.read_csv("data-2018.csv", dtype=type_descriptor)
dt_2019 = pd.read_csv("data-2019.csv", dtype=type_descriptor)

dt_2017 = dt_2017[dt_2017['Subject'] == 'History']
dt_2018 = dt_2018[dt_2018['Subject'] == 'History']
dt_2019 = dt_2019[dt_2019['Subject'] == 'History']

# dt_2017 = dt_2017.head(100)
# dt_2018 = dt_2018.head(100)
# dt_2019 = dt_2019.head(100)

# get unique list of schools in all years
ls_schools = list(
    set(dt_2017['URN'].unique()) &
    set(dt_2018['URN'].unique()) &
    set(dt_2019['URN'].unique())
)

# use 2019 data as test data
dt_test = dt_2019

# use year 2017-2018 to calculate historical grade distribution
dt_historic = pd.concat([dt_2018, dt_2019])

# config
ls_grades = ['A*', 'A', 'B', 'C', 'D', 'E', 'F']
rj = 0.7

# helper columns
dt_test["F"] = dt_test["Fail/No results"]
for grade in ls_grades:
    dt_test["R_" + grade] = dt_test[grade] / dt_test['Total entries']
dt_historic = dt_historic.replace('Supp', np.nan)
dt_historic["F"] = dt_historic["Fail/No results"]
for grade in ls_grades:
    dt_historic["R_" + grade] = dt_historic[grade] / \
        dt_historic['Total entries']

# school table
dt_schools = pd.DataFrame(ls_schools, columns=['URN'])
dt_schools["is_private"] = False

# calculate C[k,j]
for grade in ls_grades:
    # dt_schools["Ckj_" + grade] = 0.0
    # dt_schools["qkj_" + grade] = 0.0
    # dt_schools["pkj_" + grade] = 0.0
    # dt_schools["Pkj_" + grade] = 0.0
    dt_schools["Pkj_error_" + grade] = 0.0

for idx in range(len(ls_schools)):
    urn = ls_schools[idx]

    dt_school_historic = dt_historic[dt_historic['URN'] == urn]
    dt_school_test = dt_test[dt_test['URN'] == urn]

    if dt_school_test['School or college type'].iloc[0] == 'Independent School':
        dt_schools.at[idx, "is_private"] = True

    remainder_q = 1
    remainder_p = 1
    for grade in ls_grades:
        # calculate historical grade distribution values
        Ckj_value = dt_school_historic['R_' + grade].mean()
        # dt_schools.at[idx, "Ckj_" + grade] = Ckj_value

        # generate made up data for predicted grades
        # we assume that predicted grades are 20% higher then than actual grades
        R_value = dt_school_test['R_' + grade].iloc[0]
        if isinstance(R_value, float):
            # qkj - historic teacher prediction
            qkj_value = min(remainder_q, R_value * 1.2)
            remainder_q = max(remainder_q - qkj_value, 0)
            # dt_schools.at[idx, "qkj_" + grade] = value_q
            # pkj - lockdown teacher prediction
            pkj_value = min(remainder_p, R_value * 1.3)
            remainder_p = max(remainder_p - pkj_value, 0)
            # dt_schools.at[idx, "pkj_" + grade] = value_p

        # this is weak assumption based on the expectation that GCSE
        # grades will strongly correlate with A level marks
        pkj_value_GCSE = Ckj_value
        qkj_value_GCSE = R_value

        # Pkj
        Pkj_value = pkj_value
        # dt_schools.at[idx, "Pkj_" + grade] = Pkj_value
        if dt_school_test['Total entries'].iloc[0] > 15:
            Pkj_value = (1 - rj) * Ckj_value + rj * \
                (Ckj_value + qkj_value_GCSE - pkj_value_GCSE)
        dt_schools.at[idx, "Pkj_error_" + grade] = Pkj_value - R_value


# Pkj = (1-rj)Ckj + rj(Ckj + qkj - pkj)
# The variables
# n is the number of pupils in the subject being assessed
# k is a specific grade
# j indicates the school
# C[k,j] is the historical grade distribution of grade at the school (centre) over the last three years, 2017-19.
# q[k,j] is the predicted grade distribution based on the class’s prior attainment at GCSEs. A class with mostly 9s (the top grade) at GCSE will get a lot of predicted A*s; a class with mostly 1s at GCSEs will get a lot of predicted Us.
# p[k,j] is the predicted grade distribution of the previous years, based on their GCSEs. You need to know that because, if previous years were predicted to do poorly and did well, then this year might do the same.
# r[j] is the fraction of pupils in the class where historical data is available. If you can perfectly track down every GCSE result, then it is 1; if you cannot track down any, it is 0.
# CAG is the centre assessed grade.
# P[k,j] is the result, which is the grade distribution for each grade k at each school j.

dt_private_schools = dt_schools[dt_schools['is_private']]
dt_state_schools = dt_schools[dt_schools['is_private'] != True]

print("Prediction error")
for grade in ls_grades:
    print(
        grade, "private:", "{:.2%}".format(
            dt_private_schools['Pkj_error_' + grade].mean()),
        "state:",
        "{:.2%}".format(dt_state_schools['Pkj_error_' + grade].mean()),
    )

percent_missing = dt_private_schools.isnull().sum() / \
    len(dt_private_schools)

print("Missing data (private schools)",
      "{:.2%}".format(percent_missing['Pkj_error_A*']))

percent_missing = dt_state_schools.isnull().sum() / \
    len(dt_state_schools)

print("Missing data (state schools)", "{:.2%}".format(
    percent_missing['Pkj_error_A*']))


dt_schools.to_csv('test.csv')




III) Overall Results:

- which hypothesis we can confirm or need to reject? 







IV) Limitations of the study:  




V)Implications and suggestions for further research:








