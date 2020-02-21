# Sentiment Analysis in Power BI via SQL Server Machine Learning Services using R or Python 

The database you need to recreate the example used in the blog can be found in the database folder located in this repo. The example was done using R but if you want to use Python instead then you need to make sure that the Python version of the pre-trained models are installed in your database. The instructions to install the Python version can be found [here](https://bit.ly/38nROBI). Also you need to replace step 3 in the blog with the instructions that follows.

Set the values of the ***@Query*** and ***@PythonScript*** variables. You need to set the @Query variable to a string that represents the T-SQL needed for your input dataset. You also need to set the @PythonScript variable to hold the Python code that will be used to perform the sentiment analysis. The T-SQL script for the @Query variable is as follows:

```
      SET @Query = 'SELECT [id], [text] FROM [dbo].[SentimentData]'
```

It is a simple SELECT statement that grabs the data from the [dbo].[ SentimentData] table needed for our Python script. Next, we define the Python code needed to score the data. Here is the code:

```
      SET @PythonScript = '
from microsoftml import rx_featurize, get_sentiment

sentiment_scores = rx_featurize(
    data=dfInput,
    ml_transforms=[get_sentiment(cols=dict(scores="text"))])

dfOutput = sentiment_scores
'
```

The script is only three lines long because the pre-trained model does the heavy lifting. In the above Python script, the first line of code loads the necessary functions needed for this script. They are the ***rx_featurize()*** function and ***get_sentiment()*** function. The next line (actually the next three lines due to formatting) performs the sentiment analysis. The workhorse functions are the rxFeaturize() and getSentiment() functions from the microsoftml package. The rxFeaturize() function enables you to access data that has undergone a machine learning data transformation via microsoftml. It specifies the machine learning transformation that is being performed in the mlTransforms argument; which, in this example, is a getSentiment() transformation. The cols argument in getSentiment() is used to specify the columns in your dataset that you want to score. It uses a dictionary with the key of each ***key/value*** pair in the dictionary representing the name of the new column that will contain the sentiment scores and the value representing the the column that contains the text you are scoring. Please note that you can perform sentiment analysis on multiple columns at once. You just need to add the necessary key/value pairs to your dictionary using the method previously describe. The getSentiment() function will return a numeric value between 0 and 1 for each sentiment analysis it performs. The closer to 0 the value is, the more negative the sentiment, and the closer to 1 the value is, the more positive the sentiment.

You also need to make changes to ***step 4***. You just need to change the part of the code that referenced R to Python as illustrated below:

```
      EXEC sp_execute_external_script
             @language = N'Python'
            ,@input_data_1 = @Query
            ,@input_data_1_name = @InputDFName
            ,@output_data_1_name = @OutputDFName
            ,@script = @PythonScript
```
The ***SentimentAnalysisDB*** database contains two stored procedures. The ***dbo.getSentiments_R*** stored procedure uses R and ***dbo.getSentiments_Python*** stored procedure uses Python. Remember you need to make sure that ***SQL Server Machine Learning Services*** is installed in the database as well as the pre-trained models. Happy coding!
