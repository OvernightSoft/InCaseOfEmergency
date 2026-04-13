# How CanShrink works in Microsoft Access, and how to handle controls that don't shrink

Access' reports provide a great feature: shrinkable sections and controls. By setting a section’s or control’s `CanShrink` property to `True`, Access can automatically remove unused vertical space when data is missing.

There are some caveats, however. In certain layouts, sections may not shrink as expected, leading to confusing or seemingly inconsistent behavior.

This article explains **how shrinking really works** in Access reports and shows practical techniques to deal with controls—like labels and checkboxes—that don’t support shrinking.

# The fundamental rule

The fundamental rule to remember is: 

> Considering a horizontal band of a section, it will shrink only if both of these conditions are met:
>
> 1. there is at least a shrinkable control which is empty, and whose `CanShrink` property is set to `True`
> 1. there is no control blocking the shrinking. A control will NOT block shrink if:
>    1. it is invisible, or
>    1. it is shrinkable, and is empty

When these conditions are true, the horizontal band defined by the shrinkable control's Top and Botton margins collapses entirely.

If a report contained only textboxes (and no labels) `CanShrink` would work flawlessly. Unfortunately, real reports usually contain labels and other controls that cannot shrink — introducing edge cases.

Let's see what this means in practice, using this test table:

![Test Table](assets/img01.png)

# Just a textbox

The simplest case is just a textbox, shrinkable, in a shrinkable section. The report's structure is shown in the image; it includes two colored reference lines to highlight the shrinking area:

![](assets/img02.png)

The result is as expected

![](assets/img03.png)

# A textbox and its label

We now add a label

![](assets/img04.png)

The label goes against both parts of condition 2) above. It is not invisible, and it is not shrinkable: it will therefore block shrinking even when the textbox is empty, and in fact we get

![](assets/img05.png)

A frequently suggested workaround is to replace the label with a shrinkable textbox, and set its value to `=IIF(ISNULL(Fruit), NULL, "Fruit")`

This works, but it's not very elegant. Also, the "label as a textbox" trick does not solve the problem of shrinking a band when it contains other non shrinkable controls, such as checkboxes.

A better approach is to use VBA in the `OnFormat` event of the shrinkable section, to set the controls invisible when they aren't needed.

We can use this this code

![](assets/img06.png)

to obtain the wanted result. 

When the control is set as invisible, its label becomes invisible as well, so both conditions 1) and 2) are met, and the section shrinks.

# Shrinkable and not shrinkable controls

When a non shrinkable control is added

![](assets/img07.png)

to satisfy condition 2.i) above, we only need to fix the formatting procedure

![](assets/img08.png)

and again...

![](assets/img09.png)

# Labels, labels everywhere

But labels can be everywhere, so what happens when we change our layout to 

![](assets/img10.png)

In this case, we get a partial result... the band with the textboxes collapses, but the band with the labels doesn't, even if they are invisible

![](assets/img11.png)

This is happening because the band with the labels does not satisfy condition 1): there is nothing to shrink!

The solution? Add an empty, shrinkable textbox, to signal to Access that the band should be shrinked (when possible).

![](assets/img12.png)

When the labels are visible, the band won't be shrinked. But when they are hidden, the empty, shrinkable textbox will cause the shrinking of the whole band. And in fact...

![](assets/img13.png)

This small trick—adding an empty, shrinkable textbox purely as a shrink signal—is something I haven’t seen documented elsewhere. Hopefully, it will help you build cleaner, more reliable Access reports.