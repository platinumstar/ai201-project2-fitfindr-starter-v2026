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

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->
''



---

## Tool Inventory
"""
size='US 9' returns nothing, and One Size returns nothing. A size with a space never equals a single token. If a user types "size US 9", it won't match the shoe. You can leave it, but then say so in your README, or handle multi-word sizes by splitting wanted too and requiring all its pieces.
One Size items never match M. That follows your whole-token rule, which is fine. Just be aware of it.
"""
<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the listings data for items matching a description, and optionally a size and a price ceiling. Size matches by whole token, case-insensitively: M matches S/M and M/L but not US 9 or XL.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None, inclusive)
- **Returns:** A list of listing dicts, best keyword match first, at most `config.SEARCH_RESULT_LIMIT` long. Each dict has `id`, `title`, `description`, `category`, `style_tags` (list), `size`, `condition`, `price` (float), `colors` (list), `brand` (str or None) and `platform`.
- **When it has nothing:** Returns an empty list `[]`, not `None` and not an exception. The loop branches on this.

### `suggest_outfit`

- **What it does:** Given a thrifted item and the user's wardrobe, asks the model to suggest one or two outfits.
- **Inputs:** `new_item` (dict, a listing), `wardrobe` (dict with an `items` list, possibly empty)
- **Returns:** A non-empty `str` of one or two outfit suggestions, each naming wardrobe pieces the user already owns.
- **When it has nothing:** If `wardrobe["items"]` is empty, returns a non-empty `str` of general styling advice for `new_item`. Never `""`, never raises.

### `create_fit_card`

- **What it does:** Asks the model to write a short caption someone would actually post about the find.
- **Inputs:** `outfit` (str, from `suggest_outfit`), `new_item` (dict, a listing)
- **Returns:** A `str` caption of two to four sentences that mentions the item, its price and its platform.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a descriptive message `str` such as "No outfit to caption yet" instead of raising.

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

**Branch rule:** "If search_listings returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to suggest_outfit."

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->

**What moves through the session:** <!-- which fields, in what order -->

---

## Sample Run

<!-- Two things go here.

 % python -c "from tools import search_listings; r = search_listings('Y2K Baby Tee in white color'); print([i['title'] for i in r])"
['Y2K Baby Tee — Butterfly Print', 'Low-Rise Cargo Pants — Khaki', 'Mesh Long-Sleeve Top — Black', 'Platform Sneakers — White Chunky Sole', '90s Track Jacket — Navy/White Stripe', 'Graphic Tee — 2003 Tour Bootleg Style', 'Platform Mary Janes — Black Patent', 'Vintage Linen Blazer — Cream', 'Crochet Halter Top — Cream', 'Biker Shorts — Black, Shiny']

% python -c "from tools import search_listings; r = search_listings('graphic tee', max_price=30); print([(i['title'], i['price']) for i in r])"
[('Y2K Baby Tee — Butterfly Print', 18.0), ('Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('Mesh Long-Sleeve Top — Black', 15.0), ('Vintage Band Tee — Faded Grey', 19.0), ('Low-Rise Cargo Pants — Khaki', 27.0), ('Vintage Graphic Hoodie — Faded Black', 26.0)]

 % python -c "from tools import search_listings; r = search_listings('Y2K Baby Tee in white color'); print([i['title'] for i in r])"
['Y2K Baby Tee — Butterfly Print', 'Low-Rise Cargo Pants — Khaki', 'Mesh Long-Sleeve Top — Black', 'Platform Sneakers — White Chunky Sole', '90s Track Jacket — Navy/White Stripe', 'Graphic Tee — 2003 Tour Bootleg Style', 'Platform Mary Janes — Black Patent', 'Vintage Linen Blazer — Cream', 'Crochet Halter Top — Cream', 'Biker Shorts — Black, Shiny']
(.venv) ronju@Ronjus-Mac-mini ai201-project2-fitfindr-starter-v2026  % python -c "from tools import search_listings; r = search_listings('graphic tee', max_price=30); print([(i['title'], i['price']) for i in r])"
[('Y2K Baby Tee — Butterfly Print', 18.0), ('Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('Mesh Long-Sleeve Top — Black', 15.0), ('Vintage Band Tee — Faded Grey', 19.0), ('Low-Rise Cargo Pants — Khaki', 27.0), ('Vintage Graphic Hoodie — Faded Black', 26.0)]
(.venv) ronju@Ronjus-Mac-mini ai201-project2-fitfindr-starter-v2026  % python -c "from tools import search_listings; print(search_listings('designer ballgown', size='XXS', max_price=5))"
[]


% python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

**Outfit 1: Casual & Sporty**
Pair the Vintage Levi's 501s with the white ribbed tank top, layered under the oversized grey crewneck sweatshirt. Finish the look with chunky white sneakers and the black crossbody bag. 

**Outfit 2: Edgy Streetwear**
Style the jeans with the black cropped zip hoodie and the vintage black denim jacket on top. Accessorize with the brown leather belt and black combat boots.

 % python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()))"
Here are two easy ways to style vintage Levi’s 501s using wardrobe basics:

1. **Classic Casual:** Pair them with a crisp white t-shirt, a leather belt, and white sneakers for an effortless, timeless look.
2. **Elevated Everyday:** Tuck in an oversized black turtleneck or a neutral crewneck sweater and add loafers or ankle boots for a slightly sharper vibe.

     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

```

```
$ python -c "from tools import suggest_outfit; ..."

```

```
$ python -c "from tools import create_fit_card; ..."

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

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
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

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

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



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
