---
layout: post
title:  "Solve the Jumble!"
---

# Solve the Jumble!
                <p>Ah yes.  The weighty problem of solving the Jumble in the daily paper.  This is usually an easy puzzle
		to solve, although when your mind gets "jumble-blocked", it can tend to stay that way for a while.</p>
		
		<p>In the &quot;Jumble&quot; puzzles (copyrighted by David L. Hoyt) the challenge is to unscramble a set
		of words.&nbsp; The individual words are usually 5 to 8 characters long.  In the official puzzle, selected
		letters from the individual answers are unscramble to provide the punchline to a horrific pun.</p>
		
		<table style="border: 1px solid #DADAE3;" align="center">
		    <tr><td>
		
		    <table align="center" width="100%"><tr><td><form name="form1" method="post" action="/jumble_solver.php"><div align="center"><strong>Enter 'Jumbled' word:</strong></div><BR/><div align="center"><input name="scrambled" type="text" style="width: 80px; padding: 2px; border: 1px solid black" value="elbmuj"><input type="submit" name="Submit" value="Unscramble!"></div></form></td></tr></table>		    
		    </td></tr>
		</table>
		
		<p><b>How to Solve It</b></p>
		
		<p>The human brain seems to be particularly gifted at doing this.  Many solvers have had the experience
		of having the unscrambled answer for each word pop right into their head after a quick glance.&nbsp; This is followed
		by a short burst of intellectual smugness and fantasies about accepting the Nobel Prize for Extreme Cleverness.</p>
		
		<p>And yet, sometimes the unscrambling process stumbles and the letters just stare back at you, defiantly scrambled.&nbsp;
		At that point any number of tricks to jump-start the puzzle-solving process might work. Simple tactics like reversing
		the scrambled letters and giving your brain another go, or heuristics like looking for common groupings of letters
		('th', 'ing', etc.) in order to simplify the number of combinations to consider.</p>
		
		<p>This is a fun Comp Sci 101 exercise because of how inventive people can be at finding difficult solutions. On the
		other hand, if you identify the elegant, simplifying hack (or, well, have it explained to you ... I'm not proud)
		it can take your own personal smugness to all new levels.  In this case, the clever hack is described by Jon Bentley
		in the "Aha! Algorithms" essay from his book "<a href="http://amzn.to/10vbiyW">Programming Pearls</a>".</p>
		
		<p>To unscramble the individual words, the essay describes this central trick:
		sort the letters of the scrambled word into alphabetical order. Then compare this sorted arrangement of letters with a
		dictionary of words processed the same way (an alphabetical list of words each paired with a version of itself with its
		letters sorted alphabetically).</p>
		
		<p>For example, say that one of our scrambled puzzle words is &quot;lorac&quot;. We sort the letters into alphabetical
		order to obtain &quot;aclor&quot;. We then search our dictionary for any five-letter words that also sorted to the
		pattern &quot;aclor&quot;. (I used a list of about 100,000 words and found four solutions: &quot;calor&quot;,
		&quot;carlo&quot;, &quot;carol&quot;, and &quot;coral&quot;)</p>
		
		<p>The final script used in this page is written in PHP and is fairly short and simple:</p>
		<ol>
		    <li>Collect a scrambled word from the form</li>
		    <li>Lowercase the word, then create and save a 'signed' version of it.</li>
		    <li>Load the contents of the 'squashed' file into an array</li>
		    <li>Go through the array looking for a match for the 'signed' version of the scrambled word</li>
		    <li>Display the answer (if any) and prep the page for another try.</li>
		</ol>
		
		<p>As mentioned, that worked, but it wasn't very fast the first time I wrote it because of the time needed to load
		the word list (100,000 lines long).  And then I had a linear search through the list, etc.  Not efficient.</p>
		
		<p>I replaced the list with a call to a MySQL table of dictionary words and their corresponding 'signed' versions, and
		that made for a simpler and much faster script.</p>
	
