# 103 - Jinja Essentials for Workflow Automation

{% embed url="https://youtu.be/jU8OZ0exa_0" %}

{% hint style="success" %}
Sign up for our **LIVE** training sessions below!
{% endhint %}

<table data-card-size="large" data-view="cards" data-full-width="false"><thead><tr><th align="center"></th><th align="center"></th><th data-hidden data-type="content-ref"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td align="center"><mark style="color:blue;"><strong>Rewst 103</strong></mark></td><td align="center">Jinja Essentials for Workflow Automation</td><td><a href="https://calendly.com/cluck-u/rewst-103">https://calendly.com/cluck-u/rewst-103</a></td><td><a href="https://calendly.com/cluck-u/rewst-103">https://calendly.com/cluck-u/rewst-103</a></td></tr><tr><td align="center"><mark style="color:blue;"><strong>ROC AMA</strong></mark></td><td align="center">Come and well, ask me anything!</td><td><a href="https://calendly.com/cluck-u/rewst-ama">https://calendly.com/cluck-u/rewst-ama</a></td><td><a href="https://calendly.com/cluck-u/roc-ama">https://calendly.com/cluck-u/roc-ama</a></td></tr></tbody></table>

## Essentials for Jinja Automation Hands-on Exercises

Follow these steps to learn how to use Jinja techniques!

<details>

<summary>Step 1: Open the Live Editor with Sample Context Data and Jinja</summary>

**Open the Live Editor**

1. **Open** the [Live Editor with some sample data.](https://app.rewst.io/jinja-live-editor?id=4ee84407-d0c1-4a9a-94b3-84c643b8dd3f)

Here are some important keyboard shortcuts you can use throughout

1. Remove comments
   * Windows: _ctrl + /_
   * OSX: _cmd + /_
2. Render code
   * Windows: _ctrl + enter_
   * OSX: _cmd + enter_

</details>

<details>

<summary>Step 2: Start by viewing the basic CTX Attributes</summary>

**Express all the CTX Attributes**

1. **Open** the Context editor to view the JSON context.
2. **Uncomment** `{# {{ CTX }} #}`.
   * See the shortcuts in Step 1 to remove comments.
3. **Render** `{{ CTX }}`.
   * See the shortcuts in Step 1 to remove comments.
4. **Re-add** the comments to revert the code back to `{# {{ CTX }} #}`.

**Show the Value of the Week**

1. **Uncomment** `{# {{- CTX.week -}} #}`.
2. **Render** `{{- CTX.week -}}`.
   * See the shortcuts in step one to remove comments.
3. **Re-add** the comments to revert the code back to `{# {{- CTX.week -}} #}`.

**Show How many Days are in the Schedule**

1. **Uncomment** `{# {{ CTX.days | length }} #}`.
2. **Render** `{{ CTX.days | length }}`
   * See the shortcuts in step one to remove comments.
3. **Re-add** the comments to revert the code back to `{# {{ CTX.days | length }} #}`.

</details>

<details>

<summary>Step 3: Use Loops and List Comprehension</summary>

**Show the Dessert for each Weekday**

1.  **Uncomment** and **Render** the following:

    ```django

    {% raw %}
    {% for weekday in CTX.days %}
        The dessert on {{ weekday.day }} is {{ weekday.lunch.dessert }}
    {% endfor %}
    {% endraw %}


    ```
2. **Review** the Output.
3. **Re-add** the comments.

**Use List Comprehension to build a List of Strings**

1.  **Uncomment** and **Render** the following:

    ```django
    {{-
        [
            "The dessert on "~weekday.day~" is "~weekday.lunch.dessert
            for weekday in CTX.days
        ]
    -}}
    ```
2. **Review** the Output.

**Add Join Filter to Output List as text**

1.  **Type** `| join('.\n')` to the end of your code like the following:

    ```django
    {{-
        [
            "The dessert on "~weekday.day~" is "~weekday.lunch.dessert
            for weekday in CTX.days
        ] | join('.\n')
    -}}
    ```
2. **Render** the code.
3. **Review** the Output.
4. **Re-add** the comments.

</details>

<details>

<summary>Step 4: Organize List Data</summary>

**Render a List of Lists**

1.  **Uncomment** and **Render** the following:

    ```django
    {{-
        [
            weekday.lunch.sides
            for weekday in CTX.days
        ]
    -}}
    ```
2. **Review** the Output.

**Use the Flatten Filter to Output a Single List**

1.  **Type** `| flatten` to the end of your code like the following:

    ```django
    {{-
        [
            weekday.lunch.sides
            for weekday in CTX.days
        ] | flatten
    -}}
    ```
2. **Render** the code.
3. **Review** the Output.

**Use the Unique Filter to Remove Repetition**

1.  **Type** `| unique` to the end of your code like the following:

    ```django
    {{-
        [
            weekday.lunch.sides
            for weekday in CTX.days
        ] | flatten | unique
    -}}
    ```
2. **Render** the code.
3. **Review** the Output.

**Use the Sort Filter to Order Alphabetically**

1.  **Type** `| sort` to the end of your code like the following:

    ```django
    {{-
        [
            weekday.lunch.sides
            for weekday in CTX.days
        ] | flatten | unique | sort
    -}}
    ```
2. **Render** the code.
3. **Review** the Output.
4. **Re-add** the comments.

**Simplify the Output with a JSON Path Query**

1.  **Uncomment** and **Render** the following:

    ```django
    {{- CTX.days | jsonpath_query('[*].lunch.sides[*]') | sort | unique -}}
    ```
2. **Review** the Output to see that it's the same.
3. **Re-add** the comments.

**Optional: Use the Map Filter to Generate the Same List**

1.  **Type** the following in the next line:

    ```django
    {{ CTX.days | map(attribute="lunch.sides")}}
    ```
2. **Render** the code.
3. **Review** the Output.

**Optional: Add Different Filters with the List Filter to Organize**

1.  **Type** `| list` the following in the next line:

    ```django
    {{ CTX.days | map(attribute="lunch.sides") | list}}
    ```
2. **Try** out filters like `| flatten` or `| sort` to organize.
3. **Render** the code.
4. **Review** the Output.
5. **Re-add** the comments.

</details>

<details>

<summary>Step 5: Use More Advanced List Comprehension ans Filtering</summary>

**Check for a Specific Attribute**

1.  **Uncomment** and **Render** the following:

    ```django
    {{-
        [
            weekday.day
            for weekday in CTX.days
            if weekday.lunch.drink == "Milk"
        ]
    -}}
    ```
2. **Review** to see that it only shows Monday even though Friday has Chocolate Milk.
3. **Re-add** the comments.

**Ensure all Versions of the Attribute are Displayed**

1.  **Uncomment** and **Render** the following:

    ```django
    {{-
        [
            weekday.day
            for weekday in CTX.days
            if "milk" in weekday.lunch.drink | lower
        ]
    -}}
    ```
2. **Review** to see that the output includes both options.

**Add String Concatenation to Turn Data into a Sentence**

1. **Replace** the top line of this code snippet to say `weekday.lunch.drink ~ " will be served on " ~weekday.day`.
2.  **Render** the following:

    ```django
    {{-
        [
            weekday.lunch.drink ~ " will be served on " ~weekday.day
            for weekday in CTX.days
            if "milk" in weekday.lunch.drink | lower
        ]
    -}}
    ```
3. **Review** the output.

**Use the Join Operator to Combine the Two Statements**

1. **Add** `| join(" and ")` to the end of the code snippet.
2.  **Render** the following:

    ```django
    {{-
        [
            weekday.lunch.drink ~ " will be served on " ~weekday.day
            for weekday in CTX.days
            if "milk" in weekday.lunch.drink | lower
        ] | join(" and ")
    -}}
    ```
3. **Review** the output.
4. **Re-add** the comments.

</details>

<details>

<summary>Step 6: Combine List Comprehension and Loops</summary>

**Build a List and Loop Through it**

1.  **Uncomment** and **Render** the following:

    ```django
    {%- set milky_days = [
            weekday
            for weekday in CTX.days
            if "milk" in weekday.lunch.drink | lower
        ]
    -%}

    On {{ weekday.day }} you should bring a frosty RewstyCola for lunch at {{ weekday.lunch.time }} because they are serving {{ weekday.lunch.drink | lower }}.

    ```
2. **Review** the output.
3. **Re-add** the comments.

</details>

<details>

<summary>Eggstra-Credit!</summary>

**Loop Through and If And Statement and Combine with a Join Operator**

1.  **Uncomment** and **Render** the following:

    ```django
    On {{
        [
            weekday.day
            for weekday in CTX.days
            if
                "west" in weekday.weather.wind_direction
                and
                (
                    "rain" in weekday.weather.conditions | lower
                    or
                    "storm" in weekday.weather.conditions | lower
                )
        ] | join(" and ")
    }} you will need to close the darn windows!
    ```
2. **Review** the output.
3. **Re-add** the comments to revert the code.

**Loop Through and Output JSON Objects**

1.  **Uncomment** and **Render** the following:

    ```django
    {{-
        [
            {
                "day": weekday.day,
                "temp": weekday.weather.temperature
            }
            for weekday in CTX.days
            if
                weekday.weather.temperature <= 65
        ]
    -}}
    ```
2. **Review** the output.
3. **Add** a comma at the end of `"temp": weekday.weather.temperature`.
4. **Type** `"lunch": weekday.lunch` on the next line.
5.  **Render** the following:

    ```django
    {{-
        [
            {
                "day": weekday.day,
                "temp": weekday.weather.temperature,
                "lunch": weekday.lunch
            }
            for weekday in CTX.days
            if
                weekday.weather.temperature <= 65
        ]
    -}}
    ```
6. **Review** the output to see the lunch attribute added.
7. **Re-add** the comments to revert the code.

**Output an HTML Table**

1. **Uncomment** the HTML structure at the bottom of your sample code.
2. **Render** the HTML Structure.
3. **Open** a text editor and copy/paste the output.
4. **Save** the file as _Jinja-to-html.html_.
5. **Drop** the file into a browser to see the results!

</details>

## Get Credit

{% hint style="warning" %}
To get credit for completing this session offline, please [submit this form](https://app.rewst.io/form/ef9d5d9a-f4d0-4aaf-a7a0-c11a9b3b80a1).
{% endhint %}

## Additional Resources

{% hint style="info" %}
For more information on using Jinja, check out our documentation:

* [Introduction to Jinja](../../documentation/jinja/intro-to-jinja.md)
{% endhint %}