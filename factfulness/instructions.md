


> You are a coding assistant with access to the Tavily web search MCP and a code execution environment.
> I want you to:
>
> **Goal**
>
> Create a **Jupyter notebook** called `factfulness_update.ipynb` that:
>
> 1. Fetches **5 global metrics** *from the web at runtime* (no manual dataset download by you).
> 2. Analyzes how they changed, focusing especially on **2018–latest year** as an update to Hans Rosling’s *Factfulness*.
> 3. Contains enough **Markdown commentary** that I can later run `nbconvert` to get a blogpost‑ready HTML/Markdown article.
>
> The 5 metrics:
>
> 1. Extreme poverty rate (share of world population in extreme poverty)
> 2. Under‑5 child mortality rate (deaths per 1,000 live births, global)
> 3. Global life expectancy at birth (years)
> 4. Primary completion rate, female (% of relevant age group; world total or low‑income countries)
> 5. Access to electricity (% of population, global)
>
> ---
>
> ### Step 1 – Find good *programmatic* data sources (use Tavily)
>
> Use the **Tavily MCP** to search for **official, stable, machine‑readable sources** for each metric. Follow these rules:
>
> 1. **Preferred sources**
>
>    * For metrics 1, 2, 3, 5, *prefer* **Our World in Data (OWID)** or **World Bank**.
>    * For metric 4 (girls’ primary completion), use **World Bank** indicator `SE.PRM.CMPT.FE.ZS` (female primary completion rate).
> 2. **Our World in Data (OWID)**
>
>    * Use OWID’s **“grapher”** endpoints, which expose CSV/JSON data when you append `.csv` or `.json`.
>      The pattern is like:
>      `https://ourworldindata.org/grapher/share-of-population-in-extreme-poverty.csv`
>      `https://ourworldindata.org/grapher/child-mortality.csv`
>      `https://ourworldindata.org/grapher/life-expectancy.csv`
>      `https://ourworldindata.org/grapher/share-of-the-population-with-access-to-electricity.csv`
>    * Confirm this pattern against OWID’s **Chart API docs** using Tavily.
> 3. **World Bank API**
>
>    * For World Bank indicators, use the JSON API with `WLD` (world total) when possible, for example:
>      `https://api.worldbank.org/v2/country/WLD/indicator/SE.PRM.CMPT.FE.ZS?format=json&per_page=20000`
>    * If you need other indicators (e.g., `SH.DYN.MORT`, `SP.DYN.LE00.IN`, `EG.ELC.ACCS.ZS`), you may use the same pattern.
> 4. **Important constraint**
>
>    * **Do NOT** download or paste entire datasets *yourself* into the conversation.
>    * Instead, for each metric, you must:
>
>      * Identify the **URL(s)** for machine‑readable data.
>      * Then **write Python code inside the notebook** that loads the data directly from those URLs at runtime using `pandas.read_csv` or `requests`.
>
> After you’ve found candidate URLs with Tavily, briefly **list them in the analysis (for yourself)** and then move on to building the notebook.
>
> ---
>
> ### Step 2 – Design of the Jupyter notebook
>
> Create a Jupyter notebook named `factfulness_update.ipynb` with this structure:
>
> 1. **Title & intro (Markdown cell)**
>
>    * Title: “Updating Factfulness: What Happened in the Last 7 Years?”
>    * Short intro (2–4 paragraphs) explaining:
>
>      * The *Factfulness* idea: our perception is more negative than the data.
>      * This notebook will **recreate and update** a small part of Rosling’s analysis for 5 global metrics.
>      * We focus on 2000–latest, highlighting **2018–latest** as “the post‑book years”.
> 2. **Setup code cell**
>
>    * Import libraries:
>
>      ```python
>      import pandas as pd
>      import matplotlib.pyplot as plt
>      from pathlib import Path
>      ```
>    * Set a global style (optional, but keep it simple and built-in `matplotlib`).
>    * Define a `DATA_DIR = Path("data")` and create it if needed.
> 3. **One section per metric**
>
> For **each of the 5 metrics**, create a section with the following cells:
>
> **(a) Markdown – explanation of the metric**
>
> * Briefly describe:
>
>   * What the metric measures (1 short paragraph).
>   * Why Rosling cared about it / what misconception it addresses.
>   * What you expect to find in the period 2000–latest and especially 2018–latest.
>
> **(b) Code – data fetching**
>
> * Load the data **directly from the URL(s)** you identified earlier.
> * Use code like:
>
>   ```python
>   url_poverty = "https://ourworldindata.org/grapher/share-of-population-in-extreme-poverty.csv"
>   df_pov = pd.read_csv(url_poverty)
>   ```
> * Clean to a tidy format:
>
>   * Keep `World` aggregate (or relevant entity field) only.
>   * Ensure you have columns like `year`, `value` (rename as needed).
>   * Filter to a reasonable time window, e.g. `year >= 1990`.
>
> **(c) Code – quick analysis**
>
> * Compute:
>
>   * Starting value (e.g. year 2000).
>   * Value in 2018.
>   * Latest available value.
>   * Absolute and relative change between 2018 and latest.
> * Print a small summary, e.g.:
>
>   ```python
>   base = df.loc[df["year"] == 2000, "value"].iloc[0]
>   y2018 = df.loc[df["year"] == 2018, "value"].iloc[0]
>   latest_year = df["year"].max()
>   latest = df.loc[df["year"] == latest_year, "value"].iloc[0]
>   print(f"2000: {base:.2f}, 2018: {y2018:.2f}, {latest_year}: {latest:.2f}")
>   print(f"Change 2018→{latest_year}: {latest - y2018:+.2f} ({(latest - y2018)/y2018*100:+.1f}%)")
>   ```
>
> **(d) Code – visualization**
>
> * Make a simple line chart:
>
>   ```python
>   fig, ax = plt.subplots(figsize=(8, 4))
>   ax.plot(df["year"], df["value"])
>   ax.axvline(2018, linestyle="--", alpha=0.5)  # mark the book year
>   ax.set_title("Extreme poverty rate, World")
>   ax.set_xlabel("Year")
>   ax.set_ylabel("% of population (under poverty line)")
>   plt.tight_layout()
>   ```
> * Don’t depend on external styling libraries (plain matplotlib is fine).
>
> **(e) Markdown – mini-interpretation**
>
> * Write a short narrative (1–3 paragraphs) *based on the computed numbers*, e.g.:
>
>   * For each metric, explain:
>
>     * Whether things improved, stagnated, or worsened since 2000.
>     * What specifically happened **2018 → latest** (this is the “Factfulness update”).
>     * How this compares to what people *think* is happening (optional, but nice).
>
> Repeat this pattern for:
>
> 1. Extreme poverty (OWID or World Bank WLD)
> 2. Under‑5 mortality (OWID `child-mortality` or WB `SH.DYN.MORT` with WLD)
> 3. Life expectancy (OWID `life-expectancy` or WB `SP.DYN.LE00.IN` with WLD)
> 4. Primary completion, female (`SE.PRM.CMPT.FE.ZS` – world or low‑income aggregate)
> 5. Access to electricity (OWID `share-of-the-population-with-access-to-electricity` or WB `EG.ELC.ACCS.ZS`)
>
> When you use OWID, remember to append `.csv` to grapher URLs.
>
> ---
>
> ### Step 3 – Wrap‑up section
>
> At the end of the notebook, add:
>
> 1. **Markdown – “Putting the pieces together”**
>
>    * A short section that:
>
>      * Lists the 5 metrics with a one‑line summary like
>        “Extreme poverty: down from X% in 2018 to Y% in 2023”
>        “Under‑5 mortality: down from A to B per 1,000” etc.
>      * Briefly reflects on:
>
>        * Where Rosling’s optimism still holds.
>        * Where progress may have slowed or reversed.
>    * This is basically the **conclusion** of the blogpost.
> 2. **Optional: helper code cell**
>
>    * A final cell with something like:
>
>      ```python
>      # If running from command line, one could convert this notebook to HTML with:
>      # !jupyter nbconvert --to html factfulness_update.ipynb
>      ```
>
> ---
>
> ### Constraints and style
>
> * Very important:
>
>   * Do **not** manually download whole datasets or paste them into the conversation.
>   * All data access must be via URL in the notebook code cells.
> * Use **clear, simple Markdown**: headings, short paragraphs, bullet lists.
> * Assume a reader who is not a data scientist; explain metrics in plain language.
> * Prioritize correctness and reproducibility of data fetching.
>
> When you’re ready, output the full notebook content (cells in order).
> If the environment allows, actually create `factfulness_update.ipynb`; otherwise, print the cells in a way that I can easily reconstruct the notebook.

---


[1]: https://ourworldindata.org/grapher/share-of-population-in-extreme-poverty?utm_source=chatgpt.com "Share of population living in extreme poverty, 2024 - Our World in Data"
[2]: https://ourworldindata.org/grapher/child-mortality?utm_source=chatgpt.com "Child mortality rate - Our World in Data"
[3]: https://ourworldindata.org/grapher/life-expectancy?utm_source=chatgpt.com "Life expectancy - Our World in Data"
[4]: https://genderdata.worldbank.org/en/indicator/se-prm-cmpl-zs?utm_source=chatgpt.com "Primary completion rate, based on completers (%) | World Bank Gender ..."
[5]: https://ourworldindata.org/grapher/share-of-the-population-with-access-to-electricity?utm_source=chatgpt.com "Share of the population with access to electricity - Our World in Data"
[6]: https://docs.owid.io/projects/etl/api/chart-api/?utm_source=chatgpt.com "Chart API - OWID's Technical Documentation"
