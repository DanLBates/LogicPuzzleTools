# logicpuzzletools Package
last change 20-Mar-2022
### An collection of tools to aid in solving logic puzzle problems with Python 3.x .
This does not use sophisticated Artificial Intelligence methods. Instead it helps to generate
permutations and tools to filter these permuations. Finally, a ready made way of outputing 
those solution(s).
### Author: Daniel L. Bates	(dan.bates)
## The pypi module logicpuzzletools
## What kind of logic puzzle problems?
The kind of logic puzzle problem the tools help solve are also called logic grid puzzles.
The logic grid puzzle has two or more categories. Each category is a set of something, Often one
category is people with a set of names, although this is not always true. Each set has the same
cardinality. 

The elements are sometimes the same as the formulation. In this set of tools they must be unique.
If the problem has 5 girls who are friends each naming there pet after one of there 4 friends you
have to go through the gymnastics of making them different. For instance 'diane' and 'diane_pet'.

You can always find one category not to permute. This is our fixed category. It is good to pick some
category which has some order to it, like house numbers or floor numbers in a hotel (yes you can leave
out floor 13).

Categories are global or nonlocal such that all the solution code has access to them.

There are also a set of rules describing relationships between elements of disparate (or the same)
category. The are commonly solved with the upper left of a matrix which
includes the lower left to upper right diagonal. In the columns typically one category is left out.
In the rows typically a different category is left out.
The set element names label the rows and columns.

!(see[A Simple example from the Brainzilla website.](https://www.brainzilla.com/logic/logic-grid/superheroes/))

### The Einstein puzzle.
On December 17, 1962 *Life International* published a logic grid puzzle called "Who owns the zebra". This has
been called the Zebra puzzle. The problem was allegedly invented by Albert Einstein, so it is also
called the Einstein puzzle. A more recent incarnation of the problem, without the Zebra is stated here:

| Categories | House #s | colors | nationalites | drinks | Pets  | Smokes      |
| ---------- | -------- | ------ | ------------ | ------ | ----  | ------      | 
|  indexes   |    1     | yellow |    swede     | beer   | fish  | bluemasters |
|            |    2     | red    |    dane      | coffee | birds | dunhills    |
|            |    3     | green  |    german    | milk   | horses| princes     |
|            |    4     | blue   |    brit      | tea    | cats  | blends      |
|            |    5     | white  |    norwegian | water  | dogs  | pallmalls   |

Rules:
```
    There are five houses in a row in different colors.
    In each house lives a person with a different nationality.
    The five owners drink a different drink,
    smoke a different brand of cigar
    and keep a different pet, one of which is a Walleye Pike.
    
     1. The Brit lives in the red house.
     2. The Swede keeps dogs as pets.
     3. The Dane drinks tea.
     4. The green house is on the left of the white house.
              4. has to be interpreted as immediatly to the left of
              or there is more than one solution
     5. The green house owner drinks coffee.
     6. The person who smokes Pall Malls keeps birds.
     7. The owner of the yellow house smokes Dunhills.
     8. The man living in the house right in the center drinks milk.
     9. The man who smokes Blends lives next to the one who keeps cats.
    10. The Norwegian lives in the first house.
    11. The man who keeps horses lives next to the one who smokes Dunhills.
    12. The owner who smokes Bluemasters drinks beer.
    13. The German smokes Princes.
    14. The Norwegian lives next to the blue house.
    15. The man who smokes Blends has a neighbor who drinks water.
              15. has to be interpreted as lives adjacent to.  or there
              is more than one solution.
    The question is-- who owns the fish?
```

The code I give you to solve this does not use a grid. It is instead a
constraint satisfaction method. This generates permutations, Not all permutatations are completly examined
as some are pruned. There are permutations for each category. Elements are related to
elements of different categories by there index in their own category.

These permutations are taken through filters which are the rules.

Because position is important for each category we use lists instead of sets to represent them.

Here is the code using logic_puzzle_tools for the Einstein problem:
```
from logic_puzzle_tools import *
...
def einstein ():
    '''
    Einstein puzzle

    There are five houses in a row in different colors.
    In each house lives a person with a different nationality.
    The five owners drink a different drink,
    smoke a different brand of cigar
    and keep a different pet, one of which is a Walleye Pike.
    
     1. The Brit lives in the red house.
     2. The Swede keeps dogs as pets.
     3. The Dane drinks tea.
     4. The green house is on the left of the white house.
              4. has to be interpreted as immediatly to the left of
              or there is more than one solution
     5. The green house owner drinks coffee.
     6. The person who smokes Pall Malls keeps birds.
     7. The owner of the yellow house smokes Dunhills.
     8. The man living in the house right in the center drinks milk.
     9. The man who smokes Blends lives next to the one who keeps cats.
    10. The Norwegian lives in the first house.
    11. The man who keeps horses lives next to the one who smokes Dunhills.
    12. The owner who smokes Bluemasters drinks beer.
    13. The German smokes Princes.
    14. The Norwegian lives next to the blue house.
    15. The man who smokes Blends has a neighbor who drinks water.
              15. has to be interpreted as lives adjacent to.  or there
              is more than one solution.
    The question is-- who owns the fish?
    '''
    logic_puzzle_setup()
    houseposition = [1, 2, 3, 4, 5] # This is fixed Left:1 2 3 4 5:Right
    # these belong to scope of einstein
    nationalities = [None] * 5
    drinks        = [None] * 5
    housecolor    = [None] * 5
    pets          = [None] * 5
    smokes        = [None] * 5

    def choose_nationalities():
        nonlocal nationalities
        permute_nationalities = permute('norwegian, brit, swede, dane, german', nationalities)
        for p in permute_nationalities:
            category(nationalities, p)
            if fails(is_associated('norwegian', 1)):                    #R10
                continue
            choose_drinks()

    def choose_drinks ():
        nonlocal drinks
        permute_drinks = permute('coffee, milk, beer, tea, water', drinks)
        for p in permute_drinks:
            category(drinks, p)
            if fails(is_associated('dane', 'tea')):                     # R3
                continue
            if fails(is_associated('milk', 3)):                         # R8
                continue
            choose_housecolor()

    def choose_housecolor ():
        nonlocal housecolor
        permute_housecolor = permute('blue, green, white, red, yellow', housecolor)
        for p in permute_housecolor:
            category(housecolor, p)
            if fails(is_associated('brit', 'red')):                     # R1
                continue
            if fails(is_just_left_of('green', 'white')):                # R4
                continue
            if fails(is_associated('green', 'coffee')):                 # R5
                continue
            if fails(is_adjacent_to('norwegian', 'blue')):              # R14
                continue
            choose_pets()
            
    def choose_pets ():
        nonlocal pets
        permute_pets = permute('fish, birds, dogs, horses, cats', pets)
        for p in permute_pets:
            category(pets, p)
            if fails(is_associated('swede', 'dogs')):                   # R2
                continue
            choose_smokes()

    def choose_smokes ():
        nonlocal smokes
        permute_smokes = permute('blends, pallmalls, bluemasters, dunhills, princes', smokes)
        for p in permute_smokes:
            category(smokes, p)
            if fails(is_associated('pallmalls', 'birds')):              # R6
                continue
            if fails(is_associated('yellow', 'dunhills')):              # R7
                continue
            if fails(is_adjacent_to('blends', 'cats')):                 # R9
                continue
            if fails(is_adjacent_to('horses', 'dunhills')):             # R11
                continue
            if fails(is_associated('bluemasters', 'beer')):             # R12
                continue
            if fails(is_associated('german', 'princes')):               # R13
                continue
            if fails(is_adjacent_to('blends', 'water')):                # R15
                continue
            a_solution()

    def a_solution ():
        # a_solution. If not enough constraints, there will be more
        # than one. If too many constraint, there will be none.
        fmt1 = '%7s %11s %8s %11s %8s %12s'
        fmt2 = '%7d %11s %8s %11s %8s %12s'
        print(fmt1%('house #','nationality','drinks','housecolor','pet','smokes'))
        print('=' * 70)
        PrintSolution(fmt2, houseposition, nationalities, drinks, housecolor, pets, smokes)
        # who owns the fish
        print('the', nationalities[idx('fish')],'owns the fish, (the Walleye Pike)')

    fixed_cat(houseposition)
    choose_nationalities()  # einstein() back to the first function at the top
```
The soloution produced is:
```
house # nationality   drinks  housecolor      pet       smokes
======================================================================
      1   norwegian    water      yellow     cats     dunhills
      2        dane      tea        blue   horses       blends
      3        brit     milk         red    birds    pallmalls
      4      german   coffee       green     fish      princes
      5       swede     beer       white     dogs  bluemasters
the german owns the fish, (the Walleye Pike)
```
## First step if you are solving more than one puzzle.

If you are going to solve more than one puzzle, you must call
logic_puzzle_setup() for all but the first one. It will do no harm
to call this for the first puzzle as well.

The form this takes on is to loop over permutations making each permution
the category. The body of the loop tests such criteria as can be determined
by the categories that have been defined. When a rule has been broken we
continue with the next permutation. At the very end of the loop we bring another
category into play (most easily by calling another function). Rather than
actually negate each rule there is a function fails(rule) which make the
code read better but does very little. The fails(rule) is equivalent to
not (rule). This lets the code match the verbage of the rule.

The means of making a permutation is by calling the function 
category(cat : list, permutation : list). This function keeps all the
references to that particular category valid. This does an element by element
copy all the elements of permutation to the elements of category.

There is always one category you do not permute. To set the fixed category
you call the following:

fixed_cat (cat: list) -> None

Each item in cat is linked back to category cat through a dictionary.

## Making a generator for the permutations of each category.
The tool permute uses itertools.permute to create a generator for a category
and to return it. All the categories except one need to be permuted. The generator
also associates element name with the category which contains it.

def permute (things : str, cat : list) -> list:

Return a generator which gives new permutations of things.  
And map each element to the category cat.

Example:	permute_housecolor = permute('blue, green, white, red, yellow', housecolor)

## Tools for filtering.

Unlike a much earlier version of this package, you do not ever need to
specify a category.

In most of the following rule related helpers the string arg1 and arg2 are elements. They
usually to different categorys. These are predicates which return True if the relationship
is True and False otherwise. The relationships are for the indexes of the elements within
there respective categories. All these should be thought of as the relationship by
the associated fixed category. These are:


#### is_associated (arg1 : str, arg2 : str) -> bool:
Returns True when arg1 is the same as arg2 like 'green' and 'coffee'.
#### is_not_associated (arg1 : str, arg2 : str) -> bool:
Returns False when arg1 is not the same as arg2 like 'green' and 'coffee'.

For is_associated and is_not_associated to make sense, arg1 and arg2 must
be in seperate categories. Ex: is_associated('green', 'green') will always be True and
is_not_associated('green', 'green') will always be False.

#### is_just_left_of (arg1 : str, arg2 : str) -> bool:
#### is_just_below (arg1 : str, arg2 : str) -> bool:
These two are identical. Use the one which best matches the verbage of the rule.
The truth of idx(arg2) - idx(arg1) == 1.
#### is_just_right_of (arg1 : str, arg2 : str) -> bool:
#### is_just_above (arg1 : str, arg2 : str) -> bool:
These are also identical. Pick what reads the best. The truth of idx(arg2) - idx(arg1) == -1. 
#### is_below (arg1 : str, arg2 : str) -> bool:
The truth of idx(arg1) < idx(arg2).
#### is_above (arg1 : str, arg2 : str) -> bool:
The truth of idx(arg1) > idx(arg2)
#### is_adjacent_to (arg1 : str, arg2 : str) -> bool:
The truth of abs(idx(arg1) - idx(arg2)) == 1.
#### is_not_adjacent_to (arg1 : str, arg2 : str) -> bool:
The truth of abs(idx(arg1) - idx(arg2)) != 1.
#### after (arg1 : str, arg2 : str) -> bool:
The same as is_above.
#### before (arg1 : str, arg2 : str) -> bool:
The same as is_below. Use the most natural form.
#### just_after (arg1 : str, arg2 : str) -> bool:
The same as is_just_right_of.
#### just_before (arg1 : str, arg2 : str) -> bool:
The same as is_just_left_of.

#### idx (element : str) -> int:
No matter how many primitives are provided here, a problem will arise where they
do not suffice. The function idx uses the category of element and returns the
index of element within that category. With this you can build the rule you need.

## Outputing a solution
To help output the solution a function PrintSolution (fmt_str, *ListSet) is provided.
the parameter fmt_str is a C printf style format string for one row.
ListSet are categories to be the columns. An example is the a_solution function
in the einstein code.

## Watch for:
This topic is discussed in more detail in a book I have written.
I will update this with information when the book is published.
