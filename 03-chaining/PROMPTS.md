# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: [name your flow]

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1. Add a screen "{{User Shift Start}}". Match the layout of the {{Guided overview}} screen. Include a "Welcome back" message.
2. Add a screen "{{User Shifts Journal}}". Match the layout of the {{Actions log}} screen.
3. Navigation: write the logic so {{User Shift Start}} links to {{User Shift Journal}} and creates a new Day in the Journal. The Journal is afterwards filling with the list of consecutive Actions applied by the User and their respective time stamp.

Build these in order so {{User Shift Start}} is the anchor for {{User Shifts Journal}}.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the {{Site breakdown}} screen flow:
- Use skeleton screens for the {{list}} loading state.
- If no data is present, show the empty state: "{{empty-state message}}".
- On fetch failure, trigger the error state: "{{error message}}".

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
Improve the prototype to address the main usability risk identified during review:

CURRENT PROBLEM
Users are being asked to take an action ("Apply idle fee at Stirling M9 J9") before they have enough evidence to trust the recommendation. This creates hesitation and may reduce engagement because the dashboard appears to be asking for a decision before explaining the reasoning.

GOAL
Keep the guided decision approach, but provide enough evidence on the first screen for users to understand why the recommendation exists before they click.

CHANGES REQUIRED

1. Enhance the "Highest priority action required" card on the Guided Overview page

Keep the recommended action visible, but add a concise evidence summary directly within the card.

Include a section such as:

"Why this action is recommended"

With 3 to 4 short evidence bullets based on the existing scenario data:

• Average stall occupancy exceeds 5 hours
• Vehicles charge for only about 1 hour on average
• More than 80% of occupancy time is idle parking
• Current utilisation pattern indicates blocking, not charging demand

Add a short conclusion:

"Analysis indicates that increasing charger capacity would not materially improve availability. Reducing idle occupation is expected to have a greater impact."

The objective is to make the recommendation feel justified without requiring navigation to another page.

2. Rename the secondary link

Current text:
"See how we got here"

Replace with:
"Why this location was flagged and which other locations require your attention"

The new wording should make it immediately clear that the page contains:
- detailed evidence supporting the recommendation
- analysis of the flagged location
- visibility into other locations that may also need action

3. Improve hierarchy inside the priority action card

Structure should be:

Highest Priority Action Required
↓
Recommendation
↓
Evidence summary
↓
Expected outcome
↓
Primary action button

This should feel like a recommendation supported by evidence rather than an unexplained command.

4. Add an "Expected outcome" section

Example:

Expected outcome:
• Increase charger availability without adding infrastructure
• Reduce average occupancy duration
• Improve throughput at Stirling M9 J9

5. Preserve the prototype philosophy

Do NOT:
- add charts to the overview
- add large tables to the overview
- introduce filter bars
- introduce dashboard complexity

The overview must remain simple and decision-oriented.

DESIGN INTENT
The user should be able to answer the question:
"Why are you recommending this action?"
without leaving the first screen, while still having the option to navigate to the detailed analysis page for deeper investigation.

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- Giving names to each Screen for clarity and so that the AI does not change anything in the other screens.
- Creating two screens with a logic between the two.
- Simulating real-world conditions (skeleton and sand-clock while loading the data).
- Being able to simulate errors.

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

The 3rd prompt in the chain created an Error in Lovable.   
I guessed there were contradictions related to the "do NOT do ..." elements which were split.  I reworded the point 5. to this and it worked fine.     

5. Preserve the prototype philosophy

Do NOT:
- add charts to the overview
- add large tables to the overview
- introduce filter bars
- introduce dashboard complexity
- change anything else in the project 
- touch the underlying logic.
The overview must remain simple and decision-oriented.

DESIGN INTENT
The user should be able to answer the question:
"Why are you recommending this action?"
without leaving the first screen, while still having the option to navigate to the detailed analysis page for deeper investigation.


