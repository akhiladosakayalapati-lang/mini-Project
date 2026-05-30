import nltk
from nltk.corpus import brown
from collections import Counter

nltk.download('brown')

word_list = [w.lower() for w in brown.words()]

WORD_FREQ = Counter(word_list)
TOTAL = sum(WORD_FREQ.values())

def probability(word):
    return WORD_FREQ[word] / TOTAL if word in WORD_FREQ else 0

def edits1(word):
    letters = 'abcdefghijklmnopqrstuvwxyz'
    splits = [(word[:i], word[i:]) for i in range(len(word)+1)]
    
    deletes = [L + R[1:] for L, R in splits if R]
    inserts = [L + c + R for L, R in splits for c in letters]
    replaces = [L + c + R[1:] for L, R in splits if R for c in letters]
    transposes = [L + R[1] + R[0] + R[2:] for L, R in splits if len(R) > 1]

    return set(deletes + inserts + replaces + transposes)

def edits2(word):
    return set(e2 for e1 in edits1(word) for e2 in edits1(e1))
    
def known(words_set):
    return set(w for w in words_set if w in WORD_FREQ)
def candidates(word):
    return (
        known([word]) or
        known(edits1(word)) or
        known(edits2(word)) or
        [word]
    )
def correct(word):
    return max(candidates(word), key=probability)

def autocorrect(sentence):
    return " ".join(correct(word.lower()) for word in sentence.split())

if __name__ == "__main__":
    text = input("Enter a sentence: ")
    print("Corrected:", autocorrect(text))
