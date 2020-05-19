## Named Entity Extractor

Named Entity Recognition is one of the important NLP tasks and helps identifying name of a person, place, organization in a document.

BERT (Bidirectional Encoder Representations from Transformers) - is a NLP language model, BERT uses  transformer and attention mechanism to learn contextual relation between words. An encoder and a decoder is used to learn the contextual relation.  Generally Language model predict the next word in a sequence, as this strictly puts the model to process in a directly. But BERT uses Masked Language Model and Next Sentence Prediction.

Using BERT model, we can quiet easily tune it with less data to specific task.  The Model which is being used for the NER task is trained based on BERT and ner_conll2003 dataset (http://docs.deeppavlov.ai/en/master/features/models/ner.html). Constant re-training is done on top of the base model to resolve any disambiguation.

Using the model, the below document

```reStructuredText
British Grand Prix: no F1 race without quarantine exemptions - BBC Sport British Grand Prix: no F1 race without quarantine exemptions By Andrew Benson Chief F1 writer 19 May From the section Formula 1 says it would be unable to hold a British Grand Prix if personnel are not given exemptions from plans to quarantine international travellers. The UK government will "soon" impose a requirement on all arrivals from abroad to self-isolate for 14 days. An F1 spokesman said: "A 14-day quarantine would make it impossible to have a "Additionally, it has a major impact on literally tens of thousands of jobs linked to F1 and supply chains."
A DCMS spokesman said no decisions on exemptions had yet been reached. F1 has drawn up plans to ensure its races Teams will be kept apart from each other at the tracks, and stay in separate hotels, to which they will be driven in buses. In addition, all personnel would be tested before travelling and every two days while at the races. The spokesman said: "We would be travelling back to the UK on F1-only occupied aircraft and all staff would be tested, making a quarantine totally unnecessary. "If all elite sport is to return to TV, then exemptions must be provided." The government's plans also threaten football's attempts to revive the Champions League this summer.
BBC Sport understands that F1's plans to start the season with two races in Austria on 5 and 12 would be able to go ahead, but the British races planned to follow on after that would have to be dropped. They would be replaced by other races in Europe, with the teams returning to the UK for two weeks before travelling again, until the F1's season is on ice as a result of the coronavirus crisis. Ten races have so far been postponed or cancelled, but F1 chairman Chase Carey has said he is "increasingly confident" of being able to stage a season of about 16 races starting in July.
Current plans involve the two races in Austria, then two events at either Silverstone - or Hockenheim in Germany if the UK proves impossible. After that, there would be a selection of four races from the previously scheduled grands prix in Spain, Hungary, Belgium, Italy, France and the Netherlands, before moving on around the rest of the world. The plan is to try to fit in either Canada or Singapore if possible, before races in Azerbaijan and Russia and then on to China and Japan.
There would be grands prix in the US, Mexico and Brazil if the coronavirus situation allows, then Vietnam, and then the season would end with consecutive races in Bahrain on 6 December and Abu Dhabi on 13 December. However, the number of variables involved means plans remain flexible, although F1 hopes to announce a European season at least in the next two to three weeks.
```



identifies the below NER.

```
organisations=[BBC Sport, Formula 1, F1's, DCMS, F1]
people=[Chase Carey, Andrew Benson],
locations=[Hockenheim, Singapore, Hungary, Europe, Japan, Bahrain, Abu Dhabi, Russia, Spain, Silverstone, Canada, Azerbaijan, Austria, Netherlands, Vietnam, Belgium, UK, China, Brazil, Italy, Mexico, France, Germany, US])
```



To use the base model install the model using 

```shell
python -m deeppavlov install ner_conll2003_bert
```



```python
from deeppavlov import configs, build_model
from more_itertools import locate
from deeppavlov import configs, train_model
ner_model = build_model(configs.ner.ner_conll2003_bert)
```





To train the model with new data set

```python
from deeppavlov import configs, build_model
from more_itertools import locate

#configs contains a property to the path of the bert json config file, that property need to be overwritten to point to a path where the new set of train / test data.

from deeppavlov import configs, train_model
ner_model = train_model(configs.ner.ner_conll2003_bert)
ner_model.save()
```



## Entity Report

Entity report is a spring boot app, which reads the data from the No-SQL database and present it to the user on the recognized entities.