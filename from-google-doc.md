## Best Practices Draft Doc for sampling/convergence assessment & ensemble averages/uncertainties
Alan Grossfield (agrossfield on GitHub), Pascal Merz, Paul Patrone, Daniel Roe, Andrew Schultz, Daniel Siderius, Daniel M. Zuckerman [PLEASE ADD YOUR NAME IF YOU HAVE ADDED TO THE DOC!!!]
Grossfield, Alan [Alan_Grossfield@URMC.Rochester.edu]; zuckermd@ohsu.edu; Pascal.Merz@Colorado.EDU; Siderius, Daniel (Fed) [daniel.siderius@nist.gov]‎‎; ajs42@buffalo.edu‎; alan_grossfield@urmc.rochester.edu‎; daniel.r.roe@gmail.com; Patrone, Paul (Fed) [paul.patrone@nist.gov]

# Scope
This article is geared toward studies attempting to quantify observables and derive reliable error bars based on “standard” canonical sampling methods (e.g., molecular dynamics and Monte Carlo) and associated “enhanced” sampling methods.  Some problems and systems may be better studied with cruder techniques and analyses, which will not be covered here.

# Key Definitions [to be refined]
- Precision
-- The amount of variability in an estimate
- Accuracy
-- Vs nature/experiment
-- Vs. outcome of perfect/infinite simulation
- Raw data - the set of numbers that the computer program spits out as it runs
- Derived observables - quantities derived based on ‘non-trivial’ analyses based on the raw data

# CHECKLIST 
- Pre-simulation sanity checks and planning tips: There is no guarantee that any method (enhanced or otherwise) can sample any given system
See best-practices papers on simulation background and planning/setup [Link out to simulation background and preparation documents - github]
Are system timescales known experimentally and feasible computationally based on published literature?
If timescales are too long for straight-ahead MD, is an enhanced method being used for which there are precedents for systems of similar complexity?
Read a good article or book on sampling assessment (this one or a reference herein).  Understanding error is a technical endeavor.
Key concept: Connection between the equilibrium ensemble and individual trajectory (may or may not reach equilibrium); equilibrium vs. non-equilibrium.
Consider multiple runs vs. single run.  Multiple runs may be especially useful in assessing uncertainty for enhanced sampling methods.
Make initial configuration as diverse as possible  … but note that if results depend on initial configs, that implies insufficient sampling (to be pedantic, it always does if sampling is finite, but it’s about figuring out variability and confidence intervals
Look for automated construction methods for reproducibility
Check your code/method via a simple benchmark system.  [Link out to software validation doc]
- Perform a quick-and-dirty data checks which can rule out (but not ensure) sufficient sampling: Necessary vs. sufficient
Look at time series -- think in advance about what states should exist. How many transitions do you see? If you have 1 transition, you can’t talk about populations
Plot as many properties as you can think of, even if they’re not interesting
Visualize the trajectory -- look for slow motions.  BE SKEPTICAL!
Compare observable different fractions of a run (DMZ thirds idea)
Andrew: short vs. very short
Daniel R: Compare runs from different initial conditions - be sure initial conditions are ‘different enough’
- Remove an ‘equilibration’/’burn in’/transient portion of a single MD or MC trajectory and perform analyses only on remaining ‘production’ portion of trajectory.
- Consider computing a quantitative measure of global sampling [biomolecular vs. materials problems] - i.e., how many statistically independent samples do you have?
- Quantifying error in specific observables of interest [general approaches]
Key concept: Difference between between scale of variation (measured by variance) and statistical uncertainty
- If you’re using an enhanced sampling method, …
Key concept: Complexity of correlations in advanced methods

# Item 6 - Computing error in specific observables
- Basics: how to report, what goal to shoot for, significant figures
- When should you not trust uncertainties
Unknown unknowns
- If calculating a derived quantity, consider error in conversion from raw data 
Propagation of error (this doesn’t mean publishing your results!)
Taylor series expansion can handle cases where derived quantity is a direct function of measured data
Wikipedia: Propagation of uncertainty
Generate synthetic data with noise model [Paul]
Data filtering [Paul]
Bootstrapping [Andrew/Dan S]
used for cases where the derived quantity is not simple function of the measured data.
An Introduction to the Bootstrap
Need to know correlation to correctly estimate sample size
Otherwise just gives relative uncertainty
- Correlation time analysis [Dan Z]
- Block averaging [Dan Z/Alan?/Dan S] Flyvjberg and Petersen 
‘Dark uncertainty’ analysis [Paul]
- MOST OF THESE ALGORITHMS FAIL IF THE TRAJECTORY IS WAY TOO SHORT
If you miss the timescale by enough, you can’t tell
YOU HAVE TO THINK ABOUT THIS IN ADVANCE
- Link out to transport doc

# Item 7 - Enhanced sampling [Write as supplements, which can later be broken out as separate documents]
- Compare to standard sampling (e.g., straight MD) for a simple system
- Complex correlation structures in complex methods indicate comparison of multiple independent runs will be useful
Explain what to compare [Dan Z]
- Replica exchange [Daniel Roe]
Round-trips (necessary but not sufficient)
Compare ‘coordinate trajectories’ - distributions from temperature/Hamiltonian-wandering trajectories should match
Examine replica residence times
- Weighted ensemble (WE)
See this WE overview doc, particularly limitations section
Key concept: ‘Tree’ of trajectories generated by WE leads to strong correlations, requiring care
Key concept: WE simulation generically relaxes from the initial distribution toward the ultimate distribution which could be equilibrium (if no feedback/recycling or external driving) or a non-equilibrium steady state (if feedback from specified target to initial state)
Safest approach: Use multiple runs, which are fully independent.  Perform as many runs as needed to reduce the statistical uncertainty (std err of mean) for quantity of interest
For a single observable, the time course of the value can be analyzed using the usual methods of analyzing time-correlated data (see above - e.g., block-averaging)


# Original notes/discussion below here ... needs to be harvested for text

Questions
Should our document be self-contained?  That is, should we write so that a graduate student could understand and implement all the methods noted, or do we rely on published works?
I think making it self-contained would make for an enormous document and a huge amount of work.  My preference would be to go with qualitative descriptions of techniques, prescriptions and rules of thumb for good practice, and extensive references for the details. 
 
(Paul Patrone: I second this.  Another NIST colleague and I wrote a book chapter on UQ for MD meant as an introductory text for graduate students and novice modelers.  It’s close to 100 pages long and took ~ 1 year to write.  That would be too difficult for this document.)

	 	 	
(PP) The more I read this document, I feel like we need to clarify for ourselves what our scope is. “Sampling / Uncertainty Assessment” is a very vague term to me. Is this meant to be an “overview” of uncertainty? What types of uncertainties do we focus on (systematic, statistical, etc.)? Do we focus on reducing uncertainties, quantifying them, both? Assessing confidence in a result? For what purpose? Do we only discuss issues of sampling in service of quantifying uncertainties, or do questions of sampling arise independently of uncertainty assessments?

To put forth one perspective on this, I'll play devil’s advocate and beat a (dead ?) horse a bit. A few folks seem to be advocating the perspective that any discussion of sampling / convergence should not poach (too much) from other groups works, e.g. on physical validation or best practices for specific properties. I would perhaps argue a different perspective, namely that any discussion of sampling / convergence assessment can’t be divorced from the specifics of the simulation in question. (I'm just going to lump convergence / sampling / uncertainty estimates into the term “uncertainty quantification” or UQ).

Uncertainty assessment and related issues of convergence (at least as I see them), are always tied to a question that we are trying to address or decision that someone is trying to make. UQ basically builds confidence that a decision is the “best” one. That being said, I don't think it's useful to do UQ in a vacuum or do analyses that don't help to inform a decision. As an example, I don't see too much of a point in checking that I've converged to the “correct” ensemble if all I care about is computing an ensemble average. So, not only can I forget about verifying the ensemble, I can use a thermostat (e.g. Andersen) that only gives me correct ensemble average. If, on the other hand, I care specifically about fluctuations, I may want use Nose-Hoover and then verify in whatever way I can that the ensemble is correct. As a different example, if I'm trying to estimate a material property such as yield strain, then I should be able to check that the material (approximately) recovers it's shape before the onset of yield. I would argue that's part of the uncertainty assessment because if the simulation doesn't do that, then I don't have a good estimate of when the material yielded and can’t make decisions on the basis of that simulation.

So, all this being said, I see this document as a general overview of UQ issues that arise in simulations, with an eye always towards the reasons why and contexts in which we do these assessments. We need not go into detail on every material property (that's for other groups to address) but I don't think we should shy away from the greater context in which we do UQ.

Anyways, I'm happy to get off of this horse if folks see this document differently. But I do think it would be useful to clarify what we're up to here and how this document fits into the overall picture of this workshop and the other documents that folks are writing.   

(AJS) It just doesn’t seem practical to enumerate how to compute and/or diagnose trouble with every kind of property and then to use this document as the basis for a “checklist” (that was the original objective of all this) that someone would use to convince themselves that they have sampled properly and have reasonable estimates of uncertainty.  If we want people to use the resulting checklist, it has to be compact and not list stuff that’s not relevant to what they’re doing.  Several of the specific properties mentioned below do have other groups working to produce a document (and checklist) for those properties.  The group doing fluctuations (for instance) ought to have something about how to run the simulation, how to determine that the sampling is adequate and the issues (specific to fluctuations) in estimating uncertainties.  It seem our best option is to point to those documents from here and to work with them to ensure that the relevant issues are covered in one of the documents. My 2 cents.

DMZ: Here’s my view.  I do think we want to adhere, for the most part, to the original goal of producing a doc that will be useful to beginning simulators (e.g., grad student in early years).  There is consensus that the document will not explain all the theory but link out to extensive existing literature.  As others have said, inevitably there will be overlap with the documents being produced by other groups (which doesn’t seem like a real problem) but I do think it’s ok for us to offer suggestions on assessing error in specific properties of general interest where some of us may have a lot of expertise.  The more guidance we have for that beginning graduate student the better.  One clean line that I think we can draw around our efforts is to exclude underlying/systematic errors in the models/force fields that we are simulating.  I think we should stick to the issue of how the beginner knows whether to rely on their data as an accurate portrayal of the model/force field that has been assumed.  BTW I think the goal is to produce and maintain a ‘living document’ that will be updated over time, which I think is a nice model.  If one of us (or someone else) in the future wants to add discussion of analyses specific to another property, that will make the document still more helpful to the beginner.

AG: I think Paul’s idea is interesting and true, but I’d say that the solution for his vision is the aggregate of _all_ of the documents coming from this workshop -- taken together, they’re trying to cover all of the sources of error and incorrectness.  Like Dan, I think our charter was to focus on one particular area -- statistical error -- as separate from the many sources of systematic error.  To cover all of the other sources of error would require a book, which is more effort than I think we want.  

Basic ideas and themes
     What can cause errors?
Implementation errors (bad code, bad algorithms)
Force field
Finite size artifacts
Timescale mismatch (experiments all do some implicit averaging)
All of the above cause systematic errors
No matter how long you run, these problems won’t go away
Statistical errors are a separate issue
Ergodic theorem only true in infinite time limit
Finite sampling time implies statistical error
Trivial to quantify in simple case -- stderr, etc
Hard in the context of simulations with long time correlations
Must be resolved in order to tackle the causes of systematic error
Can’t know a new force field is better until you know the error bars on your “measurement”
Realization and Verification of “convergence”
Aim is to address the question: how does one ensure that the simulation is representative of thermodynamic ensemble OR whatever statistics govern the system of interest
Generally involves at least two steps
Simulating a transient/relaxation period during which we “burn in” or equilibrate system
Verification that subsequent simulations represent the desired ensemble

Start with sanity checking in advance (BEFORE DOING THE CALCULATION)
What are the relevant time scales?
Roughly how many relevant states are there?
Is it even plausible to equilibrate? To converge?  
Have systems of similar complexity been well-sampled in the past, based on ‘best practices’ checks?
Recall: most “internal consistency” measures of statistical uncertainty fail if your simulations are orders of magnitude too short.  No rescue for stupid design.

Uncertainty estimation (sometimes goes by the moniker “Uncertainty quantification”)
Aim is generally to put error bars / confidence intervals on estimates, and possibly assess inconsistencies with other “target results” (i.e. validate the result)
First task is generally to estimate uncertainties associated with the simulation itself 
Requires, e.g., understanding whether simulated data is correlated or statistically independent
Requires understanding / modeling / accounting for the timescales over which random variables may fluctuate
Requires formulating appropriate definition of uncertainty: e.g. sample variance or other statistical measures of uncertainty in a mean
Other questions that arise / must be addressed
How can we say when two averages are different? (P values)
How do we verify that simulated data is consistent with theory (more examples below)?
How do we address finite-size effects and their impact on uncertainty (this is relevant, e.g. for isothermal compressibility)?  See J. J. Salacuse, A. R. Denton, and P. A. Egelstaff, Phys. Rev. E 53, 2382 (1996).
How do we represent complicated observables, such as the radial distribution function or probability densities?  See  http://dx.doi.org/10.1063/1.4977516
Second task may be to calibrate simulation inputs, (this task may not be necessary and/or may be a part of or precede the verification steps above)
Validation against target data
Often must be addressed in the context of target data that comes with its own uncertainties
May be used to estimate quantities such as “model-form” error
May use a variety of tools, such as polynomial chaos, Kriging analyses, etc.
Point to relevant literature about uncertainty quantification; e.g. “Guide for Verification and Validation in Computational Solid Mechanics (ASME, 2006).”  Also, Ralph Smith’s book.

	


II.  Realizing and verifying convergence of a simulation
Methods for realizing convergence
Within a single simple MD trajectories
Chodera’s method http://pubs.acs.org/doi/abs/10.1021/acs.jctc.5b00784
Rule of thumb??  Divide trajectory into thirds and compare distributions of observable for the last two thirds (i.e., 2nd third vs. 3rd third, with 1st third discarded as presumed equilibration period)
Better yet -- all scalar quantities should be subject to block averaging if you’ve got one trajectory.  It’s cheap, just make it part of standard operating procedure.  STILL FAILS IF YOU’RE SUFFICIENTLY OFF ON TIME SCALE.
What about ideas based on / motivated by fluctuation dissipation theorems?  Small deviations from equilibrium often have a characteristic exponential decay time back to equilibrium.  Shouldn’t observables converge according to these decay times?  (I’m speculating a bit here)
Here we describe a method that is similar to looking for convergence in mean square of observables: https://doi.org/10.1016/j.polymer.2016.01.074
Convergence using multiple simple MD trajectories
Often useful if you know there are long-lived states (even if it’s in principle an equilibrium system)
Lets you use standard statistical approaches, treat each trajectory as 1 independent measurement
May be necessary if, for example, the system takes a long time (microseconds or longer) to transition between metastable states
May be necessary if individual simulations become irreversibly trapped in certain configurations (e.g. as simulations of crosslinked polymers do)
This is a non-equilibrium circumstance, so totally different set of problems -- you’re characterizing a single relaxation per trajectory
Methods for realizing convergence
Ensemble methods - replica exchange, etc.  (THIS IS A WHOLE OTHER BALLGAME)
Just run multiple simulations (e.g. for crosslinked polymers)
Methods for assessing convergence
Shirts’ method (along with spectral Monte Carlo) to verify that an “equilibrated” distribution is consistent with Boltzmann statistics or grand-canonical ensemble.  See M. R. Shirts, Journal of Chemical Theory and Computation 9, 909 (2013).
Method motivated by law of large numbers 
Basically, compute standard error for N measurements and stop simulating when standard error falls below threshold
NOTE: convergence methods based on total energy are tricky and ill-advised without extreme care
Recommendation is based on the observation that discrete integrators (such as Verlet methods used in most MD packages) actually sample points from a shadow-Hamiltonian that is a perturbation of the analytical one that we write down.  
As a result, the energies output by MD (which are computed from the original, unperturbed Hamiltonian) fluctuate, especially in microcanonical ensemble.  Thus, it appears that such simulations never converge to the correct ensemble.
In order to assess energy conservation, it is therefore necessary to know the shadow Hamiltonian, or at least approximate it.  This computation (whether done analytically or numerically), is very expensive. 
Alternatively, estimate order of magnitude of shadow Hamiltonian perturbation and make sure energy is conserved to within that precision
See B. Leimkuhler and C. Matthews, Molecular Dynamics with Deterministic and Stochastic Numerical Methods (Springer International Publishing, Switzerland, 2015).
Like most scalar observables, the energy is highly degenerate - many configs have same energy.  The energy also includes all degrees of freedom in a system, unlike many specific observables of interest.  Thus, despite its prominent place in statistical mechanics, the energy is unlikely to be a good indicator of overall sampling
Convergence of structural ensembles from multiple independent simulations can be assessed via “combined clustering”: see e.g. http://pubs.acs.org/doi/abs/10.1021/ct400862k (Figs 3+4)
Similarly, convergence of dynamics from multiple independent simulations can be assessed by overlap of principal component projection histograms: http://www.sciencedirect.com/science/article/pii/S0304416514003092 (Fig 3)




III. Uncertainty estimation (UQ)
Single MD trajectories
At least two types of UQ associated with raw simulated data and data analysis thereof
For raw simulated data, many techniques for estimating uncertainties
Auto-correlation analysis
For scalar properties such as pressure, temperature, etc., keep simulated data from every timestep
For system in equilibrium, assume stationary autocorrelation function; estimate in terms of sample covariance
Build uncertainty estimate in terms of associated correlation matrix
Block averaging - note that correlation time is implicit
Decorrelation time analysis - doesn’t require any user selection of observables [Lyman & Zuckerman, JPCB 2007 http://pubs.acs.org/doi/abs/10.1021/jp073061t]
Analysis according to methods proposed by Salacuse: J. J. Salacuse, A. R. Denton, and P. A. Egelstaff, Phys. Rev. E 53, 2382 (1996) and J. J. Salacuse, A. R. Denton, P. A. Egelstaff, M. Tau, and L. Reatto, Phys. Rev. E 53, 2390 (1996).

Data analysis of simulated data
Generally is used to extract material properties from raw data, such as Tg (glass-transition temp) from density-temperature curves or modulus from stress-strain data
Propagation of uncertainty can be used to estimate uncertainties of derived quantities (free energy, coexistence, etc).
Must account for correlation between quantities measured in a single simulation.
Bootstrapping can be used for cases where the derived quantity is not simple function of the measured data.
Analyses often attempt to mimic what one would do with experimental data, which raises several issues
Does the simulated data actually have characteristics of experimental data?
In the context of Tg, one can ask: does my density-temperature curve have asymptotic regimes as identified, e.g., by a hyperbola? https://doi.org/10.1016/j.polymer.2016.01.074
In the context of yield, one could ask: does the system recovers its original dimensions if allowed to relax after applying a strain that is less than the yield strain? https://doi.org/10.1016/j.polymer.2017.03.046
Does simulated data have significantly more noise than experimental data, and if so, how does that affect analysis?
Bootstrap and synthetic dataset methods can be used to estimate uncertainties associated with analysis applied to a single dataset
Multiple independent analysis methods can be applied to a dataset to check for consistency between them
Do the experiment and simulation operate on similar time scales? Experimental “shutter speed” can mean that the correct comparison is not with the ensemble generated in MD, but on some averaged/blurred version thereof.  Especially true of fluctuation quantities.

Multiple simple MD trajectories
Estimates from multiple realizations of the same system may not have overlapping error bars
“Between-simulation” or “dark-uncertainty” estimates can be used to generate a consensus mean estimate of the quantity of interest.
See, e.g. https://doi.org/10.1016/j.polymer.2016.01.074 and A. L. Rukhin, Metrologia 46, 323 (2009).
Ensemble methods - replica exchange, etc..
Connection to global sampling
Generic strategies for evaluating enhanced sampling techniques which may have complex internal correlation structures
Use of multiple independent (enhanced) simulations which are guaranteed to be independent.
Validation / comparison with target data
Can be used to generate model form error 
See, e.g. papers by Kennedy and O’Hagan
M. C. Kennedy and A. O’Hagan, Journal of the Royal Statistical Society: Series B (Statistical Methodology) 63, 425 (2001).
M. C. Kennedy, C. W. Anderson, S. Conti, and A. OHagan, Reliability Engineering & System Safety 91, 1301 (2006), the Fourth International Conference on Sensitivity Analysis of Model Output (SAMO 2004).



Global measures of sampling - should avoid input of coordinates if possible
Background: Connection to single-observable uncertainty … in principle, good global sampling should imply good ‘local’ sampling of particular observables … use idea of timescales to connect global/local
Single simple MD trajectories
Decorrelation time analysis - doesn’t require any user selection of observables [Lyman & Zuckerman, JPCB 2007 http://pubs.acs.org/doi/abs/10.1021/jp073061t]
Nemec & Hoffman http://pubs.acs.org/doi/abs/10.1021/acs.jctc.6b00823
Multiple simple MD trajectories
Effective sample size
Grossfield & co
Zuckerman & co: JCTC 2010 http://pubs.acs.org/doi/abs/10.1021/ct1002384
Also Nemec & Hoffman (same as above)




