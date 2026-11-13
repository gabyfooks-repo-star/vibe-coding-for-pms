# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: retention-and-revenue-chain

### Step 1: Expand: add the two missing screens
```
Build the next phase of this app in a strict sequence:
1. Add a 2nd screen called "revenue" where we can see the gross ticket fees from each purchase flow (mobile vs. web). Make it so that revenue is somehow hyperlinked from the main page as well. 
2. Add another screen called "retention" that demonstrates how many of these buyers are repeat users, including a definition of what retention means in this context (came back to purchase a ticket within 6 months). 
3. Navigation: write the logic so the main page, "revenue" and "retention" all link to one another and can navigate back to the main page. No dead-end states.
```

### Step 2: Behavior: handle loading, empty and error states
```
Apply the following logic constraints to the Channel Performance flow:
- Use skeleton screens for the App vs. web · All regions loading state.
- If no data is present, show the empty state: "No data. Check your filters or refresh the screen.".
- On fetch failure, trigger the error state: "Connection lost. Please refresh the dashboard.".

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine: polish one thing on one screen
```
The Revenue needs a professional tableau-style polish.
1. Start by listing the 3 biggest gaps in typography and spacing compared to Tableau.
2. Once you've identified those, resize the headers and update the interactive elements to match.

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- Naming the point/goal of each page allows the AI to hallucinate less and make decisions that are more context-based.
- If i didn't state an explicit change to a particular button, the AI did not touch it. There weren't too many unique inferences made. Not necessarily a bad thing, it took the instructions literally.

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

The new screens don't necessarily reference one another in a humanistic way. There's no natural place where the dashboard is saying "go here for more insights on X"....it provides out-of-context and literal CTAs to click into a new tab, but no clear reason or hierarchy for doing so. I told the AI I wanted all CTAs to be contextual. 
