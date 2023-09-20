# Persuasive argument prediction

After Reading Tan et al. I set out to replicate their study on persuasion observed on r/changemyview. 

There were two objectives for this exercise:

1. Assess the Malleability of the Original Poster (OP) via GPT 3.5 API's
   
2. Analyze Pair arguments made to convince an OP and assess the difference between the argument that convinced the OP versus the argument that did not also via GPT 3.5 API's

For more context, please refer to [https://chenhaot.com/papers/changemyview.html](https://chenhaot.com/papers/changemyview.html)

## 1. OP Analysis of Malleability

### Task

For this objective, I had a JSON dataset of OP opinions. From this dataset I wanted to see if I could predict which OP's had "malleable" minds with the help of GPT 3.5 and prompting

### Approach

I tried two different strategies to try to predict the malleability of OP's:

1. **Unprimed Approach** Rely on the encoded knowledge GPT 3.5 might have about open-mindedness and produce an unrimed prompt

2. **Feature Primed Approach** Tan et al. found a battery of statistically significant predictive text features that indicated an OP's malleability. I set out to prime GPT 3.5 with those features to see if I could beat the unprimed approach. Features are found below

```
FEATURES = ['#words', '#definite articles', '#indefinite articles', '#positive words', '#2nd person pronoun', '#links',
            '#negative words', '#hedges', '#1st person pronouns', '#1st person plural pronoun', '#.com links',
            'frac. links',
            'frac. .com links', '#examples', 'frac. definite articles', '#question marks', '#PDF links', '#.edu links',
            'frac. positive words', 'frac. question marks', '#quotations', 'arousal', 'valence', 'word entropy',
            '#sentences',
            'type-token ratio', '#paragraphs', 'Flesch-Kincaid grade levels', '#italics', 'bullet list', '#bolds',
            'numbered words',
            'frac. italics']
```

#### Unprimed Approach

I prompted GPT 3.5 with the following prompt

>I have a set of predictions here \n\n{examples}\n\n I want you to explain what characteristics make an OP malleable, now tell me, out of the following opinions that I have given you, how many of these op's are malleable based on your predictions. append to your explanation the number correct then list the row numbers with the variable { examples } I passed in `heldout_op_data` from `'cmv/op_task/heldout_op_data.jsonlist.bz2'` for GPT 3.5 to assess

**Example explanation output from GPT 3.5**

>"it means that the original poster (OP) had their view changed by the responses and arguments presented by others. In this context, a malleable OP refers to an OP who is open-minded and willing to consider different perspectives and potentially change their initial view based on the arguments and evidence presented. Characteristics that make an OP malleable include:\n\n1. Openness to different viewpoints: The OP is receptive to hearing different perspectives and is willing to engage in a discussion about their beliefs.\n\n2. Willingness to evaluate evidence: The OP is open to examining evidence and considering logical arguments that challenge their initial view. They are not dismissive of opposing viewpoints but rather seek to understand and evaluate the validity of alternative arguments.\n\n3. Adaptability to new information: The OP is willing to incorporate new information and insights into their thinking process. They may realize that their initial view was based on incomplete or flawed information and are open to changing their perspective based on more accurate or comprehensive information.\n\n4. Reflective and analytical thinking: The OP engages in critical thinking and reflects on their own beliefs and assumptions. They are able to consider different angles and perspectives, weighing the strengths and weaknesses of their own arguments and the arguments presented by others.\n\n5. Non-defensive attitude: The OP does not become defensive or dismissive when their beliefs are challenged. They engage in respectful and constructive discussions, valuing the exchange of ideas and insights rather than seeking to defend their position at all costs.\n\nIt is important to note that having a malleable OP does not guarantee a change in view, as the OP may still maintain their original perspective after considering different arguments. However, a malleable OP is more open to the possibility of changing their view based on the convincing arguments and evidence presented by others."

#### Feature Primed Approach

I prompted GPT 3.5 with the following prompt

>Here is a list of opinions: \n\n {examples} classify each row as delta:True or delta:False, provide an explanation of 100 characters supported by features of the text such as 
>{''.join(FEATURES)} ```{to_predict[['title', 'delta_label', 'selftext']]}``` produce your
>results in valid csv format with header '\'row\', \'delta\', \'explanation\'' for every row >explanation should talk about the stylistic features, make sure that you include the row index in >the row column of the csv. make sure you produce a valid csv"

the variable `{ examples }` in this case are 20 rows from the training set fed into the prompt in text format.
(sadly GPT 3.5 did not produce a consistent csv, although it was simple enough to engineer it to one in PyCharm)

**Sample Explanation**
>"This post argues that tougher gun laws would not have prevented yesterday's tragedy where Bryce Williams killed Alison Parker and Adam Ward. The author believes that if guns were more difficult to access, Bryce Williams would have found another means to carry out the attack. The explanation highlights the use of repetition to emphasize the author's point and the mention of specific names and details related to the tragedy."

### Analysis

**Unprimed Model Fit**

I performed an Area Under the Receiver Operating Characteristic (AUROC) analysis and got a result under 0.5. This essentially means the fit of the model was worse than what one expect from a human naively predicting.
```
In [389]: auc_score_unprimed = roc_auc_score(validation_numeric["delta_label_numeric"], validation_numeric[
     ...: "unprimed_numeric"])

In [390]: auc_score_unprimed
Out[390]: 0.48747769667477703
```

I then asked the LLM to take the explanations of each row and summarize what characteristics make the OP malleable

>The most common factors that made this group of OPs less malleable are:\n\n1. Strong opinions without mentioning openness to alternative viewpoints.\n2. Firm beliefs without expressing a desire to hear counter-arguments or be persuaded otherwise.\n3. Lack of mention of being open to changing their views.\n4. Clear stance on an issue, leaving little room for malleability.\n5. Stating a belief without indicating a willingness to question or self-reflect.\n\nThese factors suggest that a lack of openness to considering new information, perspectives, or evidence, as well as a strong emotional attachment to their opinions, contribute to the OPs being less malleable



What is interesting here is that GPT 3.5 mainly mentioned sentiment or topical features of text, (i.e. evidence, counterarguments, beliefs, opinions, tones, commonalities) rather than logical strategies i.e. (ad hominem, ethos, flattery)  or textual analysis (i.e. alliteration, first-person pronouns, active voice).

>"row,delta,explanation\n597,False,\"The text does not provide any evidence for the claim that being gay is solely due to upbringing and socialization rather than biological factors. It seems to be based on personal speculation and anecdotal experience. There are no references or supporting facts provided.\""

I suspect this might be because of the model's fine-tuning to be conversational.

**Feature Primed Model Fit**

In the feature primed model, we saw modestly better performance. per the AUROC analysis. I would like to take further samples of a similar size and see if we are statistically significant in our finding that the feature primed approach performed better than the unprimed approach. I did find a seemingly high false positive rate of `0.9610` for the unprimed approach versus `0.5428` for the feature primed approach.
```
In [391]: auc_score_feature_primed = roc_auc_score(validation_numeric["delta_labe
     ...: l_numeric"], validation_numeric["feature_primed_numeric"])

In [392]: auc_score_feature_primed
Out[392]: 0.5280129764801298
```

I then asked the LLM to take the explanations of each row and summarize what characteristics make the OP malleable

> This collection of posts all demonstrates different viewpoints and perspectives on various topics. However, they all have commonalities in how they express 
> personal opinions and beliefs without providing evidence or counterarguments. They also tend to use emotional language and may have dismissive tones towards 
> opposing arguments. Some posts mention specific events or examples to support their viewpoint, while others include personal experiences or anecdotes. 
> Overall, these posts reflect the subjective nature of personal opinions and the role that emotions can play in shaping them.

> Commonalities among the explanations of why a person's post indicates they are malleable include the use of personal beliefs or opinions, the mention of 
> supporting evidence or sources, and the inclusion of specific words or phrases that indicate a positive or negative sentiment. Additionally, some explanations
>  mention the presence of logical arguments or reasoning to support the views expressed in the posts.


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



```
Positive sentiment first person pronoun (#views#) links (#people#) positive words (#support#)
     ...: Positive sentiment negative words (#unconstitutiona#)
     ...: Negative sentiment indefinite article (#not#) negative words (#bad##hate#)
     ...: Positive sentiment positive words (#best#)
     ...: Positive sentiment positive words (#better#) negative words (#leaving#) positive and negative examples (#better# #well paying##stable#)
     ...: explanation
     ...: The text includes negative words and hedges, suggesting a change in view.
     ...: The text includes negative words and hedges, suggesting no change in view.
     ...: The text includes negative words, suggesting a change in view.
     ...: The text includes negative words, hedges, and a quotation, suggesting a change in view.
     ...: The text includes negative words and hedges, suggesting a change in view.
     ...: The text includes positive words and a URL, suggesting no change in view.
     ...: The text includes positive words and a URL, suggesting a change in view.
     ...: The text does not include any keywords, suggesting no change in view.
     ...: The text does not include any keywords, suggesting no change in view.
     ...: The text does not include any keywords, suggesting no change in view.
     ...: explanation
     ...: This post expresses racist views towards Gypsies, using negative words and stereotypes.
     ...: This post argues against the idea of equal value for all humans, using subjective measures.
     ...: This post discusses the idea of restricting immigration based on ethnic history, without expressing explicit racism.
     ...: This post argues that philosophy is not distinct from the scientific method, using reasoning and examples.
     ...: This post discusses the ethics of killing Hitler using a time machine, considering potential consequences.
     ...: explanation
     ...: Progressive minimum wage system is proposed, similar to progressive tax system.
     ...: Becoming an online entertainer is seen as a risky career due to uncertainties of success.
     ...: There are competing opinions on whether transgender people should be allowed to choose restrooms.
     ...: The word \'bulbous\' is argued to be grosser than the word \'moist\' because it has no positive connotations and requires a more complicated mouth shape to say.
     ...: explanation
     ...: The text includes the words \'I believe\' indicating a personal opinion.
     ...: The text includes negative words like \'annoying\', \'stupid\', and \'assholes\', indicating a negative viewpoint.
     ...: The text includes negative words like \'inconsiderate\', \'unthinkable\', and \'asshole\', indicating a negative viewpoint.
     ...: The text includes negative words like \'annoying\', \'wake\', and \'fucking\', indicating a negative viewpoint.
     ...: The text includes negative words like \'problems\', \'violence\', and \'fighting\', indicating a negative viewpoint.
```
