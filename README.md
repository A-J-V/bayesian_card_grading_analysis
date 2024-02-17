# A Bayesian Analysis of Pokemon Card Grade Stability

## Synopsis

Among trading card enthusiastics, it is common to send cards to a company to have the card examined and assigned a grade based on the evaluated condition as a paid service. PSA, Professional Sports Authority, is the most popular and prestigious grading company (by far).
Cards are typically graded on a scale of 1 - 10, and cards that receive the highest grades very often see a substantial boost in market value compared to cards that are either ungraded or have been assigned a low grade.
As an example, a 1st Edition Alakazam from the Pokemon Trading Card Game Base Set has a market value, as of early 2024, of approximately $900 when graded as a PSA 8, $1900 as a PSA 9, and $19000 as a PSA 10.

The extreme differences in value between these discrete labels, and the fact that the grades are ultimately subjective judgement calls made by PSA staff through a process with virtually no transparency, has raised concerns over the years.
The two most frequently expressed concerns can be posed as two questions:
* Does PSA intentionally suppress the number of rare cards graded at a 10 in order to artificially boost their value and therefore the perceived prestige of the company?
* Are grades stable and assigned via well-defined and highly controlled criteria, or are they largely subjective and the result of the whims of whoever graded them?

The first concern has a lot of believers and a lot of individuals who label it a conspiracy theory. A popular YouTube video in [year] claimed that the proportion of PSA 10s among highly valuable cards, such as Derek Jeter, are lower that the proportion of PSA 10s overall, therefore PSA does engage in "population control."
This method is pretty easily dismissed as invalid due to selection bias--very rare and valuable cards such as the Derek Jeter card are more likely to be sent in for grading regardless of the cards position, while less valuable and easily obtained cards are typically only sent in if the owner is confident that they will receive an excellent grade.
As a result, there is severe selection bias in terms of which cards get sent in for grading, and it is very reasonable to expect a priori that more rare and valuable cards in poor condition will be sent in than all other cards in poor condition.
A controlled test of this is infeasible due to the cost of obtaining such rare and valuable cards.

However, the second question, which to my knowledge has never been explicitly tested, is more tractable. This project attempts to answer the question by providing a robust estimate of how stable PSA's grades are.

## Methodology

To address the question of how stable PSA's grades are, I conducted a blinded repeated-measures study of PSA's grading habits and analyzed them using a Bayesian Hierarchical Model.

The study used repeated-measures by taking the same group of graded cards, cracking their protective slabs, then sending them back in for grading. We therefore receive multiple grades on the same cards and are able to see if they change.

The study is single-blinded in the sense that the staff who are assigning the grades are not aware that the cards have actually already received grades previously. This should eliminate any bias due to the grader having a previous grade to reference for the cards.

There were a total of 40 Wizards of the Coast era Pokemon cards used in this study, each card receiving 3 grades. It is important to note that this gives us some weak estimates of the variability in grades within each card, but not an overall estimate as to the stability of the company's grades overall.

We should also consider that it is reasonable to expect that different cards exhibit different grade stability. That is to say, some cards may be "obvious 9s" while others are harder to pin down and may lean more on the subjectivity of the grader.

In order to take these facts into consideration and integrate the information spread across all of the cards' grades into a single estimate about the company's overall stability, a Bayesian Hierarchical Model was created. For simplicity, I selected "probability of grade change," P, as the estimand. This variable is as it sounds--the probability that a card's grade will change if it is sent in for a blind regrade. We allow each card to have it's own P. We model the regrading process as a binomial variable where the number of regrades is n and the probability of the event is P. We model P as itself being a random variable sampled from a Beta distribution representing our uncertainty about the company's overall probability of grade change. By using Bayesian inference, we can assign some appropriate priors and estimate the posterior of the company's "grade change distribution" after taking the collected data into consideration.

### Selecting Priors
As we are using Bayesian inference, we need to set priors, specifically:
* Each card's observations are modeled using a Binomial distribution. The P parameter of each card's distribution requires a prior.
* Each P parameter will be sampled from the same prior, a Beta distribution that we choose to represent the company's overall uncertainty around grade changes.
* This Beta distribution is reparameterized from the usual alpha and beta parameters to mean and concentration parameters. These are hyperparameters, and they require hyperpriors.
* I select the hyperprior on the mean hyperparameter to be Beta(3, 6), and I select the hyperprior on the concentration hyperparameter to be Normal(12, 2).

Since there are so many different terms flying around in Bayesian inference, it's good to pause and consider what, exactly, we actually care about determining. The primary goal of this project is to estimate the probability of a card changing grade at the company level. The company level is represented as a Beta distribution, the prior for each card's P. That prior has a hyperprior distribution that represents our uncertainty around it's mean. If we had to pick one focus of this project, it's that, because it is an estimate of the company's central tendency regarding their grade changes. Secondarily, the concentration hyperprior is worth considering since it represents the dispersion.

To expound on this a little more, we have measured a lot of cards and seen how their grades change. To integrate all of this information, we've set a Beta distribution at the company level to allow for different cards to have different P parameters. What we actually care about though isn't the cards or necessarily the range of reasonable Beta distributions for the company, but rather the mean of that beta. The one we're calling the mean hyperprior that we assigned as Beta(3, 6). THAT hyperprior is going to be updated via Bayesian inference and allow us to use the data to get a stronger estimate on the company's mean probability of changing a grade if someone resubmits a card. The card estimates are just byproducts of the inference, the company's beta distribution is an intermediate step to integrate the data from all of the cards and frame the problem, and the hyperprior mean is the core of the study that we are focused on. If it's low, it means that all cards graded by the company have an average probability of changing grade that is low. If it's high, it means that all cards graded by the company have an average probability of changing grade that is high. The concentration reflects whether this differs much between cards. A high concentration means a low variance, which means probability of grade changes doesn't really differ by card. A low concentration means a high variance, indicating that different cards may have substantially different probabilities of changing grade.

Again, if we wanted to focus on one thing, it's the mean hyperprior and how it is updated after we run the inference.

To defend my choice of these hyperpriors, let's examine some prior predictive simulations in Plot 1. This allows us to visualize the implications of our prior (or in this case, hyperprior) selections to ensure that they cover the reasonable range of possibilities.

<p align="center">
  <img src="https://github.com/A-J-V/pokemon_card_grading_analysis/assets/72227828/7ab33697-ec48-411d-9e85-8b81a596eec0" alt="Prior Predictive Plot"><br>
  <em>Plot 1: This prior specification allows for the full range of plausible beliefs</em>
</p>

To create this plot, we draw several sample means and concentrations from the Beta(3, 9) and Normal(12, 2) hyperprior distributions, respectively. For each pair of these hyperparameters that we draw, we plug them into a Beta distribution and plot it. For example, if we sampled 0.4 from the Beta(3, 9) and 12 from Normal(12, 2) hyperpriors, we would plot a Beta(mean=0.4, concentration=12) distribution. By plotting several of these distributions on a single axis, we can see the implications of the hyperpriors that we specified. The main goal of this is to given ourselves a sanity check and ensure that we've encoded beliefs that are within the realm of reason. In this case, I have tuned my hyperprior selections so that, as seen in the plot, they allow for the possibility that PSA's overall grade change probability distribution can be mostly found concentrated near 0.0 (indicating very good stability, most cards don't change when regraded), although it is believable that they may also have some probability up toward 0.7 or 0.8, indicating extreme volatility. This restricts the beliefs to what is possible without professing any more specific belief on where the true distribution should be. The mean hyperprior (the one we care about most) can also be seen in Plot 2.

<p align="center">
  <img src="https://github.com/A-J-V/pokemon_card_grading_analysis/assets/72227828/864c2175-4345-4e05-b005-f0c42092da54" alt="Mean Prior Distribution"><br>
  <em>Plot 2: This is the mean prior distribution, an encoding of what I find most likely before seeing any data.</em>
</p>

### Performing the Inference
The Bayesian inference for this project was performed using NumPyro. I used the Hamiltonian Monte Carlo algorithm to sample from the posterior distribution, and drew 2000 samples across 4 chains with a warmup of 1000. The full code to run the model is included in the linked repository. The mathematical specification of the model is included as the end of this post for the interested reader. The results are shown below (Figure 1), indicating no divergences and an a Gelman-Rubin convergence diagnostic value of 1.0 for all parameters (this is good).

<p align="center">
  <img src="https://github.com/A-J-V/pokemon_card_grading_analysis/assets/72227828/caa8e5f1-d6ff-40eb-8380-5fd36ae244bd" alt="Summary of Sampling from the Posterior"><br>
  <em>Figure 1: Summary of sampling from the Posterior.</em>
</p>

In this circumstance, we care about this image mainly from a diagnostic perspective to check that the model converged and doesn't show any signs of problems. The results are discussed in the next section.

## Results

To understand the results, we can plot some posterior predictive simulations (Plot 3). This is similar to what we did with the prior predictive simulations, but these are now drawing the mean and concentration from the posterior distributions, which have taken the observed data into consideration.

<p align="center">
  <img src="https://github.com/A-J-V/pokemon_card_grading_analysis/assets/72227828/ab9b6ece-cc61-4045-9cee-71a94fcee0e6" alt="Posterior Predictive Plot"><br>
  <em>Plot 3: The range of plausible posterior distributions has shrunken as the data reduced our uncertainty.</em>
</p>

We can see that the range of distribution are less spread out than they were in the prior simulation. Taking the data into consideration has reduced our uncertainty.

The contrast is more blatant if we depict the prior mean of the company's distribution (the one we care about most) with it's posterior (how it changed after we ran inference) (Plot 4). This is the core result of the project. It's clear that while we encoded our prior beliefs to allow for a wide range of plausible outcomes, after seeing the data, we can be fairly certain that the actual range of plausible mean grade change probability is somewhere between 0.25 and 0.45, with the posterior mode (maximum a posteriori) sitting around 0.35.

<p align="center">
  <img src="https://github.com/A-J-V/pokemon_card_grading_analysis/assets/72227828/71efada6-3b85-484e-98b1-af6e8b3949c3" alt="Prior vs. Posterior Mean Distributions"><br>
  <em>Plot 4: A visual comparison of the prior versus posterior .</em>
</p>

In my view, this is quite a finding. The interpretation is that the average card sent to PSA has a 25% - 45% chance of being assigned a different grade upon each blind resubmission. Not exactly a vote of confidence in the integrity of the company's grading procedures, to say the least. It's worth noting, however, that the concentration is fairly high, being somewhere between 9 and 15. This indicates that the probability of changing grade may vary substantially by card--some cards may rarely change, others may get a different grade almost every time they're submitted.

### Limitations and Comments

It's important to point out some limitations of the study and comment on a few interesting findings.

First of all, all cards in this study were Wizards of the Coast era Pokemon cards from the Base Set, Base Set II, Jungle Set, or Fossil Set. Out of the enormous number of different types of cards, this represents a tiny selection. I find it unlikely but at least hypothetically possible that other types of cards have different grading procedures resulting in more (or less) stable grades.

Second, all cards were already graded, and most were graded as an 8 or a 9 initially. The reason for this is that cards with very low grades aren't very valuable or interesting, and most of the concern and importance of the question is how cards that could plausibly be viewed by some as deserving a high grade can change depending on the subjectivity of the grader.

Third, it is worth noting the types of grade changes that occurred and the context around the grades. 
* The cards were acquired from different places at different times, so it is unknown when they were initially graded.
* Most cards only moved up or down one grade at a time. There were a two 1st edition cards initially graded as 8s that dropped to 5s.
* Interestingly, only one card in the entire sample ever exceeded it's initial grade. A holographic Muk card that start as an 8 was regraded as a 9 (twice). Every other grade change dropped below the initial grade.
* This isn't necessarily evidence, but it is curious and makes me wonder more seriously how much the company's grading criteria has changed over the years, as many claim. Did they get stricter, therefore most old grades were downgraded upon regrading?
* Not a single Wizards of the Coast card that was included in this experiment was upgraded to a grade 10. I submitted 2 modern cards on a whim with one of the resubmission batches. Both immediately received grade 10. It is entirely plausible that this is a coincidence, but the fact that 40 old (valuable) cards, many of which are visually flawless, were regraded twice, saw many grade changes, yet never never saw a single 10, while the two modern (non-valuable) cards that were no more flawless in appearance were granted immediate 10s makes me wonder.

Fourth, due to the high cost and low speed of grading, each card was graded 3 times, which is only 2 regrades per card. We had 40 cards, so there were a total of 120 grades and 80 regrades, and this allows us to get a meaningful estimate. Nevertheless, it is still far from an ideal sampling situation, and it would have been preferably if each card could have gotten 4-6 regrades.

Fifth, the reason that the study chose a binary outcome at the card level (either the grade changes or it does not change) was for simplicity. In reality, some cards (although very few) changes more than 1 grade at a time (such as the two 8s that became 5s after a single regrade). We could have made a more granular study, but it would have substantially increased the complexity and is unlikely to have improved the result.

Sixth, I'd like to comment that there is no reason to assume that PSA uses a linear grading scale. That is to say, there is no reason that the "difficulty" of a card moving from a 7 to an 8 should be the same as moving from an 8 to a 9. So there is not necessarily anything suspicious about having a small number of 10s, or seeing some grades change but never reach a 10. Any suspicious I would have comes from the lack of visually flawless old cards not being upgraded to a 10 while common modern cards immediately became 10s.

Seventh, when you submit your cards for grading with PSA, you are required to accept terms and conditions, which includes the following:
"Grading involves individual judgments that are subjective and require the exercise of professional opinion, which can change from time to time. Therefore, PSA makes no warranty or representation and shall have no liability whatsoever to Customer for the grade assigned by PSA to any item..." so while PSA and card dealers pretend to authority (it's literally in their name) in their marketing, when it comes to legality, they get a lot more shy and admit that what they're doing is subjective and they won't stand by their own grades.

Finally, I didn't make this study to attack PSA. I feel confident that any major grading company has similar issues. I conducted this study out of curiousity and after seeing people spend near six figures on certain Pokemon cards that were graded as perfect during the pandemic. I selected PSA purely because they're by far the largest in the grading industry.

