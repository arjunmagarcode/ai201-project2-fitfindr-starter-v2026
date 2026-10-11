# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## Milestone 1 Notes

I ran the environment check, inspected the listing and wardrobe fields, reviewed
six full listings, checked the example queries, and ran the starter agent. The
starter ran successfully and correctly reported that the planning loop was not
built yet.

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr accepts a natural-language request such as "vintage graphic tee under
$30, size M." It searches thrift listings using the requested description, size,
and maximum price. It selects a matching item, suggests outfits using the user's
wardrobe, and creates a short caption for the item. If no listing matches, it
stops and tells the user what search details to change.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the listings dataset for items matching a description and optional size and price ceiling.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None).
- **Returns:** A list of listing dictionaries containing `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`, sorted with the best matches first.
- **When it has nothing:** Returns an empty list `[]`.

### `suggest_outfit`

- **What it does:** Suggests one or two outfits using the selected listing and the user's wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict).
- **Returns:** A non-empty string containing outfit suggestions.
- **When it has nothing:** If the wardrobe is empty, returns general styling advice for the selected item.

### `create_fit_card`

- **What it does:** Creates a short social-media-style caption for the selected item and outfit.
- **Inputs:** `outfit` (str), `new_item` (dict).
- **Returns:** A two-to-four-sentence caption mentioning the item, price, platform, and overall style.
- **When it has nothing:** If the outfit string is empty, returns a descriptive message instead of crashing.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, the agent stores an
actionable message in `session["error"]` and stops before calling the model
tools. Otherwise, it stores the first result in `session["selected_item"]`,
passes that item to `suggest_outfit`, and then passes the outfit and the same
item to `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regular expressions extract a maximum price after
phrases such as `under` or `below` and a size after `size`. The remaining text
becomes the listing description.

**What moves through the session:** The parsed description, size, and maximum
price go into `session["parsed"]`; search results go into
`session["search_results"]`; the first result goes into
`session["selected_item"]`; then the outfit string and fit-card caption go into
`session["outfit_suggestion"]` and `session["fit_card"]`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

Outfit:   Here are two practical outfit suggestions using your new Y2K baby tee:
Outfit 1: Classic Y2K Streetwear using baggy straight-leg jeans, chunky white sneakers, and a black crossbody bag.
Outfit 2: Edgy Contrast using a vintage black denim jacket, wide-leg khaki trousers, and black combat boots.

Fit card: Serving major 2000s pop princess energy with this Y2K Baby Tee — Butterfly Print! Styled with baggy denim and chunky sneakers for the ultimate nostalgic streetwear vibe. Snagged this absolute steal on Depop for just $18.00!

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([item['id'] for item in search_listings('graphic tee', max_price=30)])"
['lst_017', 'lst_002', 'lst_033', 'lst_006', 'lst_015', 'lst_011']

```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()))"
**Look 1: Casual Streetwear**
* **Pieces:** Oversized graphic tee, white leather retro sneakers, and a canvas tote bag.
* **Styling Direction:** Lean into the vintage, laid-back vibe.

**Look 2: Elevated Casual**
* **Pieces:** A fitted black baby tee or tank top, an oversized black blazer, and black loafers or ankle boots.
* **Styling Direction:** Contrast the casual denim with sharp tailoring.

```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats the character of broken-in denim. I just scored these Vintage Levi's 501 Jeans — Medium Wash for $38.0 over on Depop, and they have an effortless streetwear vibe. Paired with crisp white sneakers, this is my go-to uniform all season long!

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked for help turning the three required tools into a
     precise Tool Inventory with typed inputs, specific return values, and empty
     cases.
- *What came back:* The suggested specification required `search_listings` to
     return listing dictionaries and `[]` when there were no matches, while the
     model-backed tools had defined strings and explicit empty-input behavior.
- *What I changed:* I added those specifications to the README before building
     the tools, including whole-token size matching for values such as `M` and
     `S/M`.

**Moment 2**

- *What I asked for:* I asked for help implementing the planning loop so it
     would branch on empty search results and carry the selected listing through
     session state.
- *What came back:* The suggested design used regular expressions for price and
     size parsing, stored every tool result in the session, and stopped with an
     actionable message when search returned an empty list.
- *What I changed:* I implemented that design in `agent.py::run_agent`, then
     verified that matching queries complete all three tools and impossible queries
     leave `session["fit_card"]` as `None`.

**Moment 3**

- *What I asked for:* I asked AI to help interpret the repeated fit-card
     outputs and identify a single change that addressed the observed miss.
- *What came back:* The miss was at the model-output step: the model sometimes
     paraphrased the selected listing title even though the tool and session
     state were correct.
- *What I changed:* I tightened the `create_fit_card` prompt to require the
     exact title, numeric price, and platform spelling, then reran all five
     scenarios five times and recorded the before and after reports.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Full three-tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Item in session matches item passed on | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions title, price, and platform | 4 of 5 | PASS | PASS | PASS | PASS | FAIL | MET (4/5) |
| 5. Over-budget query returns nothing over the limit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
Source: results/run_2026-10-11_1837_before.md, produced by run_eval.py::main
Loop: agent.py::run_agent

matching query completes, Try 1:
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
Fit card:
Found the ultimate Y2K Baby Tee — Butterfly Print for just $18 over on depop,
and I am officially obsessed! It’s giving total early 2000s nostalgic
streetwear when paired with baggy denim and chunky sneakers. Snagged a serious
wardrobe staple today! 🦋✨

impossible query stops early, Try 1:
- stopped early: yes — No listings matched those filters. Try changing the description, size, or maximum price.
- selected_item: (none)
- search_results: 0

selected item handoff, Try 1:
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
Trace: [1] search_listings (via MCP), [2] suggest_outfit, [3] create_fit_card

fit card content, Try 5:
Fit card:
Found this dreamy Y2K Butterfly Baby Tee and I am officially obsessed with the
nostalgic early-2000s energy! Styled it with baggy denim and chunky sneakers for
the ultimate casual street vibe. Snagged it on Depop for just $18—such a good score. 🦋✨

price ceiling, Try 1:
- selected_item: Mesh Long-Sleeve Top — Black ($15.0, depop)
- search_results: 6
All six returned listings were at or below $30.
```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

1. **Criterion 1 — MET (5/5):** The MCP search call returned results and the
     loop reached `suggest_outfit` and `create_fit_card` on every try. No miss.
2. **Criterion 2 — MET (5/5):** The search tool returned `[]` every time. The
     empty-results branch in `agent.py::run_agent` set an actionable error and
     returned before either model tool. No miss.
3. **Criterion 3 — MET (5/5):** The session selected the first MCP result on
     every run, and the trace showed that same item entering `suggest_outfit`.
     No session handoff miss occurred.
4. **Criterion 4 — MET (4/5):** The miss was in the model output step on Try
     5. The caption called the item “Y2K Butterfly Baby Tee” instead of the
     selected title “Y2K Baby Tee — Butterfly Print”; price and platform were
     present. The tools and session were correct, but the prompt only said to
     mention the title and allowed the model to paraphrase it.
5. **Criterion 5 — MET (5/5):** Search filtering returned six items for the
     `$30` query and every recorded price was at or below the ceiling. No miss.

The three deliberate failure checks were also recorded. An impossible search
returned: “No listings matched those filters. Try changing the description,
size, or maximum price.” An empty wardrobe returned general styling advice and
still produced a fit card. With `GEMINI_API_KEY=invalid-key-for-unit-test`, the
agent returned: “The outfit model is unavailable: The model rejected your API
key. Check GEMINI_API_KEY in your .env file, or create a fresh key at
aistudio.google.com. Check your API key or network connection and try again.”
No raw traceback was shown.



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
[1] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: 10 items: Y2K Baby Tee — Butterfly Print, Vintage Band Tee — Faded Grey, Graphic Tee — 2003 Tour Bootleg Style … +7 more
[2] suggest_outfit
     in:  dict with keys: new_item, wardrobe
     out: Here are two practical outfit suggestions using your new Y2K baby tee…
[3] create_fit_card
     in:  dict with keys: outfit, new_item
     out: Serving major 2000s pop princess energy with this Y2K Baby Tee — Butterfly Print…
```

**Empty search**

```
[1] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: [] (empty)
[2] search branch
     in:  dict with keys: search_results
     out: No listings matched those filters. Try changing the description, size, or maximum price.
     →    empty results: stopping before model tools
```

On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->

`mcp_server.py` now exposes `search_listings` with typed `description`, `size`,
and `max_price` inputs. `agent.py::run_agent` calls it through
`mcp_client.call_tool`; the other two tools remain direct calls. `python
mcp_client.py` listed the tool and matching queries returned the same listing
shape as the direct implementation.

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

I changed only the `create_fit_card` prompt in `tools.py` to require the exact
listing title, numeric price, and platform spelling once each.

**Which failure it was meant to fix:**

Criterion 4's baseline miss came from model paraphrasing the selected title.
The search tool and session were already correct, so tightening the prompt was
the smallest change aimed at the model-output step.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Full three-tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Item in session matches item passed on | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions title, price, and platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Over-budget query returns nothing over the limit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Did it help, and how do I know:**
Yes. The same five scenarios ran five times again in
`results/run_2026-10-11_1845_after.md`. Criterion 4 rose from 4/5 to 5/5;
criteria 1, 2, 3, and 5 stayed at 5/5. The change helped the diagnosed model
output problem without changing the search or loop behavior.
<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->
No acceptance criterion remained missed after the improvement. The model can
still vary its wording, and the invalid-key path is only manually triggered,
not part of the five scored scenarios. A next improvement would add automated
assertions over the trace and failure messages, but I stopped after the one
prompt change required for this unit so the measurement stays attributable.


<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
