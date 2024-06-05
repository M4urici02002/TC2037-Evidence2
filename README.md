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
S --> NP VP SP
SP --> Conj NP VP SP | ε
```
Define Acceptable Strings
The grammar like this should accept strings such as:
- "el niño alegre corre rápidamente."
- "él y ella están en la ciudad."
- "la niña lee el libro"
- "ella lee lentamente"
- "la niña lee sobre la mesa"

Invalid strings might include:
- "corre el niño rápidamente."
- "la ciudad sobre el libro"
- "el niño salta pero es triste"

###Implementation
The implementation was made in python, using the library of NLTK that help me making a lot easier the way to present the syntax analisis of the Spanish language, the code is below and also add the google colab so it can be tested with valid or invalid sentences

```python
# Import necessary modules from nltk for natural language processing
import nltk
from nltk import CFG  # Context-Free Grammar module for defining grammars
from nltk.tokenize import word_tokenize  # Tokenization function to split text into tokens
from nltk.tree import Tree  # Import Tree for enhanced visual representation of parse trees
nltk.download('punkt')  # Ensure the 'punkt' tokenizer models are downloaded for tokenization

# Define the grammar using a multi-line string
# This grammar defines how sentences in Spanish can be structured syntactically
grammar = CFG.fromstring("""
  S -> NP VP SP 
  SP -> Conj S |  
  NP -> Det N | Det AP N | Pron | NP PP | NP Conj NP 
  VP -> V NP | V PP | VP Conj VP | V Adv | V S | V NP PP  
  PP -> P NP  
  AP -> Adj AP | Adj 
  Det -> 'el' | 'la' | 'los' | 'las' | 'un' | 'una' 
  N -> 'niño' | 'niña' | 'libro' | 'ciudad' | 'parque' | 'mesa' | 'cine'  
  P -> 'en' | 'sobre' | 'bajo' | 'contra' | 'al'  
  Pron -> 'él' | 'ella' | 'ellos' | 'ellas' | 'lo' | 'la' | 'le'  
  V -> 'corre' | 'salta' | 'piensa' | 'cree' | 'ha' | 'están' | 'van' | 'es'  
  Adj -> 'alegre' | 'triste' | 'grande' | 'pequeño' | 'interesante'  
  Adv -> 'rápidamente' | 'lentamente' | 'ayer' | 'hoy'  
  Conj -> 'y' | 'o' | 'pero' | 'porque' | 'que'  
""")

# Create a parser instance from the defined grammar using NLTK's ChartParser
parser = nltk.ChartParser(grammar)

# Function to parse and validate a sentence according to the defined grammar
def parse_and_validate(sentence):
    # Tokenize the input sentence using NLTK's word_tokenize, set to Spanish for correct tokenization of Spanish text
    tokens = word_tokenize(sentence, language='spanish')
    parsed = False  # Initialize parsed status as False
    print("\nSpanish sentence that is parsing:", sentence)  
    try:
        # Attempt to parse the tokenized sentence, generating parse trees
        for tree in parser.parse(tokens):
            tree.pretty_print()  # Visually display the parse tree using pretty_print
            parsed = True  # Set parsed status to True if at least one parse tree is generated
    except ValueError as error:
        print(error)  # Print any errors encountered during parsing

    if parsed:
        print("The sentence is valid.")  # Sentence conforms to the grammar
    else:
        print("Sentence is invalid.")  # No valid parse tree could be generated

# Interactive loop for user to input sentences for parsing
while True:
    user_input = input("Please enter a sentence 'exit' to quit: ")  # Prompt for user input
    if user_input.lower() == 'exit':  # Check if user wants to exit the program
        break  # Exit the loop and end the program
    parse_and_validate(user_input)  # Process the user input through the parse and validate function

```
### Analysis
The grammar developed for this project is classified as a Context-Free Grammar (CFG), which is positioned at Level 2 in the Chomsky Hierarchy. This classification is strategically chosen to balance complexity and computational efficiency, making it particularly suitable for parsing natural languages like Spanish.
