# Probabilistic Record Linkage Model for Entity Resolution

##### Intent: 

Our objective is to facilitate the creation and upkeep of a `person entity` table. </br>
This entails distinguishing between records in incoming files that correspond to individuals already present in our table and those that denote the creation of a new person entity.

##### Approach:

Without a unique identifier, accurately categorizing pairs of records as belonging to the same entity becomes difficult. We utilize `probabilistic record linkage`, a method that leverages non-unique variables such as name and address to assess the likelihood that a pair of records corresponds to the same entity. </br>
This approach considers supporting and opposing evidence for a match, assigning appropriate weights to each factor. The term "probabilistic" underscores the inherent uncertainty of this process, which relies on evaluating the overall balance of evidence.</br>

We implement the `Fellegi-Sunter Probabilistic Linkage Model`. This model was introduced in Ivan Fellegi and Alan Sunter's seminal 1969 [paper](https://courses.cs.washington.edu/courses/cse590q/04au/papers/Felligi69.pdf).




 🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  **THEORY**  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖  -  🤖 


##### problem statement:  
Consider two samples, A and B, denoted by their respective elements a and b. It is assumed that A and B originate from the same population albeit with errors and incompleteness due to factors such as sampling methods or timing.

A and B represent the union of two disjoint sets:

`Matched Set (M): {(a,b), a = b, a ∈ A, b ∈ B}`</br>
`Unmatched Set (U): {(a, b); a ≠ b, a ∈ A, b ∈ B}`

Our task is to classify each record pair (a,b) as belonging to either the Matched Set (M) or the Unmatched Set (U).



##### comparison space: (Γ)
Γ represents the set containing all pairwise comparisons between elements of sets A and B. A significant hurdle in record linkage arises from the scale of the problem, as the size of Γ increases roughly quadratically with the number of records. This necessitates substantial computational resources and incurs high costs. To mitigate this challenge, we employ a method known as "blocking" to confine Γ to a subset Γ', where pairs outside the block are deemed Not Matched.

##### comparison vector: (γ)
The comparison vector (γ) serves as the metric of similarity, representing the evidence that our model evaluates when determining whether a pair of records constitute a match.</br> 
For each pair (a, b) within Γ, we create a comparison vector γ[a,b], which comprises various comparisons and their corresponding levels (e.g., 'first name: exact match'). The decision function \\(d(γ)\\) then assesses the evidence contained within each comparison vector to make a prediction regarding whether a pair qualifies as a "match".

##### match weights: ω
The "partial match weight" (\\(ω_i\\)) quantifies the relative importance of each piece of evidence \\(γ_i\\) in influencing decision d(γ)

The partial match weight \\(ω_i\\) is derived from the conditional probabilities of \\(γ_i\\)
* m(γ): the probability of observing \\(γ_i\\) given the records are a match, calculated as the sum of the product of the probability of observing each comparison element P(γ[a, b]) and the conditional probability of the match P((a,b)|M). 
  * P(γ_i|Match) `m(γ) = Σ P(γ[a, b])⋅ P((a,b)|M)`
* u(γ): the probability of observing \\(γ_i\\) given the records are not a match, calculated as the sum of the product of the probability of observing each comparison element P(γ[a, b]) and the conditional probability of the non-match P((a,b)|U).
  * P(γ_i|Not a Match) `u(γ) = Σ P(γ[a, b])⋅ P((a,b)|U)`

We are interested in the relative size of m and u (\\(\frac{m}{u}\\)) aka the Bayes Factor (K). The partial match weight (\\(ω_i\\)) is a log transformation of K to make it additive: ω = log₂(K). The final match weight is the sum of the individual partial match weights.


The final prediction is made by combining the final match weight (ω) with the prior match weight (\\(ω_λ\\)) and converting it into a probability using a logistic transformation. (??)
##### probability two records are a match
We calculate the posterior probability that two records (a,b) are a match given evidence γ using Bayes' Theorem.</br>
`Posterior probability`: the updated belief that (a,b) are a match given γ, calculated as the product of the likelihood and the prior, divided by the evidence.
\\(\frac{likelihood \bullet prior}{evidence}= \frac{m \bullet λ}{P(γ)} = \frac{m \bullet λ}{m \bullet λ + u \bullet (1-λ) }\\)
* `Prior probability` (λ): the probability that two records drawn at random are a match.
* `Likelihood` (m): the probability of observing the evidence γ given that the records are a match.





Model Assumptions:
1. (a, b) are randomly selected from population A × B, therefore the comparison vector γ[a, b] is a random variable
2. the attributes we are comparing are independent conditional independence between attributes, also known as naive Bayes.
