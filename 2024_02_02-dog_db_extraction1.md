
# Queries that Test Relationships Between Test Completion and Features of Dogs.

[Original **SQL script** can be accessed here](https://github.com/lugmenn/Insights-for-Dognition/blob/main/02_Dogs_v_CompletedTests.sql)

A database provided by Dognition was examined with the use of **MySQL** to generate insights to find out how could the company retain their users and increase the number of customers using the platform.
The general content of the database can be read thoroughly [here](assets/p04-dataextraction/00_Dognition-Data-Set-Description.pdf).

![schema](assets/p04-dataextraction/DognitionDB_schema.png)

It contained information about the registered users, their registered dogs, their exam answers and completed tests logs, as well as customer reviews about the testing program and activity throughout the website. Sensitive personal data from the users (such as names, e-mails, phone numbers and full address) was not included in the dataset for analysis.

## Findings

The Dognition Team wanted to now if the number of completed tests were dependent on some dog characteristics. The main findings were:

* The dog personalities' groups didn´t show a difference in the completed tests totals.
  
* The same observation is applicable when evaluating the dog's breed purity.

* It´s not possible to assume with certainty whether the dog's breed group has a direct influence or not.

* However, it was found the Herding and Sporting groups had the highest number of completed tests.

* Dognition could decide to explore more deeply these groups and develop marketing strategies for owners of these dog breed types, as well as create special content for them.

* The neutered dogs showed to have 1 to 2 more completed tests than those who weren't. The analysis should be extended to find if the difference persists when other variables are paired.
  
## Method

The following queries intended to address the issues related to "Features of Dogs" that could potentially influence the number of tests the dogs were likely to complete.

First, the SQL module and the database were imported into the workspace.

```python
%load_ext sql
%sql mysql://studentuser:studentpw@localhost/dognitiondb
%sql USE dognitiondb
```

     * mysql://studentuser:***@localhost/dognitiondb
    0 rows affected.
    []


## Personality influence over the number of tests completed 

The Dognition personality dimensions represent distinct combinations of characteristics assessed by the Dognition tests.  It is certainly possible that certain personalities of dogs might be more or less likely to complete tests.  

To test the relationship between Dognition personality dimensions and test completion totals, it was necessary a query that would output a summary of the number of tests completed by dogs that have each of the Dognition personality dimensions.

First, I explored the different categories dogs can be classified into by their personality (dimension).

```sql
%%sql
SELECT DISTINCT dimension
FROM dogs;
```

     * mysql://studentuser:***@localhost/dognitiondb
    11 rows affected.





<table>
    <tr>
        <th>dimension</th>
    </tr>
    <tr>
        <td>charmer</td>
    </tr>
    <tr>
        <td>protodog</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>einstein</td>
    </tr>
    <tr>
        <td>stargazer</td>
    </tr>
    <tr>
        <td>maverick</td>
    </tr>
    <tr>
        <td>socialite</td>
    </tr>
    <tr>
        <td>ace</td>
    </tr>
    <tr>
        <td>expert</td>
    </tr>
    <tr>
        <td>renaissance-dog</td>
    </tr>
    <tr>
        <td></td>
    </tr>
</table>



Given that the query's output showed there are null values, further exploration into those cases was required.

Next, a summary for the number of tests completed by each unique dog, alogside its personality, was created.


```sql
SELECT DISTINCT
      d.dog_guid AS dogID,
      d.dimension AS dimension,
      COUNT(c.created_at) AS tests_completed
FROM dogs d JOIN complete_tests c
    ON d.dog_guid=c.dog_guid
GROUP BY dogID
LIMIT 10;
```

     * mysql://studentuser:***@localhost/dognitiondb
    10 rows affected.





<table>
    <tr>
        <th>dogID</th>
        <th>dimension</th>
        <th>tests_completed</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>charmer</td>
        <td>21</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>protodog</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>2</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>11</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>einstein</td>
        <td>31</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>stargazer</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>maverick</td>
        <td>27</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>protodog</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27c1c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>einstein</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27c5be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>socialite</td>
        <td>20</td>
    </tr>
</table>


After obtaining the number of tests completed by each individual dog, then the data was aggregated to know how many tests were completed by dogs in a certain dimension or personality group.

The previous query was used as a subquery to create a temporary table with the relevant data.



```sql
SELECT
   TestsPerDog.dimension AS personality, 
   AVG(TestsPerDog.tests_completed) AS AvgCompleteTests
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID, 
        d.dimension AS dimension, 
        COUNT(c.created_at) AS tests_completed
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY personality;
```

     * mysql://studentuser:***@localhost/dognitiondb
    11 rows affected.





<table>
    <tr>
        <th>personality</th>
        <th>AvgCompleteTests</th>
    </tr>
    <tr>
        <td>None</td>
        <td>6.9416</td>
    </tr>
    <tr>
        <td></td>
        <td>9.5352</td>
    </tr>
    <tr>
        <td>ace</td>
        <td>23.3878</td>
    </tr>
    <tr>
        <td>charmer</td>
        <td>23.2594</td>
    </tr>
    <tr>
        <td>einstein</td>
        <td>23.2171</td>
    </tr>
    <tr>
        <td>expert</td>
        <td>23.3926</td>
    </tr>
    <tr>
        <td>maverick</td>
        <td>22.8199</td>
    </tr>
    <tr>
        <td>protodog</td>
        <td>22.9336</td>
    </tr>
    <tr>
        <td>renaissance-dog</td>
        <td>23.0157</td>
    </tr>
    <tr>
        <td>socialite</td>
        <td>23.1194</td>
    </tr>
    <tr>
        <td>stargazer</td>
        <td>22.7368</td>
    </tr>
</table>



The output returned 11 rows of dimension categories, which include a null-value category (labeled as _None_) and an empty category (non-null or "").

To check out how many unique DogIDs are included in each of those categories empty, the following query was used:


```sql
SELECT
    TestsPerDog.dimension AS personality, 
    COUNT(TestsPerDog.dogID) AS NumDogs
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID, 
        d.dimension AS dimension, 
        COUNT(c.created_at) AS tests_completed
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      WHERE dimension IS NULL OR dimension=""
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY personality;
```

     * mysql://studentuser:***@localhost/dognitiondb
    2 rows affected.





<table>
    <tr>
        <th>personality</th>
        <th>NumDogs</th>
    </tr>
    <tr>
        <td>None</td>
        <td>13705</td>
    </tr>
    <tr>
        <td></td>
        <td>71</td>
    </tr>
</table>


The Dognition testing platform assigns each dog a personality dimension after the initial Assessment for completing the first 20 tests. This means the dogs with the null-value as dimension are those that did not complete the required number of tests fo getting a label.

However, the dogs with the empty dimension values could indicate there were some odd conditions as to why they weren't assigned a label. For this reason, it was appropiate to find out if there were some common characteristics among those dogs.

```sql
SELECT DISTINCT 
    d.dog_guid AS dogID,
    d.breed, 
    d.weight, 
    d.dimension, 
    d.exclude,
    MIN(c.created_at) AS first_test, 
    MAX(c.created_at) AS last_test, 
    COUNT(c.created_at) AS completed_tests
FROM dogs d JOIN complete_tests c
    ON d.dog_guid=c.dog_guid
WHERE d.dimension=""
GROUP BY dogID
LIMIT 10;
```

     * mysql://studentuser:***@localhost/dognitiondb
    10 rows affected.





<table>
    <tr>
        <th>dogID</th>
        <th>breed</th>
        <th>weight</th>
        <th>dimension</th>
        <th>exclude</th>
        <th>first_test</th>
        <th>last_test</th>
        <th>completed_tests</th>
    </tr>
    <tr>
        <td>fd45154c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Retriever</td>
        <td>30</td>
        <td></td>
        <td>0</td>
        <td>2013-05-23 07:06:21</td>
        <td>2013-07-02 12:15:18</td>
        <td>17</td>
    </tr>
    <tr>
        <td>fd51daac-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Dachshund</td>
        <td>10</td>
        <td></td>
        <td>1</td>
        <td>2014-10-21 18:53:02</td>
        <td>2014-10-21 19:10:07</td>
        <td>3</td>
    </tr>
    <tr>
        <td>fd5d7d3a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Border Collie-Labrador Retriever Mix</td>
        <td>50</td>
        <td></td>
        <td>0</td>
        <td>2013-11-16 02:26:15</td>
        <td>2013-11-16 02:38:57</td>
        <td>4</td>
    </tr>
    <tr>
        <td>fd680124-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Belgian Tervuren</td>
        <td>70</td>
        <td></td>
        <td>1</td>
        <td>2014-11-10 21:21:06</td>
        <td>2014-12-16 01:13:28</td>
        <td>13</td>
    </tr>
    <tr>
        <td>fd699c28-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pembroke Welsh Corgi</td>
        <td>20</td>
        <td></td>
        <td>1</td>
        <td>2014-09-19 17:42:37</td>
        <td>2014-09-22 17:58:25</td>
        <td>4</td>
    </tr>
    <tr>
        <td>fd6a7774-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Chihuahua</td>
        <td>0</td>
        <td></td>
        <td>1</td>
        <td>2014-10-06 00:57:46</td>
        <td>2014-10-09 22:55:51</td>
        <td>2</td>
    </tr>
    <tr>
        <td>fd6bf766-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd</td>
        <td>50</td>
        <td></td>
        <td>1</td>
        <td>2014-10-06 01:54:49</td>
        <td>2014-10-30 02:16:12</td>
        <td>14</td>
    </tr>
    <tr>
        <td>fd6cfd96-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>60</td>
        <td></td>
        <td>1</td>
        <td>2014-10-10 01:01:21</td>
        <td>2014-10-10 12:33:52</td>
        <td>4</td>
    </tr>
    <tr>
        <td>fd6d1182-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Portuguese Water Dog</td>
        <td>60</td>
        <td></td>
        <td>1</td>
        <td>2014-10-10 13:22:58</td>
        <td>2014-10-10 13:36:17</td>
        <td>3</td>
    </tr>
    <tr>
        <td>fd6d1ea2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Labrador Retriever</td>
        <td>50</td>
        <td></td>
        <td>1</td>
        <td>2014-10-06 15:28:42</td>
        <td>2014-10-23 20:24:20</td>
        <td>7</td>
    </tr>
</table>



Note: _The query without the LIMIT clause brought back 71 dog IDs with empty NON-null-values in the Dimension column._

A quick inspection of the output showed most of the dogs with an empty dimension value ("") had a exclude flag (value "1") in the exclude column. The DogIDs flagged with a 1 in that column were not meant to be included in the monitoring by the Dognition team, since they could have been flagged for many reasons, including them being fake data for platform testing.

Because of this, those entries (subjects with non-NULL empty strings in the dimension column and _1_ in the exclude column) were excluded from the analysis.

NOTE: _the 'exclude' column could have had a value or 1, 0 or a null-value. The Dognition team recognized a NULL-value as a valid one, so the query WHERE exclude!=1 wouldn't have been appropiate because then, the output would have excluded those null values._


```sql
SELECT
    TestsPerDog.dimension AS personality,
    AVG(TestsPerDog.tests_completed) AS AvgCompleteTests,
    COUNT(TestsPerDog.dogID) AS NumberOfDogs
FROM (SELECT 
        DISTINCT d.dog_guid AS dogID, 
            d.dimension AS dimension, 
            COUNT(c.created_at) AS tests_completed
        FROM
            dogs d JOIN complete_tests c
        ON
            d.dog_guid=c.dog_guid
        WHERE 
            (d.dimension IS NOT NULL AND d.dimension<>"") AND 
            (d.exclude IS NULL OR d.exclude=0)
        GROUP BY 
            dogID
     ) AS TestsPerDog
GROUP BY personality;
```

     * mysql://studentuser:***@localhost/dognitiondb
    9 rows affected.





<table>
    <tr>
        <th>personality</th>
        <th>AvgCompleteTests</th>
        <th>NumberOfDogs</th>
    </tr>
    <tr>
        <td>ace</td>
        <td>23.5100</td>
        <td>402</td>
    </tr>
    <tr>
        <td>charmer</td>
        <td>23.3594</td>
        <td>626</td>
    </tr>
    <tr>
        <td>einstein</td>
        <td>23.2385</td>
        <td>109</td>
    </tr>
    <tr>
        <td>expert</td>
        <td>23.4249</td>
        <td>273</td>
    </tr>
    <tr>
        <td>maverick</td>
        <td>22.7673</td>
        <td>245</td>
    </tr>
    <tr>
        <td>protodog</td>
        <td>22.9570</td>
        <td>535</td>
    </tr>
    <tr>
        <td>renaissance-dog</td>
        <td>23.0410</td>
        <td>463</td>
    </tr>
    <tr>
        <td>socialite</td>
        <td>23.0997</td>
        <td>792</td>
    </tr>
    <tr>
        <td>stargazer</td>
        <td>22.7968</td>
        <td>310</td>
    </tr>
</table>



At first glance, the results didn't suggest there was a great influence of the dogs' personalities over the number of tests dogs had completed.

For more accuracy, a statistical testing over these results could be executed to prove if there is a significant difference between the different groups. For now, it could be a better idea for the Dognition team to put their effort into analyzing different aspects to improve Dognition completion and usage rates.



##  Dog breeds influence over the total of tests completed

To evaluate the impact a dog's breed had on the number of completed tests, I ran one analysis assessing the Breed Group and one more with Breed Type.

Similarly to the evaluation of the personality types, first, I wanted to find out how many breed groups were registered in the database.

```sql
SELECT DISTINCT breed_group
FROM dogs;
```

     * mysql://studentuser:***@localhost/dognitiondb
    9 rows affected.





<table>
    <tr>
        <th>breed_group</th>
    </tr>
    <tr>
        <td>Sporting</td>
    </tr>
    <tr>
        <td>Herding</td>
    </tr>
    <tr>
        <td>Toy</td>
    </tr>
    <tr>
        <td>Working</td>
    </tr>
    <tr>
        <td>None</td>
    </tr>
    <tr>
        <td>Hound</td>
    </tr>
    <tr>
        <td>Non-Sporting</td>
    </tr>
    <tr>
        <td>Terrier</td>
    </tr>
    <tr>
        <td></td>
    </tr>
</table>



The query's output showed there were null values among the breed group field, a variable that was explored to find out those dogs characteristics.

```sql
SELECT DISTINCT 
     d.dog_guid AS dogID, 
     d.breed, 
     d.weight,
     d.exclude,
     MIN(c.created_at) AS first_test, 
     MAX(c.created_at) AS last_test, 
     COUNT(c.created_at) AS completed_tests
FROM dogs d JOIN complete_tests c
    ON d.dog_guid=c.dog_guid
WHERE d.breed_group IS NULL
GROUP BY dogID;
```

     * mysql://studentuser:***@localhost/dognitiondb
    10 rows affected.





<table>
    <tr>
        <th>dogID</th>
        <th>breed</th>
        <th>weight</th>
        <th>exclude</th>
        <th>first_test</th>
        <th>last_test</th>
        <th>completed_tests</th>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>50</td>
        <td>None</td>
        <td>2013-02-05 18:57:05</td>
        <td>2013-02-05 22:38:01</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27c5be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Shih Tzu-Poodle Mix</td>
        <td>0</td>
        <td>None</td>
        <td>2013-02-05 21:44:38</td>
        <td>2013-02-10 03:33:37</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27c74e-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Pembroke Welsh Corgi Mix</td>
        <td>40</td>
        <td>None</td>
        <td>2013-02-06 04:45:28</td>
        <td>2014-01-06 05:58:13</td>
        <td>14</td>
    </tr>
    <tr>
        <td>fd27c956-7144-11e5-ba71-058fbc01cf0b</td>
        <td>German Shepherd Dog-Nova Scotia Duck Tolling Retriever Mix</td>
        <td>30</td>
        <td>None</td>
        <td>2013-05-17 17:45:46</td>
        <td>2013-06-14 23:42:53</td>
        <td>11</td>
    </tr>
    <tr>
        <td>fd27cea6-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>10</td>
        <td>None</td>
        <td>2013-02-06 04:44:50</td>
        <td>2013-02-06 04:48:29</td>
        <td>2</td>
    </tr>
    <tr>
        <td>fd27d0b8-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Australian Shepherd-German Shepherd Dog Mix</td>
        <td>90</td>
        <td>None</td>
        <td>2013-02-07 05:15:48</td>
        <td>2013-12-20 21:03:18</td>
        <td>21</td>
    </tr>
    <tr>
        <td>fd27d248-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Golden Doodle</td>
        <td>70</td>
        <td>None</td>
        <td>2013-02-09 05:49:46</td>
        <td>2013-02-09 06:10:11</td>
        <td>6</td>
    </tr>
    <tr>
        <td>fd27d4dc-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>30</td>
        <td>None</td>
        <td>2013-02-10 03:28:12</td>
        <td>2013-07-20 02:12:37</td>
        <td>28</td>
    </tr>
    <tr>
        <td>fd27d9fa-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mixed</td>
        <td>90</td>
        <td>1</td>
        <td>2014-09-24 15:10:03</td>
        <td>2014-09-24 21:23:37</td>
        <td>20</td>
    </tr>
    <tr>
        <td>fd27dc52-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Mudi</td>
        <td>20</td>
        <td>None</td>
        <td>2014-10-06 22:21:56</td>
        <td>2014-10-06 22:24:02</td>
        <td>2</td>
    </tr>
</table>



Note: _The query without the LIMIT clause brought back 8816 dog IDs with missing data on the the breed_group column._

There was no pattern to be found on the first evaluation of the output. This means there was no good reason for this specific trait to be excluded from the analysis.

The next step was to find the relationship between breed group and the number of tests completed.

```sql
SELECT
    TestsPerDog.breed_group AS breed_group,
    COUNT(TestsPerDog.dogID) AS NumberOfDogs,
    SUM(TestsPerDog.tests_completed) AS TotalTests,
    AVG(TestsPerDog.tests_completed) AS AvgCompleteTests
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID, 
        d.breed_group AS breed_group, 
        COUNT(c.created_at) AS tests_completed
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      WHERE d.exclude IS NULL OR d.exclude=0
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY breed_group
ORDER BY AvgCompleteTests DESC;
```

     * mysql://studentuser:***@localhost/dognitiondb
    9 rows affected.





<table>
    <tr>
        <th>breed_group</th>
        <th>NumberOfDogs</th>
        <th>TotalTests</th>
        <th>AvgCompleteTests</th>
    </tr>
    <tr>
        <td></td>
        <td>179</td>
        <td>3536</td>
        <td>19.7542</td>
    </tr>
    <tr>
        <td>Herding</td>
        <td>1774</td>
        <td>19952</td>
        <td>11.2469</td>
    </tr>
    <tr>
        <td>Sporting</td>
        <td>2470</td>
        <td>27149</td>
        <td>10.9915</td>
    </tr>
    <tr>
        <td>Working</td>
        <td>865</td>
        <td>8854</td>
        <td>10.2358</td>
    </tr>
    <tr>
        <td>None</td>
        <td>8564</td>
        <td>87568</td>
        <td>10.2251</td>
    </tr>
    <tr>
        <td>Hound</td>
        <td>564</td>
        <td>5674</td>
        <td>10.0603</td>
    </tr>
    <tr>
        <td>Non-Sporting</td>
        <td>964</td>
        <td>9659</td>
        <td>10.0197</td>
    </tr>
    <tr>
        <td>Terrier</td>
        <td>780</td>
        <td>7748</td>
        <td>9.9333</td>
    </tr>
    <tr>
        <td>Toy</td>
        <td>1041</td>
        <td>9073</td>
        <td>8.7157</td>
    </tr>
</table>


As it is visible on the generated table, Herding and Sporting groups turned out to have the highest number of completed tests.

A suggestion for the Dognition team would be to deepen the analysis further specificaly into these breeds for creating marketing strategies or create specific content for them, as well as study who the owners of those dogs are.

To showcase the top 4 breed groups with the most number of completed tests for a report the previous query was adapted:


```sql
SELECT
    TestsPerDog.breed_group AS breed_group,
    COUNT(TestsPerDog.dogID) AS NumberOfDogs,
    SUM(TestsPerDog.tests_completed) AS TotalTests,
    AVG(TestsPerDog.tests_completed) AS AvgCompleteTests
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID, 
        d.breed_group AS breed_group, 
        COUNT(c.created_at) AS tests_completed
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      WHERE (d.exclude IS NULL OR d.exclude=0)
         AND
        d.breed_group IN ("Sporting","Hound","Herding","Working")
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY breed_group
ORDER BY AvgCompleteTests DESC;
```

     * mysql://studentuser:***@localhost/dognitiondb
    4 rows affected.





<table>
    <tr>
        <th>breed_group</th>
        <th>NumberOfDogs</th>
        <th>TotalTests</th>
        <th>AvgCompleteTests</th>
    </tr>
    <tr>
        <td>Herding</td>
        <td>1774</td>
        <td>19952</td>
        <td>11.2469</td>
    </tr>
    <tr>
        <td>Sporting</td>
        <td>2470</td>
        <td>27149</td>
        <td>10.9915</td>
    </tr>
    <tr>
        <td>Working</td>
        <td>865</td>
        <td>8854</td>
        <td>10.2358</td>
    </tr>
    <tr>
        <td>Hound</td>
        <td>564</td>
        <td>5674</td>
        <td>10.0603</td>
    </tr>
</table>



Next, the breed types (breed purity) were examined.  


```sql
SELECT DISTINCT breed_type
FROM dogs;
```

     * mysql://studentuser:***@localhost/dognitiondb
    4 rows affected.





<table>
    <tr>
        <th>breed_type</th>
    </tr>
    <tr>
        <td>Pure Breed</td>
    </tr>
    <tr>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
    </tr>
    <tr>
        <td>Cross Breed</td>
    </tr>
    <tr>
        <td>Popular Hybrid</td>
    </tr>
</table>


-- And in a similar approach, I tried to find the relationship between the breed type and the number of tests completed.


```sql
SELECT
    TestsPerDog.breed_type AS breed_type,
    COUNT(TestsPerDog.dogID) AS NumberOfDogs,
    SUM(TestsPerDog.tests_completed) AS TotalTests,
    AVG(TestsPerDog.tests_completed) AS AvgCompleteTests
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID,
        d.breed_type AS breed_type,
        COUNT(c.created_at) AS tests_completed
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      WHERE d.exclude IS NULL OR d.exclude=0
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY breed_type
ORDER BY AvgCompleteTests DESC;
```

     * mysql://studentuser:***@localhost/dognitiondb
    4 rows affected.





<table>
    <tr>
        <th>breed_type</th>
        <th>NumberOfDogs</th>
        <th>TotalTests</th>
        <th>AvgCompleteTests</th>
    </tr>
    <tr>
        <td>Popular Hybrid</td>
        <td>634</td>
        <td>6874</td>
        <td>10.8423</td>
    </tr>
    <tr>
        <td>Cross Breed</td>
        <td>2884</td>
        <td>30573</td>
        <td>10.6009</td>
    </tr>
    <tr>
        <td>Pure Breed</td>
        <td>8865</td>
        <td>92291</td>
        <td>10.4107</td>
    </tr>
    <tr>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>4818</td>
        <td>49475</td>
        <td>10.2688</td>
    </tr>
</table>



The output didn't show an appreciable difference between number of tests completed by dogs of different breed types, however, statistical testing is required to have certainty.
    
  
## Influence of the breed and neutering over the number of tests completed

To explore the results found above a little further, the breed's type were relabeled according to "Pure_Breed" and "Not_Pure_Breed".  


```sql
SELECT
    TestsPerDog.PureBreed AS PureBreed,
    COUNT(TestsPerDog.dogID) AS NumberOfDogs,
    SUM(TestsPerDog.NumberOfTests) AS TotalTests,
    AVG(TestsPerDog.NumberOfTests) AS AvgCompleteTests
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID,
        COUNT(c.created_at) AS NumberOfTests,
        d.breed_type,
        CASE d.breed_type
            WHEN "Pure Breed" THEN "Pure_Breed"
            ELSE "Not_Pure_Breed"
            END AS PureBreed
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      WHERE d.exclude IS NULL OR d.exclude=0
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY PureBreed
ORDER BY AvgCompleteTests DESC
LIMIT 10;
```

     * mysql://studentuser:***@localhost/dognitiondb
    10 rows affected.





<table>
    <tr>
        <th>dogID</th>
        <th>NumberOfTests</th>
        <th>breed_type</th>
        <th>PureBreed</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>21</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>20</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>2</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>11</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>31</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27b948-7144-11e5-ba71-058fbc01cf0b</td>
        <td>20</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27ba1a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>27</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27bbbe-7144-11e5-ba71-058fbc01cf0b</td>
        <td>20</td>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>Not_Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27c1c2-7144-11e5-ba71-058fbc01cf0b</td>
        <td>20</td>
        <td>Pure Breed</td>
        <td>Pure_Breed</td>
    </tr>
    <tr>
        <td>fd27c5be-7144-11e5-ba71-058fbc01cf0b</td>
        <td>20</td>
        <td>Cross Breed</td>
        <td>Not_Pure_Breed</td>
    </tr>
</table>



Then, it was possible to find the the number of tests completed by breed type and whether or not the dogs were neutered.


```sql
SELECT
    TestsPerDog.PureBreed AS PureBreed,
    TestsPerDog.neutered AS Neutered,
    COUNT(TestsPerDog.dogID) AS NumberOfDogs,
    SUM(TestsPerDog.NumberOfTests) AS TotalTests,
    AVG(TestsPerDog.NumberOfTests) AS AvgCompleteTests
FROM (SELECT DISTINCT 
        d.dog_guid AS dogID,
        d.dog_fixed AS neutered,
        d.breed_type,
        CASE d.breed_type
            WHEN "Pure Breed" THEN "Pure_Breed"
            ELSE "Not_Pure_Breed"
            END AS PureBreed,
        COUNT(c.created_at) AS NumberOfTests
      FROM dogs d JOIN complete_tests c
        ON d.dog_guid=c.dog_guid
      WHERE d.exclude IS NULL OR d.exclude=0
      GROUP BY dogID
     ) AS TestsPerDog
GROUP BY PureBreed, Neutered;
```

     * mysql://studentuser:***@localhost/dognitiondb
    6 rows affected.





<table>
    <tr>
        <th>PureBreed</th>
        <th>Neutered</th>
        <th>NumberOfDogs</th>
        <th>TotalTests</th>
        <th>AvgCompleteTests</th>
    </tr>
    <tr>
        <td>Not_Pure_Breed</td>
        <td>None</td>
        <td>97</td>
        <td>969</td>
        <td>9.9897</td>
    </tr>
    <tr>
        <td>Not_Pure_Breed</td>
        <td>0</td>
        <td>592</td>
        <td>5139</td>
        <td>8.6807</td>
    </tr>
    <tr>
        <td>Not_Pure_Breed</td>
        <td>1</td>
        <td>7647</td>
        <td>80814</td>
        <td>10.5681</td>
    </tr>
    <tr>
        <td>Pure_Breed</td>
        <td>None</td>
        <td>135</td>
        <td>1118</td>
        <td>8.2815</td>
    </tr>
    <tr>
        <td>Pure_Breed</td>
        <td>0</td>
        <td>1687</td>
        <td>15822</td>
        <td>9.3788</td>
    </tr>
    <tr>
        <td>Pure_Breed</td>
        <td>1</td>
        <td>7043</td>
        <td>75351</td>
        <td>10.6987</td>
    </tr>
</table>



Eventhough a dog's breed_type doesn't seem to have a strong relationship with how many tests a dog completed, neutered dogs, on average, seemed to finish 1-2 more tests than non-neutered dogs. It is reasonable to explore this variable along with other characteristics, and assess whether it is because of the dogs' characteristics or because of their owners behavior and their own interest in completing more tests. 

Another interesting factor to study was whether the breed type could have had effects on the time tests were completed. This time, the standard deviation was also calculated.



```sql
SELECT 
    d.breed_type AS BreedType,
    AVG(TIMESTAMPDIFF(minute,e.start_time, e.end_time)) AS AvgDuration,
    STDDEV(TIMESTAMPDIFF(minute,e.start_time, e.end_time)) AS SDDuration
FROM dogs d JOIN exam_answers e
  ON d.dog_guid=e.dog_guid
WHERE
    TIMESTAMPDIFF(minute,e.start_time, e.end_time)>0
GROUP BY breed_type;
```

     * mysql://studentuser:***@localhost/dognitiondb
    4 rows affected.





<table>
    <tr>
        <th>BreedType</th>
        <th>AvgDuration</th>
        <th>SDDuration</th>
    </tr>
    <tr>
        <td>Cross Breed</td>
        <td>11810.3230</td>
        <td>59113.45580229881</td>
    </tr>
    <tr>
        <td>Mixed Breed/ Other/ I Don&#x27;t Know</td>
        <td>9145.1575</td>
        <td>48748.626840777506</td>
    </tr>
    <tr>
        <td>Popular Hybrid</td>
        <td>7734.0763</td>
        <td>45577.65824281632</td>
    </tr>
    <tr>
        <td>Pure Breed</td>
        <td>12311.2558</td>
        <td>60997.35425304078</td>
    </tr>
</table>



Many of the standard deviations have larger magnitudes than the average duration values.  This suggests  there are outliers in the data that are significantly impacting the reported average values, so the average values are not likely trustworthy. These data should be exported to R or Python for more sophisticated statistical analysis.


```
