# python
In this assignment, you'll be working with messy medical data and using regex to extract relevant infromation from the data.

Each line of the dates.txt file corresponds to a medical note. Each note has a date that needs to be extracted, but each date is encoded in one of many formats.

The goal of this assignment is to correctly identify all of the different date variants encoded in this dataset and to properly normalize and sort the dates.

Here is a list of some of the variants you might encounter in this dataset:

04/20/2009; 04/20/09; 4/20/09; 4/3/09
Mar-20-2009; Mar 20, 2009; March 20, 2009; Mar. 20, 2009; Mar 20 2009;
20 Mar 2009; 20 March 2009; 20 Mar. 2009; 20 March, 2009
Mar 20th, 2009; Mar 21st, 2009; Mar 22nd, 2009
Feb 2009; Sep 2009; Oct 2010
6/2008; 12/2009
2009; 2010
Once you have extracted these date patterns from the text, the next step is to sort them in ascending chronological order accoring to the following rules:

Assume all dates in xx/xx/xx format are mm/dd/yy
Assume all dates where year is encoded in only two digits are years from the 1900's (e.g. 1/5/89 is January 5th, 1989)
If the day is missing (e.g. 9/2009), assume it is the first day of the month (e.g. September 1, 2009).
If the month is missing (e.g. 2010), assume it is the first of January of that year (e.g. January 1, 2010).
Watch out for potential typos as this is a raw, real-life derived dataset.
With these rules in mind, find the correct date in each note and return a pandas Series in chronological order of the original Series' indices.

For example if the original series was this:

0    1999
1    2010
2    1978
3    2015
4    1985
Your function should return this:

0    2
1    4
2    0
3    1
4    3
Your score will be calculated using Kendall's tau, a correlation measure for ordinal data.

This function should return a Series of length 500 and dtype int.

# python_code
# In[1]
  import pandas as pd

  doc = []
  with open('dates.txt') as file:
      for line in file:
          doc.append(line)

  df = pd.Series(doc)
  df.head(10)
  
  # In[2]
    def date_sorter():

      # Your code here
   import re
   import datetime

   df = pd.DataFrame(doc, columns=['text'])

   df['text'] = df['text'].apply(lambda x: x.strip('\n'))
   d1 = df['text'].str.findall(r'\d{1,2}[/-]\d{1,2}[/-]\d{2,4}')
   d1 = d1.reset_index()
   d2 = df['text'].str.findall(r'\d{1,2}[/-]\d{4}')
   d2 = d2.reset_index()
   d3 = df['text'].str.findall(r'\d{4}')
   d3 = d3.reset_index()
   d4 = df['text'].str.findall(r'\d{2}(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*\d{4}')
   d4 = d4.reset_index()
   d5 = df['text'].str.findall(r'(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*\d{4}')
   d5 = d5.reset_index()
   d6 = (d1, d2, d3, d4, d5)
   dataset = list(d6)


   for x in df['text']:
      if r'\d{4}' in x:
          dataset = 'January 1, '+ r'\d{4}'
          x = dataset
      elif r'\d{1,2}[/-]\d{4}' in x:
          datasplit = x.split('/')
          dataset = datasplit[0]+'/1/'+datasplit[1]
          x = dataset
      elif r'\d{1,2}[/-]\d{1,2}[/-]\d{2}' in x:
          datasplit = x.split('19')
          dataset = datasplit[0]+datasplit[1]+'19'+datasplit[2]
          x = dataset
      dataset = datetime.datetime(dataset).strftime("%m/%d/%Y")


   df['date'] = dataset
   fun = lambda date: datetime.strptime(date, "%m/%d/%Y")
   df['index'] = sorted(range(len(dataset)), key=lambda x : fun(dataset[x]))
   sdf = df
   sdf.drop('text', axis=1,inplace=True)
   answer = list(df['index'])
   final_answer = pd.Series(answer)
   return dataset
