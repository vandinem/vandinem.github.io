---
layout: post
title:  "Solve the Jumble!"
tags: [puzzles, python, sqlite, php, sql]
---

# Solve the Jumble!
The &quot;Jumble&quot; puzzle (copyrighted by [David L. Hoyt](https://www.davidlhoyt.com/)) is usually easy to solve, although when your mind gets "jumble-blocked", it can tend to stay that way.
		
The challenge is to unscramble a set of words.  The individual words are usually 5 to 8 characters long.  In the official puzzle, selected letters from the individual answers unscramble to provide the punchline to a dependably horrific pun.
		
## How to Solve It

The Jumble is a satisfying puzzle. With a little focus, the answer usually comes easily.  Many solvers have had the experience of having the unscrambled answer for each word pop right into their head after a quick glance.

But sometimes the unscrambling process stumbles and the letters just stare back at you, defiantly scrambled. At that point any number of tricks to jump-start the puzzle-solving process might work. Simple tactics like reversing the scrambled letters, or looking for common letter groupings ('th', 'ing', etc.) ... reorder or simplify the problem.

This is a fun Comp Sci 101 exercise because of how inventive people can be at finding difficult solutions. Generating lots of letter permutations and look-ups is an invitation to inefficiency and slow going.

On the other hand, if you identify a clever transformation (or, well, have it explained to you ... I'm not proud) the resulting script is elegant and fast.

In this case, the clever hack is described by Jon Bentley in the "Aha! Algorithms" essay from his book [Programming Pearls](https://bookshop.org/books/programming-pearls/9780201657883).

To unscramble the individual words, sort the letters of the scrambled word into alphabetical order. Then compare this sorted arrangement of letters with a dictionary of words processed the same way (an alphabetical list of words each paired with a version of itself with its letters sorted alphabetically).

For example, say that one of our scrambled puzzle words is "lorac". We sort the letters into alphabetical order to obtain "aclor". Then we search our dictionary for any five-letter words that also sorted to the pattern "aclor". From a list of 10,000 words I found three solutions:

1. carlo
2. carol
3. coral

In my latest implementation of this, I decided to first create my reference word list as a SQLite file.  This has the advantages of being fast, and deployable in lots of environments.  The following Python script creates the SQLite file 'puzzlewords' which includes the table 'wordlist'.  I identify a couple of URLs with useful English word lists to process. (The 10,000 word lists often overlook words that seem common enough; 100,000 word lists often contain unusable noise).

(The repository [github.com/vandinem/jumble](https://github.com/vandinem/jumble) contains these code samples, as well as the SQLite file of words)

### wordlistsql.py

```python

	import urllib.request
	import sqlite3

	# --------------------------------------------------------------------
	# Get a sqlite table ready for data

	database = "./puzzlewords.sqlite"

	try:
		conn = sqlite3.connect( database )
	except Error as e:
		print( e )
		exit()

	cursor = conn.cursor()

	# Creating table
	table = """CREATE TABLE IF NOT EXISTS wordlist( word VARCHAR( 30 ), sorted VARCHAR( 30 ), UNIQUE( word, sorted ) );"""
	cursor.execute( table )

	# --------------------------------------------------------------------
	# Get the words at https://www.mit.edu/~ecprice/wordlist.10000, and store
	# each word and its sorted version in our sqlite table.

	# url  = "https://www.mit.edu/~ecprice/wordlist.10000"
	url  = "http://www-personal.umich.edu/~jlawler/wordlist"
	file = urllib.request.urlopen( url )

	maxlen = 0;

	ctr = 0

	for line in file:
		decoded_line = line.decode( "utf-8" )

		if ( decoded_line[ 0 ] == '#' ): # skip comments
			continue

		word = decoded_line.rstrip()
		word = word.lower()

		letters  = list( word )

		sortword = "" . join( sorted( letters ) )

		# print( f"{ word }, { sortword }" )

		try:
			cursor.execute( "INSERT OR IGNORE INTO wordlist ( word, sorted ) VALUES ( ?, ? );", ( word, sortword ) )
		except:
			print( f"Failed INSERT to wordlist (DB open in browser?): { word }, { sortword }" )
			conn.close()
			exit()

		if ( len( word ) > maxlen ):
			maxlen = len( word )

		ctr = ctr + 1


	conn.commit()

	conn.close()

	print( f"Read the page and found { ctr } words and max length is { maxlen }" )

```

With this database available, we can write a terminal script for solving words:

### jumble.py
```python

	import sqlite3
	import re

	database = "./puzzlewords.sqlite"

	try:
		conn = sqlite3.connect( database )
	except Error as e:
		print( e )
		exit()

	cursor = conn.cursor()

	while True:

		str = input( "\nEnter a Jumble word? " )

		str = str.lower()

		if ( str == 'q' ):
			print( "\n" )
			break
		else:

			str = re.findall("[\da-z]*", str)[0]

			letters  = list( str )

			str = "" . join( sorted( letters ) )

			if ( len( str ) < 3 ):
				print( "Come on guy ... at least three letters?" )

			cursor.execute( f"SELECT word FROM wordlist WHERE sorted = '{ str }'" )

			rows = cursor.fetchall()

			if ( len( rows ) == 0 ):
				print( "No solutions found!" )
				continue

			for row in rows:
				print( f"{ row[ 0 ] }" )

```

Or copy the puzzlewords.sqlite file into your PHP web site, and you can use an HTML form to interact with it. (No doubt something similar will work with Javascript, coding language of the damned.)  For PHP, make sure that the sqlite interface is installed.

### jumble.php
```php

	<!DOCTYPE HTML>
	<html lang="en">
	<head>
	    <title>Solve the Jumble</title>
	</head>
	<body>

	    <form name="form1" method="post" action="<?php echo $_SERVER["PHP_SELF"]; ?>">
			<strong>Enter 'Jumbled' word:</strong><BR/>

			<input 
				name="scrambled" 
				type="text" 
				style="width: 80px; padding: 2px; border: 1px solid black" 
				value="<?php echo $scrambled; ?>"
			>
			<input 
				type="submit" 
				name="Submit" 
				value="Unscramble!"
			>

		</form>

		<?php

		    if ( isset( $_POST[ "scrambled" ] ) && ( strlen( $_POST[ "scrambled" ] ) > 0 ) ) {

				process_jumble( $_POST[ "scrambled" ], 1 );

			}
			else {

				jumble_form( "elbmuj" , 0 );

			}

		?>

	</body>
	</html>

	<?php 

		function jumble_form( $scrambled, $flag ) {

			global $wordlist;

			if ( $flag ) {

				$ctr = count( $wordlist );

				if ( isset( $_POST[ "scrambled" ] ) && ( $ctr == 0 ) ) {

					print "<strong>Nothing found!</strong><BR>";

				}
				else {

					print "<BR/>";

					for ( $i = 0;$i < $ctr;$i++ ) {

						print ( $i + 1 ) . ": <strong>$wordlist[$i]</strong><BR/>";

					}

				}

			}

		}

		function process_jumble( $scrambledStr ) {

			global $wordlist;

		$db = new SQLite3( './puzzlewords.sqlite', SQLITE3_OPEN_READONLY);

			$wordlist = array();

			$chars  = str_split( strtolower( $scrambledStr ) );

		sort( $chars );

		$sorted = join( "", $chars );

			// Get any rows from the wordlist with a matching 'sorted' key

			$res = $db -> query( "SELECT word FROM wordlist WHERE sorted=\"$sorted\" ORDER BY word" );

			while ( $row = $res -> fetchArray() ) {

				$wordlist[] = $row[ 0 ];

			}

		} 

		jumble_form( $scrambledStr, 1 );

	?>
```
