# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
searches through the listings.json to find items that match the user input using keyword descriptions, size, and price limits. 

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): keywords on what type of item the user is looking for in  terms of style or fashion
- `size` (str): optional size to limit the item selection by
- `max_price` (float): sets a price limit for the item search 

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
returns a list of listings with stronger matches being clsoer to the top. each listing has the fields id, title, decsription, category, style_tags, size, condition, price, colors, brand, platform. 
**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
it returns an empty list if nothing matches and  returns some sort of error message letting the user know what happened, stops the loop right here. 
---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
it takes in a specific item and the current wardrop to suggest 1 outfit combination and style. 
**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): listing dict aka the item the user is suggested to buy
- `wardrobe` (dict): wardrobe dict with wardrobe items

**What it returns:**
<!-- Describe the return value -->
string with outfit suggestions and styling tips
**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
if the wardrobe is empty or no outfit can be suggestion, the outfit suggestion piece does not occur, but the styling part still happens and offers general styling advice. 
---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
give a caption based on the suggested item to buy and the suggesetd outfit
**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (...): the outfit suggestion return by suggest_outfit

**What it returns:**
<!-- Describe the return value -->
a short string that can be used as a caption for the specific outfit
**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
returns "Couldn't create a fit card"
---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---
### Stretch Feature: Retry Logic with Fallback

**What it does:**
If search_listings returns no results and a size filter was applied, the agent
automatically retries without the size constraint and informs the user what was adjusted.

**What it returns:**
If the retry finds results, returns listings with a retry_note string explaining what was adjusted. If retry also returns nothing, falls through to the normal error message.

**What happens if it fails or returns nothing:**
If the retry still returns an empty list, the agent sets session["error"] to "No listings found for description under price. Try a broader description or higher price." and stops.

### Stretch Feature: Price Comparison Tool

**What it does:**
Compares the selected item's price against the average price of other listings
in the same category to estimate whether the deal is good, fair, or overpriced.

**Input parameters:**
- `item` (dict): the selected listing dict from search_listings

**What it returns:**
A string summarizing the price verdict, e.g. "$18 vs. avg $22 for tops (14 listings) — fair price." Returns a descriptive message if no comparable
listings exist.

**What happens if it fails or returns nothing:**
If no listings share the same category, returns "No comparable listings found to evaluate this price." without crashing.
## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->
search_listings is always first, if its empty then stop the loop and give an error message. If its not empty, return the results and call suggest_outfit. suggest_outfit returns session outfit_suggestion. then this is used to call create_fit_card. this completely session is returned to user. 
---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->
a single session dict is created for each session aka each runagent call. this stores the selected_item, outfit_suggestion, fit_card, and error. This is used to call information in subsequent steps. 
---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | No listings found. Please look for something different or change the size or price constraints. |
| suggest_outfit | Wardrobe is empty | Wardrobe is empty. Here is some general styling advice: |
| create_fit_card | Outfit input is missing or incomplete | Couldn't create a fit card. |

---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     ASCII art, a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html), or an embedded
     sketch are all fine. You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->

---
User query(description, size, max_price, wardrobe) -> run_agent() -> search_listings(desciption, size, max_price) (if results empty, return ERROR and stop, else return slected_item) -> suggest_outfit(selected_item, wardrobe) (return outift suggestion or styling advice) -> create_fit_card (outfit_suggestion, selected_item) (return error or caption) -> return session to user
## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**
Give claude the the different tools and their descriptions from above and aks it to implement the function. I'll expect it to produce the correct output at each tool and test by setting sample queries. 
**Milestone 4 — Planning loop and state management:**
Give claude the architecture and state management and ask it to implement run_agent. Check differnet types of queries to make sure it returns errors and responses both. 
---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->
the first rool called is search_listings with the keyword vintage graphic tee and max_price 30. It goes through listings.json and returns what is matched to vintage graphic tee and is under $30. It stores the top result as selected_item. 

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->
the selected_Item was returned from step 1 and now suggest_outfit is called with the selecteditem and get example wardroble. This sends the selected item and user's wardrobe to the LLm and gets back an outift suggestion stored as outfit_suggestion. 
**Step 3:**
<!-- Continue until the full interaction is complete -->
next the outfit_suggestion is taken from step 2 and create_fitF_card is called which generates a caption for the outfit and stored in fit_card. 
**Final output to user:**
<!-- What does the user actually see at the end? -->
the user gets the found item, outfit suggestion, and fit card. 