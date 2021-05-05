Finding Patterns - Market Basket Analysis Using Association Rules
================
Emma Grossman
5/5/2021

# Understanding association rules

A set, or **itemset** is formed by groups of one or more items that
appear in the data with some regularity. The result of **market basket
analysis** is a collection of **association rules** that depict patterns
found in the relationships among items in the itemsets. For example, an
itemset like {bread, peanut butter, jelly } might create an association
rule like {peanut butter, jelly } \(\rightarrow\) {bread}. So, if peanut
butter and jelly are purchased together, that implies that bread is also
likely to be purchased.

Association rules are not used for prediction. Another characteristic of
market basket is that there is no need to train the data; the program is
unleashed on the data and hopefully, interesting associations will be
found. Because of this, it is difficult to measure how well the market
basket algorithm performs.

## The Apriori algorithm for association rule learning

With 100 items, there would be \(2^{100}\) possible itemsets, so to
eliminate too much work, extremely rare events are ignored.

The **Apriori** algorithm searches large datasets fo rules.

Strengths:

  - can work with large amounts of data
  - rules are understandable
  - helpful in data mining and discovering unexpected knowledge in
    databases

Weaknesses:

  - not helpful for small databases
  - effort to separate true insight from common sense
  - could draw spurious conclusions from random patterns

## Measuring rule interest - support and confidence