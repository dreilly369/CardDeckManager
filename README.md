# CardDeckManager
Handle using a Playing card deck for 2fa

## Motivation ##
A fantastic demo of the concept at a recent conference. For more on that read the blog posting relating to this code at
http://the-it-ninja.blogspot.com/2016/03/pick-card-playing-cards-as-trust-tokens.html

The idea being the unique ordering of a deck of cards can be sed to derive a seond form of authentication. A simple example is given in the blog post above so I wont repeat it here. Instead I will focus on the practical applications. The system operates in two states. The First state is a Python Class to be imported into any application that you want to require the challenge in.

The second state is a command line tool that servers a few different maintenance/testing functions. The most important of which is providing a command line solver for a given challenge on a given deck. More details on this below.

## Glossary
*Challenger* - The server side that wants verification of identity.

*Responder* - The person or system requesting authentication.

*Deck* - Either a physical deck of cards or a *.dck text file representing the current state of a deck of cards to the server.

*Card* - Any one of the suit-value pairs present in a 52 card deck of Bicycle playing cards.

*Shift* - The action of dealing cards face down from the top of the deck onto the table (reversing their order) then placing the remainder of the deck on top of the packet on the table. 

*Shift size* - The number of cards to move from top to bottom in the shift. A good min/max shift size would be 1/8

*Expected Card* - The card on the bottom of the server's deck after a given shift.

*Actual Card* - The card at the botton of the responder's deck after a given shift.

*Challenge length* - The number of shifts to occur in any given challeng. During Testing I have found 4 to be a good length but long is also possible.

*Challenge* - A string of numbers representing the shift sizes to make. An example challenge might be: 3, 2, 1, 6 which means do a shift of size 3 and input the resulting card, then do a shift of size 2 and input that card, the same for 1 card and 6 cards. 

*Response* - the collection of cards that represent the user's answers to all the shifts in the challenge. These can either be full card names, or abbreviated. For instnace the above challenge: 3,2,1,6 could generate the response: 4H,3D,KS,4C (4 of hearts, 3 of diamonds, King of Spades, 4 of Clubs).

*Expected Response* - the collection of cards that represents the server's expected response to a given challenge string. 

##CardDeck Class
The actual class that encompasses the logic is relatively simple. It does not store the deck state internally. Instead it leaves that up to your program. 

###Methods
These are the method descriptions for the class. They should give you a better idea how you can implement this in your own project. 

*shift_cards(self, List state, Integer shift)* - Perform the shift operation on a given shift size. Returns a liust representing the new state of the cards.

*pick_challenge(self, Integer length)* - Get a new Challenge String of a given length. Returns a string of integers each one between 1-9 to represent the shift sizes of the challenge.

*solve_challenge(self, List deck, List challenge)* - Given a deck (as a list) and a challenge (as a list) produce a list of cards that satisfies the challenge. Returns a set which consists of the response and the new deck state as so [resonse, new_state]

*save_state(self, List state, String saveto)* - Saves the given state to the given file. Really just a convienience method. Used after initialization or a successful authentication. No return.

*shuffle_deck(self,List raw)* - Given a raw deck state, randomly pop cards out into a new pseudo-randomly ordered list. Shuffling will break the synchronicity of two Decks if a shuffle is applied to either of the decks. Good for the initialization phase though.  Returns a list of strings representing the newly shuffled order of the deck. 

*build_deck(self)* - Builds an unshuffled default deck from a suits list and a values list that is the product of the two lists. 4 suits * 13 values = 52 unique cards. Used to start a new deck in the initialization phase. Returns a List of strings representing the raw unshuffled deck of cards (similar to opening a brand new deck, but not the same ordering as that).

*load_deck(self, String deckfile)* - Loads a deck into a list from a given file. It does not store the deck internally, though it leaves that to your program. Returns a List of strings representing the saved state of a deck of cards in top down ordering. used after a deck is created to maintain synchronicity between to deck files (or a deck file and a physical deck)

## Creating dck files
This is pretty simple. From the command line just do:

    python CardDeckManager.py -i username.dck
Now a .dck file exists with the given name. Anyone with a matching setup physcal deck of cards can manually solve the challenges. The deck itself records the state. It is easy to get out of synch because of human error using a single file and physical deck method. That has lead me to explore using two files and programatically keeping them in synch.
So I can alternately use the commands: 

    python CardDeckManager.py -i username.dck && cp username.dck username-usercopy.dck
Which creates two copies of the key. I just copy that file to a second machine. Now both machines are in synch.

    python CardDeckManager.py -i username.dck && scp username.dck ssh-login@remote-system:~/username-usercopy.dck
Which creates the File and immediately passes a copy to a remote machine using th secure copy (scp) program.

Finally, you can call this in a loop from your own script to genrate batches of .dck files at a time.

## Stand alone vs. Import
using the CardDeckManager as a stand alone app allows you to use it from most server side languages. You can also use it to do maintenance tasks like I mentioned above. If you want to use the application in a stand-alone way but stil lcall it programmatically I recommend cleaning up the debug output some. Importing the class into your own Python applications is the most powerful way to use the CardDeckManager. You can use the functions in the main part as a guide to how you might structure your code, but a high level overview is given below.

## Work flow for adding a Card Deck Challenger to your Application.
1. Import CardDeck into your application
2. Create a .dck file for each identity you want to authenticate
3. Add calls from your current pre-authentication code to *pick_challenge* and display this to the user. Save the challenge so you can look it up during authentication. I use .chl files, but you can easily write a sqlite db or similar to do this.
4. Add a response entry form so users can respond to challenges
5. Add calls in your authentication code to *load_deck* and *solve_challenge* to generate the expected response
6. compare the response to the expected response
5. Add calls to your post authentication code to handle saving the new deck state in the even of a successful authentication. A failed login should remove the previous challenge, but leave the deck state uneffected.


## Workflow as a Client Side Solver
Since the application has the ability to solve supplied challenges, it can be used in place of a physical deck of cards with a copy of the .dck file on a second system. From the command line you can call:

    python CardDeckManager.py -l username.dck -c 1,2,3,4

Which will print the expected response to the console. You can then enter this response into the application's response form and be on your way.This can even be set to run on a Microcontroller like a middle tier Arduino.  (32u4 chip or above). I am currently testing an even smaller purpose built Hardware thast will allow you to load a .dck file onto an MicroSD and use that to solve from.

Alternately you can add the client-side solver functionality to your program simply by copying the .dck file to use onto the client machine. Once that is available you can follow this pattern:

1. Add calls in your authentication code to *load_deck*.
2. Add a method to input challenges.The need to be split into individual shifts so using a list or an easily tokenized string is a good option.
3. Add calls in your authentication code to *solve_challenge*.
4. Add code to update the .dck file after a successful login to maintain synchronictity with the server's deck.
