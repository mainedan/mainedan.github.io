---
title: How to Make a Dynamic Calendar in LibreOffice Calc
author: Mainedan
date: 2026-09-28 21:38:22 -0700
categories:
  - LibreOffice
  - Tutorials
tags:
  - libreoffice
  - calc
  - calendar
  - dynamic-template
  - spreadsheet
---

## title: How to Make a Dynamic Calendar in LibreOffice Calc

## Step-by-Step Guide: How to Make a Dynamic Calendar in LibreOffice Calc

This guide outlines the complete process for creating an editable, dynamic calendar template in LibreOffice Calc that automatically updates when you select a new month or year.

---

## Step 1: Initial Page & Document Setup

1. **Create a New Spreadsheet**:
   - Open LibreOffice Calc.
   - Select **File** -> **New** -> **Spreadsheet** from the menu.

2. **Configure Page Settings and Print Area**:
   - Go to **File** -> **Print Preview**-> **Format Page**
   - In the **Page** tab:
     - Select paper format (e.g., **US Letter** or **A4**).
     - Set orientation to **Landscape**.
     - Set desired margins (e.g., `0.75"` on all sides).
     - Click **OK**. Dashed lines will appear indicating the printable area boundaries.

3. **Remove Headers and Footers**:
   - Go to **File** -> **Print Prevew** -> Format Page
   - In the **Header** tab, uncheck **Header on**.
   - In the **Footer** tab, uncheck **Footer on**.
   - Click **OK**.

![LibreOffice Calc Page Style dialog](/assets/images/cal/Page-Style.png)

![LibreOffice Calc Format Page dialog with the Footer tab selected.](/assets/images/cal/Footer.png)

![LibreOffice Calc Format Page dialog with the Header tab selected.](/assets/images/cal/Header.png)

## Step 2: Set Up Worksheet Tabs**

1. Double-click the existing sheet tab (`Sheet1`) and rename it to **`Month Calendar`**.
   - Add a second sheet and name it **`Settings`**.

2. **Populate Month List**:
   - Switch to the **`Settings`** sheet.
   - In Column A (cells `A1` to `A12`), type the names of the 12 months sequentially (e.g., `January` through `December`).

3. **Add Data Validity (Dropdown List)**:
   - Switch back to the **`Month Calendar`** sheet.
   - Select cell **`A1`**.
   - Go to **Data** -> **Validity…**
   - In the **Criteria** tab:
     - Under **Allow**, choose **Cell Range**.
     - In the **Source** field, enter: `$Settings.$A$1:$A$12`
   - Click **OK**. Cell `A1` now features a drop-down arrow to select any month.

![LibreOffice Calc Data Validity dialog](/assets/images/cal/Validity.png)

---

## Step 3: Layout Header & Adjust Dimensions

1. **Format Month Header**:
   - Merge the top cells (e.g., `A1:C1`) to provide space for the month title.
   - Increase font size (e.g., size **40**).
   - Left  Align

2. **Calculate & Set Column Widths**:
   - Determine width per column based on paper width and margins:
     Open your calculator application. For me it’s (11-2*0.75)/7=1.3571. Note: only use the first 2 decimals, LibreOffice will round the number anyway.
     *(Example for 11" US Letter Landscape with 0.75" margins: $(11 - 1.5) / 7 = 1.3571" \approx 1.35"$.)*
   - Select headers for the first 7 columns (**A** through **G**).
   - Right-click and choose **Column Width…**.
   - Enter your calculated width (e.g., `1.35"`) and click **OK**. (The print boundary dashed line should align right after Column G).

3. **Add Year and Day Names**:
   - In cell **`G1`**, enter a year manually (e.g., `2026`) or use the dynamic formula:

     ```ods
     =YEAR(TODAY())
     ```

   - Format `G1` with the same font and size as the month header.
   - In row 2 (cells `A2:G2`), enter the weekday names (e.g., `Sunday` through `Saturday`).

---

## Step 4: Configure Dynamic Calendar Formulas

1. **Calculate Starting Date of Calendar Grid**:
   - To make the grid dynamic, cell **`A3`** (first cell of week 1) must calculate the first date shown in the calendar grid (which may fall in the previous month if the 1st of the month isn't a Sunday):

     ```ods
     =DATE(G1,MATCH(A1,$Settings.A1:A12,0),1) - (WEEKDAY(DATE(G1,MATCH(A1,$Settings.A1:A12,0),1)) - 1)
     ```

     - `DATE(G1, MATCH(A1, $Settings.A1:A12, 0), 1)` retrieves the 1st day of the selected month/year.
     - `WEEKDAY(...)` calculates the day index (1 = Sunday, 7 = Saturday).
     - Subtracting `WEEKDAY(...) - 1` offsets the calendar back to the preceding Sunday.

2. **Populate Grid Formulas**:
   - **Week 1 Row (`Row 3`)**:
     - Cell `B3`: `=A3+1`
     - Cell `C3`: `=B3+1` (drag/copy across to `G3`).
   - **Row 4**: Leave empty (used as event/note space under Week 1).
   - **Week 2 Row (`Row 5`)**:
     - Cell `A5`: `=G3+1`
     - Cell `B5`: `=A5+1` (drag/copy across to `G5`).
   - **Subsequent Weeks**: Skip alternating rows for event spaces, setting each new date row relative to the row above it:
     - Cell `A1`: `=G5+1`
     - Cell `B7`: `=A7+1` (drag/copy across to `G7`). Continuue to populate the calendar for a total of 6 lines
3. \. Quick Copy/Paste Shortcut
      - To avoid retyping formulas for the remaining weeks, select the entire date row for Week 2 (**A5:G5**) and copy it. Paste it directly into the date rows for the remaining weeks (**Row 7, Row 9, Row 11, and Row 13**). LibreOffice Calc will - automatically adjust the relative cell references for you.

---

## Step 5: Format Date Displays & Row Heights

1. **Format Date Display Code**:
   - Select all date cells (`A3:G3`, `A5:G5`, `A7:G7`, `A9:G9`, `A11:G11`, `A13:G13`).
   - Right-click and choose **Format Cells…**.
   - Under the **Date** category, change the **Format Code** to: **`D`** (displays only the day number).
   - Adjust font styling (e.g., size **12**, **Bold**).

2. **Calculate & Set Event Row Heights**:
   - Calculate remaining vertical space for event rows:
     $$\text{Event Row Height} = \frac{\text{Page Height} - (2 \times \text{Margin}) - \text{Header/Date Row Heights}}{6}$$
     *(Example for 8.5" page height with 1.5" total margins and 2.28" existing row height: $(8.5 - 1.5 - 2.28) / 6 = 0.786" \approx 0.78"$.)*
   - Select the 6 event rows (rows `4`, `6`, `8`, `10`, `12`, `14`).
   - Right-click the row numbers -> **Row Height…** -> set to `0.79"` and click **OK**.

---

## Step 6: Grid Borders & Final Formatting

1. **Apply Cell Borders**:
   - Select calendar date and event cells.
   - Right-click and select **Format Cells…** -> **Borders** tab.
   - Apply outer boundaries and inner gridlines.
   - To create seamless boxes for each day, set the **bottom border** of date rows to **None** and the **top border** of event rows to **None**.

2. **Preview and Print**:
   - Choose **File** -> **Print Preview** to verify page layout and margins.
   - Exit preview with `Escape` or **Close Preview**.

## Step 7: Conditional Formatting (Fade Out Extra Month Dates)

1. **Set Up Boundary Calculations in `Settings` Sheet**:
   - Switch to the **`Settings`** sheet.
   - In cell **`C1`**, type label: `First Day of the Month`.
   - In cell **`C2`**, enter formula to calculate the start date of the active month:

     ```ods
     =DATE($'Month Calendar'.G1, MATCH($'Month Calendar'.A1, A1:A12, 0), 1)
     ```

   - In cell **`D1`**, type label: `Last Day of the Month`.
   - In cell **`D2`**, enter formula to calculate the end date of the active month:

     ```ods
     =EDATE(C2, 1) - 1
     ```

   - Make sure the format of the cell is set to **Date ->**Format -(12/01/99)

2. **Select Calendar Date Ranges**:
   - Switch back to the **`Month Calendar`** sheet.
   - Select all date header rows while holding `Ctrl`:
     `A3:G3, A5:G5, A7:G7, A9:G9, A11:G11, A13:G13`

3. **Create and Customize the "Faded" Cell Style**:
   - Press **`F11`** (or go to **Styles** -> **Manage Styles**) to open the Styles sidebar.
   - Click the **Cell Styles** icon at the top of the sidebar.
   - Right-click **Default** (or any empty space in the style list) and select **New…** (or right-click **Faded** and select **Modify…** if it already exists).
   - In the **General** tab, enter **`Faded`** in the **Name** box.
   - In the **Font Effects** tab, set **Font color** to a light muted shade (e.g., **Light Gray** or **Gray 4**).
   - Click **OK** to save the new style.

4. **Configure Conditional Formatting Rules**: Monthly Calendar sheet
   - Go to **Format** -> **Conditional** -> **Condition…** -> **More Rules**
   - Set **Condition 1**:
     - **Cell value** -> **is less than** -> `$Settings.$C$2`
     - **Apply Style**: Select **`Faded`**
   - Click **Add** to create **Condition 2**:
     - **Cell value** -> **is greater than** -> `$Settings.$D$2`
     - **Apply Style**: Select **`Faded`**
   - Confirm the target **Cell Range** is set to:
     `A3:G3,A5:G5,A7:G7,A9:G9,A11:G11,A13:G13`
   - Click **OK**. Days belonging to the previous or next month will now automatically display in grayed-out/faded text.

   ## Step 8: Add & Highlight National Holidays (Optional)

5. **Create Holiday Table in `Settings` Sheet**:
   - Switch to the **`Settings`** sheet.
   - In Column **F** (cells `F2:F20`), enter your list of national holiday dates (e.g., `2026-01-01`, `2026-07-04`, `2026-12-25`).
   - In Column **G** (cells `G2:G20`), enter the corresponding holiday names (e.g., `New Year's Day`, `Independence Day`, `Christmas`).

6. **Create a "Holiday" Cell Style**:
   - Press **`F11`** to open the Styles sidebar.
   - Right-click in the style list and select **New…**.
   - In the **Organizer** tab, set **Name** to **`Holiday`**.
   - In the **Background** tab, pick a soft accent color (e.g., **Light Red** or **Soft Yellow**).
   - In the **Font Effects** / **Font** tab, apply bold text or a dark accent color.
   - Click **OK**.

7. **Configure Conditional Formatting Rule for Holidays**:
   - Switch back to the **`Month Calendar`** sheet.
   - Select all calendar date cells (`A3:G3, A5:G5, A7:G7, A9:G9, A11:G11, A13:G13`).
   - Go to **Format** -> **Conditional** -> **Condition…** (or **Manage…**).
   - Click **Add** to create a new rule:
     - Change condition type from **Cell value** to **Formula is**.
     - Enter formula:

       ```ods
       COUNTIF($Settings.$F$2:$F$20, A3) > 0
       ```

     - **Apply Style**: Select **`Holiday`**.
   - Click **OK**. Any date on the calendar matching a date in your holiday table will now automatically highlight.

8. **Automatically Display Holiday Names in Event Rows (Optional)**:
   - To show the holiday title directly in the box under the date, select event row cell **`A4`** (under date `A3`) and enter:

     ```ods
     =IFERROR(VLOOKUP(A3, $Settings.$F$2:$G$20, 2, FALSE), "")
     ```

   - Copy or drag this formula across to `G4`, and repeat for event rows `6`, `8`, `10`, `12`, and `14`.
   - When a holiday occurs, its name will automatically populate in the event space below the date!

---

## Step 9: Create an Annual Master View (3x4 Grid Layout)

1. **Add a New Worksheet Tab**:
   - Right-click the sheet tab list at the bottom and select **Insert Sheet…**.
   - Name the new worksheet **`Annual View`**.

2. **Set Up Master Year Header**:
   - In cell **`A1`**, enter the label: `Year`.
   - In cell **`B1`**, enter the target year: `2026` (or `=YEAR(TODAY())`).
   - Format **`A1:B1`** with bold text and larger font size as the primary chronological anchor for the document.

3. **Construct the 3x4 Month Grid Spans (7-Column Mini Blocks)**:
   - To lay out all 12 months cleanly on a single sheet, organize the mini calendars into **3 columns across** and **4 rows down**. Because each month requires 7 columns (Sunday–Saturday) plus spacing:
     - **Column 1 Block (Spreadsheet Cols A–G)**:
       - **January**: Title merged `A3:G3`, Day Headers `A4:G4`, Date Grid `A5:G10`.
       - **April**: Starts below January at merged `A12:G12` (Day Headers `A13:G13`, Date Grid `A14:G19`).
       - **July**: Starts below April at merged `A21:G21` (Day Headers `A22:G22`, Date Grid `A23:G28`).
       - **October**: Starts below July at merged `A30:G30` (Day Headers `A31:G31`, Date Grid `A32:G37`).
     - **Column 2 Block (Spreadsheet Cols I–O, leaving Column H as a spacer)**:
       - **February**: Starts at merged `I3:O3` (Day Headers `I4:O4`, Date Grid `I5:O10`).
       - **May**: Starts at merged `I12:O12` (Day Headers `I13:O13`, Date Grid `I14:O19`).
       - **August**: Starts at merged `I21:O21` (Day Headers `I22:O22`, Date Grid `I23:O28`).
       - **November**: Starts at merged `I30:O30` (Day Headers `I31:O31`, Date Grid `I32:O37`).
     - **Column 3 Block (Spreadsheet Cols Q–W, leaving Column P as a spacer)**:
       - **March**: Starts at merged `Q3:W3` (Day Headers `Q4:W4`, Date Grid `Q5:W10`).
       - **June**: Starts at merged `Q12:W12` (Day Headers `Q13:W13`, Date Grid `Q14:W19`).
       - **September**: Starts at merged `Q21:W21` (Day Headers `Q22:W22`, Date Grid `Q23:W28`).
       - **December**: Starts at merged `Q30:W30` (Day Headers `Q31:W31`, Date Grid `Q32:W37`).

4. **Configure Mini Month Block Formulas (Detailed Cell-by-Cell Example for January)**:
   - **Month Title Header** (Cell `A3` or merged `A3:G3`):

     ```ods
     =DATE($B$1, 1, 1)
     ```

     - Right-click `A3` -> **Format Cells…** -> set Date Format Code to **`MMMM`** (displays "January").
   - **Weekday Headers** (Row 4): Enter day abbreviations across cells `A4:G4`:

     | Cell | `A4` | `B4` | `C4` | `D4` | `E4` | `F4` | `G4` |
     | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
     | **Label** | `S` | `M` | `T` | `W` | `T` | `F` | `S` |

   - **Week 1 First Sunday Calculation** (Cell `A5`):

     ```ods

    =DATE($B$1, 1, 1)-(WEEKDAY(DATE($B$1, 1, 1))-1)

     ```
   - **Horizontal & Vertical Grid Increments**:
     - **Horizontal Addition (`+1`)**: In cell `B5`, enter `=A5+1`. Copy/drag across `C5:G5` (`=B5+1`, `=C5+1`, etc.) to populate Monday–Saturday.
     - **Vertical Addition (`+7`)**: In cell `A6` (Sunday of Week 2), enter `=A5+7`. Copy/drag row 5's horizontal formulas across `B6:G6`.
     - **Remaining Weeks**: Copy row 6 formulas down through rows 7–10 to complete January's grid.

5. **Replicate Block Formulas Across the 3x4 Layout**:
   - Repeat the mini month block formulas across the remaining 11 month blocks, updating the month index parameter in `DATE($B$1, Month_Number, 1)` for each month position (1 through 12).

6. **Apply Conditional Formatting**:
   - Apply conditional formatting (`< DATE($B$1, Month, 1)` and `> EDATE(DATE($B$1, Month, 1), 1) - 1`) to automatically fade out neighboring month dates in each mini grid.
   - Updating the year in **`B1`** automatically updates all 12 mini month blocks across the entire 3x4 grid!
