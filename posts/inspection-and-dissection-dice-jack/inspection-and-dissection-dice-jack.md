---
title: "Inspection & Dissection: Dice-Jack"
date: 2019-09-24T21:06:18
draft: false
featuredImage: static/postimages/10/dicejack1.png
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Game Development, Project Release]
---
As part of a course we had to make a game in 2 days that involved random dice and be made as a Windows forms application with Visual Basic. So we had to make a game in something that was the furthest you could get from a game engine with a language I had never used before. But hey, [Rogue](https://en.wikipedia.org/wiki/Rogue_%28video_game%29) was made in 1980 and the graphics had to be ASCII so it could be worse.

The rest of the class generally stuck to the brief with 3 dice that you rolled and you got points for each dot.

![dicejack 2](static/postimages/10/dicejack2.png)

As you can see, its rather simple. Theres 4 picture boxes, 4 labels, 2 buttons and a listbox. Dice are rolled, the dots are added up and the score is added to the listbox. After 5 rolls the game ends and the player is given their total score. But to know how I added further features we need to look at how this prototype was made.

# Behind the prototype

```vbnet

Dim Die1, Die2, Die3 As Integer
Dim DicePoints As Integer
Dim random As Random
Dim RollCount As Integer = 0
```

So we have our variables declared in the convoluted VB way. Die1, Die2, and Die3 and the scores for the corresponding dice. DicePoints is the total of those 3 scores and rollcount keeps track of how many times the dice were rolled. Random is just an object we use to generate random numbers

```vbnet
Private Sub tmrTimer_Tick(sender As Object, e As EventArgs) Handles tmrTimer.Tick
        'Increments the load bar every tick
        random = New Random
        prbLoad.Increment(4)
        If prbLoad.Value = 100 Then
        ...
        Else
            'Sets dice to random number when the timer ticks before the load is full
            Die1 = random.Next(1, 7)
            Die2 = random.Next(1, 7)
            Die3 = random.Next(1, 7)
 
            MatchImage(Die1, pbxDie1)
            MatchImage(Die2, pbxDie2)
            MatchImage(Die3, pbxDie3)
 
        End If
```

On the board is a timer (inventfully called tmrTimer), it tick every 0.1s. Each tick it adds a bit onto the load bar at the bottom and randomizes the images on the dice. MatchImage is my own subroutine but all it does is take an integer and applies the correct face for it to whichever picture box I pass it.

Once the bar gets to 100, the fun begins.

```vbnet
If prbLoad.Value = 100 Then
            'Adds the necessary points on the random dice when the load bar 
            'reaches 100
            tmrTimer.Stop()
            prbLoad.Value = 0
            RollCount += 1
            DicePoints = Die1 + Die2 + Die3 
```

We stop the timer ticking, we empty the load bar and we increment our roll counter before adding up all the spots on the dice, pretty simple.

```vbnet
'Adds Score to list
lstScore.Items.Add("Roll " & RollCount & ": " & DicePoints)
GameScore += DicePoints
DicePoints = 0
```

Almost done, we add our points to the listbox with some formatting. Add the points to the total score for the game and then set it back to 0.

```vbnet
If RollCount >= 5 Then
    MsgBox("Your game score was " & GameScore,, "Game Over")
    tmrTimer.Stop()
    btnStart.Enabled = False
    btnReset.Enabled = True
 
End If
```

Then we check whether they have reached the magic 5 rolls, if so we tell them their score and let them reset the game

There is also some initialisation code that assigns images and there are event handlers that make the buttons work but this is the main logic. If you would like to see how that stuff works, you can find it at the [github repo](https://github.com/LukeBriggsDev/Dice-Alpha)

# The Evolution

The following won’t include too many code snippets since it is slightly more extensive (the rushed deadline also means it resembles a tin of Heinz spaghetti in a tumble dryer but we’ll keep that between us)

![dicejack 1](static/postimages/10/dicejack1.png)

Lets break this down.

First we have a background that was shamelessly made in powerpoint.

![dicejack 3](static/postimages/10/dicejack3.png)

The rules, scores, and title are all labels. The bar on the right is made up of the same button and listbox items as before but with its properties edited to change colour, font and border style.

**The Rules have been changed to the following:**

- The goal is to get as close to 21 points without going over
- There is a Player and a Dealer
- Each contestant has 2 dice
- The player goes first
- Each dice spot is worth 2 points
- If either the player or dealer goes over 21, they lose the game
- After the first roll, if the player has not gone bust, they can choose to ‘stand’ or ‘hit’
- If they ‘hit’, the dice are rolled again and the player can choose to add the points of either die or the total of both dice
- The player must add a score after hitting, even if it means they will go ‘bust’
- In the event of a tie, the game goes to the dealer

The rules are based on those by [James Yates](http://www.chessandpoker.com/dice_blackjack.html)

Instead of a big chunk of code I’m going to show you the process of a turn through our good old friend the flow chart. They can be intimidating but I’ve tried to make it in a way that is easy to follow and you should be ale to get a grasp of what the code is doing behind the scenes.

![flowchart](static/postimages/10/flowchart.png)

There you go, who needs a hundred lines of bolognese when you can have pastel shapes?

Those who played the game may also notice this is the first time I’ve included a computer-controlled opponent, ‘The Dealer’. The dealer is like the player but has the advantages of being able to go second and win in the result of a draw. The dealer’s choice code is as follows:

```vbnet
If DealerCanChoose1 And Not (DealerCanChoose2 Or DealerCanChooseBoth) Then
    DealerScore += Die1 * 2
 
ElseIf DealerCanChoose2 And Not (DealerCanChoose1 Or DealerCanChooseBoth) Then
    DealerScore += Die2 * 2
 
ElseIf DealerCanChooseBoth Then
    DealerScore += (Die1 + Die2) * 2
 
ElseIf DealerCanChoose1 And DealerCanChoose2 Then
    If Die1 > Die2 Then
        DealerScore += Die1 * 2
    Else
        DealerScore += Die2 * 2
    End If
End If
 
If Not (DealerCanChoose1 Or DealerCanChoose2 Or DealerCanChooseBoth) Then
    'Dealer goes bust if no options are available
    If Die1 < Die2 Then
        DealerScore += Die1 * 2
    Else
        DealerScore += Die2 * 2
    End If
    DealerStand()
    tmrDealer.Stop()
 
ElseIf DealerScore > GameScore Then
    'Dealer stands if their score is more than the player's
    DealerStand()
Else
    DealerRoll()
End If
```

Before this code I run checks that see which options are available after a dealer rolls (these are the DealerCanChoose variables). Here there is a series of conditionals that look at the options and picks the best one for the dealer.

If there is only one option available, then the dealer will pick that one. If there is more than one option available then the dealer will pick whichever will get them the highest score that turn. If there are no options available then the dealer is bust and they will stand. The dealer will also stand if they have a score higher than that of the player. The dealer will continue to roll as long as they are lower than the player and their current score is under 17.

And thats the game! The quickest project I’ve ever done and although its rough around the edges the gameplay is solid and I got to grips with windows form’s implementation of an event-driven program. You can find the full source code [here](https://github.com/LukeBriggsDev/Dice-Jack)
