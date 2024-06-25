# Title: MRC 40 "The Time Curve Staking MOR For Compute Providers"

### Link for Discussion on Discord: 
https://discord.com/channels/1151741790408429580/1251995364693774336

### Status: 
In Discussion

## Authors / Discord Name: 
Anon 866, David Johnston (Smart Agents), Anton (antonb), Anon (lachsbagel) Christopher (storm.father), Jon (jonisjon), Maxwell (scott_b_), Jeff (Jabo38), Luke Stokes, Nebuchadnezzar Crew 

### Authors of Proposals On Similar Time Related Subjects: 
- mechaverse specifically from MRC 36 https://github.com/MorpheusAIs/MRC/blob/main/MRC36.md
- With extensive feedback and alternative proposals from: kehndry

## Category: 
MRI #1 Smart Contracts Reference Implementation

## Rationale: Time Equals Aliegnment
To implement economic alignment of Compute Providers, it is necessary to implement a functionality where users can specify for what period they want to lock the MOR token claiming, in return the user will receive an increased MOR reward and more reputation.

- 1. Staked MOR can inform the likelihood of the Compute provider being selected. Weight in Compute Provider Repution.
- 2. Stake MOR can inform the maximum reward a Compute Provider can earn. So for example if 100 MOR are Staked for 1 Year, then the most the Compute Provider can earn is 100 MOR during 1 year of providing Compute.

## Dependencies: 
None.

## New Weights Requested: 
30,000 Weights

## Existing Weights: 
None.

## Deliverables: Smart Contract Updates 
The Distribution Smart Contract specifically with have a the two functions added.
1. MOR Time Delay function (restrict MOR claims during certain block heights).
2. Power Multiple calculation (update MOR reward calculation).

## Qualification:
Same open source developers that developed the Morpheus Smart Contracts thus far.

## Timelines
3 to 4 weeks from June 21st 2024

## Analysis & Models:
- **Emissions Curve Calculator 7.21.2024**
https://docs.google.com/spreadsheets/d/1xTY7keBdPo2Nzm35Wdmu7ngP3NVIDebR/edit?usp=share_link&ouid=108805586783812761772&rtpof=true&sd=true

- **MOR Power Table**
https://docs.google.com/spreadsheets/d/1uEjozAcnEt-IWaSsu_BbYPRMUCkbwjwv/edit?usp=share_link&ouid=108805586783812761772&rtpof=true&sd=true

## Realization
Final MOR reward calculation
When  claim of  the MOR tokens are locked, there will be a multiplier for the user final rewards. Thus the final reward will be calculated by the formula:

final_reward = standard_reward * multiplier

Where the standard_reward: calculated according to the existing rules.

## Realization For Calculating the Multiples
Final MOR reward calculation
When  claim of  the MOR tokens are locked, there will be a multiplier for the user final rewards. Thus the final reward will be calculated by the formula:

final_reward = standard_reward * multiplier

Where the standard_reward: calculated according to the existing rules.

Calculation of the multiplier
The multiplier is calculated using the following formula:

multiplier = (end - now)/now + 1;

Where the end: MOR that potentially will be in circulation at the end of lock period for the current group.
Where the now: MOR that is potentially in circulation at the time of transaction execution for the current group.

## Using Tanh Hyperbolic Tangent for this Function in Solidity (Included in the Smart Contract)
- Below is the function for calculating the Power with a multiple cap of ~10.7.
- It works over 16 years: July 25, 2024 12pm UTC to January 26, 2040 12pm UTC
- Power Multiple cap reflects a 6 year delay on Claim locks.
- A Contributor can lock for the full 16 years, however they gain no additional Multiple beyond the first 4 years (7.46 max). 

**Function:**
def power_relative(staking_begin_unixtime, staking_end_unixtime):
    power_max=16.61327546
    period_start_unix=1721908800  # July 25, 2024 12:00 UTC
    period_end_unix=2211192000    # January 26, 2040 12:00 UTC

    val = power_max * (np.tanh(2 * ((staking_end_unixtime - period_start_unix) / (period_end_unix - period_start_unix))) - np.tanh(2 * ((staking_begin_unixtime - period_start_unix) / (period_end_unix - period_start_unix))))
    val = min(10.7, val) # keeps value below or equal to 10.7
    val = max(1.0, val) # keeps value above or equal to 1

    return val

## Using of multiplier
A multiplier can be applied at deposit, if the user specifies locking period. Or with a separate function on the smart contract - lockClaim(). The lock period is specified in seconds, it can be any interval.
When a multiplier is applied, the user's share of the stETH pool increases, depending on the multiplier.

## Example Multiples in Chart
![ExampleMORPowerMultiples6Year](https://github.com/MorpheusAIs/MRC/assets/1563345/ef16bf76-2de4-4c13-911e-514fb2a0c3b0)
**Presumes a July 25th 2024 Start Date**

## Restrictions
To implement such functionality, we need to carry a number of constraints and understand the important points:
- the claim lock period can be set by the user or administrator (for non-automatic groups) at any time;
- the claim lock period cannot be decreased; 
- the claim lock period can be increased. At the time of the transaction, the new multiplier will be applied.
## Changes to Smart Contracts
The Distribution contract and related interfaces will change. Updates to the smart contract on the network will need to be made.

## Timelines
4 to 6 weeks from June 21st 2024.

## APPENDIX: Reference Materials Including Analysis Reviewed For MRC
In order to design the optimal system, it is important to identify the key objectives to be accomplished. A properly designed system should focus its efforts on these objectives that it is meant to control and not expand outside its scope.

## MOR Compute Tokenomics - or the design by which Compute Providers are determined eligible - notes the following objectives:
- Fidelity:
- Accuracy
- Precision 
- Consistency
- Reliability
- Faithful to requests
- Deter Adversarial Conditions - remove incentive or ability for the system to be gamed and non-value add.
- Align Compute Providers with long-term success of Morpheus.
- Create a fair environment that encourages participation from a variety of Compute Providers
- Design a system that has the ability to scale up over time, as outlined in the phased approach below
- When demand increases, high-quality compute resources are incentivized to join the network. As a result, competition drives prices downward.

To ensure that proper analysis is conducted in determining the optimal system, this paper serves to identify a broad scope of viable methods, defining the basics of how they would operate, and laying out the pros and cons of each. Only after ample efforts have been taken to explore as many possibilities as possible, can a thoughtful recommendation be made.

### As such, the following options are ones that have been identified and will be explored throughout this paper:
- Minimum requirement stake (# of MOR) 
- Minimum requirement stake (% of supply)
- Top X stakers 
- Top X stakers, inclusive of multiplier
- Proportional to IPS (? or some other measurable) metric
- Equal to X month earnings - variable
**- Fixed lockup equal to eligible earnings - Judged Most Fair, Scalable and Simplest To Implement**
- Entrance Fee - flat rate, scaled option, or bidding
- Add-On: No rewards for first X number of days
- Add-On: Stake + Burn

As each of the above items has its own pros and cons, the ultimate design may likely be a combination of several of these approaches. As much development is still underway, it allows Morpheus to take a slow and calculated approach to roll-out. This ensures that ample testing is conducted, and sufficient feedback is digested, that by the ultimate design will evolve over time as the ecosystem grows and matures. Not to mention, the design decisions made here will impact a wide variety of other aspects of Morpheus, and also be impacted by those other aspects as well. All of this requires thoughtful consideration of not only the direct impacts, but the second, third, and fourth order effects. Morpheus could consider a phased roll-out, something similar to the outline noted in the table below.

### Exploration of Viable Methods
In the following section, each method will be briefly described, additional notes will be added, and finally pros and cons will be documented. These outlines are meant to provide the general background of each method in consideration. Subsequent efforts will dive much deeper into each, and ultimately reach an overall recommendation as to which method, or methods, provide Morpheus with the best outcome.

### Minimum Requirement Stake (# of MOR)
Brief Description: A specific amount of MOR is noted as the requirement to be eligible as a Compute Provider. 
Additional Notes: None.
Pros: 
Simplistic approach
Static amount that gives Compute Providers certainty of if they’ll be eligible or not
Cons: 
Doesn’t differentiate between large-scale and small-scale Compute Providers
Sets a cap on the number of Compute Providers. I.e. if 10,000 MOR stake is required and only 1,000,000 MOR are circulating, you could only have 100 providers.

### Minimum Requirement Stake (% of MOR)
Brief Description: A specific amount of MOR is noted as the requirement to be eligible as a Compute Provider. This amount will increase alongside the emission schedule of MOR, ensuring that Compute Providers must maintain their relative share of the MOR tokens in order to be eligible.
Additional Notes: None.
Pros
Simplistic approach
Static amount that gives Compute Providers certainty of if they’ll be eligible or not
Provides constant buy pressure as Compute Providers must obtain MOR at the inflation rate
Cons
Doesn’t differentiate between large-scale and small-scale Compute Providers
Sets a cap on the number of Compute Providers. I.e. if 1% MOR stake is required, you could only have 100 providers.

### Top X Stakers
Brief Description: No specific amount of MOR is defined to be considered eligible. Willing Compute Providers stake MOR and the ones who are the top X stakers are the ones considered eligible Compute Providers.  
Additional Notes: This would be at the Compute Provider level, i.e. the sum of all nodes they operate.
Pros
Creates a competitive environment which operates as a free market
Consistent demand for MOR from Compute Providers 
Cons
Discourages, or outright eliminates, ability for small-scale providers to participate
No incentive for providers to stake more than needed. I.e. If the top 10 stakers are eligible, they will do their best to always be 1 MOR ahead of the 11th highest staker.
Would Compute Providers who stake, but don’t make the ‘active set’ get their MOR unstaked?

### Top X Stakers with Multiplier
Brief Description: No specific amount of MOR is defined to be considered eligible. Willing Compute Providers stake MOR and the ones who are the top X stakers are the ones considered eligible Compute Providers. The only difference here is that the length of the staking matters. I.e. A provider who stakes 100 MOR for 1 year may be higher ranked than one who stakes 200 MOR for 1 month.
Additional Notes: This would be at the Compute Provider level, i.e. the sum of all nodes they operate.
Pros
Creates a competitive environment which operates as a free market 
Consistent demand for MOR from Compute Providers 
Encourages longer term lockups
Cons
Discourages, or outright eliminates, ability for small-scale providers to participate
No incentive for providers to stake more than needed. I.e. If the top 10 stakers are eligible, they will do their best to always be 1 MOR ahead of the 11th highest staker.
Would Compute Providers who stake, but don’t make the ‘active set’ get their MOR unstaked?

### Proportional to IPS (or some other Compute Metric)
Brief Description: The IPS (or some other Compute Metric) will determine the relative size of potential earnings that a Compute Provider will be bringing to the network. This directly ties to the earning power of that particular provider.
Additional Notes: None
Pros
Net reduction in circulating supply during the lockup period as an amount equal to expected earnings would be immediately locked up.
Proportional to size, allowing large and small providers to participate
Cons: 
Based on estimate - as IPS does not necessarily translate directly to earnings

### Proportional to Potential X Month Earnings - Variable
Brief Description: Compute Providers are required to stake MOR equal to the amount they anticipate earning based on a calculation when they bring a new node online. 
Additional Notes: This would be be similar to the Proportional to IPS method, but would differ depending on some open questions regarding the pricing and inference cost procedures.
Pros
Net reduction in circulating supply during the lockup period as an amount equal to expected earnings would be immediately locked up.
Proportional to size, allowing large and small providers to participate
Cons: 
Based on estimate - Providers could wind up earning less, or more, than the staked amount based on a variety of factors

### Fixed Lock Equal to Earnings
Brief Description: Compute Providers are required to stake MOR equal to the amount they want to be eligible to earn in an X month period. For example, a Compute Provider who stakes 10,000 MOR can earn up to 10,000 MOR from the Compute bucket.
Additional Notes: Providers could continue to add to their stake if they realize they are nearing their capped amount.
Pros
Whereas the proportional methods are based on future estimates that may not be realized (or may be over realized), this fixed approach ensures that a Compute Provider does not earn more than they have staked.
Net reduction in circulating supply during the lockup period as an amount equal to expected earnings would be immediately locked up.
Cons: 
If a Compute Provider earns all the allowable MOR quickly, then they may not want to stake additional, and less capacity is available making the remaining capacity more expensive… however, a profitable environment would likely encourage Compute Providers to obtain and stake additional MOR.

### Entrance Fee
Brief Description: Pay to play… it would cost X amount of MOR (which goes to the Protection Fund, PoL, or gets burned) in order to obtain a ‘license’ to operate. Could be different tiers as well - i.e. 100 MOR for a week, 1000 MOR for a year. Another consideration could be that it is reward based… Pay 100 MOR and you get to be a provider until you earn 200 MOR. 
Additional Notes: This could be a flat fee or a dynamic one based on a variety of metrics
Pros
Reduction of supply of MOR, enhancing the scarcity and value
Cons
Might discourage Compute Providers to outlay the upfront cost, given that the return is less certain than in other methods

### Add-On: No Rewards for First X Days
Brief Description: Compute Providers must operate for X days successfully before they become eligible for rewards
Additional Notes: Providers would be constantly, or randomly sampled, for testing during this process. This would likely be just an add-on to a more robust method
Pros 
Proving ground that doesn’t require payment of MOR while Compute Providers are tested
Cons
Similar to all other methods, but ongoing testing would be needed to ensure compliance doesn’t stop after the testing phase

### Add-On: Stake + Burn
Brief Description: While not a standalone concept, this could be implemented as an add-on to any other methodology.
Additional Notes: Creates a burn mechanism for shorter staked duration (i.e. Stake for 52 weeks and no burn, but each week less than 52 you stake, you burn 0.1% of the MOR)
Pros
Encourages long-term staking
Cons
Potentially makes spikes in demand harder to track - in situations where more providers may come online during a spike in demand, those same providers might be discouraged to lock up for extended periods thus increasing cost of inference to users.
