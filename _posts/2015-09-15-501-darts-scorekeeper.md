---
layout: post
title: 501 Darts Scorekeeper
description: Keep score and track statistics for your darts game with your iOS device.

---

![placeholder](/img/501iPhoneScreen.png "501")

501 is an iOS app for keeping track of score during a one or two-player standard 501 [darts](https://en.wikipedia.org/wiki/Darts#Games) game. This app keeps track of a single dart game (leg) at a time and records statistics for one-player gameplay.

## Inspiration

My friend and I have played darts over the summers for several years now, and we finally had the brilliant idea of developing our own app for recording score during a dart leg. It's often a pain to write the scores on a whiteboard and do mental calculations mid-game, hence the conception of the app. My friend had originally pioneered a C++ program to effeciently manage the score for a dart leg with user-input error-checking and score bound-checking. This led to my development of a user interface with statistics and settings.

## Game

![placeholder](/img/Dartboard.png "Created by Tijmen Stam from Wikipedia")

A dartboard consists of twenty different circular segments connected at a center called the double bullseye (DB). Darts is a turn-based score-shedding throwing game. The objective is to reduce your score to 0, while only being able to throw three darts per turn. A turn is also called a "visit" (to the dartboard). Standard tournaments start the score at 501 for both players, and each takes a turn until one of the players has reduced his or her score to zero. The score must land on exactly zero, and the user's last dart thrown must land in the double ring. The act of reducing one's score to zero is called an <code>out</code>.

## Implementation: One-Player

501 was written in Objective-C and works on all iOS devices able to run iOS 8.4. When the app launches, a check is made to see which player mode was last used (one-player or two-player), then the correct mode is loaded. By default, the game loads one-player mode. Several elements comprise the one-player view: <code>reset leg button</code>, <code>statistics navigation button</code>, <code>settings navigation button</code>, the <code>score text display</code>, the <code>number-of-darts-used-this-turn stepper control</code>, the <code>user-score-input text field</code>, the <code>score submit button</code>, and the default iOS <code>number pad</code>.

### Reset Leg

You have the option of resetting the current leg for whatever reason, discarding any statistics that were in progress. A UIAlertController manages the conditional flow of pressing the red ✕ in the corner—confirming if you really want to reset the game. If you confirm the action, it resets the score, and reverts statistics to the way they were before that leg began.

### Statistics

Through the use of NSUserDefaults, a small database can be taken advantage of for any app. Seven unique strings are stored in NSUserDefaults for calculating all of the statistics:

<dl>
  <dt># of Games Won</dt>
  <dd>Implemented with a count variable.</dd>

  <dt>3-Dart Avg.</dt>
  <dd>Implemented by dividing net score (sum of scores of all turns ever thrown) by the <code># of Games Won</code>.</dd>

  <dt># of 180s</dt>
  <dd>Implemented with a count variable.</dd>

  <dt>Highest Out<sup id="fnref:fn-out_explanation"><a href="#fn:fn-out_explanation" class="footnote">1</a></sup></dt>
  <dd>Implemented by comparing each out as it happens with the current statistic.</dd>

  <dt>Out Avg.</dt>
  <dd>Implemented by dividing net out (sum of scores of all outs ever thrown) by the <code># of Games Won</code>.</dd>

  <dt>Least Darts per Game</dt>
  <dd>Implemented by comparing net darts for a single game with the current statistic.</dd>

  <dt>Darts per Game Avg.</dt>
  <dd>Implemented by dividing net darts (count of all darts ever thrown) by <code># of Games Won</code>.</dd>
</dl>

Quantities such as net score, net out, and net darts are all kept track of in the backend portion of the app.

### Settings

There are two controls in the settings: Player mode and Reset Statistics. If you change the current player mode, the app discards the current game's progress (including statistics in-progress) and begins a new leg for the specified number of players.

Reset Statistics simply removes statistic-related values for the statistic keys in NSUserDefaults.

### Backend

The backend for one-player mode receives the <code>leg score</code>, <code>turn score</code>, and <code>number of darts used this turn</code>. With this information, it checks for illegal input (number over 180 or number reducing leg score to too low of a value). This is where the statistics are collected and maintained.

## Implementation: Two-Player

The implementation for two-player gameplay is similar to one-player, except that the current player's score is highlighted. Statistics for two-player gameplay are currently unwritten. This would require some form of user-identification in order to keep track of whose statistics belonged to whom. Implementing user-identification is a possible addition to this app for future updates.

### Backend

The backend for two-player mode is the same as one-player mode, but without the statistic checks and assignments.

{% highlight objc %}
// abridged code for two-player backend
if (turnScore > 180) {
    UIAlertView* alert = [[UIAlertView alloc] initWithTitle:@"Illegal score" message:@"Must be less than or equal to 180" delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil];
    [alert show];
    return 0; // illegal input
  }
  if (turnScore == legScore) {
    return 2; // victory
  }
  if (legScore - turnScore < 2) {
    UIAlertView* alert = [[UIAlertView alloc] initWithTitle:@"Bust" message:@"" delegate:nil cancelButtonTitle:@"Darn" otherButtonTitles:nil];
    [alert show];
    return 1; // legal input, the user busted, and no score reduction takes place
  }
  scoreDisplay.text = [NSString stringWithFormat:@"%d", legScore - turnScore];
  return 1; // legal input
{% endhighlight %}

## Fruits

<img src="/img/501AppIcon.png" alt="501 icon" height="75" width="60">

You can view the code on [GitHub](https://github.com/matthewkotila/501).

<div class="footnotes">
  <ol>
    <li id="fn:fn-out_explanation">
      <p>The out is the score of the player before throwing his or her last turn. The higher an out is, the less possible combinations a player has for winning in that turn. <a href="#fnref:fn-out_explanation" class="reversefootnote">↩</a></p>
    </li>
  </ol>
</div>

<hr>
