# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**

I picked 4 of 5 because search is a plain keyword match. A query like "find t-shirt" won't match a listing titled "Baby Tee", so some phrasings will miss even though the item exists. My test queries are phrased the way a user would type them, not copied from the data.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**

I picked 5 of 5 because this path never calls the model. When search returns an empty list, the loop stops at a plain check, so all five tries give the same result. My test query is "designer ballgown size XXS under $5", where no word matches any listing.

---

## 3. Something about state

In 5 of 5 runs of "looking for a vintage graphic tee under $30", the `id` of `session["selected_item"]` equals the `id` of `session["search_results"][0]`, and equals the `id` of the `new_item` that `suggest_outfit` received. A reader can check this by printing those three ids after the run.

**Why this target:**

I picked 5 of 5 because this is just a value passed from one step to the next, with no model involved, so nothing varies.


---

## 4. Something about the fit card

One try is one set of three runs on the same item. A try passes only if all of these hold:

- every caption contains the item's price and platform name;
- every caption is 2 to 4 sentences;
- the three captions are not all word-for-word identical.

The price counts if the number appears (e.g. 18.0); the platform counts if its name appears, ignoring capitalization (e.g. depop).

Target: 5 of 5 tries pass.

**Why this target:**

I picked 5 of 5 because the checks don't depend on exact wording. They only look for the price number, the platform name and a sentence count, so the model's random variation shouldn't cause a miss. My prompt tells the model to include the price and platform.

---

## 5. Your choice

With get_empty_wardrobe(), the agent completes all three tools and returns a non-empty fit card, with no exception raised, in 4 of 5 tries.

**Why this target:**

I picked 4 of 5 because the empty-wardrobe branch itself is a plain check that I control, so it should behave the same every time. But this path still makes two model calls, one in `suggest_outfit` and one in `create_fit_card`, and a model can fail or return something unexpected. One miss in five is a fair allowance for that.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
