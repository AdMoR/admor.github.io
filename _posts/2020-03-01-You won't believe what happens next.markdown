---
description: And other catchy titles
tags: gpt2 text generation finetuning
img: https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse2.mm.bing.net%2Fth%3Fid%3DOIP.lgqyyFHbouXXr55PUFEQfwHaEK%26pid%3DApi&f=1
---


### Let's generate headlines with GPT2


- Step 1 : Find your dataset

[Thanks Kaggle](https://www.kaggle.com/chrisfilo/onion-or-not) : Headlines from the Onion

Process it : 

```python
f = open("OnionOrNot.csv")
f.readline()
data = filter(lambda line: len(line) > 0 and len(line.split(",")) == 2, f.readlines())
processed_data = map(lambda line: line.strip().split(",")[0] + "\n" if line.strip().split(",")[1] else "", data)
from functools import reduce
dumped_data = reduce(lambda a, b: a + b, processed_data)
with open("onion_processed.txt", "w") as g:
    g.write(dumped_data)
```

- Step 2 : Finetune the model

Unfortunately the full GPT2 is a bit too big to be trained easily. We can go for a smaller model.

[Follow the steps on this repo](https://github.com/minimaxir/gpt-2-simple)

Enabling the GPU mode, brings a lot of value in the training speed.

- Step 3 : Generation time : 

Early generation : 

```
And if you've ever had the chance to play a hidden game, I promise you, you will make the best ones.
And if you've ever had the chance to play a hidden game, I promise you, you will make the best ones.
And if you have the chance to play a hidden game, I promise you, you will make the best ones.
I promise you, you will make the best ones.
```

Quite interestingly, the limited training time brings a very crippled model even if the loss decreased a lot in about 10 steps.


After the 300 steps : 

```
Man with 5 cats claims he's the new face of furry fandom
```


### My best picks

```
News: Incredible! Donald Trump Has Apologized For Taking His Daughter’s Phone Call During His Janitorial Degree Convocation

Sugar Rush founder: Use of high fructose corn syrup increases obesity risk

Facebook to Fund Ads Exploring How Users Are Adrift On Internet Of Things

Report: 87% Of Humans Would Rather Be Sexually Attracted To Robots

Woman calls 911 to complain about the weather: 'I am in a bad mood right now'

Life: Environmental Experts Recommend Cutting Down On Spiked Mistletoe
```


Non hand picked sample from 1000 iterations : 

```
======== SAMPLE 1 ========
Quosed Chinese thief pulls sled down Saskatchewan trail thinking it is a pet
N.D. legislator sees potential in all New Yorkers by watching governor
Pizza delivery man gets $75 after being wrongly imprisoned for 31 years
Woman arrested for ‘havingboos’ booby-trapped her car during a raid on a meth lab
Parents arrested after arresting their kids for not bringing their kids to school. First time in 14 years: DWP finds toddler had had sex with police.
Hang-gliding cop caught after trying to take phone from car during police chase
Rival Gangs Claim They Protect Each Other From Fundamentalism Of American Capitalism
Sumerians Warn Hungarians They're Going To Be Tyrants For Hire After Vote
Russian Porn Filter Blocks XXX-Catered-For Views of its Cinemascope
Life: Environmental Experts Recommend Cutting Down On Spiked Mistletoe
Mighty Wall Of Skyscraper Stared Huge On Skulls
Man Named Vincent D’Onofrio Beautiful Wall Of Being Hept for 7 Years 
Woman Caught Making Cat Scare Party During Whole Fucking Trailblazing Son's High School Merger
Police: Air-to-Face Compliment During Traffic Stop Becomes Twitter Topic
‘Hangry disgruntled ex-girlfriend stolen for church shooting site’: 5 Signs Your Pregnant-Party-Proud-Uncles
Lucky Break For Wendy Baldwin: Mother Gets To Have Rest Of Wife’s Dead Family Over Video Game Boss During PAX Prime
‘Miracle on Ice’ Players Wondering If They Can Still Make It To Final Fantasy Game This Year
North Pole zoo displays horrifying ‘illusion of a normal day’ in life
"Muslim man arrested for slapping girlfriend."
New Haven Globetrotters warn travelers coming from Australia
Nordstrom Not Even Tugging Into Every Order Now
‘Miracle On Ice’ Players Wondering If Frozen Is Still Real After 5 Minutes
Woman charged in Walmart burglary note left on car says it was written by her
Mormons Say More on Their Page because They Can't Find Much Comment There
Texas Woman Accused of Shooting at Her Own 3-Year-Old Years Ago
‘Miracle On Ice’ Players Wondering If They Can Still Make It To Final Fantasy Game This Year
Woman calls 911 to complain about the weather: 'I am in a bad mood right now'
Trump Assures Nation That Defunds Within 2 Years
Loch Ness Monster ‘absolutely’ not ‘a flatulent-like creature’
"Republican candidate: ""Scampy collides with bicycle during campaign rally"""
Klingon language of names proposed to tackle school shootings delayed by ministers quoting Oxford English"
Study: Sharks Much Ado About Size Of Small Deer Skip Olympics
Elderly Woman Begins Dedication To Her Hive Mind
Lucky Bastard Restaurant’s Chandelier blasts 'air-ghost' power of 3500rpm
Korean Onion Socialites Seemto Have Discovered American Dead In Lake
Man shot sleeping while trying to sleep during armed robbery
Trump Attempts To Ease Tensions With Intelligence Division Seemingly Utilizing Classified Defense Data
Drama queen sues mother of boy killed by buggy during groom exchange
Man On First Day Out Of office has couple of days off before officially launching PAC
Dying Tea Party Defends Bill To End Child Support Peanuts
Man rescued from Florida playground after being bitten by snake for a year gets DUI charge
Lofty Opulence Business Admits It Needs College Fix Of Things To Keep Profit
Punk Band Shoots 1996 Video Game Advance For Badass Rock Fans
New Census Report Finds There Still 1 In 30 Americans
Woman Found Her Ex-Partner Shooting Swords From A Riflesights Sword Belt
Trump Administration Denies Obama Used Military Positions In Promotion Of Syrian President
Tornado Creeped Out By New High-Pitched Rumbling Outside Bush’s Florida Convention
'I am very much a Toronto' - Ikea vandalizes Japanese Ikea Ikea Window
Police: Home Might Now Be Completing Construction Excavations On Front Porch Where Charged With Jail
'We will notify the government when they take down this site': Residents fight Google strike against unpopular social media site
News: Major Bombshell: WikiLeaks Has Revealed That Tim Allen Is The Producer Of The Adventurous Love Story ‘El Chapo’
Gross Yearbook Typing Alleged To Coworkers In December 2016
Woman Sent Home After Driving Drunk For 7 Months Finally Gets Sent Back To Jail
Man dressed as giant penis calls police to complain about giant penis.
News: Stepping Up To The Plate: Crayola Has Announced That All Of Its Markers Will Now Inject You With Epinephrine
Mariana Airlines Offering Rising Diversified Diet Menu Features For Participants 22 Years Of Ag
```

The quality remains globally poor as some examples are non sensical or stutter on certain words.

This is a known issue of smaller version of GPT2