# Decoding Biases in AI: the Ofqual algorithm scandal

# I) Introduction: 

In 2020, Ofqual, the English regulator of qualifications, exams and tests in England produced an algorithm trying to combat glade inflation and moderate teacher-predicted grades for A levels, after examinations were cancelled in the COVID-19 pandemic.

The algorithm, officially titled "Ofqual's Direct Centre Performance model" works as follows. It is based on the record of school being assessed. Firstly, the examination centre of each school provides a list of teacher predicted grades (centre assessed grades, CAGs). The students of each class is then placed in rank order on the basis of their predicted grade. Secondly, for classes with more than 15 students, the historical results of the school were also consulted. Going back three years, the average number of students getting each grade (A*, A, B, C, D, E, and U, with A* being the highest attainable and U denoting "unclassified, or in other words "fail") is being added into the calculation. Thirdly, if this data was available, the calculation could be further enhanced on the basis of the historical data of the class's prior attainment in their GSCE's. (Hern 2020b).

In his study, Haines found that the Ofqual algorithm skewes the grades as well upwards as downwards (Haines 2020). In other words, it hollows out the middle of the grade distribution for students, pushing certain students up and other students down based on their school's historical data. The main aim of this study is to find out whether the algorithm is negatively biased towards all schools or whether there is a difference between state and private schools. This brought us to our main hypothesis:

H1. The algorithm either strongly pushes results downwards across all schools or pushes the results of independent schools upwards. 

The main finding of our analysis is that the algorithm had a tendency to push results upwards. There was a slightly stronger upward trend for independent schools, but this was not statistically greatly significant. However, given that very small classes’ data were masked by the government for data protection reasons, almost 2x as many independent schools’ data was missing from the calculation than maintained school data, indicating that the difference might be stronger in real life.

Our study underlines the relevance of a thorough ethicial assessment of algorithms used by both public and private institutions, _especially_ in an educational setting. In addition, it brings again under attention the relevance of adequate technical training and digital skills of public administration officials who are, in case of lack of knowledge, often still highly depend on third party providers that might have less for the ethical implications of their algorithm.

# II) Methodological approach:

# Import the relevant libraries 

    import pandas as pd
    import numpy as np
    
# Name Categories and variable types

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

# Load the data files (historical school and subject data from England and Wales, from 2017 to 2019)

    dt_2017 = pd.read_csv("data-2017.csv", dtype=type_descriptor)
    dt_2018 = pd.read_csv("data-2018.csv", dtype=type_descriptor)
    dt_2019 = pd.read_csv("data-2019.csv", dtype=type_descriptor)

    dt_2017 = dt_2017[dt_2017['Subject'] == 'History']
    dt_2018 = dt_2018[dt_2018['Subject'] == 'History']
    dt_2019 = dt_2019[dt_2019['Subject'] == 'History']

    # dt_2017 = dt_2017.head(100)
    # dt_2018 = dt_2018.head(100)
    # dt_2019 = dt_2019.head(100)
    

# Link schools to their unique ID number  

    ls_schools = list(
        set(dt_2017['URN'].unique()) &
        set(dt_2018['URN'].unique()) &
        set(dt_2019['URN'].unique())
     )

# Use 2019 data as the test data (as 2020 results are unavailable)

    dt_test = dt_2019

# Use year 2017-2018 to calculate historical grade distribution

    dt_historic = pd.concat([dt_2018, dt_2019])

    ls_grades = ['A*', 'A', 'B', 'C', 'D', 'E', 'F']
       rj = 0.7

# Create helper columns

    dt_test["F"] = dt_test["Fail/No results"]
       for grade in ls_grades:
       dt_test["R_" + grade] = dt_test[grade] / dt_test['Total entries']
    dt_historic = dt_historic.replace('Supp', np.nan)
    dt_historic["F"] = dt_historic["Fail/No results"]
    for grade in ls_grades:
       dt_historic["R_" + grade] = dt_historic[grade] / \
          dt_historic['Total entries']
          
          
# Create school table 

    dt_schools = pd.DataFrame(ls_schools, columns=['URN'])
    dt_schools["is_private"] = False


# We calculate C[k,j]

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
    
        
 # We calculate the historical grade distribution values
 
        Ckj_value = dt_school_historic['R_' + grade].mean()
        # dt_schools.at[idx, "Ckj_" + grade] = Ckj_value

        
       
 # Calculate qkj and pkj
 
These were not published by the UK government but based on Ofqual’s own studies, we assumed that in normal times, predicted grades are about 20% higher than actual grades, this is based on the unsubstantiated but presumed assumption that GCSE grades, on which predicted grades are usually based, correlate strongly with A level result. 
 
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
        
        pkj_value_GCSE = Ckj_value
        qkj_value_GCSE = R_value
 

        # Pkj
        Pkj_value = pkj_value
        # dt_schools.at[idx, "Pkj_" + grade] = Pkj_value
        if dt_school_test['Total entries'].iloc[0] > 15:
            Pkj_value = (1 - rj) * Ckj_value + rj * \
                (Ckj_value + qkj_value_GCSE - pkj_value_GCSE)
        dt_schools.at[idx, "Pkj_error_" + grade] = Pkj_value - R_value

        Pkj = (1-rj)Ckj + rj(Ckj + qkj - pkj)
       


# The algorithm 

The algorithm is defined as follows:

        # n is the number of pupils in the subject being assessed
        
        # k is a specific grade
        
        # j indicates the school
        
        # C[k,j] is the historical grade distribution of grade at the school (centre) over the last three years, 2017-19.
        
        # q[k,j] is the predicted grade distribution based on the class’s prior attainment at GCSEs. This is where the pupils' own ability comes in. A class with mostly 9s (the top grade) at GCSE will get a lot of predicted A*s; a class with mostly 1s at GCSEs will get a lot of predicted Us. This is where the pupils' own ability comes in.
        
        # p[k,j] is the predicted grade distribution of the previous years, based on their GCSEs. You need to know that because, if previous years were predicted to do poorly and did well, then this year might do the same.
        
        # r[j] is the fraction of pupils in the class where historical data is available. If you can perfectly track down every GCSE result, then it is 1; if you cannot track down any, it is 0.
        
        # CAG is the centre assessed grade.
        
        # P[k,j] is the result, which is the grade distribution for each grade k at each school j.


# Identify and divide schools into private and public 

     dt_private_schools = dt_schools[dt_schools['is_private']]
     dt_state_schools = dt_schools[dt_schools['is_private'] != True]

     print("Prediction error")
     for grade in ls_grades:
         print(
            grade, "private:", "{:.2%}".format(
            dt_private_schools['Pkj_error_' + grade].mean()),
        "state:",
        "{:.2%}".format(dt_state_schools['Pkj_error_' + grade].mean()),
    

# Calculate overall missing data percentage differences between private and state schools

      percent_missing = dt_private_schools.isnull().sum() / \
           len(dt_private_schools)

      print("Missing data (private schools)",
           "{:.2%}".format(percent_missing['Pkj_error_A*']))

      percent_missing = dt_state_schools.isnull().sum() / \
           len(dt_state_schools)

     print("Missing data (state schools)", "{:.2%}".format(
           percent_missing['Pkj_error_A*']))


     dt_schools.to_csv('test.csv')




# III) Overall Results:


> Prediction error 

    # A* private: 1.45% / state: 0.36%

    # A private: 3.69% / state: 1.43%

    # B private: 2.17% / state: 2.90%

    # C private: -3.36% / state: 0.67%

    # D private: -3.19% / state: -3.05%

    # E private: -1.03% / state: -2.04%

    # F private: -0.26% / state: -0.54%

> Missing data (private schools) 28.54%

> Missing data (state schools) 16.50%

Our main hypothesis was:

H1. The algorithm either strongly pushes results downwards across all schools or pushes the results of independent schools upwards. 

a.	Our findings allow us to reject the hypothesis that the algorithm pushes results downwards.

b.	We find that algorithm seems to raise grades slightly however. An effect that is a stronger for private schools (see the 3.69% increase as opposed to 1.43% for state schools), however, given that we are below 5% differences, statistical relevance is low.

# IV) Limitations of the study:  

Firstly, the study unfortunately lacked a non-negligible amount of real historical data. A lot of schools could not by law publish their data as some classes were simply too small, which could have had some drawbacks in terms of the privacy of the concerned students, who would have likely been recognized by their results. This eventually forced us to resort to use an abundant amount of proxy data. It becomes clear that as well a relative lack of independent school data makes results less accurate and might - although not necessarily - hide an even greater advantage for private schools under the Ofqual algorithm. For instance, the overprediction and underprediction of grades were based on the assumptions made by former Ofqual studies were it was assumed that on average teachers tended to over predicted their own students grades by 20 to 40 per cent higher, compared to what they would in reality score at the exam.

Secondly, the predicted grades data were not published by the government. Where the predicted grades are officially based on GCSE results, which are available, it is worth noting that pupil movement between KS4 (GCSEs) and KS% (A-levels) between schools would make the historical GCSE data only partially accurate. That being said, had we had more time, we would have used school GCSE data as the next best option in our calculations. 

Thirdly, because of the low differences between the different grade distributions for private and state schools, the results in this study can not be generalised. Nevertheless, the findings of our study co-align with earlier studies that analysed the Ofqual algorithm (Haines 2020; Hern 2020a; Hern 2020b; Birch 2020; James 2020; Nye and Thompson 2020).

# V) Implications and suggestions for further research:

> Policy Implications 

Firstly, the use of educational algorithms that give great value to historical school data, as the Ofqual algorithm did, brings with it the danger of favouring certain types of school above others, failing to take personal student characteristics into account. First, we find, although statistically insignificant, some evidence that the algorithm slightly advantages independent schools. This is a finding in line with earlier studies of the algorithm (Haines 2020; Nye and Thompson 2020). Second, we find that the 15 pupil-limit for using CAG (teacher predicted grades) is a clear advantage to schools with small class sizes, which is very substantially more common among independent schools.

Secondly, even if the algorithm is not significantly biased in its treatment of independent and state schools, it must be stressed that the schools-based allocation of grades to individual students is enormously inappropriate. Pupils are graded by their own and school's statistical background, not their own work and merit. Much like with criminal justice, one cannot do away with trial based on the individual case/personal actions in favour of just sentencing anyone who is ‘statistically likely’ to commit a crime. This approach makes it impossible for students from unlikely/disadvantaged students to significantly outperform their predictions, even though this happens, especially among immigrant students who tend to attend state schools. 

All in all, it is the assigning life-changing grades to individuals based on statistical data that makes this policy and therefore the algorithm inappropriate. Algorithms and AI have great potential to aid governments but it must be realised that not all governmental functions are suitable for automatisation. 

>  Suggestions for further research

Firstly, since we only based our studies on students results in history it could be potentially enriching to apply the algorithm for other school subjects and determine whether certain subjects are more affected than others. For example, the Guardian found that the algorithm has different accuracy levels for the subjects "English" and "Maths" (Hern 2020).

Secondly, Ofqual is only on example among others. So it would also be interested to make a cross comparison between different countries educational algorithm's systems. We could for instance take the example of the former APB algorith in France. 

Thirdly, we exclusively based our comparison on the binary distinction between independent/ private schools and state schools. Therefore It would be appropriate to enlarge this analysis and make a more detailed separation between schools types. 


# Acknowledgments

Gabor Győrfi, Tom Haines (University of Bath) and Peter Kemp (King's College London).

We are especially grateful for Tom Haines and Peter Kemp who have very kindly helped to point us in the right direction regarding available datasets and Gabor Gyorfi who very kindly helped us get started with Visual Studio Code and some of the more advanced aspects of Python coding.

# Sources:

Adams et al. (2020). A-level results: almost 40% of teacher assessments in England downgraded. The Guardian. Available online at: https://www.theguardian.com/education/2020/aug/13/almost-40-of-english-students-have-a-level-results-downgraded.

Birch, J. (2020). Exam Result Scandal. Harvard News Analysis. Available online at: https://www.harvard.co.uk/why-we-cant-blame-tech-for-the-exam-result-scandal/.

Haines, T. S. F. (2020). A-Levels: The Model is not the Student. Personal blog. Available online at: https://thaines.com/post/alevels2020.

Hern, A. (2020a). Do the maths: why England's A-level grading system is unfair. The Guardian. Available online at: https://www.theguardian.com/education/2020/aug/14/do-the-maths-why-englands-a-level-grading-system-is-unfair.

Hern A. (2020b). Ofqual's A-level algorithm: why did it fail to make the grade? The Guardian. Available online at: https://www.theguardian.com/education/2020/aug/21/ofqual-exams-algorithm-why-did-it-fail-make-grade-a-levels. 

James, R. (2020). Pkj = (1-rj)Ckj + rj(Ckj + qkj – Pkj). Flourish. Available online at: https://flourishworld.com/pkj-1-rjckj-rjckj-qkj-pkj/.

Nye, P. and Thomson, D. (2020). A-level results 2020: Why independent schools have done well out of this year's awarding process. FFT Education Datalab. Available online here: https://ffteducationdatalab.org.uk/2020/08/a-level-results-2020-why-independent-schools-have-done-well-out-of-this-years-awarding-process/.

Quinn, B. and Adams, R. (2020). England exam rows timeline: was Ofqual warned of algorithm bias? https://www.theguardian.com/education/2020/aug/20/england-exams-row-timeline-was-ofqual-warned-of-algorithm-bias.

Raw data: https://www.compare-school-performance.service.gov.uk/download-data?currentstep=region&downloadYear=2016-2017&regiontype=all&la=0
