## Essex Management Skills

### Skills Entity Cloud
![Skills Entity Cloud](https://raw.githubusercontent.com/essexmgmt/skills/master/entity_cloud.png)

### Requirements
* Python3.7
* Pandas
* xlswriter
* xlrd
* NLTK

### Getting started
`pip install -r requirements`

### Skills Analysis


```python
# Install a pip package in the current Jupyter kernel
import sys
!{sys.executable} -m pip install -r requirements.txt >> /dev/null
```


```python
nltk.download('punkt')
nltk.download('averaged_perceptron_tagger')
```


```python
"""
Extract Transform and Loads skills data
"""
import os
import pandas as pd
import zipfile as zf

from datetime import datetime


class ETLSkills:
    """
    Returns one dataframe of aggregated skills from an archived zip file of xlsx skills
    """
    skills_archive = "skills_essex.zip"

    def __init__(self):
        self.dfs = []

    def extract(self):
        skills_archive = self.skills_archive
        for fname in os.listdir(f"temp_extract_{skills_archive}"):
            if fname[-4:] == "xlsx":
                print(f"Reading excel {fname} ...")
                self.dfs.append(pd.read_excel(f"temp_extract_{skills_archive}/"+fname))
        return pd.concat(self.dfs, sort=True)

    def write_to_file(self, dfs):
        tmstmp = datetime.now().strftime("%Y_%m_%d_%H_%M_%S")
        writer = pd.ExcelWriter(f'./aggregate_skills_{tmstmp}.xlsx', engine='xlsxwriter')
        sheet_names = ['sheet1']
        for df, sheet_name in zip(dfs, sheet_names):
            df.to_excel(writer, sheet_name=sheet_name)
        writer.save()

if __name__ == "__main__":
    etl = ETLSkills()
    df = etl.extract()
    
```


```python
# aggregates non-empty cells of skills
skills = []
for index, row in df.iterrows():
    for i in range(0, len(row)):
        if not pd.isna(row[i]):
            skills.append(row[i])
len(skills)
```


```python
# tokenize and extract entities
from nltk import word_tokenize
tokens = word_tokenize(" ".join(skills))
tokens[:10]
```


```python
# filter out stop words, punctuation, qualifliers, and quantifiers
from nltk.corpus import stopwords 
stop_words = set(stopwords.words('english'))
articles = ['a', 'the', 'an', 'I', ]
directions = ['details','list','level','rows','skills']
qualifiers = ['advanced', 'intermediate', 'beginner', 'exploratory', 'expert', 'certified', 'experience', 'basic', 'good', 'some', 'fluent']
quantifiers = [str(n) for n in range(0,50)]
not_in_context = ['yr', 'yrs', 'years', 'e.g', 'w/', "`s", ]

filters = [stop_words, directions, qualifiers, quantifiers, not_in_context] 
def filter_tokens(filters):
    filtered_tokens = tokens
    for token_filter in filters:
        filtered_tokens = [w for w in filtered_tokens if not w.lower() in token_filter and w.isalpha()] 
    return filtered_tokens

filtered_tokens = filter_tokens(filters)
```


```python
# parts of speech tagging
post_tokens = nltk.pos_tag(filtered_tokens)
entities = []
verbs = []
concepts = []
remainder = []
for token in post_tokens:
    # entity extraction
    if token[1] in ['NN', 'NNP', 'NNPS', 'NNS']:
        entities.append(token[0])
    elif token[1] in ['JJ']:
        concepts.append(token[0])
    elif token[1] in ['VBD', 'VBG', 'VBN', 'VBP']:
        verbs.append(token[0])
    else:
        remainder.append(token)
```


```python
csv_entities = ",".join(entities)
```


```python
# Future analysis TODO:
# tf-idf (skill frequency : # technical entities with that skill describes a skill area)
# LSA / LDA clustering of skills to identify similarity between skills and transferrability
    # if you know A, recommend learning a similar skill B
# identify qualifiers to associate as rank weights for each skills polarity
# entity classification by specified categories [FRONTEND, BACKEND, DATABASES, CLOUD, BIOLOGY, GENETICS, CHEMISTRY, NLP, ML, etc]
```
[Skills NLP Analysis IPython Notebook](https://raw.githubusercontent.com/essexmgmt/skills/master/analysis.ipynb)


### About the Skills Data
Skills data to generate this analysis is stored locally and not provided in this public repository.

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/essexmgmt/skills/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Need to get in touch? Check out our website ![Essex Management](essexmanagement.com) or [contact](anna.lu@essexmanagement.com) and we’ll help you sort it out.
