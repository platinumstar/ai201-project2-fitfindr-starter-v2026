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

FitFindr is a thrift-shopping assistant. A user types a request in plain language, like "vintage graphic tee under $30", and the agent searches a set of 40 secondhand listings, filtering by price and size and ranking by keywords. It picks the best match, asks the model for one or two outfits that use pieces from the user's own wardrobe (or general styling ideas if the wardrobe is empty), and writes a short social-media-style caption for the find. The user gets back the item with its price and platform, the outfit ideas, and the caption, or, if nothing matches, a message saying which filters to change.

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

- **What it does:** Searches the listings data for items matching a description, and optionally a size and a price ceiling. Size matches by whole token, case-insensitively: M matches S/M and M/L but not US 9 or XL. A size with a space (such as "US 9") never matches, and "One Size" items never match a letter size.
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

**Branch rule:** 
"If search_listings returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to suggest_outfit."

**Where it lives:** 
`agent.py::run_agent`

**How the query is parsed:** 
two regular expressions (SIZE_RE and PRICE_RE) find the size and price, and those phrases are cut out, leaving the description. The price pattern needs a `$` after "under", "below", "up to" or "max", and the size pattern needs the word "size" before the size, so "jeans under 40 dollars" is not parsed for price.

**What moves through the session:** 
query, parsed, search_results, selected_item, outfit_suggestion, fit_card, and error only if the run stops early.

---

## Sample Run

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  Outfit:   **Outfit 1 (Casual Y2K):** Pair the butterfly baby tee with the **baggy straight-leg jeans**, **brown leather belt**, and **chunky white sneakers**. Throw on the **black cropped zip hoodie** for a layered look.
**Outfit 2 (Contrast Mix):** Style the pink/purple butterfly tee with the **wide-leg khaki trousers** and **chunky white sneakers**, topped with the **vintage black denim jacket** to ground the pastel tones.
  Fit card: Just scored this absolute dream of a Y2K baby tee on Depop for only $18! I can't wait to style the butterfly print with some baggy denim and a cute zip-up hoodie for the ultimate throwback fit.
2 model calls this session, 367 prompt + 148 output tokens
```

**The three tools, tested one at a time**

`search_listings`, a query that matches, then one that does not:

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import search_listings; print(search_listings('designer ballgown', size='XXS', max_price=5))"
[]
```

`suggest_outfit`, with the example wardrobe, then with an empty wardrobe:

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
**Outfit 1: Casual & Classic**
Pair the Vintage Levi's with the **white ribbed tank top** tucked in, the **brown leather belt**, and **chunky white sneakers**. Layer the **vintage black denim jacket** on top for a cool, effortless finish. 

**Outfit 2: Cozy Streetwear**
Style the jeans with the **oversized grey crewneck sweatshirt** half-tucked, finished off with **black combat boots** and the **black crossbody bag** for an easy, everyday look.
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()))"
**1. The Classic Casual:** Pair them with a tucked-in plain white t-shirt, a leather belt, and white canvas sneakers. 

**2. Elevated Denim:** Dress them up with an oversized black blazer layered over a ribbed tank top, finished with black ankle boots.
```

`create_fit_card` (run with `AI201_CACHE=0` so the answer is fresh, not cached):

```
$ AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[10]))"
Obsessed with these new khaki low-rise cargo pants I just scored on Poshmark! They only cost $27, which is an absolute steal for how comfy they are. I can’t wait to style them with denim and crisp white kicks for my next weekend errand run.
```

---

## How I Used AI

I used Claude mostly as a tutor to simplify the existing code and milestone expectations.

Claude explained each milestone section to me, and I built the tools and the loop from those explanations. Then I asked Claude to attack my work and explain why something did not look right.

**Moment 1**

- *What I asked for:* I asked Claude to check my `create_fit_card` tool and test it on several items, including one test with the cache turned off.
- *What came back:* Some of the captions included incorrect information. For example, one caption said "$12 on Depop" even though the item cost $27 and was listed on Poshmark. Another caption incorrectly mentioned Vinted. My prompt did not include the item's price or platform, so the model filled in the missing details by guessing.
- *What I changed:* With my approval, Claude added the correct price and platform to the prompt and told the model to summarize the outfit in one phrase and limit the response to two to four sentences. I then tested four items again, and all four captions matched the actual data.

**Moment 2**

- *What I asked for:* I asked Claude to run my agent and verify that the sample commands worked before adding them to the README.
- *What came back:* It found two errors. Running `app.py ask` caused an `UnboundLocalError` because some lines had accidentally been placed outside the `if` block in `run_agent`. Also, running `suggest_outfit` with an empty wardrobe caused a `NameError` because the fit-card prompt had been placed in the wrong function.
- *What I changed:* With my approval, Claude moved the lines back inside the `if` block and restored the prompt for the empty-wardrobe case. After those changes the commands ran successfully, so I copied the actual output into the README.

**Moment 3**

- *What I asked for:* I had Claude argue against my own MET verdict on criterion 4, since my captions said "$18" but the criterion literally said "18.0."
- *What came back:* Claude pointed out that strictly reading my own wording, none of my captions matched, which would flip it to MISSED.
- *What I changed:* I kept it as MET — the criterion was really checking that a reader can see the price, not an exact string, so I didn't touch the prompt.

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
| 4. Fit card mentions the item's price | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Over-budget query returns nothing over the limit | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Full evidence for all 25 tries: [`results/run_2026-10-12_1205_before.md`](results/run_2026-10-12_1205_before.md).

**Real output for each criterion, one try**, naming the file and function
that produced it:

**Criterion 1 — `agent.py::run_agent`, query `"vintage graphic tee under $30"`, try 1**

```
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- ids: selected_item=lst_002 · search_results[0]=lst_002 · suggest_outfit received=lst_002

Fit card:
Obsessed with this little butterfly tee I just scored! It's giving major 2000s energy and I can't wait to style it with baggy denim or contrast it with some edgy trousers. Grabbed it for $18 on depop and it's easily my new favorite piece in the rotation.
```

**Criterion 2 — `agent.py::run_agent`, query `"designer ballgown size XXS under $5"`, try 1**

```
- stopped early: yes — No listings matched "designer ballgown". You could raise your price
  limit (you searched under $5), or try a different size (you searched size XXS), or use
  simpler or different keywords.
- selected_item: (none)
- ids: selected_item=None · search_results[0]=None · suggest_outfit received=None
```

**Criterion 3 — `agent.py::run_agent`, query `"looking for a vintage graphic tee under $30"`, try 1**

```
- ids: selected_item=lst_002 · search_results[0]=lst_002 · suggest_outfit received=lst_002
```

**Criterion 4 — `tools.py::create_fit_card`, same item, try 1, three repeat runs**

```
Run 1: Obsessed with this butterfly print baby tee I just scored on depop for only $18! It's
giving major Y2K nostalgia and is so easy to style for either a baggy denim street look or
tailored trousers with combat boots. Size S/M fits like a dream, so grab it before I change my
mind and keep it for myself!

Run 2: Just scored this precious butterfly tee and I am obsessed with the nostalgic colorway!
It's up on my Depop right now for only $18 in a size S/M. I'm totally planning to throw it on
with some relaxed denim and a dark jacket for effortless off-duty vibes, or dress it up with
utility trousers and rugged boots. Run, don't walk, because you guys know these early 2000s
gems never stay in my shop long!

Run 3: Obsessed with this little butterfly tee I just scored for only $18 on depop! It's giving
total early 2000s pop star energy, and I honestly can't wait to style it. I'm thinking about
either throwing on some baggy denim and chunky kicks for an effortless daytime look, or layering
it under a cropped hoodie with trousers for something a bit more edgy.
```

**Criterion 5 — `agent.py::run_agent`, `get_empty_wardrobe()`, query `"denim jacket under $50"`, try 1**

```
- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- ids: selected_item=lst_007 · search_results[0]=lst_007 · suggest_outfit received=lst_007

Fit card:
Score of the century secured on Poshmark for just $42! I am so obsessed with this light wash
cropped denim jacket and can't wait to style it for everything from running errands to casual
date nights.
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

**Happy path**

```
$ python app.py ask 'vintage graphic tee under $30' --trace
[1] parse query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] branch
      →    results found: continuing
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: **Outfit 1 (Casual Y2K):** Pair the butterfly baby tee with the **baggy straight-leg jeans**, **brown leather …
[5] create_fit_card
      in:  **Outfit 1 (Casual Y2K):** Pair the butterfly baby tee with the **baggy straight-leg jeans**, **brown leather …
      out: Just scored this absolute dream of a Y2K baby tee on Depop for only $18! I can't wait to style the butterfly p…
```

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5' --trace
[1] parse query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[3] branch
      →    no results: stopping
```

The empty-search trace stops at step 3, so it is shorter than the happy path: `suggest_outfit` and `create_fit_card` are never called.

**The three failures, triggered one at a time**

- *Empty search:* No listings matched "designer ballgown". You could raise your price limit (you searched under $5), or try a different size (you searched size XXS), or use simpler or different keywords.
- *Empty wardrobe* (`python app.py ask 'vintage graphic tee under $30' --empty-wardrobe`): no crash and no empty string. `suggest_outfit` returned two looks built from general basics, and `create_fit_card` still produced a caption.
- *Model unavailable* (a fake key for one run, on a query not already cached): the trace ends at step 4, `model unavailable`, and the agent says: "I found a match, but couldn't reach the AI model to write the outfit. The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com. Once that is fixed, run your search again."

**On the MCP move:** I moved `search_listings` onto MCP. In `mcp_server.py` it is registered with `@mcp.tool()`, typed inputs and a description, and `python mcp_client.py` lists it. In `run_agent()` I replaced the direct call with `call_tool("search_listings", {...})`, passing the same three values from `session["parsed"]`. Nothing behaved differently afterwards: the list returned over MCP equals the list from the direct call, `brand: None` values survive, and an empty search still returns `[]`, so the branch still works.

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

Added retry-on-overload logic to `generate.py::generate` (lines ~328–347): a
`503 UNAVAILABLE` / "high demand" response from the model is now treated the
same as a rate limit — retried up to `config.MAX_RETRIES` (5) times with
backoff — instead of immediately raising `ModelUnavailable` and ending the
run.

**Which failure it was meant to fix:**

While building out the remaining scenarios, my first full 5-scenario run
(`results/run_2026-10-12_0133.md`, before this fix existed) hit transient
`503 UNAVAILABLE` errors from the model on one try each for criteria 1, 3,
and 4 — all the same root cause, not three separate bugs. I made the retry
fix before formally recording my official "Run Log — Before," so that table
already reflects the fixed code and reads 5/5 on every criterion.

**Timeline note:** because of that ordering, my before-run and after-run are
expected to look the same — both ran on the already-fixed code. The real
before/after comparison is `results/run_2026-10-12_0133.md` (pre-fix, 3
scenarios, two 503-caused stops) against `results/run_2026-10-12_1205_before.md`
(post-fix, all 5 scenarios, zero 503s). I'm reporting this honestly rather
than re-staging a fix that already happened.

### Run Log — After

`python run_eval.py --label after` — see [`results/run_2026-10-12_1532_after.md`](results/run_2026-10-12_1532_after.md).

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Full three-tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Item in session matches item passed on | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions the item's price | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Over-budget query returns nothing over the limit | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Did it help, and how do I know:**

Yes, based on the earlier evidence — before the fix, `results/run_2026-10-12_0133.md`
hit transient 503s on one try each for criteria 1, 3, and 4. After the fix, two
separate 5-try batches on the fixed code (`1205_before`, 25 tries, and this
`1532_after` run, another 25 tries — 50 tries total) came back with zero
model-availability stops. The after-run doesn't differ numerically from the
before-run because both ran on the already-fixed code — the improvement is
visible in the older, pre-fix log, not in a before/after delta under
identical code.

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

All five criteria read MET, but criteria 1, 3, and 4 only ever ran against
one item (`lst_002`, the Y2K Baby Tee) across all 5 tries. A clean 5/5 on
the same item each time is weaker evidence than 5/5 across 5 different
items — it proves the loop and the prompt work for *this* item's shape
(it has a brand-less field, a short description, a round price), but not
that they hold up for an item with, say, a missing `brand` or a price with
cents. I'd rewrite the criterion 1, 3, and 4 scenarios in `scenarios.py` to
cycle through a handful of different items instead of repeating the same
query, and re-run. I stopped here because the current run log already
satisfies this unit's requirement, and reworking the scenarios would mean
another full before/after cycle I didn't have time left for this unit.

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
