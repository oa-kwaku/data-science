# Experiement Analysis

## OP Analysis of Malleability

### Task

### Approach

#### Naive Approach

````
"If an OP has delta=True, it means that the original poster (OP) had their view changed by the responses and arguments presented by others. In this context, a malleable OP refers to an OP who is open-minded and willing to consider different perspectives and potentially change their initial view based on the arguments and evidence presented. Characteristics that make an OP malleable include:\n\n1. Openness to different viewpoints: The OP is receptive to hearing different perspectives and is willing to engage in a discussion about their beliefs.\n\n2. Willingness to evaluate evidence: The OP is open to examining evidence and considering logical arguments that challenge their initial view. They are not dismissive of opposing viewpoints but rather seek to understand and evaluate the validity of alternative arguments.\n\n3. Adaptability to new information: The OP is willing to incorporate new information and insights into their thinking process. They may realize that their initial view was based on incomplete or flawed information and are open to changing their perspective based on more accurate or comprehensive information.\n\n4. Reflective and analytical thinking: The OP engages in critical thinking and reflects on their own beliefs and assumptions. They are able to consider different angles and perspectives, weighing the strengths and weaknesses of their own arguments and the arguments presented by others.\n\n5. Non-defensive attitude: The OP does not become defensive or dismissive when their beliefs are challenged. They engage in respectful and constructive discussions, valuing the exchange of ideas and insights rather than seeking to defend their position at all costs.\n\nIt is important to note that having a malleable OP does not guarantee a change in view, as the OP may still maintain their original perspective after considering different arguments. However, a malleable OP is more open to the possibility of changing their view based on the convincing arguments and evidence presented by others."
````

- describe the dataset to LLM
- feed LLM OP text and delta examples
- loop through and ask the LLM to predict-then-explain how it arrived at the answer

### Analysis

#### Prompt

```
training_prompt = f"Here is a list of opinions: \n\n { examples }"

    to_predict = evaluation_data()
    prediction_prompt = "classify each row as" + \
                        "delta:True or delta:False, provide ab explanation of 100 characters supported by features" +\
                        f"of the text such as {''.join(FEATURES)} +" \
                        f"```{ to_predict[['title', 'delta_label', 'selftext']] }```"

    format_prompt =\
        "produce your results in valid csv format with header '\'row\', \'delta\', \'explanation\'' for every row" +\
        "explanation should talk about the stylistic features, make sure that you include the row index in" +\
        "the row column of the csv. make sure you produce a valid csv"
```

#### Output
```
"This post argues that tougher gun laws would not have prevented yesterday's tragedy where Bryce Williams killed Alison Parker and Adam Ward. The author believes that if guns were more difficult to access, Bryce Williams would have found another means to carry out the attack. The explanation highlights the use of repetition to emphasize the author's point and the mention of specific names and details related to the tragedy.\""

```

####Accuracy
```
In [223]: op_analysis.measure_accuracy(op, val_df, True)
Index(['row', 'delta', 'explanation'], dtype='object')
      row  delta  ... delta_label  correct
0    60.0  False  ...       False    False
1  1030.0   True  ...       False    False

[2 rows x 5 columns]
Out[223]: 50.0
```

GPT 3.5 works hard to consider the concepts in the argument rather than the stylistic choices event when prompted otherwise 
```
"row,delta,explanation\n597,False,\"The text does not provide any evidence for the claim that being gay is solely due to upbringing and socialization rather than biological factors. It seems to be based on personal speculation and anecdotal experience. There are no references or supporting facts provided.\""
```
### Further Investigation

TK


## Pair Analysis of Effective Argument Features

### Task

- use large language models to predict which of two similar counterarguments will succeed in changing the same view
- study found that style features and interplay features have predictive power
- The feature with the most predictive power of successful persuasion is the dissimilarity with the original post in word usage

### Approach

#### Prompt
```
training_prompt = f"Here is an opinion with an argument that convinced an OP and an argument that did not" + \
                              f" convince the OP. OP will give a ∆ to the winning argument" + \
                              f" so don't factor that in `{evaluation_data()[num_predicted: num_predicted + BATCH_SIZE]}`"

task = f" summarize the difference in the positive and negative arguments by talking about the" +\
           f" contrast in the following features {','.join(FEATURES)} only talk about the top 3 features that" +\
           f"were significantly present in the positive argument but not the negative and limit the output to 100 words"
```

### Analysis

```
The most common text features mentioned as differentiators between positive and negative arguments are:

- Higher number of words
- Presence of definite and indefinite articles
- Higher number of positive words
- Use of 2nd person pronoun "you"
- Inclusion of a link
- Presence of positive words
- Use of question marks
- Inclusion of examples
```
Note: Sample Size was 2

### Further Investigation

Get an idea of proportion, what percentage of results employed each of the following strategies?
