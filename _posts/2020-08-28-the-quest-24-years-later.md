---
layout: post
title:  The Quest, 24 Years Later.
categories: []
excerpt: In 1996, AOL included a game called The Quest along with the AOL 2.0 CD. It was a point-and-click game filled with tie-ins to AOL, and contained content from Cartoon Network and other brands. The objective of the game was to answer ten "riddles" based on the in-game content and linked demos. The reward for completing the quest was... an hour of AOL. Back in 1996, I was unable to progress beyond the first one or two riddles.
---

AIn 1996, AOL included a game called The Quest along with the AOL 2.0 CD. It was a point-and-click game filled with tie-ins to AOL, and contained content from Cartoon Network and other brands. The objective of the game was to answer ten "riddles" based on the in-game content and linked demos. The reward for completing the quest was... an hour of AOL. Back in 1996, I was unable to progress beyond the first one or two riddles. My CD was lost to time, along with, it seemed, everyone else's. I began a quest of my own, to find a copy of this game and finish it. I found maybe half a dozen references to the game around the Internet, some of which were others who could barely remember the game. I archived every reference I could. I'd largely given up my search, but recently I took another look and found that someone [found](https://lostmediawiki.com/The_Quest!_(found_Cartoon_Network_point-and-click_game;_mid-1990s)){:target="_blank"} the game and put it online. I was ecstatic - I downloaded the game, and set up a Windows 95 virtual machine to play it.

All at once, my memories returned. I activated the interface where you answer the riddles. First one, easy. Second one, I didn't quite understand how you were supposed to know the answer, but I knew where in the game you were supposed to get it, giving me nine possibilities. I tried them one by one, until I found it. The third riddle comes on screen; I instantly know that I never got past this point in 1996. 24 years older, surely I could find the answer now. I started poking through the game, realizing that I was tracing the same steps I'd taken 24 years earlier, looking in the same places. I couldn't find it. This was a "fill in the blank" riddle, given a ridiculous recipe that "serves 13,000". I punched the other ingredients into Google, and found a number of recipes that used them - but none that were the one on screen. I started trying _their_ ingredients as answers, one by one. After many guesses, I arrived at the correct answer: _nutmeg_. I'd finally made some progress. Riddle number four was a simple yes/no question. I guessed it, and got it correct. More progress! Riddle number five - I had absolutely no idea. I found nothing in the game that seemed like it could be relevant, and nothing on the Internet either. I worried that the answer might be lost in some long-defunct AOL keyword page. It was time for an alternative approach.

I thought I might reverse engineer the game. I took a look through its files, and after some research, identified that the game code was baked into DXR files: an output of Macromedia Director, a logical choice for this type of software, back in the 90s. In all this time, nobody has made a decompiler for Director's DXR format, which I found described as a stack-based virtual machine. This seemed like a dead end. I browsed some of the text file based resources, but didn't find any leads there. Finally I tried searching for the answers that I was able to get as strings in the DXR binaries. Success! I located a fragment of game data that contained the answers I'd located so far, and it was obvious that the remaining answers were right beside them. I entered the remaining riddle answers. I could tell that I _never_ would have guessed all but one or two of them. I have no idea how anybody beat this game - maybe nobody has.

Without further ado, here are the answers, which I cheated to get:
![](images/content/QuestAnswers.png)

1. Ramses II
2. Arthur
3. Nutmeg
4. Yes
5. Fish
6. Banjo
7. Joker
8. Chile
9. Club KidSoft
10. Sudan

24 years later, The Quest is complete.
