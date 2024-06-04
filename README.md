# TC2037 - Evidence 2
## Generating and Cleaning a Restricted Context-Free Grammar for Spanish
### Overview of the Spanish Language

The spanish language is a romance language that is mainly spoken in Spain and in the Americas. Spanish was originated from the a Vulgar Latin of the Roman soldiers on the Iberian peninsula around the the 3rd century A.D. The language is characterized by its subject-verb-object order, although it can be flexible fue to the inflectional systema. This allows various syntactical constructions without loss of clarity. 

### Key Elements of Spanish Syntax
- Nouns: Classified as masculine or feminine, and can be singular or plural (e.g., "el" for masculine singular, "los" for masculine plural).
- Adjectives: Must agree with the nouns they describe in terms of gender and number and usually follow the nouns.
- Prepositions: Utilized to express relationships such as location, direction, or time (e.g., "a", "ante", "bajo").
- Sentence Structure: Typically uses an SVO order but is quite flexible to emphasize different parts of the sentence.

### Grammar Specifications

A grammar includes a set of rules from which we can derive strings. These rules are effectively statements of logical equivalence of the form: ψ → ω, where ψ and ω are strings.

Here is a very simple example of a grammar to generate the string "the dog saw a man in the park": 

```mathematica
S -> NP VP
NP -> Det N | Det N PP
PP -> P NP
VP -> V NP | VP PP
Det -> 'the' | 'a'
N -> 'man' | 'dog' | 'park'
P -> 'in' | 'with'
V -> 'saw' | 'ate' | 'walked'
```


### Terminal Elements:
- Verbs: correr, saltar, comer, jugar.
- Adjectives: alegre, triste, pequeño, alto.
- Adverbs: rapidamente, ayer, hoy, mañana.
- Conjuctions: pero, y, porque.
- Prepositions: a, ante, bajo, con, contra.
- Pronouns: el, ella, los, las, un, una. 

### Non-Terminal Elements
These symbols have been expanded to handle more grammatical structures:
- S (Sentence)
- NP (Noun Phrase)
- VP (Verb Phrase)
- PP (Prepositional Phrase)
- AP (Adjective Phrase)
- AdvP (Adverb Phrase)

## Create a grammar that recognizes Spanish language
```mathematica
S (Sentence) -->  NP VP | S Conj S | S PP
NP (Noun Phrase) --> Det N | Det AP N | Pron | NP PP | NP Conj NP
VP (Verb Phrase) --> V NP | V AP | V AdvP | V PP | VP Conj VP
PP (Prepositional Phrase) --> P NP
AP (Adjective Phrase) --> Adj | Adj AP
AdvP (Adverb Phrase) --> Adv | Adv AdvP
Conj (Conjunction) --> 'y' | 'o' | 'pero' | 'porque'
Det (Determiner) -->  'el' | 'la' | 'los' | 'las' | 'un' | 'una'
N (Noun) --> 'niño' | 'niña' | 'libro' | 'ciudad'
P (Preposition) --> 'en' | 'sobre' | 'bajo' | 'contra'
Pron (Pronoun) --> 'él' | 'ella' | 'ellos' | 'ellas' | 'lo' | 'la' | 'le'
V (Verb) -->  'corre' | 'salta' | 'piensa' | 'cree' | 'ha'
Adj (Adjective) --> 'alegre' | 'triste' | 'grande' | 'pequeño'
Adv (Adverb) -->  'rápidamente' | 'lentamente' | 'ayer' | 'hoy'
```

### Challenges in Grammar
#### Eliminating Ambiguity
Ambiguity arises when a string has multiple valid parse trees. This is resolved by refining recursive rules and clearly defining the use of adjectives and adverbs.

```mathematica
AP --> Adj | Adj AP
AdvP --> Adv | Adv AdvP
```

#### Handling Left Recursion
Left recursion occurs in a context free grammar when a non-terminal symbol includes itself as the first 
symbol in one of its productions, potentially leading to infinite recursion during the parsing, which most 
parsers can't handle. 

Original:
```mathematica
S --> S Conj S | NP VP
```
Transformed to eliminate left recursion:
```mathematica
S --> NP VP S' 
S' --> Conj NP VP S' | ε
```
Define Acceptable Strings
The grammar like this should accept strings such as:
- "El niño alegre corre rápidamente."
- "Ella cree que la ciudad grande es hermosa."
- "Él y ella están en la ciudad."

Invalid strings might include:
- "Corre el niño rápidamente." (Incorrect order)
- "Ella cree rápidamente." (Semantic mismatch)



```python
# Import the nltk module and CFG class for working with context-free grammars
import nltk
from nltk import CFG

# Define a context-free grammar using a multiline string.
grammar = CFG.fromstring("""
  S -> 'NP' 'VP' S'
  S' -> 'Conj' 'NP' 'VP' S' | ''
  NP -> 'Det' 'N' | 'Det' 'AP' 'N' | 'Pron' | 'NP' 'PP' | 'NP' 'Conj' 'NP'
  VP -> 'V' | 'V' 'NP' | 'V' 'PP' | 'VP' 'Conj' 'VP'
  PP -> 'P' 'NP'
  AP -> 'Adj' | 'Adj' 'AP'
  Det -> 'el' | 'la' | 'los' | 'las' | 'un' | 'una'
  N -> 'niño' | 'niña' | 'libro' | 'ciudad'
  P -> 'en' | 'sobre' | 'bajo' | 'contra'
  Pron -> 'él' | 'ella' | 'ellos' | 'ellas' | 'lo' | 'la' | 'le'
  V -> 'corre' | 'salta' | 'piensa' | 'cree' | 'ha'
  Adj -> 'alegre' | 'triste' | 'grande' | 'pequeño'
  Adv -> 'rápidamente' | 'lentamente' | 'ayer' | 'hoy'
  Conj -> 'y' | 'o' | 'pero' | 'porque'
""")

# Create a parser using the defined grammar
parser = nltk.ChartParser(grammar)

# List of sentences to parse and validate against the grammar
sentences = [
    "El niño alegre corre rápidamente",
    "Ella cree que la ciudad grande es hermosa",
    "Él y ella están en la ciudad",
    "Corre el niño rápidamente"
]

# Validate sentences using the parser
for sentence in sentences:
    parsed = False
    print("\nTrying to parse:", sentence)
    # Attempt to parse each sentence and generate parse trees
    for tree in parser.parse(sentence.split()):
        print(tree)
        parsed = True
    # If no parse trees were generated, the sentence is invalid according to the grammar
    if not parsed:
        print("Sentence is invalid.")

```
