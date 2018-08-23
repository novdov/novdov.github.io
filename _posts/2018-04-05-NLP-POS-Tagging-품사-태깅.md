---
title: ‘Stanford Pos Tagger를 이용한 POS Tagging'
date: 2018-04-05 17:24:27
description: Stanford Pos Tagger를 이용해 POS tagging 방법을 간단하게 알아봅니다.
categories:
- NLP
tags:
- machine learning
---

###### 2018. 04. 05

## POS Tagging (품사 태깅)

- Stanford Pos Tagger를 다운받고 modelr과 jar파일의 경로를 지정한 뒤, pos_tagger로 불러온다.
- Stanford Pos Tagger Download Link: [Stanford Log-linear Part-Of-Speech Tagger](https://nlp.stanford.edu/software/tagger.html)

```python
from nltk.tag import StanfordPOSTagger
from nltk.tokenize import word_tokenize

STANFORD_POS_MODEL_PATH = "압축 푼 디렉토리/stanford-postagger-full-2018-02-27/models/english-bidirectional-distsim.tagger"
STANFORD_POS_JAR_PATH = "압축 푼 디렉토리/stanford-postagger-full-2018-02-27/stanford-postagger-3.9.1.jar"

pos_tagger = StanfordPOSTagger(STANFORD_POS_MODEL_PATH, STANFORD_POS_JAR_PATH)
```

- 분석할 text를 불러오고, `word_tokenize`로 품사 태깅한다.
- Article Source: [Zuckerberg: Facebook made a “huge mistake” — but I can fix it](https://www.axios.com/)

```python
text = """Facebook CEO Mark Zuckerberg acknowledged a range of mistakes on Wednesday, 
including allowing most of its two billion users to have their public profile data scraped by outsiders. 
However, even as he took responsibility, he maintained he was the best person to fix the problems he created."""

tokens = word_tokenize(text)
print(tokens)
print()
print(pos_tagger.tag(tokens))
```

결과는 다음과 같다.

- 품사 태깅 약어 정보: [Alphabetical list of part-of-speech tags used in the Penn Treebank Project](https://www.ling.upenn.edu/courses/Fall_2003/ling001/penn_treebank_pos.html)

```python
['Facebook', 'CEO', 'Mark', 'Zuckerberg', 'acknowledged', 'a', 'range', 'of', 'mistakes', 'on', 'Wednesday', ',', 'including', 'allowing', 'most', 'of', 'its', 'two', 'billion', 'users', 'to', 'have', 'their', 'public', 'profile', 'data', 'scraped', 'by', 'outsiders', '.', 'However', ',', 'even', 'as', 'he', 'took', 'responsibility', ',', 'he', 'maintained', 'he', 'was', 'the', 'best', 'person', 'to', 'fix', 'the', 'problems', 'he', 'created', '.']

[('Facebook', 'NNP'), ('CEO', 'NNP'), ('Mark', 'NNP'), ('Zuckerberg', 'NNP'), ('acknowledged', 'VBD'), ('a', 'DT'), ('range', 'NN'), ('of', 'IN'), ('mistakes', 'NNS'), ('on', 'IN'), ('Wednesday', 'NNP'), (',', ','), ('including', 'VBG'), ('allowing', 'VBG'), ('most', 'JJS'), ('of', 'IN'), ('its', 'PRP$'), ('two', 'CD'), ('billion', 'CD'), ('users', 'NNS'), ('to', 'TO'), ('have', 'VB'), ('their', 'PRP$'), ('public', 'JJ'), ('profile', 'NN'), ('data', 'NNS'), ('scraped', 'VBN'), ('by', 'IN'), ('outsiders', 'NNS'), ('.', '.'), ('However', 'RB'), (',', ','), ('even', 'RB'), ('as', 'IN'), ('he', 'PRP'), ('took', 'VBD'), ('responsibility', 'NN'), (',', ','), ('he', 'PRP'), ('maintained', 'VBD'), ('he', 'PRP'), ('was', 'VBD'), ('the', 'DT'), ('best', 'JJS'), ('person', 'NN'), ('to', 'TO'), ('fix', 'VB'), ('the', 'DT'), ('problems', 'NNS'), ('he', 'PRP'), ('created', 'VBD'), ('.', '.')]
```

- 명사와 동사만 따로 떼어서 봐도 기사의 맥락을 이해할 수 있다.

```python
noun_and_verbs = []
for token in pos_tagger.tag(tokens):
    if token[1].startswith("V") or token[1].startswith("N"):
        noun_and_verbs.append(token[0])
print(', '.join(noun_and_verbs))
```

```python
Facebook, CEO, Mark, Zuckerberg, acknowledged, range, mistakes, Wednesday, including, allowing, users, have, profile, data, scraped, outsiders, took, responsibility, maintained, was, person, fix, problems, created
```

