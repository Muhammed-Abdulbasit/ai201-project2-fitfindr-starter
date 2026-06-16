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
This function searches the available clothing listings based on the user's description, preferred style, and budget. It filters the listings to find items that match the user's criteria.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): A textual description of the item the user is looking for (e.g., "vintage graphic tee").
- `size` (str): The user's clothing size (e.g., "M", "L", "32", etc.).
- `max_price` (float): The maximum price the user is willing to pay for the item.

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
A list of matching listing dicts, sorted by relevance (best match first).
Each listing dict has the following fields:
id, title, description, category, style_tags (list), size, condition, price (float), colors (list), brand, platform

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
If no listings match the user's criteria, the agent should return a message indicating that no items were found and suggest broadening the search criteria (e.g., increasing the max price, being less specific in the description, or considering different styles). The agent can also offer to suggest outfit ideas based on the user's existing wardrobe if they have one.

---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
This function takes a new clothing item and the user's existing wardrobe to suggest outfit combinations. It looks for items in the wardrobe that complement the new item in terms of style, color, and occasion.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): A dict representing the new clothing item, with fields like id, title, description, category, style_tags, size, condition, price, colors, brand, platform.
- `wardrobe` (dict): A dict representing the user's existing wardrobe, where keys are item ids and values are item dicts with the same structure as `new_item`.

**What it returns:**
<!-- Describe the return value -->
A list of suggested outfit combinations, where each combination is a dict containing the new item and complementary items from the wardrobe. Each combination should include details on why the items go well together (e.g., matching style tags, complementary colors).

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
If the wardrobe is empty, the agent should offer general styling advice for the new item based on its attributes (e.g., "This vintage graphic tee would pair well with high-waisted jeans and sneakers for a casual look"). If no complementary items are found in the wardrobe, the agent can suggest general outfit ideas or recommend specific types of items to look for that would go well with the new item.

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
This function generates a short, shareable outift caption for a thrift find, similar to how users create "fit cards" on social media. It takes an outfit combination and produces a catchy caption that highlights the key pieces and the overall vibe of the outfit.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (str): The outfit suggestion string from suggest_outfit().
- `new_item` (dict): The listing dict for the thrifted item, containing fields like title, description, style_tags, colors, etc.

**What it returns:**
<!-- Describe the return value -->
A 2–4 sentence string usable as an Instagram/TikTok caption. The caption should creatively highlight the new thrifted item and how it fits into the overall outfit, using engaging language and relevant style tags.

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
If the outfit data is incomplete, the agent should return a descriptive error message string and not raise an exception. The message should indicate what information is missing and suggest how to fix it (e.g., "Outfit data is incomplete: missing style tags for the new item. Please ensure the listing includes style tags to generate a fit card caption.").

---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->
The planning loop starts by analyzing the user's query to determine their needs and preferences. It first checks if the query contains specific criteria for a clothing item (e.g., "vintage graphic tee under $30"). If it does, the loop calls `search_listings()` with the relevant parameters extracted from the query. If `search_listings()` returns results, the loop then calls `suggest_outfit()` for the top result to generate outfit suggestions based on the user's wardrobe. Finally, it calls `create_fit_card()` to generate a catchy caption for the suggested outfit. The loop is complete when the final output (the fit card caption) is generated and ready to be presented to the user. If at any point a tool fails (e.g., no listings found, empty wardrobe), the loop handles the failure accordingly by providing feedback to the user and suggesting next steps (e.g., broadening search criteria, offering styling advice without specific listings, etc.).

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->
The agent maintains a session state that tracks the user's query, search results, wardrobe items, and outfit suggestions. When `search_listings()` is called, the results are stored in the session state under a key like `search_results`. When `suggest_outfit()` is called, it accesses both the `search_results` and the user's `wardrobe` from the session state to generate outfit suggestions. The output from `suggest_outfit()` is then stored in the session state as `outfit_suggestions`, which is accessed by `create_fit_card()` to generate the final caption. This structured state management allows for seamless data flow between tools and ensures that each tool has access to the necessary information to perform its function effectively.

---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | Returns a message indicating no matching items were found and suggests broadening the search criteria. |
| suggest_outfit | Wardrobe is empty | Offers general styling advice without specific listings. |
| create_fit_card | Outfit input is missing or incomplete | Returns a descriptive error message indicating what information is missing and suggests how to fix it. |

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

**Milestone 4 — Planning loop and state management:**

---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->

**Step 3:**
<!-- Continue until the full interaction is complete -->

**Final output to user:**
<!-- What does the user actually see at the end? -->
