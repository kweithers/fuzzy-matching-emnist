# Fuzzy Matching Service

Employees write a six character ID on the physical application card, and this is scanned by a machine to collect the application information. For a small number of applications, the ID on an application does not match any valid employee ID. This is likely due to an error in the [Optical Character Recognition](https://en.wikipedia.org/wiki/Optical_character_recognition).

This repository is offers a solution to remedy this issue. There are several ways to go about finding the closest match to a string, also known as ["Fuzzy Matching"](https://en.wikipedia.org/wiki/Approximate_string_matching). This repository uses the [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance), which counts the number of edits needed to get from one string to another. An edit is either a *substitution, insertion, or deletion*.

For example, the strings 'AWJGFE' and 'AWJ6FE' have a Levenshtein Distance of 1. The only edit needed is to substitute the G for a 6. 

### Finding the best match

For each application with an unmatched ID, we calculate the Levenshtein Distance between the ID on the application and every possible employee ID. 

In some cases, there is a single record with Levenshtein Distance of 1. We choose this as the best match.

If there is not a match with a distance of 1 or 2, we say that this record has no match. This is likely not just a simple character recognition problem.

### Tiebreaker 

In several cases, there are a few (2-4) matches with a Levenshtein Distance of 1. 

For example, the unmatched string 'AWWYCH' has two possible matches: 'AWWTCH' and 'AW3YCH'. We either must substitute a T for a Y, or substitute a 3 for a W. **How do we decide between these options?**

Several characters have another that are very similar: O and 0, I and 1, B and 8, etc. We need a measure of *character similarity* to determine the best match in these cases.

##### Character Similarity 

To generate some measure of character similarity, we use the [EMNIST Dataset](https://www.nist.gov/itl/products-and-services/emnist-dataset). This contains thousands of images of each alphanumeric character. 

We next perform some dimensionality reduction on the data. This allows us to find linear combinations of features (in this case, pixels) that are meaningful representations of the dataset. In this case, we use [Singular Value Decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition) to do this. For this problem, 100 feature components are kept. 

Once we have these new features generated by SVD for each record, we can take the average value for each feature grouped by each character. That is to say, we find the average of these features for each character.

Now, each character is represented by a 100 feature vector. The distance between these vectors represents character similarity. In two dimensions (using the two features with the largest singular values), it looks like this:

![alt text](https://github.com/kweithers/FuzzyMatchingService/blob/master/CharacterSimilarity.png)

Going back to the previous example, we must decide between 'AWWTCH' and 'AW3YCH' for the unmatched string 'AWWYCH'. 'T' and 'Y' are closer in similarity than '3' and 'W', so we chose the best match as 'AWWTCH'.

We now have a programmatic way to determine the best match in the case of a Levenshtein Distance tie.