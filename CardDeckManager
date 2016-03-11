#!/usr/bin/env python2
#encoding: UTF-8

# Use a deck of cards and random pseudo-random selection to generate
# a strong second form of authentication
import random
from optparse import OptionParser


cards = ["A","K","Q","J",2,3,4,5,6,7,8,9,10]
suits = ["Clubs","Hearts","Spades","Diamonds"]
class CardDeck:
    def shift_cards(self,state,shift):
        packet = state[0:shift]

        # print "Packet: ",packet
        rpack = []
        while packet:
            n = packet.pop(len(packet)-1)
            rpack.append(n)
        new_state = state[shift:]+rpack
  
        return new_state
    
    # challenge is an ordered list of ints or strings representing ints
    # denoting the number of cards to shift each time
    def pick_challenge(self,length):
        chal = []
        for i in range(0,length):
            chal.append(str(random.randint(1,9))) # between 1 and 9 shifts for each position in the token length
        chal_str = ",".join(chal)
        return chal_str

    # challenge is an ordered list of ints or strings representing ints
    # denoting the number of cards to shift each time
    def solve_challenge(self,deck,challenge):
        overwrite = deck
        shifts = challenge
        expected = []
        for k in shifts:
            overwrite = self.shift_cards(overwrite,int(k))

            expected.append(overwrite[-1:][0])
        return [expected,overwrite]
    
    def save_state(self,state,saveto):
        with open(saveto,"w") as f:
            f.write("\n".join(state))
            f.close()

    def shuffle_deck(self,raw):
        mixed = []
        while raw:
            index = random.random.randint(0,len(raw)-1)
            mixed.append(raw.pop(index))
        return mixed

    def build_deck(self):
        output = []
        for v in cards:
            for s in suits:
                output.append("%s-%s" % (v,s))
        return output

    def load_deck(self,deckfile):
        #print "opening %s" % deckfile
        try:
            with open(deckfile,"r") as f:
                data = f.read()
                f.close()
            return data.split("\n")
        except:
            return False

if __name__ == "__main__":
    parser = OptionParser()
    parser.add_option("-l", "--loadState", dest="loadFile", default=False,
                      help="Load the staste from a file")
    parser.add_option("-i", "--init-deck", dest="reInit", default=False,
                      help="Reshuffle the deck and print output.")
    parser.add_option("-c", "--challenge", dest="chalStr", default=False,
                      help="Enter a challenge string like: 4,3,4,1 must be used with -l")
        
    (options, args) = parser.parse_args()
    
    cd = CardDeck()
    
    if options.reInit is not False:
        deck = cd.build_deck() # Build the eck of cards
        for i in range(0,3):
            deck = cd.shuffle_deck(deck) # shuffle the deck 10 times
        cd.save_state(deck,options.reInit)
        # print the deck so it can be human consumed
        for c in deck:
            print c
        print "Finished Initializing Deck"
        exit(0)
        
    elif options.loadFile:
        deck = cd.load_deck(options.loadFile)
        
        if options.chalStr:
            expect = cd.solve_challenge(deck,options.chalStr.split(","))
            print expect[0]
            exit(0)
    else:
        print "Need something to do, Boss. Try -h"
        exit(0)
    # Define password length
    pass_shifts = 4
    overwrite = deck
    expected = []
    chall = []
    for i in range(0,pass_shifts):
        shift_sz = random.random.randint(1,8) # randomly choose the number of cards to shift to the bottom
        overwrite = cd.shift_cards(overwrite,shift_sz)
        #print "New overwrite ",overwrite
        last_i = len(overwrite)-1
        crd = overwrite[last_i]
        print "Card to expect %s" % crd
        expected.append(crd)
        chall.append(str(shift_sz))
    print "Challenge: %s" % (", ".join(chall))
    print expected
    
