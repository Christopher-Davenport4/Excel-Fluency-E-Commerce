# Decisions

A running log of business and project decisions. Date, decision, reason.

---

**22 September 2026. Duplicate reviews, keep the most recent review per order.**
547 orders have more than one review. I would say keep the most recent review because one would expect if you're leaving another review, that's your up-to-date opinion. That might lose out on people's initial reactions to an issue like things taking too long, but that would likely still get folded into the second review. The most recent review is chosen by review_answer_timestamp.

**24 September 2026. 2017 chosen as the working subset for Tiers 1 through 5.**
I chose 2017 as the working subset for Tiers 1 through 5. The data runs from 9/4/2016 to 10/17/2018, so 2017 is the only full year. Choosing a full year, January through December, is important as it will allow us to look at sales across all seasons. Choosing only a section of the year limits our scope and potentially the generalizability of our analyses. It also avoids the partial months at the start and end of the data, which can look like a spike or a collapse in sales when really the data just started or stopped being collected. The subset has 45,101 orders.
